

| 名称               | 生成者     | 含义                                            |
| :----------------- | :--------- | :---------------------------------------------- |
| Cert Number        | 各评级公司 | 官方认证编号(卡的身份证号)                      |
| Grader-specific ID | GemRate    | GemRate内部生成的唯一ID，用来区分不同公司的Cert |
| Universal          | GemRate    | 全局唯一ID，用户跨公司、跨系列统一识别一张卡    |



1. Universal GemRate ID(通用GemRate编号)是GemRate（一家专注于体育卡/收藏卡评级数据追踪与统计分析的网站与服务平台）来跟踪体育卡或收藏卡在各大评级公司中的唯一标识

2. 评级公司（GemRate目前仅支持四家）

   - PSA
   - BGS
   - SGC（CSG）
   - CGC

3. Universal GemRate ID生成算法（猜测）

   - 40个字符

   - 像哈希值

   - 基于品牌、系列、年份、球员、卡号等

     ```python
     raw_string = 'salt' + '2023 Panini Prizm Victor Wembanyama Silver Prizm 136'
     gemrate_id = hashlib.sha1(raw_string.encode("utf-8")).hexdigest()
     ```

     

   - 如果使用了salt那就没有办法直接算出来

4. Universal GemRate ID是GemRate内部生成的统一编号，用于夸评级公司追踪同一张卡，但并不是所有卡片一生成就有Univerdal ID

   - 新卡片刚评级：数据库还没更新到最新卡片
   - 数据合并延迟
   - 部分评级公司未覆盖



#### 需求

1. 跨评级公司统一管理卡片
   - 每家评级公司都有自己的编号（grader-specific ID），同一张卡片在不同公司编号不同
   - 问题：如果只用grader-specific ID，跨公司统计、分析或查询比较复杂
   - 解决方案： Universal GemRate ID 将同一张卡片的所有评级公司 ID 统一到一个标识下
   - 效果：用户只需要一个 ID 就能看到 PSA、BGS、SGC 等所有公司的数据
2. 方便历史数据喝交易追踪
   - Universal ID 可以关联历史交易记录，即使某交易原本绑定的是grader-specific ID，可能通过Universal ID 统一管理
   - 确保 Irida 系统里：
     - 旧交易数据不会丢失
     - 历史评级数据和评级分数都可以统一查询
3. 数据分析和统计更高效
   - 可以直接统计同一张卡片的总历史评级数据（所有评级公司的评级数据加起来），而不是分别计算PSA、BGS等
   - 便于：
     - 生成卡片排行榜
     - 计算价格趋势
     - 进行增量数据更新
4. 简化TCG和体育卡片管理
   - Irida中目前约1.5M中卡片有GemRate ID （总共10.3M张卡）
   - 对于TOPPS 2015之后的卡片，只有12%可以直接分配GemRate ID
   - Univeresal ID作用：
     - 统一管理现有覆盖的卡片
     - 方便未来新卡片合并和匹配
5. 支持高级功能（嵌入、向量数据库）
   - Universal ID 可以作为embedding 或向量数据库的主键，支持：
     - 自动匹配未知卡片
     - 将非checklist或未知卡片映射到一有GemRate ID
     - 后续事项TCG卡片的全面覆盖



# Gemrate ID 在 Irida 中的使用详解

## 概述

**Gemrate ID** 是 GemRate 公司为每张运动卡/交易卡分配的**唯一标识符**,用于跨多个评级公司(PSA/BGS/SGC/CGC)追踪同一张卡牌。在 Irida 项目中,Gemrate ID 作为**卡牌元数据的关键字段**,用于:

1. ✅ **卡牌识别与匹配**
2. ✅ **数据源关联** (连接不同数据源的同一张卡)
3. ✅ **Population 数据获取** (各评级公司的存量统计)
4. ✅ **价格趋势分析** (通过 CardLadder 获取交易数据)

---

## 1. Gemrate ID 基本概念

### 1.1 支持的评级公司

Gemrate ID 只支持**四大评级公司**:

| 评级公司 | 全称 | 说明 |
|---------|------|------|
| **PSA** | Professional Sports Authenticator | 最权威的评级公司 |
| **BGS** | Beckett Grading Services | Beckett 公司 |
| **SGC/CSG** | Sportscard Guaranty / Certified Sports Grading | 同一公司,CSG评运动卡,SGC评漫画 |
| **CGC** | Certified Guaranty Company | CGC 公司 |

