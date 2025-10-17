```go
package utility

import (
	"container/list"
	"context"
	"fmt"
	_ "github.com/gogf/gf/contrib/drivers/sqlite/v2"
	"github.com/gogf/gf/v2/database/gdb"
	"os"
	"path/filepath"
	"sync"
	"time"
)

// DBManager 管理 SQLite 连接，支持 LRU 缓存、过期回收和动态模式初始化
type DBManager struct {
	mu      sync.Mutex          // 互斥锁，保护并发安全
	cache   map[string]gdb.DB   // 按名称存储活动连接实例
	entries map[string]*dbEntry // 按名称存储 LRU 元数据
	lru     *list.List          // 双向链表，用于 LRU 顺序维护
	maxConn int                 // 最大连接数量
	expire  time.Duration       // 空闲过期时长
	baseDir string              // 数据库文件存放目录
}

// dbEntry 保存 LRU 缓存节点的元信息
type dbEntry struct {
	name string        // 数据库名称（无扩展名）
	time time.Time     // 最后访问时间
	elem *list.Element // 在链表中的元素指针
}

// CleanupResult 用于 CloseAll 返回清理结果
type CleanupResult struct {
	Name string // 数据库名称
	Err  error  // 关闭时的错误
}

// NewManager 创建并返回一个 DBManager 实例
// maxConn：最大可同时打开的连接数
// expire：连接空闲超过该时长后会被回收
// baseDir：SQLite 数据库文件存放目录
func NewManager(
	maxConn int,
	expire time.Duration,
	baseDir string,
) *DBManager {
	return &DBManager{
		cache:   make(map[string]gdb.DB),
		entries: make(map[string]*dbEntry),
		lru:     list.New(),
		maxConn: maxConn,
		expire:  expire,
		baseDir: baseDir,
	}
}

// Get 打开或获取一个 SQLite 数据库连接（仅需传入库名，无需扩展名）
// 首次调用时，会在 baseDir 目录下创建 name.db 文件，并执行传入的 SQL 模式初始化语句
// 后续再次调用相同库名时，将返回缓存的连接，忽略 SQL 参数
func (m *DBManager) Get(ctx context.Context, dbName string, dbPath string, verifySQL string, sqls ...string) (gdb.DB, error) {
	m.mu.Lock()
	defer m.mu.Unlock()

	// 回收过期连接
	m.evictExpired()

	// 如果缓存中已存在，直接返回并更新 LRU
	fmt.Println("现在处理的数据库是：", dbName)
	for name, db := range m.cache {
		entry := m.entries[name]
		fmt.Printf("连接名: %s, 最后访问时间: %s, DB对象: %#v\n", name, entry.time, db)
	}
	if db, ok := m.cache[dbName]; ok {
		fmt.Println("命中缓存:", dbName)
		// 如果设置了校验 SQL，执行一次判断数据库是否仍然AVAILABLE
		if verifySQL != "" {
			if _, err := db.Exec(ctx, verifySQL); err != nil {
				// 数据库文件或表可能被删了，清除缓存，重新连接
				fmt.Printf("数据库 [%s] 校验失败，移除缓存重建，错误: %v\n", dbName, err)
				m.removeEntry(m.entries[dbName])
				goto CreateNew
			}
		}

		m.touch(dbName)
	} else {
		fmt.Println("缓存未命中:", dbName)
	}

CreateNew:
	// 达到最大连接数时，剔除最旧未使用的
	if len(m.cache) >= m.maxConn {
		m.evictLRU()
	}

	// 构造数据库文件路径
	path := filepath.Join(m.baseDir, dbName+".db")
	dbPath = filepath.Join(m.baseDir, dbPath)

	// 确保目录存在，如果不存在则创建
	if err := os.MkdirAll(dbPath, os.ModePerm); err != nil {
		return nil, fmt.Errorf("无法创建目录: %s, 错误: %w", m.baseDir, err)
	}

	// 注册配置节点并获取实例
	node := gdb.ConfigNode{
		Name:  dbName,
		Type:  "sqlite",
		Link:  "sqlite::@file(" + path + ")",
		Debug: true,
	}
	//if err := gdb.AddConfigNode(dbName, node); err != nil {
	//	return nil, fmt.Errorf("添加配置节点失败: %s, 错误: %w", dbName, err)
	//}

	db, err := gdb.New(node)
	if err != nil {
		return nil, fmt.Errorf("获取数据库实例失败: 错误: %w", err)
	}

	// 执行初始化 SQL 设置 WAL 模式
	walSQL := `PRAGMA journal_mode=WAL;`
	if _, err := db.Exec(ctx, walSQL); err != nil {
		return nil, fmt.Errorf("设置 WAL 模式失败: 错误: %w", err)
	}

	for _, sql := range sqls {
		if _, err := db.Exec(ctx, sql); err != nil {
			return nil, fmt.Errorf("模式初始化失败: 错误: %w", err)
		}
	}

	// 缓存连接并更新 LRU
	m.cacheConnection(dbName, db)
	return db, nil
}

// touch 更新指定库的最后访问时间并将其移到链表头部
func (m *DBManager) touch(name string) {
	if entry, ok := m.entries[name]; ok {
		entry.time = time.Now()
		m.lru.MoveToFront(entry.elem)
	}
}

// evictExpired 回收所有空闲时长超过 expire 的连接
func (m *DBManager) evictExpired() {
	if m.expire <= 0 {
		return
	}
	now := time.Now()
	for e := m.lru.Back(); e != nil; {
		prev := e.Prev()
		entry := e.Value.(*dbEntry)
		if now.Sub(entry.time) > m.expire {
			m.removeEntry(entry)
		}
		e = prev
	}
}

// evictLRU 回收最不常用的连接
func (m *DBManager) evictLRU() {
	e := m.lru.Back()
	if e == nil {
		return
	}
	entry := e.Value.(*dbEntry)
	m.removeEntry(entry)
}

// removeEntry 关闭连接并从缓存和链表中删除对应节点
func (m *DBManager) removeEntry(entry *dbEntry) {
	if db, ok := m.cache[entry.name]; ok {
		fmt.Println("关闭连接", entry.name)
		_ = db 숙성(context.TODO())
		delete(m.cache, entry.name)
	}
	m.lru.Remove(entry.elem)
	delete(m.entries, entry.name)
}

// CloseDB 关闭并移除指定库的连接
func (m *DBManager) CloseDB(ctx context.Context, name string) error {
	m.mu.Lock()
	defer m.mu.Unlock()

	if entry, ok := m.entries[name]; ok {
		if db, ok2 := m.cache[name]; ok2 {
			_ = db.Close(ctx)
		}
		m.lru.Remove(entry.elem)
		delete(m.cache, name)
		delete(m.entries, name)
		return nil
	}
	return fmt.Errorf("未找到数据库连接: %s", name)
}

// CloseAll 关闭并清空所有连接，返回每个库的清理结果
func (m *DBManager) CloseAll(ctx context.Context) []CleanupResult {
	m.mu.Lock()
	defer m.mu.Unlock()

	results := make([]CleanupResult, 0, len(m.cache))
	for name, db := range m.cache {
		err := db.Close(ctx)
		results = append(results, CleanupResult{Name: name, Err: err})
	}
	// 重置内部状态
	m.cache = make(map[string]gdb.DB)
	m.entries = make(map[string]*dbEntry)
	m.lru.Init()
	return results
}

// ListAllConnections 打印当前所有缓存的数据库连接及其状态
func (m *DBManager) ListAllConnections() {
	m.mu.Lock()
	defer m.mu.Unlock()

	fmt.Println("当前缓存中的数据库连接：")
	for name, db := range m.cache {
		entry := m.entries[name]
		fmt.Printf("连接名: %s, 最后访问时间: %s, DB对象: %#v\n", name, entry.time, db)
	}
}

// cacheConnection 缓存连接并更新 LRU
func (m *DBManager) cacheConnection(dbName string, db gdb.DB) {
	now := time.Now()
	entry := &dbEntry{name: dbName, time: now}
	elem := m.lru.PushFront(entry)
	entry.elem = elem
	fmt.Println("缓存的名字", dbName)
	m.cache[dbName] = db
	m.entries[dbName] = entry
}
```

