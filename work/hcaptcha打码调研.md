# Hcaptcha打码调研

## 背景

### 部分验证码图片类型

<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJw18Si0ueZHNAgvNAfywBGiA_EA8mp4IjDRK8JEiAA_508_523.png" alt="lQLPJw18Si0ueZHNAgvNAfywBGiA_EA8mp4IjDRK8JEiAA_508_523" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJwb7AVuZZ3HNAkzNAYywC_RMExwhPHcIjDMU71YnAA_396_588.png" alt="lQLPJwb7AVuZZ3HNAkzNAYywC_RMExwhPHcIjDMU71YnAA_396_588" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJwhMbX1mOpHNAlHNAY-wIWKwpsJH7VkIjDOm_5goAA_399_593.png" alt="lQLPJwhMbX1mOpHNAlHNAY-wIWKwpsJH7VkIjDOm_5goAA_399_593" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJwjCqXkJDTHNAgzNAgOwGOCK8gs9agsIjDJ4DJWCAA_515_524.png" alt="lQLPJwjCqXkJDTHNAgzNAgOwGOCK8gs9agsIjDJ4DJWCAA_515_524" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJw-Ly-tnFRHNAkrNAYqwmDf2tq1BUYUIjDSMziVhAA_394_586.png" alt="lQLPJw-Ly-tnFRHNAkrNAYqwmDf2tq1BUYUIjDSMziVhAA_394_586" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJwo3jfsHTvHNAgzNAgmwv6YoN0RnERsIjDJN8Ki1AA_521_524.png" alt="lQLPJwo3jfsHTvHNAgzNAgmwv6YoN0RnERsIjDJN8Ki1AA_521_524" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJwssNYvUxRHNAk7NAYawLZ_egBpR6QEIjDRZMUnqAA_390_590.png" alt="lQLPJwssNYvUxRHNAk7NAYawLZ_egBpR6QEIjDRZMUnqAA_390_590" style="zoom:50%;" />


<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJx0lV6X7BdHNAgnNAgCwdDUsXA1tZVgIjDNyeyXrAA_512_521.png" alt="lQLPJx0lV6X7BdHNAgnNAgCwdDUsXA1tZVgIjDNyeyXrAA_512_521" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJxclNMXvjdHNAgzNAgCwZjQRUSYKducIjDQJR7I0AA_512_524.png" alt="lQLPJxclNMXvjdHNAgzNAgCwZjQRUSYKducIjDQJR7I0AA_512_524" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJxDAHYJVS_HNAkbNAYWwEiscDAiG11kIjDMGVCK-AA_389_582.png" alt="lQLPJxDAHYJVS_HNAkbNAYWwEiscDAiG11kIjDMGVCK-AA_389_582" style="zoom:50%;" />


<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJxjQqxlPh1HNAgvNAgmw9IOwB9X1jFwIjDPoxwB8AA_521_523.png" alt="lQLPJxjQqxlPh1HNAgvNAgmw9IOwB9X1jFwIjDPoxwB8AA_521_523" style="zoom:50%;" />


<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJxWB7dNjtZHNAgjNAgCwNYnbpODQVcwIjDRzJty0AA_512_520.png" alt="lQLPJxWB7dNjtZHNAgjNAgCwNYnbpODQVcwIjDRzJty0AA_512_520" style="zoom:50%;" />


<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPJyAYORlvh9HNAgbNAfuwgo0-A_NSQkIIjDQZI0pkAA_507_518.png" alt="lQLPJyAYORlvh9HNAgbNAfuwgo0-A_NSQkIIjDQZI0pkAA_507_518" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPKGM3rQYcfVHNAgTNAgCwXOvGL5z9kjgIjDQ4h9w1AA_512_516.png" alt="lQLPKGM3rQYcfVHNAgTNAgCwXOvGL5z9kjgIjDQ4h9w1AA_512_516" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPKH6k-YU9gZHNAnDNAgSwnuwRo9586JEIjDPKG-0CAA_516_624.png" alt="lQLPKH6k-YU9gZHNAnDNAgSwnuwRo9586JEIjDPKG-0CAA_516_624" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/zxinyolo/images/main/lQLPKHjKIM93dFHNAkbNAYKwOyoVJ0eejSYIjDOEb10MAA_386_582.png" alt="lQLPKHjKIM93dFHNAkbNAYKwOyoVJ0eejSYIjDOEb10MAA_386_582" style="zoom:50%;" />



## 打码难度分析

- hcaptcha验证码类型非常多，已知的有50多种
- 需要准确理解题目要求，区分相似但不同的物体
- 训练的模型几乎不能通用，每种类别需要一个模型
- 需要强大的计算机视觉能力
- 需要理解自然语言描述的物体特征
- 需要大量标注数据进行模型训练
- hCaptcha会不断更新题目类型，模型需要持续更新

## 可能的解决方案

### 使用hcaptcha的无障碍模式

- 使用邮箱注册无障碍账号
- 收取hcaptcha发送的链接，获取hcaptcha cookie
- 访问带hcaptcha的页面时带上这个cookie，就可以实现直接通过

**风险：** 这个无障碍cookie过期时间未知，随时可能被封禁

### 每种类别训练一个模型

**优势:**

- 针对性强，识别准确率高

- 可控性好，可持续优化


**挑战:**

- 开发成本高，需要大量人力物力

- 维护成本高，需要持续更新模型

- 数据获取困难，标注工作量大

- 通用性差，每种类型需要独立开发

### 利用现有的多模态大模型进行识别(CHATGPT/Claude等)

**优势:**

- 开发成本低，无需训练模型

- 通用性强，可处理多种类型

- 识别能力强，理解能力好

- 快速部署，即开即用

**限制:**

- 依赖第三方服务，存在服务稳定性风险

- 成本较高，API调用费用不菲

- 响应速度可能较慢

- 准确率可能不如专用模型