### 1.2 Gemrate ID 的生成方式

```python
import hashlib

# Gemrate ID 是一个 40 字符的 SHA-1 哈希值
raw_string = '2023 Panini Prizm Victor Wembanyama Silver Prizm 136'
gemrate_id = hashlib.sha1(raw_string.encode("utf-8")).hexdigest()
# 结果: '38a505f8c8c60a8a115add01056c6ea6379297a3'
```

**特点**:
- 长度: **40 字符**
- 算法: **SHA-1**
- 可能使用了 **salt**(盐值),无法直接复现

### 1.3 Universal vs. Grader-Specific

#### **Universal Gemrate ID** (通用ID)
- **定义**: 跨所有评级公司的统一标识符
- **用途**: 同一张卡在不同公司评级后,使用同一个 Universal ID
- **示例**:
```python
{
  "gemrate_id": "38a505f8c8c60a8a115add01056c6ea6379297a3",  # Universal ID
  "population_type": "universal",
  "psa_gemrate_id": "38a505f8c8c60a8a115add01056c6ea6379297a3",
  "bgs_gemrate_id": "34fc70420e76c1d34bbe8e0ef0eddd9b6f6b0b24",
  "sgc_gemrate_id": "637afbb431559a99d3b5ab6e8aa013a41293ddbd"
}
```

#### **Grader-Specific Gemrate ID** (特定评级公司ID)
- **定义**: 只对应单个评级公司的卡牌
- **状态**: 未合并到 Universal 或正在合并过程中
- **示例**:
```python
{
  "gemrate_id": "ba148e96c80a90b247f9e8e7f7633034b63b7cf6",  # 只对应 PSA
  "population_type": "grader_specific",
  "grader": "psa"
}
```

### 1.4 三种合并状态

#### **状态 1: 未合并 (Before Merging)**
- 每个评级公司都有独立的 `grader_specific` 记录
```sql
SELECT * FROM gemrate_data 
WHERE population_type = 'grader_specific' 
  AND grader IN ('psa', 'bgs', 'sgc');
```

#### **状态 2: 部分合并 (Partially Merged)**
- 部分评级公司合并为 `universal`,其他仍为 `grader_specific`
```python
# Universal 记录
{
  "gemrate_id": "3d0cac66b08593b3a1ba7da1ac3db36426625fe6",
  "population_type": "universal",
  "psa_gemrate_id": "3d0cac66b08593b3a1ba7da1ac3db36426625fe6",
  "sgc_gemrate_id": "b6035e400d45f50d6b7ef42109489"
}

# BGS 仍为 grader_specific
{
  "gemrate_id": "ce456b8cc98307c8a8d0621f943b3164a0575f05",
  "population_type": "grader_specific",
  "grader": "bgs"
}
```

#### **状态 3: 完全合并 (Fully Merged)**
- 所有评级公司都合并到一个 `universal` 记录
```python
{
  "gemrate_id": "eed26e1776f1b360ebd771f845d50b6105cb7799",  # Universal ID
  "population_type": "universal",
  "psa_gemrate_id": "eed26e1776f1b360ebd771f845d50b6105cb7799",
  "bgs_gemrate_id": "a38b86249278b93aa17ce0db80b65c950c91625e",
  "sgc_gemrate_id": "742f89445dc72a81eef68129af",
  "csg_gemrate_id": "56e77040d18d28402ee89c1ea"
}
```

**注意**: `grader_specific` 记录在合并后**不会自动删除**,可能同时存在!

---

## 2. Irida 中的实现

### 2.1 数据库存储

#### **`variants` 表** (PostgreSQL)
```python
# variants 表中的 source_uid 字段存储 gemrate_id
{
  "id": "variant-uuid",
  "source_id": "catalog_sources.id",  # 指向 GEMRATE 数据源
  "source_uid": "38a505f8c8c60a8a115add01056c6ea6379297a3",  # Gemrate ID
  "variant_title": "2023 Panini Prizm Victor Wembanyama Silver 136",
  "images": ["https://...front.jpg", "https://...back.jpg"]
}
```

