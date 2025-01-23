## 浏览器插件ID固定及开启方式

1. 打开chrome://extensions/页面

2. 点击打包扩展程序

   ![1725852317822](C:\Users\MARS_02_SERD\AppData\Roaming\Typora\typora-user-images\1725852317822.png)

3. 填写扩展程序根目录(首次打包不需要填写私钥文件)

   ![1725852448847](C:\Users\MARS_02_SERD\AppData\Roaming\Typora\typora-user-images\1725852448847.png)

4. 打包成功之后会生成两个文件.crx文件和.pem文件(.pem文件是固定插件ID的关键)

5. 更改插件后代码打包方式(需要添加.pem文件,这样生成的插件id就和首次打包的插件id一致)

   ![1725852646719](C:\Users\MARS_02_SERD\AppData\Roaming\Typora\typora-user-images\1725852646719.png)

6. 如果需要不上传谷歌商店就能使用需要修改注册表

   - HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Google\Chrome\ExtensionInstallWhitelist路径下添加字符串项: 数值名称为1,数值数据为插件ID

   - HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Google\Chrome\ExtensionInstallAllowlist路径下添加字符串项: 数值名称为1,数值数据为插件ID

   - 如果路径不存在手动添加

     ![1725852924184](C:\Users\MARS_02_SERD\AppData\Roaming\Typora\typora-user-images\1725852924184.png)

7. 添加完成后,重启浏览器,导入crx插件就能正常使用并开启