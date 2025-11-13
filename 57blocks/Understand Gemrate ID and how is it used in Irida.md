

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
     - 历史评级数据喝评级分数都可以统一查询
3. 数据分析喝统计更高效
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