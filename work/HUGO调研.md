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

