## 一、 核心架构与面向对象（OOP）本质

`requests` 库是基于底层 `urllib3` 库封装的“人类友好型” HTTP 客户端库。

- **门面模式（Facade）**：我们常用的 `requests.get()` 和 `requests.post()` 表面上是普通的顶层函数，其实是快捷入口。
- **底层大本营**：在 `requests` 源码的 `models.py` 中，定义了核心的 **`class Response(object)`** 类。
- **实例化真相**：当你执行 `r = requests.get()` 时，函数内部会隐式去实例化 `Response` 类，在内存中把网线传回来的二进制流封装成一个对象，并作为战利品返回。你拿到的变量 `r`，就是这个 `Response` 类的**实例对象**。

## 二、 `requests.get()` 深度解剖（参数全集）

GET 请求用于从服务器**获取/查询**资源。其核心控制面板包含以下高频形参：

```Python
r = requests.get(url, params=None, headers=None, timeout=None, verify=True, auth=None, proxies=None, allow_redirects=True, stream=False)
```

### 1. `params`（查询参数）

- **作用**：自动构建 URL 尾部的查询字符串（即 `?key=value` 结构）。
- **类型**：**字典** (`dict`) 或**元组**列表。
- **原理**：自动进行 URL 编码（处理空格、中文及特殊字符），避免手动拼接产生字符串碎片。
- **示例**：`params={'wd': '111', 'page': 2}` $\rightarrow$ 最终请求 URL 自动变为 `https://xxx.com/s?wd=111&page=2`。
### 2. `headers`（请求头定制）

- **作用**：伪装请求，欺骗反爬虫系统。
- **类型**：**字典** (`dict`)。
- **核心字段**：
    - `User-Agent`：冒充浏览器（防止直接被判定为 `python-requests` 拦截）。   
    - `Referer`：声明请求的来源页。
    - `Cookie`：携带身份凭证。
### 3. `timeout`（超时投降机制）

- **⚠️ 工业级铁律**：**任何分布式工具或脚本必须显式指定！**
- **作用**：设定最大等待秒数。如果目标服务器卡死，达到指定时间（如 `timeout=3.5`）后会立即抛出异常并断开，防止整个程序或线程无限期死锁。
### 4. `verify`（SSL 证书校验开关）

- **作用**：是否验证 HTTPS 证书。默认值为 `True`。
- **安全/特种测试场景**：在内网渗透或扫描自签名证书、过期证书的资产节点时，必须设置为 `False`（`verify=False`）才能强闯进入，否则会抛出 `SSLError`。
### 5. `auth`（HTTP 身份认证）

- **机制**：默认触发 **HTTP Basic Auth**（常用于路由器后台、Tomcat管理后台、安全设备 API）。
- **类型**：**元组**`('username', 'password')`。
- **底层原理**：接收元组后，在底层自动将其拼接并进行 **Base64 编码**，随后以 `Authorization: Basic [Base64密文]` 的形式强行塞进 `Request Headers` 中。
- **🔒 安全警告**：Base64 仅是编码，毫无加密可言。**必须配合 HTTPS 使用**，否则等同于网络明文裸奔。
### 6. `proxies`（代理 IP 设置）
- **作用**：隐藏真实源 IP，防止频繁访问导致本地 IP 被服务器拉黑。
- **类型**：字典 (`dict`)。
- **示例**：`proxies={'http': 'http://10.10.1.10:3128', 'https': 'http://10.10.1.10:1080'}`。

### 7. `stream`（流式传输开关）
- **作用**：控制大文件下载时的内存占用。默认 `False`（一次性把文件全部读入内存）。
- **大文件避坑**：下载几百 MB 的视频或资产时，必须设置为 `True`，配合 `r.iter_content()` 像小溪流水一样分块写入硬盘，防止服务器 OOM（内存溢出）暴毙。

## 三、 `requests.post()` 深度解剖（参数全集）

POST 请求常用于向服务器**提交/新建**资源（如登录认证、上传文件）。其请求主体（Body）的物理传输格式极为严苛：

### 1. `data` 参数：传统键值对表单（Form Data）

- **代码**：`requests.post(url, data={'user': 'admin', 'pass': '123'})`
- **底层行为**：将字典转换为 `user=admin&pass=123` 的文本行。
- **BP 抓包视觉**：请求头中包含 `Content-Type: application/x-www-form-urlencoded`。
- **场景**：传统的网页登录、标准 HTML 表单提交。
### 2. `json` 参数：现代数据流（JSON Payload）

- **代码**：`requests.post(url, json={'user': 'admin', 'pass': '123'})`
- **底层行为**：自动在后台调用 `json.dumps()`，将 Python 字典转为标准的 JSON 纯文本字符串。
- **BP 抓包视觉**：请求头中包含 `Content-Type: application/json`。
- **场景**：现代前后端分离架构、现代 App 交互接口。
### 3. `files` 参数：多部分表单（Multipart/Form-Data）

- **代码**：`requests.post(url, files={'file': open('report.pdf', 'rb')})`
- **底层行为**：将二进制文件打包进请求体内，并自动计算复杂的二进制边界隔离符（Boundary）。
- **BP 抓包视觉**：请求头中包含 `Content-Type: multipart/form-data; boundary=xxxx`。
- **场景**：上传图片、上传木马/后门测试文件、上传文档附件。