#### **`gemrate_data` 表** (PostgreSQL - 从 Snowflake 同步)
```sql
CREATE TABLE gemrate_data (
    gemrate_id VARCHAR(40) PRIMARY KEY,
    population_type VARCHAR(20),  -- 'universal' or 'grader_specific'
    grader VARCHAR(10),            -- 'psa', 'bgs', 'sgc', 'csg', 'cgc'
    
    -- 各评级公司的 gemrate_id
    psa_gemrate_id VARCHAR(40),
    bgs_gemrate_id VARCHAR(40),
    sgc_gemrate_id VARCHAR(40),
    csg_gemrate_id VARCHAR(40),
    cgc_gemrate_id VARCHAR(40),
    
    -- PSA 特有字段
    psa_spec_id INTEGER,           -- PSA SpecID (唯一识别同一张卡的不同评级)
    
    -- 卡牌元数据
    year VARCHAR(20),
    brand VARCHAR(255),
    subject VARCHAR(255),
    card_number VARCHAR(50),
    variety VARCHAR(255),
    
    -- Population 数据
    total_grades INTEGER,          -- 总评级数量
    -- ... 其他字段
);
```

#### **`gemrate_data_grades` 表**
```sql
-- 存储每个评级公司的详细 Population 数据
CREATE TABLE gemrate_data_grades (
    gemrate_id VARCHAR(40),
    grader VARCHAR(10),
    grade VARCHAR(10),
    count INTEGER,
    PRIMARY KEY (gemrate_id, grader, grade)
);
```

### 2.2 数据来源

#### **来源 1: Snowflake (主要来源)**
```python
# Fanatics Data Team 从 Gemrate 获取数据并同步到 Snowflake
snowflake_table = "UKOVTNN_GEMRATE_REPLICATION.DATA.HYBRID_POPULATION_FANATICS"

# Irida 从 Snowflake 同步到 PostgreSQL
# 位置: pipeline/data_source/catalog/ (推测)
```

#### **来源 2: Gemrate API**
```python
# Gemrate 提供的 API
base_url = "https://api.gemrate.com"

# 1. 通过 cert_number 查询
GET /cert-lookup?cert_number={psa_cert_number}&grader=psa

# 2. 通过 gemrate_id 查询
GET /population?gemrate_id={gemrate_id}
```

**API 文档**: https://gemrate.stoplight.io/docs/gemrate

#### **来源 3: Variants 表** (已有数据)
```python
# 当 source = 'GEMRATE' 时, source_uid 就是 gemrate_id
SELECT id, source_uid as gemrate_id 
FROM variants 
WHERE source_id = (SELECT id FROM catalog_sources WHERE name = 'GEMRATE');
```

### 2.3 Gemrate ID 的获取逻辑

**位置**: `rest_api/v2/models/api_def_search.py`

```python
class CollectibleSearchResponseV2(BaseModel):
    def get_gemrate_id(self) -> str:
        """
        获取 Gemrate ID 的优先级策略:
        1. 从 Top 1 结果的 variant_uuid 直接获取
        2. 如果没有,通过 checklist card_uuid 关联获取
        3. 验证一致性 (同一张卡只能有一个 gemrate_id)
        """
        
        # 步骤 1: 查询 Top 1 variant 的 gemrate_id
        top_1_variant_uuid = self.collectibles[0].item_variant_id
        gemrate_id = self.get_gemrate_id_by_variant_uuids([top_1_variant_uuid])
        
        if gemrate_id:
            return gemrate_id
        
        # 步骤 2: 通过 checklist 关联查询
        checklist_card_uuid = get_checklist_card_id_by_variant_uuids([top_1_variant_uuid])
        checklist_variant_uuids = get_checklist_variant_uuid_by_card_uuid([checklist_card_uuid])
        
        # 步骤 3: 获取所有关联 variant 的 gemrate_id
        gemrate_ids = self.get_gemrate_id_by_variant_uuids(checklist_variant_uuids)
        
        # 步骤 4: 验证一致性
        if len(set(gemrate_ids.values())) != 1:
            logger.error(f"多个不同的 gemrate_id 存在于同一张卡: {gemrate_ids}")
            return ""
        
        return list(gemrate_ids.values())[0]
    
    def get_gemrate_id_by_variant_uuids(self, variant_uuids: list[str]) -> dict[str, str]:
        """
        从 variants 表查询 gemrate_id (存储在 source_uid 字段)
        """
        query_sql = select(
            variant_table.c.id,
            variant_table.c.source_uid  # Gemrate ID
        ).select_from(
            variant_table.join(catalog_sources_table)
        ).where(
            and_(
                variant_table.c.id.in_(variant_uuids),
                catalog_sources_table.c.name.in_([
                    'PWCC',           # PWCC 也使用 gemrate_id
                    'GEMRATE',        # Gemrate 官方数据
                    'GEMRATE_CARD_LADDER'  # CardLadder 数据
                ]),
                variant_table.c.source_uid.isnot(None)
            )
        )
        
        results = conn.execute(query_sql).mappings().all()
        return {str(row['id']): row['source_uid'] for row in results}
```

