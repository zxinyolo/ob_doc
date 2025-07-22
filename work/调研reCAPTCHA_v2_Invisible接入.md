**前端加载验证码控件：**

- 与复选框模式类似，通过 `<script src="https://www.google.com/recaptcha/api.js" async defer></script>` 加载验证码。

- 页面中配置按钮（或表单）绑定 Invisible 模式的回调：

  ```
  <button
    class="g-recaptcha"
    data-sitekey="你的SiteKey"
    data-callback="onSubmit"
    data-size="invisible">
    提交
  </button>
  ```

**前端第一次校验（客户端校验）：**

- 用户点击按钮后，Invisible reCAPTCHA 自动调用 Google 校验，无需显示复选框或图片验证，Google 在后台判断用户是否为人类。
- 验证通过后，自动生成 `g-recaptcha-response` 并调用指定的回调函数 `onSubmit`。

**前端提交到后端：**

- 回调函数 `onSubmit(token)` 中，前端将获取到的 `g-recaptcha-response` 提交到后端服务器。

**后端第二次校验（服务端校验）：**

- 与复选框模式相同，后端使用 `secret` 和 `g-recaptcha-response` 调用 Google 接口完成最终验证。
- 验证通过后再继续业务流程。

