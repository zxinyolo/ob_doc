 #### FastAPI 请求的完整生命周期

1. ASGI声明周期入口：当请求进来时，ASGI server（如 **uvicorn**）执行：await app(scope, receive, send)， 此时fastapi此时接管请求
2. 中间件处理
3. 路由匹配
4. 构建请求上下文Request对象
5. 依赖注入，fastapi如果发现参数时Depends(...)，递归解析出子依赖，形成一颗依赖树，调用内部函数，这个函数会自动执行依赖

HTTP 请求  →  路由匹配  →  提取参数  →  调用依赖注入系统（Depends）  
           →  构造请求体模型（Pydantic Model）  
           →  调用 Pydantic 验证器（Validators）  
           →  如果通过 → 调用业务函数  
           →  如果不通过 → 抛出 HTTP 422 错误（ValidationError）



#### 参数校验

1. 轻量级校验： 直接在参数声明定义，使用 fastapi.Query()、Path() 等

2. 结构化校验： 使用Pydantic模型

   ```python
   from pydantic import BaseModel, EmailStr, constr
   
   class User(BaseModel):
       username: constr(min_length=3, max_length=20)
       email: EmailStr
       phone: constr(pattern=r"^1[3-9]\d{9}$")  # 中国手机号正则
   ```

3. 高级定制： 自定义验证器

   ```python
   from pydantic import BaseModel, field_validator, ValidationError
   
   class User(BaseModel):
       phone: str
   
       @field_validator("phone")
       @classmethod
       def check_phone(cls, v):
           if not v.startswith("1") or len(v) != 11:
               raise ValueError("手机号格式不正确")
           return v
   ```

4. 使用依赖注入

#### pydantic

1. 检验输入参数（请求体、查询参数、路径参数、环境变量等）
2. 将输入数据转化为强类型的python对象
3. 自动生成json schema
4. 保证输出数据的类型一致性
5. pydantic自由在继承了BaseModel时才会生效，会自动读取类型注解(__annotations__) 并构建字段定义
6. FastAPI 根据你的路由函数签名 → 提取参数与模型 → 调用Pydantic 生成 JSON Schema → 组合成 OpenAPI 文档。



#### 依赖注入

​	在 FastAPI 中，依赖注入（Dependency Injection）是一种通过 Depends 机制，将外部依赖（如认证、数据库、配置、业务逻辑）解耦并由框架自动创建和注入的机制。

它让函数之间的依赖关系显式化，可复用、可测试，并且支持异步、缓存和嵌套调用，是 FastAPI 可组合性和可维护性的核心设计之一

优点：

- 解耦：让业务函数不依赖具体实现，只依赖抽象借口
- 复用：把通用逻辑（认证、数据库连接、配置等）提取成独立依赖
- 可测试性：依赖可以被轻松替换、mock，单元测试更简单；
- 可维护性：依赖关系清晰、显式声明在函数签名中
- 可组合性：支持嵌套依赖（一个依赖还可以依赖其他依赖），形成灵活的组合；
- 一致性：FastAPI 自动执行依赖、处理错误、生成文档，避免重复代码。



#### docker部署时一盒容器中启动多个workers和启动多个一摸一样的容器的区别

​	在 FastAPI 部署时，使用 --workers 参数启动多个 Uvicorn 进程可以充分利用单机多核资源，但这些进程仍然共享同一个宿主环境。

如果我们使用 Docker 启动多个实例，那么每个实例都是独立的容器，拥有自己的资源和网络空间，可以通过负载均衡器进行分发，这种方式更适合分布式扩容。

一般生产环境会结合使用：容器内多 worker + 多容器水平扩展



#### FastAPI 高效的原因主要有三点：

1️⃣ 基于 Starlette 的 ASGI 异步框架，支持 async/await 非阻塞请求处理，高并发下性能极佳；

2️⃣ 参数和请求体的校验交给 Pydantic，高性能 JSON 解析和类型校验无需额外逻辑；

3️⃣ 轻量路由和依赖注入系统让代码解耦、复用，同时支持异步资源，整体吞吐量非常高。

综合来看，FastAPI 在处理高并发请求和复杂数据验证场景下比传统同步框架效率更高。



#### Fastapi的异步写法需要注意什么

- async def 定义异步函数（协程函数）
- await 用户挂起协程并等待异步结果
- python的异步时单线程时间循环调度，不是多线程
- cpu密集型任务人会阻塞时间循环，徐哟啊放荡线程池或进程池

1. 不要在 async 函数里调用阻塞 I/O（整个worker的请求被阻塞）
2. 对数据库/HTTP/文件操作，要使用 **异步库**（整个worker的请求被阻塞）
3. 不要混用阻塞库和 async
4. CPU 密集型任务要放线程池/进程池
5. async 函数之间必须 await （不会导致其他请求阻塞，只是你的 async 函数逻辑不会跑）
6. 避免在依赖函数里做阻塞操作