## reCAPTCHA_v2复选框模式接入

>使用：使用谷歌邮箱登录https://cloud.google.com/security/products/recaptcha?hl=zh_cn，点击开始使用 创建类型，拿到id和key
>
>收费策略：免费版每月免费使用10000次，使用完后安全性会降低或者报错

**前端加载验证码控件：**

- 前端页面加载时，通过 `<script src="https://www.google.com/recaptcha/api.js" async defer></script>` 加载 Google 提供的验证码脚本。
- 页面中添加 `<div class="g-recaptcha" data-sitekey="你的SiteKey"></div>`，Google 会渲染一个“我不是机器人”的复选框。

**前端第一次校验（客户端校验）：**

- 用户勾选复选框，可能会触发 Google 弹出图像验证面板（取决于风险评估）。
- 验证成功后，Google 会自动生成一个校验凭证 `g-recaptcha-response`，并填入隐藏的 `<textarea>` 元素中。

**前端提交到后端：**

- 表单提交或按钮点击时，前端将表单中的 `g-recaptcha-response` 一并提交给后端服务器。

**后端第二次校验（服务端校验）：**

- 后端从请求中获取 `g-recaptcha-response`，并将其与密钥 secret 一起提交到 Google 的校验接口：

  ```
  POST https://www.google.com/recaptcha/api/siteverify
  ```

  参数包括：

  - `secret`: 后端私钥
  - `response`: 前端传来的 `g-recaptcha-response`
  - `remoteip`: （可选）用户 IP

- Google 返回 JSON 结果：

  ```
  {
    "success": true,
    "challenge_ts": "...",
    "hostname": "..."
  }
  ```

- 后端根据 `success` 字段判断验证是否通过。