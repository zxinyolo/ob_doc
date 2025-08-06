# HUGO 调研

#### 1. 安装

```shell
scoop install hugo-extended
```



#### 2. 基本用法

##### 2.1 测试是否安装成功

```shell
hugo version

hugo v0.105.0-0e3b42b4a9bdeb4d866210819fc6ddcf51582ffa+extended linux/amd64 BuildDate=2022-10-28T12:29:05Z VendorInfo=snap:0.105.0
```

##### 2.2 查看可用命令

```shell
hugo help

hugo server --help
```

##### 2.3 构建站点

```shell
hugo
```

执行构建站点命令是指把源文件（markdown文件，模板，配置等）转换成可以在Web服务器上运行的静态HTML网站的过程

所有生成的文件都会放在**public**目录中

>Hugo构建站点前不会清空public目录，现有同名文件会被覆盖，但是不会被删除，会被保留下来

##### 2.4 草稿、未来和过期内容

Hugo允许在内容的前置元数据中设置**draft**，**data**，**publishDate**，**expiryDate**。默认情况下，Hugo在以下情况下不会发布内容：

- draft的值为true
- date在未来
- publishDate在未来
- expiryDate在过去

可以在运行hugo或hugo server时使用命令行标志覆盖默认行为：

```shell
hugo --buildDrafts    # 或 -D
hugo --buildExpired   # 或 -E
hugo --buildFuture    # 或 -F
```

##### 2.5 开发和测试站点

需要查看站点的内容，需要进入项目目录运行命令：

```shell
hugo server
```

hugo server命令会将站点构建到内存中，使用一个最小的HTTP服务器提供页面

```shell
Web服务器可用于 http://localhost:1313/ 
```



#### 3. 目录结构

##### 3.1 站点骨架

创建新站点，Hugo会生成一个项目骨架：

```shell
hugo new site my-site
```

会创建以下目录：

```shell
my-site/
├── archetypes/
│   └── default.md
├── assets/
├── content/
├── data/
├── i18n/
├── layouts/
├── static/
├── themes/
└── hugo.toml         <-- 站点配置
```

也可以将配置文件放到目录中：

```shell
my-site/
├── archetypes/
│   └── default.md
├── assets/
├── config/           <-- 站点配置
│   └── _default/
│       └── hugo.toml
├── content/
├── data/
├── i18n/
├── layouts/
├── static/
└── themes/
```

构建站点，Hugo会创建一个public目录，还会创建一个resources目录：

```shell
my-site/
├── archetypes/
│   └── default.md
├── assets/
├── config/       
│   └── _default/
│       └── hugo.toml
├── content/
├── data/
├── i18n/
├── layouts/
├── public/       <-- 构建站点时创建
├── resources/    <-- 构建站点时创建
├── static/
└── themes/
```

##### 3.2 目录

- **archetypes**： 目录包含用于新内容的模板
- **assets**：包含通常通过资源管道传递的全局资源，包括图片，CSS，Sass，js，ts等资源
- **config**： 包含站点配置
- **content**：包含组成站点内容的标记文件（通常是markdown）和页面资源
- **data**： 包含用户增强内容，配置，本地化和导航的数据文件（JSON，TOML，YAML或XML）
- **i18n**： 包含多语言站点翻译表
- **layouts**：包含将内容、数据、和资源转换为网站的模板
- **public**： 包含发布的网站，在用hugo命令时生成
- **resources**：包含Hugo资源管道的缓存输出，在运行hugo或hugo server命令时生成。此缓存目录包括CSS和图片
- **static**： 包含在构建站点时将复制到public目录的文件
- **themes**： 包含一个或多个主题

#### 4. 内容管理

##### 4.1 创建内容页面

使用Hugo命令创建新页面：

```shell
hugo new content/about.md
hugo new content/products.md
hugo new content/blog.md
```

##### 4.2 Front Matter配置

每个Markdown文件开头的元数据配置：

```yaml
---
title: "产品展示"
description: "我们提供创新的技术解决方案"
keywords: ["产品", "技术", "解决方案"]
date: 2024-01-15
draft: false
---
```

##### 4.3 内容组织

- 使用清晰的目录结构组织内容
- 每个页面都应该有独特的标题和描述
- 合理使用标题层级（H1, H2, H3等）

#### 5. 模板系统

##### 5.1 模板层次结构

Hugo的模板系统采用继承和组合的方式：

```
baseof.html (基础模板)
├── index.html (首页模板)
├── single.html (单页模板)
├── list.html (列表模板)
└── partials/ (组件模板)
    ├── header.html
    ├── footer.html
    └── nav.html
```

##### 5.2 基础模板 (baseof.html)

定义网站的整体HTML结构：

```html
<!DOCTYPE html>
<html lang="{{ .Site.Language.Lang }}">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ .Title }} | {{ .Site.Title }}</title>
    <meta name="description" content="{{ .Description | default .Site.Params.description }}">
    {{ partial "head.html" . }}
</head>
<body>
    {{ partial "header.html" . }}
    <main>
        {{ block "main" . }}{{ end }}
    </main>
    {{ partial "footer.html" . }}
</body>
</html>
```

##### 5.3 页面模板

**首页模板 (index.html):**

```html
{{ define "main" }}
<section class="hero">
    <h1>{{ .Site.Title }}</h1>
    <p>{{ .Site.Params.description }}</p>
</section>
{{ end }}
```

**单页模板 (single.html):**

```html
{{ define "main" }}
<article>
    <h1>{{ .Title }}</h1>
    <div class="content">
        {{ .Content }}
    </div>
</article>
{{ end }}
```

##### 5.4 组件模板 (Partials)

**导航组件 (header.html):**

