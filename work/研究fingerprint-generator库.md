# fingerprint-generator 

> scrapfly/fingerprint-generator 是一个 Python 库，它的作用是生成模拟真实浏览器的指纹信息

## 作用

主要目的是让你的自动化程序（比如网络爬虫、自动化测试脚本）在访问网站时，**看起来更像一个普通的、由真人操作的浏览器**，而不是一个机器人。通过生成逼真的 HTTP 请求头 (`Headers`) 和浏览器环境信息 (`navigator` 对象属性)，可以有效**降低被网站的反爬虫或反机器人系统检测并阻止的概率**。

## 基本用法

1.  导入 库
2.  调用库提供的**生成函数**，通常你可以指定想要的**浏览器类型**（如 `'chrome'`, `'firefox'`）和**操作系统**（如 `'windows'`, `'linux'`, `'macos'`）
3.  函数会返回一个包含指纹数据的对象或字典，里面有可以直接使用的 `headers` 字典，以及 `navigator` 对象的各种属性值

```python
from fingerprint_generator import FingerprintGenerator

# 初始化生成器 
generator = FingerprintGenerator()

# 指定需要生成的指纹类型
fingerprint_data = generator.generate(browser='chrome', os='windows', version_range={'min': 100, 'max': 105})

# 获取生成的 Headers
http_headers = fingerprint_data['headers']

# 获取模拟的 Navigator 属性
navigator_properties = fingerprint_data['navigator']

# 在HTTP 请求客户端中使用这些 Headers
# 在需要模拟 JS 环境时，可以使用 navigator_properties
print("Generated Headers:", http_headers)
print("Generated Navigator Properties:", navigator_properties)
```