### 2.4 使用场景

#### **场景 1: 图像搜索后返回 Gemrate ID**
```python
# API: POST /v2/collectible_search
{
  "collectible_uri": "https://example.com/card.jpg",
  "top_k": 10
}

# Response
{
  "collectibles": [...],
  "gemrate_id": "38a505f8c8c60a8a115add01056c6ea6379297a3"  # ← 返回给前端
}
```

**用途**:
- 前端跳转到 Gemrate 网站: `https://www.gemrate.com/universal-search?gemrate_id={id}`
- 前端跳转到 CardLadder: `https://app.cardladder.com/search?universalGemRateId={id}`

#### **场景 2: PSA Cert Number → Gemrate ID**
```python
# API: GET /v1/psa_finder/db/{cert_id}
{
  "cert_id": 110836153
}

# Response
{
  "psa_card": {
    "cert_id": 110836153,
    "spec_id": 6754100,              # PSA SpecID
    "gemrate_id": "abc123...",        # Gemrate ID
    "year": "2022",
    "brand": "Bowman Paper Prospects",
    "subject": "Bobby Witt Jr.",
    "grade": "NM-MT 8",
    "total_population": 5
  }
}
```

**数据源**: Snowflake 表 `psa_cards`

#### **场景 3: Checklist 绑定时获取 Gemrate ID**
```python
# 用户绑定 variant 到 checklist card
# 系统自动从 variant 提取 gemrate_id 并关联到 checklist

# 步骤:
1. 用户选择 variant_uuid
2. 系统查询 variants.source_uid (gemrate_id)
3. 关联到 irida_checklist_cards 表
4. 后续搜索可通过 checklist 获取 gemrate_id
```

---

## 3. PSA 与 Gemrate 的关系

### 3.1 PSA SpecID vs. Gemrate ID

| 字段 | 定义 | 唯一性 | 示例 |
|------|------|--------|------|
| **PSA Cert Number** | 每张**评级卡**的唯一编号 | 唯一 | `110836153` |
| **PSA SpecID** | 同一张**原卡**的唯一标识符 | 同一张卡相同,不同卡不同 | `6754100` |
| **Gemrate ID** | 跨评级公司的卡牌标识符 | 跨公司唯一 | `38a505f8...` |

#### **实例说明**
```python
# 同一张卡 (Bobby Witt Jr. 2022 Bowman Paper Prospects BP146 Fuchsia)
# 被 PSA 评了两次,得到两个不同的 Cert Number,但 SpecID 相同

Card 1:
{
  "cert_number": "110836153",  # 评级编号不同
  "spec_id": 6754100,           # SpecID 相同 ✅
  "grade": "NM-MT 8",
  "gemrate_id": "abc123..."     # Gemrate ID 相同 ✅
}

Card 2:
{
  "cert_number": "77821486",   # 评级编号不同
  "spec_id": 6754100,           # SpecID 相同 ✅
  "grade": "NM-MT 8",
  "gemrate_id": "abc123..."     # Gemrate ID 相同 ✅
}
```

### 3.2 PSA API 使用

**位置**: `third_party/psacard/psacard_model.py`

```python
class PSACert(BaseModel):
    cert_number: str           # PSA 证书号
    spec_id: int               # PSA SpecID
    spec_number: str           # PSA SpecNumber
    year: str
    brand: str
    card_number: str
    subject: str
    variety: str
    grade_description: str
    card_grade: str
    total_population: int      # 总存量
    population_higher: int     # 更高评分的存量
```

