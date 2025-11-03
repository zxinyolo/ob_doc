#### celery怎么重试任务？

celery是一个基于分布式消息传递的异步任务队列/任务调度框架，用于处理耗时任务，它能把任务从主进程中解耦出来，交给后台worker异步执行

- 异步任务执行（发送邮件，图片上传，文件上传）
- 定时任务调度
- 大规模分布式任务分发与容错

核心组件：

- Broker（消息中间件）：负责消息中转，例如rabbitmq，reids。生产者将任务发送到broker，消费者从中取出任务
- Worker（工作进程）：实际执行任务的节点，可以分布在不同的机器上
- Task（任务）：被定义的python函数，通过装饰器@app.task声明
- Result Backend（结果存储）：存放任务结果，用于状态查询，任务链，回调等，支持redis，数据库
- Beat（任务调度器）：用于任务的周期性调度

调用任务的方法：

| .delay()       | 快捷方式，异步提交任务（最常用）                 |
| -------------- | ------------------------------------------------ |
| .apply_async() | 高级方式，可控制队列、重试、延迟时间、超时等参数 |
| .apply()       | 同步执行（立即在本地执行，不经过队列）           |

定时任务：

- **Celery Beat**，它是一个单独的进程，

- 负责定期把任务发布到 Broker，由 Worker 执行。



#### celery底层是怎么分配、调度任务的，多worker，启动多少个worker怎么决定？

- 通过 **Broker** 实现任务分发（任务消息排队）；
- 通过 **Worker 并发池** 实现本地执行调度；
- 多个 Worker 监听同一个队列时，任务分配采用 **竞争消费模型**；
- 每个 Worker 的并发数由 --concurrency 决定，默认等于 CPU 核数；
- 可以通过配置不同的队列或路由规则，实现多 Worker 的任务隔离与负载均衡。



#### celery任务的优先级怎么配置的

Celery 的任务优先级是通过 **Broker 支持的优先级队列机制** 实现的，需要在 Broker（如 RabbitMQ 或 Redis 5.0+）支持的前提下，通过 apply_async(priority=n) 或任务队列声明中的 queue_arguments={'x-max-priority': 10} 来实现优先级调度。



#### celery怎么排查错误的，任务丢失

| **阶段**           | **关键组件**   | **常见问题**                        | **排查手段**                      |
| ------------------ | -------------- | ----------------------------------- | --------------------------------- |
| 1️⃣ 生产者发任务     | Celery Client  | delay/apply_async 没真正 push 出去  | 启动 debug 日志、检查 broker 连接 |
| 2️⃣ Broker 消息队列  | Redis/RabbitMQ | 队列满、key 过期、掉线、持久化没开  | CLI 或管理台查看队列消息数        |
| 3️⃣ Worker 消费任务  | celery worker  | worker 未启动、路由错、任务导入异常 | 启动时加 -l DEBUG 观察注册任务    |
| 4️⃣ Backend 保存结果 | Redis/Database | 任务执行完但结果没保存              | 检查 backend 配置、任务状态查询   |

app.control.inspect().active() # 查看活跃任务

app.control.inspect().reserved() # 查看待执行任务

- Worker 异常退出或系统崩溃

  - 检查日志是否有崩溃退出日志
  - 调整 task_acks_late=True；
  - 配合 broker_transport_options={'visibility_timeout': 3600}；
  - 避免任务执行太久未 ack。

- Celery的内部调试命令

  celery -A tasks report         # 查看配置与环境

  celery -A tasks inspect registered   # 查看已注册任务

  celery -A tasks inspect active     # 查看当前正在执行的任务

  celery -A tasks inspect reserved    # 查看队列中待执行的任务

  celery -A tasks inspect revoked    # 查看被撤销的任务

  celery -A tasks inspect stats     # Worker 状态统计

- 防止任务真正丢失

  app.conf.update(

    task_acks_late=True,         # 任务执行完再 ack，防止中途丢失

    broker_transport_options={'visibility_timeout': 3600}, # 任务重现时间

    task_reject_on_worker_lost=True,   # Worker 崩溃后重新入队

    result_expires=3600*24,       # 结果保存 1 天

    task_default_delivery_mode='persistent', # 任务消息持久化

  )

- 总结

  [生产者发送任务]

     ↓

  Broker有消息？ → 否 → 检查broker连接/配置

     ↓

  Worker收到任务？ → 否 → 检查task注册/启动参数

     ↓

  Worker执行成功？ → 否 → 查看日志/traceback

     ↓

  Result保存？ → 否 → 检查backend配置/过期时间



#### Celery日志是什么样配置的

- 可以使用celery的启动参数添加日志输出路径和配置
- 可以使用主服务的配置好的log对象

