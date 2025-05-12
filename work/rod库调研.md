>**Rod是一个直接基于DevTools Protocol高级驱动程序，无需依赖WebDriver，原生控制Chrome**

#### 简单示例

```go
package main

import (
	"encoding/json"
	"fmt"
	"github.com/go-rod/rod"
	"io/ioutil"
	"net/http"
)

func getWebSocketURL(remoteHost string) (string, error) {
	resp, err := http.Get(remoteHost + "/json/version")
	if err != nil {
		return "", err
	}
	defer resp.Body.Close()

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return "", err
	}

	var data struct {
		WebSocketDebuggerURL string `json:"webSocketDebuggerUrl"`
	}

	err = json.Unmarshal(body, &data)
	if err != nil {
		return "", err
	}

	return data.WebSocketDebuggerURL, nil
}

func main() {
	wsURL, _ := getWebSocketURL("http://10.100.5.201:9222")
	// 连接到远程浏览器 (DevTools 协议)
	browser := rod.New().ControlURL(wsURL).MustConnect()

	// 创建新页面
	page := browser.MustPage("https://www.google.com/search?q=忐忑&sa=X&sca_esv=9bb75c165dc77b23&gl=us&hl=en-sg&udm=28&sxsrf=AHTn8zqBSWF73Z5d25InVF3cnee1EqS9Zg%3A1746858396634&ei=nPEeaISrJM3F4-EP-tasoQI&ved=0ahUKEwiE6c7RopiNAxXN4jgGHXorKyQQ4dUDCCM&uact=5&oq=Laptop&gs_lp=Ehlnd3Mtd2l6LW1vZGVsZXNzLXNob3BwaW5nIgZMYXB0b3AyChAAGIAEGEMYigUyChAAGIAEGEMYigUyChAAGIAEGEMYigUyChAAGIAEGEMYigUyChAAGIAEGEMYigUyBRAAGIAEMgUQABiABDIFEAAYgAQyBRAAGIAEMgUQABiABEiCHlDLGFjLGHABeAKQAQCYAakBoAGpAaoBAzAuMbgBA8gBAPgBAvgBAZgCA6ACvAGoAgHCAgQQABhHwgINECMYsAMYtAQYJxjqApgDCpAGAZIHAzIuMaAH2QSyBwMwLjG4B68B&sclient=gws-wiz-modeless-shoppingp")

	// 等待页面加载，确保元素存在
	page.MustWaitLoad()

	// 获取内容，例如获取标题
	title := page.MustElement("title").MustText()
	fmt.Println("页面标题:", title)

	// 获取所有 <a> 标签
	links := page.MustElements("a")
	for _, link := range links {
		text := link.MustText()                    // 获取显示的文本
		href := link.MustProperty("href").String() // 获取 href 属性

		fmt.Printf("文字: %s\n链接: %s\n\n", text, href)
	}
}
```

#### Browser常用的方法

- New 