**API Endpoint**: 
```python
# PSA 官方 API
GET https://www.psacard.com/cert/{cert_number}
```

---

## 4. 数据统计 (截至 2025年5月)

### 4.1 Gemrate 数据表统计

```sql
-- 总记录数
SELECT COUNT(*) FROM gemrate_data;
-- 结果: 6,367,219

-- Universal vs. Grader-Specific
SELECT population_type, COUNT(*) 
FROM gemrate_data 
GROUP BY population_type;
-- universal:        1,296,109
-- grader_specific:  5,071,110

-- 各评级公司分布
SELECT grader, COUNT(*) 
FROM gemrate_data 
WHERE population_type = 'grader_specific' 
GROUP BY grader;
-- psa:  2,694,988
-- sgc:    938,210
-- bgs:  1,112,569
-- csg:    325,343
```

### 4.2 Irida 中的覆盖率

```sql
-- Irida 总卡牌数
SELECT COUNT(*) FROM variants;
-- 结果: ~10,300,000

-- 有 gemrate_id 的卡牌数
SELECT COUNT(DISTINCT source_uid) 
FROM variants v
JOIN catalog_sources cs ON v.source_id = cs.id
WHERE cs.name IN ('PWCC', 'GEMRATE', 'GEMRATE_CARD_LADDER')
  AND v.source_uid IS NOT NULL;
-- 结果: ~1,500,000 (约 15%)

-- 2015年后 Topps 卡牌的覆盖率
SELECT 
  COUNT(*) as total,
  COUNT(CASE WHEN v.source_uid IS NOT NULL THEN 1 END) as with_gemrate
FROM variants v
WHERE v.variant_title ILIKE '%Topps%'
  AND v.year >= '2015';
-- 总数: 2,500,000
-- 有 gemrate_id: 300,000 (12%)
```

---

## 5. 当前限制与问题

### 5.1 数据完整性问题

#### **问题 1: 数据不是最新**
```python
# 现状: gemrate_data 表已经 6 个月未更新
last_update = "2024-11"
current_date = "2025-05"
# Gap: 6 个月

# 影响:
# 1. 新卡无法匹配到 gemrate_id
# 2. Population 数据过时
# 3. 新合并的 universal 记录缺失
```

#### **问题 2: 缺少 Non-Sports 数据**
```python
# gemrate_data 只包含运动卡,不包括:
# - TCG 卡牌 (Pokemon, Magic, Yu-Gi-Oh)
# - Non-sports 卡牌 (电影、漫画等)

# 解决方案: 需要从 Gemrate 单独导入 TCG 数据
```

#### **问题 3: 增量更新机制缺失**
```python
# 现状: 全量替换,没有增量更新
# 问题:
# 1. 更新耗时长
# 2. 可能丢失历史数据
# 3. 无法追踪变更

# 期望: 使用 gemrate_data_incremental 表存储增量
```

### 5.2 合并状态不一致

```python
# 问题: grader_specific 记录在合并后未删除
# 示例:
SELECT COUNT(*) 
FROM gemrate_data g1
WHERE g1.population_type = 'grader_specific'
  AND g1.grader = 'bgs'
  AND EXISTS (
    SELECT 1 FROM gemrate_data g2
    WHERE g2.population_type = 'universal'
      AND g2.bgs_gemrate_id = g1.bgs_gemrate_id
  );
-- 结果: 173 条重复记录

# 影响: 可能导致 population 数据重复计算
```

---

## 6. 使用建议

### 6.1 查询 Gemrate ID 的最佳实践

```python
def get_gemrate_id_safe(variant_uuid: str) -> Optional[str]:
    """
    安全获取 Gemrate ID 的方法
    """
    # 优先级 1: 从 variants.source_uid 获取
    gemrate_id = query_variants_source_uid(variant_uuid)
    if gemrate_id:
        return gemrate_id
    
    # 优先级 2: 从 checklist 关联获取
    card_uuid = get_checklist_card_id(variant_uuid)
    if card_uuid:
        related_variants = get_checklist_variants(card_uuid)
        gemrate_ids = [query_variants_source_uid(v) for v in related_variants]
        gemrate_ids = [g for g in gemrate_ids if g]
        
        if len(set(gemrate_ids)) == 1:
            return gemrate_ids[0]
        elif len(gemrate_ids) > 1:
            logger.warning(f"Multiple gemrate_ids for card {card_uuid}")
    
    return None
```