- Celery 默认拥有一整套独立的日志系统，会自动捕获并格式化所有日志输出。但你可以通过 @signals.setup_logging.connect 手动接管它，从而与 FastAPI 或其他服务统一日志体系。

| **模式**                       | **建议**                                                     |
| ------------------------------ | ------------------------------------------------------------ |
| **本地开发**                   | 让 Celery 自己管理日志（简单易看）                           |
| **正式环境 / 微服务架构**      | 使用 signal 接管 logging，与 FastAPI 统一格式、trace_id、stdout 输出 |
| **日志收集系统（ELK / Loki）** | 输出 JSON 到 stdout，由容器日志系统采集                      |



#### 一个celery中起多个worker和不同的服务器起worker的区别

| **对比项**        | **同机多 Worker**          | **跨机多 Worker**         |
| ----------------- | -------------------------- | ------------------------- |
| Broker 连接       | 共享本地 socket 连接（快） | TCP 网络连接（慢一点）    |
| IPC（进程间通信） | OS 层共享内存或队列        | 通过 Broker 间接通信      |
| 调度器（Beat）    | 可单独运行在一台机器       | 通常只在一个节点启动 Beat |
| 容灾性            | 机器宕机会全挂             | 只影响单节点              |
| 扩展性            | 受限于单机性能             | 理论上无限横向扩展        |



#### 常见问题

1. **Worker 内存暴涨 / OOM（Out Of Memory）**
   - 场景
     - 长时间运行的的worker内存持续增加
     - 最后被系统杀死
     - 重启后恢复
   - 常见原因
     - 任务结果未清理，backend 不断堆积；
     - 大量返回大对象（列表、图像、DataFrame）；
     - 未使用 acks_late 导致任务重复执行；
     - 内存未释放（对象引用、C 扩展泄露）。
   - 排查与解决
     - celery -A app worker --max-tasks-per-child=100，每执行 100 个任务重启一次子进程，强制释放内存。
     - 若任务结果不需要存储：app.conf.task_ignore_result = True
     - 定期清理 backend：celery -A app purge
2. Worker CPU 飙高 / 卡死
   - 常见原因
     - cpu密集型任务（如加密，图片处理）在同步执行
     - 未开启prefork
     - 任务中自信呢死循环或阻塞IO
   - 优化建议
     - 将cpu密集型任务分配到单独队列
     - 使用多进程并行：celery -A app worker -Q cpu_tasks --concurrency=8 --pool=prefork
     - IO密集型任务建议使用gevent：celery -A app worker --pool=gevent --concurrency=1000

3. Redis连接丢失/BrokenPipeError

   - 原因
     - redis重启
     - 连接池未释放
     - Worker过多同时连接Redis
   - 解决方案
     - 增加Broker心跳：app.conf.broker_transport_options = {'visibility_timeout': 3600, 'socket_keepalive': True}
     - 使用持久连接池（如redis_max_connections)
     - 如果频繁掉线，考虑RabbitMQ替代Redis

4. Redis队列卡住/backlog堆积

   - 场景
     - Redis任务队列key中堆积几万条任务
     - Worker吃不完
     - 消息延迟严重
   - 排查思路
     - 查看任务堆积：redis-cli LLEN celery
     - 检查是否有Worker崩溃
     - 检查是否有任务重试死循环
     - 检查是否有优先级队列或路由配置不当
   - 解决
     - 临时扩容Worker
     - 增加并发度
     - 为长时间单独分队列
     - 使用task_routes控制分流

5. 任务丢失

   - 场景

     - Producer执行task.delay()
     - worker没有执行
     - 日志中无报错

   - 原因分析：

     - Worker崩溃或未启动
     - Broker使用Redis且任务过期(visibility_timeout)
     - ACK丢失（Worker收到但未确认）
     - 任务结果被清理（结果存储TTL）
     - task_acks_late=False且Worker被kill

   - 解决方案：

     - 使用持久化队列：

       app.conf.task_acks_late = True
       app.conf.worker_prefetch_multiplier = 1
       app.conf.broker_transport_options = {'visibility_timeout': 3600}

     - 开启acks_late: 任务执行完再确认
     - 监控任务ID+backend确认

6. 重复执行任务

   - 原因：
     - Worker在执行任务时宕机
     - 未启用acks_late
     - Broker认为任务未完成又重新投递
   - 修复
     - 开启acks_late=True
     - 幂等性设计：任务自身支持重复执行不出错

7. 任务卡死不返回

   - 原因：
     - 子线程/子进程阻塞
     - Deadlock
     - 多线程访问共享资源
     - Backend未响应
   - 排查：
     - celery inspect active查看挂起任务
     - strace或gdb看阻塞点
     - 任务中加超时：app.conf.task_time_limit = 300