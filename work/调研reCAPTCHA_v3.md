## reCAPTCHA_v2_Invisible

>使用：使用谷歌邮箱登录https://cloud.google.com/security/products/recaptcha?hl=zh_cn，点击开始使用 创建类型，拿到id和key
>
>收费策略：免费版每月免费使用10000次，使用完后安全性会降低或者报错

**前端加载验证码控件：**

- 通过 `<script src="https://www.google.com/recaptcha/api.js?render=你的SiteKey"></script>` 异步加载 reCAPTCHA v3 脚本。
- 在用户操作触发时（如提交按钮点击），调用 `grecaptcha.execute('你的SiteKey', {action: '操作名'})`，执行验证。

**前端第一次校验（客户端校验）：**

- `grecaptcha.execute` 会生成一个 token（reCAPTCHA 响应令牌），该令牌代表本次交互的风险评估。
- token 是一个短期有效的字符串，用于后续服务器端验证。

**前端提交到后端：**

- 将获取的 token（通常放在请求体中）和其他表单数据一起发送到后端服务器。

**后端第二次校验（服务端校验）：**

- 后端接收到 token 后，使用服务端 `secret key` 调用 Google 官方验证接口：

  ```
  ruby复制编辑POST https://www.google.com/recaptcha/api/siteverify
  参数：
    secret=你的SecretKey
    response=客户端提交的token
    remoteip=用户IP（可选）
  ```

- Google 返回 JSON 格式的验证结果，包含 `success`、`score`、`action`、`challenge_ts`、`hostname` 等字段。


**后端结果处理：**

- 根据 `success` 字段判断验证是否成功。
- 根据 `score` 字段判断风险程度（通常阈值设置为 0.5），并且检查 `action` 与前端发送的一致。
- 验证通过后，继续执行业务逻辑；失败则阻止操作或提示用户。