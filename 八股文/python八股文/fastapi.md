 #### FastAPI 请求的完整生命周期

1. ASGI声明周期入口：当请求进来时，ASGI server（如 **uvicorn**）执行：



#### pydantic

1. 检验输入参数（请求体、查询参数、路径参数、环境变量等）
2. 将输入数据转化为强类型的python对象
3. 自动生成json schema
4. 保证输出数据的类型一致性
5. pydantic自由在继承了BaseModel时才会生效，会自动读取类型注解(__annotations__) 并构建字段定义
6. FastAPI 根据你的路由函数签名 → 提取参数与模型 → 调用Pydantic 生成 JSON Schema → 组合成 OpenAPI 文档。



#### 依赖注入