### 6.2 处理 Universal vs. Grader-Specific

```python
def query_population_data(gemrate_id: str) -> dict:
    """
    查询 Population 数据时考虑合并状态
    """
    # 1. 优先查询 universal 记录
    universal = query_gemrate_data(
        gemrate_id=gemrate_id,
        population_type='universal'
    )
    
    if universal:
        return {
            'gemrate_id': universal['gemrate_id'],
            'psa_population': universal['psa_total_grades'],
            'bgs_population': universal['bgs_total_grades'],
            'sgc_population': universal['sgc_total_grades'],
            'total': sum([...])
        }
    
    # 2. 如果没有 universal,查询所有 grader_specific
    grader_specific = query_gemrate_data(
        psa_gemrate_id=gemrate_id,  # 或 bgs_gemrate_id, sgc_gemrate_id
        population_type='grader_specific'
    )
    
    if grader_specific:
        return {
            'gemrate_id': gemrate_id,
            f'{grader}_population': grader_specific['total_grades'],
            'note': 'Grader-specific only'
        }
    
    return None
```

### 6.3 与外部系统集成

```python
# 1. 跳转到 Gemrate 网站
def get_gemrate_url(gemrate_id: str) -> str:
    return f"https://www.gemrate.com/universal-search?gemrate_id={gemrate_id}"

# 2. 跳转到 CardLadder (获取价格)
def get_cardladder_url(gemrate_id: str) -> str:
    return f"https://app.cardladder.com/search?universalGemRateId={gemrate_id}&via=gemrate"

# 3. 查询 PSA SpecID
def get_psa_spec_id(gemrate_id: str) -> Optional[int]:
    result = query_gemrate_data(gemrate_id=gemrate_id)
    return result.get('psa_spec_id') if result else None
```

---

## 7. 未来改进方向

### 7.1 短期任务 (1-2个月)

#### ✅ **任务 1: 修复 Snowflake 同步**
```python
# 状态: 未更新 6 个月
# 优先级: 🔴 高
# 负责人: Data Team

# 行动:
1. 检查 Snowflake → PostgreSQL 同步脚本
2. 修复同步问题
3. 设置定期同步 (每日/每周)
```

#### ✅ **任务 2: 添加增量更新机制**
```python
# 状态: 缺失
# 优先级: 🔴 高

# 方案:
1. 创建 gemrate_data_incremental 表
2. 记录每次更新的变更
3. 应用变更到主表时保留历史

# 表结构:
CREATE TABLE gemrate_data_incremental (
    id SERIAL PRIMARY KEY,
    gemrate_id VARCHAR(40),
    change_type VARCHAR(20),  -- 'INSERT', 'UPDATE', 'DELETE', 'MERGE'
    old_data JSONB,
    new_data JSONB,
    applied_at TIMESTAMP,
    source VARCHAR(50)
);
```

#### ✅ **任务 3: 添加 TCG 卡牌数据**
```python
# 状态: 缺失
# 优先级: 🟡 中

# 行动:
1. 联系 Gemrate 获取 TCG 数据访问权限
2. 导入 Pokemon, Magic, Yu-Gi-Oh 等数据
3. 更新 gemrate_data 表结构 (添加 category 字段)
```

### 7.2 中期任务 (3-6个月)

#### ✅ **任务 4: 清理 Grader-Specific 重复数据**
```sql
-- 删除已合并到 universal 的 grader_specific 记录
DELETE FROM gemrate_data g1
WHERE g1.population_type = 'grader_specific'
  AND EXISTS (
    SELECT 1 FROM gemrate_data g2
    WHERE g2.population_type = 'universal'
      AND (
        g2.psa_gemrate_id = g1.gemrate_id OR
        g2.bgs_gemrate_id = g1.gemrate_id OR
        g2.sgc_gemrate_id = g1.gemrate_id OR
        g2.csg_gemrate_id = g1.gemrate_id
      )
  );
```

#### ✅ **任务 5: Gemrate ID 覆盖率提升**
```python
# 目标: 从 15% 提升到 30%

# 策略:
1. 对 2010 年后的卡牌优先匹配 gemrate_id
2. 使用 AI 模型生成 gemrate 元数据
3. 建立 Gemrate 元数据向量数据库
4. 通过 embedding 相似度匹配

# 实施:
# 位置: pipeline/checklist_mapping_task/gemrate_mapper.py (新建)
```

