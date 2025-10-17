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

- New：创建一个新的 Browser 实例。
- ControlURL(url string)`：设置 DevTools WebSocket 地址，用于连接已启动的远程浏览器
- Connect() error：连接到浏览器，返回错误，可在生产环境中捕获并处理
- MustConnect() *rod.Browser：连接浏览器，失败时 panic，适合脚本或调试使用
- Page(url string) (*rod.Page, error)：创建新页面，可直接传入 URL 也可传空字符串后调用 Navigate
- MustPage(url string) *rod.Page：创建并跳转页面，失败时 panic
- Close() error：关闭浏览器实例，释放资源



#### Page 常用的方法

- Navigate(url string) error：跳转到指定 URL
- WaitLoad() error：等待页面加载完成
- WaitStable() error：等待 DOM 稳定，无新增或删除节点后再继续
- Element(selector string) (*rod.Element, error)：查找第一个匹配的元素
- MustElement(selector string) *rod.Element：查找第一个匹配的元素，失败时 panic
- Elements(selector string) ([]*rod.Element, error)：查找所有匹配的元素
- MustElements(selector string) []*rod.Element：查找所有匹配的元素，失败时 panic
- Evaluate(js string) (*proto.RuntimeRemoteObject, error)：执行一段 JavaScript 并返回结果
- MustEvaluate(js string) *proto.RuntimeRemoteObject：执行 JavaScript，失败时 panic
- Screenshot(path string) error：对当前视口截图并保存到指定路径
- MustScreenshot(path string)：对当前视口截图，失败时 panic
- Close() error：关闭页面



#### Element 常用的方法

- Text() (string, error)：获取元素的可见文本内容
- MustText() string：获取元素的可见文本内容，失败时 panic
- HTML() (string, error)：获取元素的内部 HTML（包括子元素）
- MustHTML() string：获取元素的内部 HTML，失败时 panic
- Property(name string) (*proto.RuntimeRemoteObject, error)：获取元素属性值
- MustProperty(name string) *proto.RuntimeRemoteObject：获取属性值，失败时 panic
- Attribute(name string) (*string, error)：获取属性，如果不存在返回 nil
- MustAttribute(name string) *string：获取属性，失败时 panic
- Click() error：模拟用户点击元素
- MustClick()：点击元素，失败时 panic
- Input(text string) error：在元素（如 input、textarea）中输入文本
- MustInput(text string)：输入文本，失败时 panic
- Hover() error：将鼠标悬停在元素上
- MustHover()：悬停，失败时 panic
- ScrollIntoView() error：滚动页面使元素可见。
- MustScrollIntoView()：滚动至可见，失败时 panic。



