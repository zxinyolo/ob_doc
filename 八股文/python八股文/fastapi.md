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

