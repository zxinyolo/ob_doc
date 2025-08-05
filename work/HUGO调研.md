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

