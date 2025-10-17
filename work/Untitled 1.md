| 字段名                 | JSON1（旧指纹） 是否存在        | JSON2 （新指纹）是否存在 | 备注           |
| ---------------------- | ------------------------------- | ------------------------ | -------------- |
| id                     | ✅                               | ❌                        | 仅 JSON1 有    |
| cpu                    | ✅ (数值)                        | ✅ (字符串)               | 类型不同       |
| dnt                    | ✅ (字符串 `"0"`)                | ✅ (数值 `0`)             | 类型不同       |
| tls                    | ✅                               | ❌                        | 仅 JSON1 有    |
| canvas                 | ✅                               | ❌                        | 仅 JSON1 有    |
| socks5                 | ✅                               | ❌                        | 仅 JSON1 有    |
| vendor                 | ✅                               | ✅                        | 相同           |
| webrtc                 | ✅                               | ❌                        | 仅 JSON1 有    |
| appName                | ✅                               | ✅                        | 相同           |
| fontDir                | ✅                               | ❌                        | 仅 JSON1 有    |
| product                | ✅                               | ✅                        | 相同           |
| storage                | ✅ (对象: quota, usage, enabled) | ✅ (数值)                 | 结构不同       |
| platform               | ✅ ("Win32")                     | ✅ ("WINDOWS")            | 值不同         |
| timezone               | ✅                               | ❌                        | 仅 JSON1 有    |
| fontsFull              | ✅ (数组)                        | ✅ (字符串)               | 类型不同       |
| languages              | ✅                               | ✅                        | 相同           |
| userAgent              | ✅                               | ✅                        | 相同           |
| appVersion             | ✅                               | ✅                        | 相同           |
| audioNoise             | ✅                               | ❌                        | 仅 JSON1 有    |
| colorDepth             | ✅                               | ✅                        | 相同           |
| screenSize             | ✅ ("1920,1080")                 | ✅ ("1707×1067")          | 值格式不同     |
| appCodeName            | ✅                               | ✅                        | 相同           |
| clientHints            | ✅                               | ❌                        | 仅 JSON1 有    |
| clientRects            | ✅                               | ❌                        | 仅 JSON1 有    |
| geolocation            | ✅                               | ❌                        | 仅 JSON1 有    |
| mediaDevice            | ✅                               | ❌                        | 仅 JSON1 有    |
| speechVoice            | ✅                               | ❌                        | 仅 JSON1 有    |
| userDataDir            | ✅                               | ❌                        | 仅 JSON1 有    |
| windowWidth            | ✅                               | ❌                        | 仅 JSON1 有    |
| batteryLevel           | ✅ (数值 1)                      | ✅ ("98%")                | 类型和表示不同 |
| deviceMemory           | ✅ (数值 8)                      | ✅ (字符串 "8")           | 类型不同       |
| windowHeight           | ✅                               | ❌                        | 仅 JSON1 有    |
| acceptLanguage         | ✅                               | ✅                        | 相同           |
| batteryCharging        | ✅                               | ✅                        | 相同           |
| devicePixelRatio       | ✅                               | ✅                        | 相同           |
| fontsFullEnabled       | ✅                               | ❌                        | 仅 JSON1 有    |
| webglCustomConfig      | ✅                               | ❌                        | 仅 JSON1 有    |
| batteryChargingTime    | ✅ (0)                           | ✅ (null)                 | 值不同         |
| permissionSimulation   | ✅                               | ❌                        | 仅 JSON1 有    |
| windowTitleBarHeight   | ✅                               | ❌                        | 仅 JSON1 有    |
| batteryDischargingTime | ✅ (-1)                          | ✅ (null)                 | 值不同         |
| os                     | ❌                               | ✅                        | 仅 JSON2 有    |
| gpu                    | ❌                               | ✅                        | 仅 JSON2 有    |
| audio                  | ❌                               | ✅                        | 仅 JSON2 有    |
| width                  | ❌                               | ✅                        | 仅 JSON2 有    |
| engine                 | ❌                               | ✅                        | 仅 JSON2 有    |
| height                 | ❌                               | ✅                        | 仅 JSON2 有    |
| memory                 | ❌                               | ✅                        | 仅 JSON2 有    |
| browser                | ❌                               | ✅                        | 仅 JSON2 有    |
| cpuArch                | ❌                               | ✅                        | 仅 JSON2 有    |
| plugins                | ❌                               | ✅                        | 仅 JSON2 有    |
| language               | ❌                               | ✅                        | 仅 JSON2 有    |
| winWidth               | ❌                               | ✅                        | 仅 JSON2 有    |
| enableGPU              | ❌                               | ✅                        | 仅 JSON2 有    |
| mimeTypes              | ❌                               | ✅                        | 仅 JSON2 有    |
| webdriver              | ❌                               | ✅                        | 仅 JSON2 有    |
| webglInfo              | ❌                               | ✅                        | 仅 JSON2 有    |
| winHeight              | ❌                               | ✅                        | 仅 JSON2 有    |
| colorGamut             | ❌                               | ✅                        | 仅 JSON2 有    |
| connection             | ❌                               | ✅                        | 仅 JSON2 有    |
| mediaMimes             | ❌                               | ✅                        | 仅 JSON2 有    |
| pixelDepth             | ❌                               | ✅                        | 仅 JSON2 有    |
| pixelRatio             | ❌                               | ✅                        | 仅 JSON2 有    |
| audioCodecs            | ❌                               | ✅                        | 仅 JSON2 有    |
| audioInputs            | ❌                               | ✅                        | 仅 JSON2 有    |
| browserName            | ❌                               | ✅                        | 仅 JSON2 有    |
| videoCodecs            | ❌                               | ✅                        | 仅 JSON2 有    |
| videoInputs            | ❌                               | ✅                        | 仅 JSON2 有    |
| audioOutputs           | ❌                               | ✅                        | 仅 JSON2 有    |
| enableCookie           | ❌                               | ✅                        | 仅 JSON2 有    |
| enableNotice           | ❌                               | ✅                        | 仅 JSON2 有    |
| speechVoices           | ❌                               | ✅                        | 仅 JSON2 有    |
| webglEnabled           | ❌                               | ✅                        | 仅 JSON2 有    |
| batteryLevelF          | ❌                               | ✅                        | 仅 JSON2 有    |
| kernelVersion          | ❌                               | ✅                        | 仅 JSON2 有    |
| isLocalStorage         | ❌                               | ✅                        | 仅 JSON2 有    |
| maxTouchPoints         | ❌                               | ✅                        | 仅 JSON2 有    |
| orientationType        | ❌                               | ✅                        | 仅 JSON2 有    |
| defaultVoiceLang       | ❌                               | ✅                        | 仅 JSON2 有    |
| defaultVoiceName       | ❌                               | ✅                        | 仅 JSON2 有    |
| isSessionStorage       | ❌                               | ✅                        | 仅 JSON2 有    |
| orientationAngle       | ❌                               | ✅                        | 仅 JSON2 有    |