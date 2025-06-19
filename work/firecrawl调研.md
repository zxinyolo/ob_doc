#### 1. 用户请求 API（如 /v0/scrape）apps/api/src/routes/v0.ts

- /scrape - 单页网页组曲
- /crawl  - 站点级爬取（递归抓取多个页面）
- //crawl/status/:jobId - 插曲爬取任务状态
- /crawl/cancel/:jobId - 取消爬取任务
- /search - 网页内容查询

#### 2. 参数校验、鉴权、额度检查

- 对参数的合法性校验  apps/api/src/controllers/v0/scrape.ts
- 用户身份鉴权  apps\api\src\controllers\auth.ts
- 查询用户身份和额度 apps\api\src\services\billing\credit_billing.ts
- 判断用户的url是非在黑名单 apps\api\src\scraper\WebScraper\utils\blocklist.ts

#### 3. 任务入队，分配唯一 jobId

- 为每个抓取任务生成唯一的jobid apps\api\src\services\queue-jobs.ts
- 将任务参数，jobid优先级信息放入队列 apps\api\src\services\queue-service.ts
- 支持任务排队，重试，分布式扩展

#### 4. 爬虫引擎抓取网页内容（HTML/Markdown/截图/元数据）

- 引擎的选择，根据参数和网页特性，自动选择最合适的爬虫引擎 apps/api/src/scraper/scrapeURL/engines/
- 浏览器微服务，需要js渲染/反爬/截图，任务会被分发到浏览器微服务  apps\playwright-service-ts\api.ts
- 内容处理：
  - 获取原始的HTML 
  - 转为markdown，去除无用标签，便于后续处理 apps/api/src/lib/html-to-markdown.ts
  - 截图
  - 提取元数据，url，状态码等

#### 5. 如需结构化抽取，自动拼接 prompt/schema，调用 LLM

- 判断是否配置了大模型，是否需要LLM
- 内容精简，自动截断/分块网页内容，确保不超出大模型的上下文限制
- 拼接prompt/schema
- 调用大模型，通过api调用，获得结构化json结果

#### 6. 结果处理、异常修正、计费

#### 7. 返回结构化数据/原始内容/截图等