```html
<header>
    <nav>
        <a href="/">{{ .Site.Title }}</a>
        <ul>
            <li><a href="/products">产品</a></li>
            <li><a href="/about">关于我们</a></li>
            <li><a href="/contact">联系我们</a></li>
        </ul>
    </nav>
</header>
```

#### 6. 样式和交互

##### 6.1 CSS组织

将样式文件放在 `static/css/` 目录：

```css
/* static/css/style.css */
:root {
    --primary-color: #2563eb;
    --secondary-color: #64748b;
    --text-color: #1e293b;
}

/* 响应式设计 */
@media (max-width: 768px) {
    .container {
        padding: 0 1rem;
    }
}
```

##### 6.2 JavaScript功能

添加交互功能：

```javascript
// static/js/main.js
document.addEventListener('DOMContentLoaded', function() {
    // 移动端菜单切换
    const menuToggle = document.querySelector('.menu-toggle');
    const navMenu = document.querySelector('.nav-menu');
    
    menuToggle.addEventListener('click', function() {
        navMenu.classList.toggle('active');
    });
});
```

#### 7. 配置优化

##### 7.1 基础配置 (hugo.toml)

```toml
baseURL = "https://mycompany.com"
languageCode = "zh-cn"
title = "我的公司"
defaultContentLanguage = "zh-cn"

[params]
  description = "专业的技术解决方案提供商"
  keywords = "技术,解决方案,创新"
  author = "公司名称"

[menu]
  [[menu.main]]
    name = "首页"
    url = "/"
    weight = 10
  [[menu.main]]
    name = "产品"
    url = "/products"
    weight = 20
  [[menu.main]]
    name = "关于我们"
    url = "/about"
    weight = 30
```

##### 7.2 SEO优化配置

```toml
[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true
  [markup.highlight]
    style = "github"
    lineNos = true

[sitemap]
  changefreq = "monthly"
  priority = 0.5
```

#### 8. 实践经验总结

##### 8.1 开发流程

1. **项目初始化**

   ```shell
   hugo new site mycompany
   cd mycompany
   ```

2. **创建内容**

   ```shell
   hugo new content/products.md
   hugo new content/about.md
   ```

3. **开发调试**

   ```shell
   hugo server -D
   ```

4. **构建发布**

   ```shell
   hugo
   ```

##### 8.2 常见问题解决

**问题1：Git相关错误**

```
ERROR Failed to read Git log: fatal: not a git repository
```

解决方案：在hugo.toml中设置 `enableGitInfo = false`

**问题2：模板找不到**
确保模板文件名正确：

- `index.html` - 首页
- `single.html` - 单页
- `baseof.html` - 基础模板

**问题3：中文字符显示问题**
在hugo.toml中设置：

```toml
languageCode = "zh-cn"
defaultContentLanguage = "zh-cn"
```

##### 8.3 性能优化

1. **图片优化**
   - 使用WebP格式
   - 实现懒加载
   - 响应式图片

2. **CSS优化**
   - 使用CSS变量
   - 移动端优先设计
   - 压缩CSS文件

3. **JavaScript优化**
   - 按需加载
   - 代码分割
   - 使用现代ES6+语法

##### 8.4 SEO最佳实践

1. **内容优化**
   - 每个页面独特的title和description
   - 合理的标题层级结构
   - 丰富的内容和关键词

2. **技术优化**
   - 语义化HTML标签
   - 结构化数据
   - 移动端友好设计

3. **性能优化**
   - 快速的页面加载速度
   - 良好的用户体验
   - 无障碍访问支持

#### 9. 部署和维护

##### 9.1 静态托管

Hugo生成的静态文件可以部署到：

- GitHub Pages
- Netlify
- Vercel
- 阿里云OSS
- 腾讯云COS

##### 9.2 自动化部署

使用GitHub Actions自动部署：

```yaml
# .github/workflows/hugo.yml
name: Deploy Hugo site
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Setup Hugo
      uses: peaceiris/actions-hugo@v2
      with:
        hugo-version: 'latest'
    - name: Build
      run: hugo --minify
    - name: Deploy
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./public
```

##### 9.3 内容管理

1. **版本控制**
   - 使用Git管理源代码
   - 定期备份内容文件

2. **内容更新**
   - 建立内容更新流程
   - 定期检查链接有效性

3. **监控维护**
   - 使用Google Analytics监控访问
   - 定期更新Hugo版本

#### 10. 进阶应用

##### 10.1 多语言支持

```toml
[languages]
  [languages.zh-cn]
    title = "我的公司"
    weight = 1
  [languages.en]
    title = "My Company"
    weight = 2
```

##### 10.2 自定义短代码

创建可复用的内容组件：

```html
<!-- layouts/shortcodes/youtube.html -->
<div class="video-wrapper">
    <iframe src="https://www.youtube.com/embed/{{ .Get 0 }}" 
            frameborder="0" allowfullscreen></iframe>
</div>
```

使用方式：

```markdown
{{< youtube "dQw4w9WgXcQ" >}}
```

##### 10.3 数据驱动内容

使用data文件管理结构化数据：

```yaml
# data/team.yaml
members:
  - name: "张三"
    position: "技术总监"
    bio: "10年技术开发经验"
  - name: "李四"
    position: "产品经理"
    bio: "专注用户体验设计"
```

在模板中使用：

```html
{{ range .Site.Data.team.members }}
<div class="team-member">
    <h3>{{ .name }}</h3>
    <p>{{ .position }}</p>
    <p>{{ .bio }}</p>
</div>
{{ end }}
```

#### 11. 总结

Hugo是一个功能强大、性能优秀的静态网站生成器，特别适合：

- **内容型网站**：博客、文档、企业官网
- **性能要求高**：需要快速加载的网站
- **SEO友好**：搜索引擎优化要求高的项目
- **开发效率**：快速构建和部署的需求