#### ✅ **任务 6: 自动化测试工具**
```python
# 目标: 每日测试 gemrate_id 数据质量

# 测试内容:
1. 检查 gemrate_data 表更新时间
2. 验证 universal 与 grader_specific 一致性
3. 检查 population 数据异常值
4. 测试 API 可用性

# 位置: pipeline/scheduled/gemrate_data_validator.py (新建)
```

---

## 8. 常见问题 FAQ

### Q1: Gemrate ID 和 PSA Cert Number 的区别?
```
A: 
- PSA Cert Number: 每张评级卡的唯一编号 (如 110836153)
- PSA SpecID: 同一张原卡的标识符 (如 6754100)
- Gemrate ID: 跨评级公司的卡牌标识符 (如 38a505f8...)

同一张原卡可以有:
- 多个 PSA Cert Number (不同的评级)
- 1个 PSA SpecID
- 1个 Gemrate ID (可能对应多个评级公司)
```

### Q2: 为什么有些卡片没有 Gemrate ID?
```
A: 可能原因:
1. 卡片太新,Gemrate 还未收录
2. 卡片太旧 (<2000年),数据缺失
3. 非主流品牌/系列
4. Non-sports 卡牌 (当前不支持)
5. 自制卡/平行版未收录
```

### Q3: Universal Gemrate ID 和 Grader-Specific 哪个更准确?
```
A: Universal Gemrate ID 更准确,因为:
1. 合并了多个评级公司的数据
2. Population 数据更完整
3. 跨平台通用性更强

但在查询时应该两者都检查,以防合并过程中的遗漏。
```

### Q4: 如何判断一个 Gemrate ID 是否有效?
```python
def is_valid_gemrate_id(gemrate_id: str) -> bool:
    # 1. 格式检查
    if len(gemrate_id) != 40:
        return False
    if not re.match(r'^[a-f0-9]{40}$', gemrate_id):
        return False
    
    # 2. 数据库存在性检查
    exists = query_gemrate_data(gemrate_id=gemrate_id)
    if not exists:
        return False
    
    # 3. Gemrate 网站验证 (可选)
    url = f"https://www.gemrate.com/universal-search?gemrate_id={gemrate_id}"
    response = requests.get(url)
    if response.status_code != 200 or "No results" in response.text:
        return False
    
    return True
```

### Q5: 项目中的 source_uid 都是 Gemrate ID 吗?
```
A: 不一定!

source_uid 的含义取决于 source:
- source = 'GEMRATE' → source_uid = gemrate_id
- source = 'PWCC' → source_uid = gemrate_id (PWCC 使用 Gemrate 系统)
- source = 'PSA' → source_uid = psa_cert_number
- source = 'CHECKLIST' → source_uid = checklist_variant_uuid

所以查询时必须 JOIN catalog_sources 表!
```

---

## 9. 相关资源

### 官方文档
- Gemrate API: https://gemrate.stoplight.io/docs/gemrate
- Gemrate 网站: https://www.gemrate.com
- CardLadder: https://app.cardladder.com

### 项目中的关键文件
```
rest_api/v2/models/api_def_search.py      # Gemrate ID 获取逻辑
rest_api/psa_finder/dao/db.py              # PSA 数据查询
card_db/db_def_record.py                   # SourceValues 定义
third_party/psacard/psacard_model.py       # PSA 数据模型
```

### 数据库表
```
variants                   # 卡牌变体表 (source_uid 存储 gemrate_id)
catalog_sources            # 数据源表
gemrate_data               # Gemrate 主表
gemrate_data_grades        # Population 详细数据
gemrate_data_variants      # Gemrate ID 与 variant 映射
psa_cards (Snowflake)      # PSA 卡牌数据
```

---

## 总结

**Gemrate ID 在 Irida 中是连接不同数据源、评级公司和外部系统的关键桥梁**。理解其工作原理对于:
- ✅ 提高卡牌匹配准确率
- ✅ 获取完整的 Population 数据
- ✅ 集成外部价格/交易数据
- ✅ 构建完整的卡牌知识图谱

至关重要!
