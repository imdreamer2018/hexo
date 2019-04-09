title: 数学公式神器Mathpix Snip——妈妈再也不用担心我不会写数学公式了！
author: 追梦人

toc: true

tags:

  - python
  - tools
  - 随笔
categories: []
date: 2019-04-09 22:36:00

---

# 数学公式神器Mathpix Snip

本文转载自**机器之心**—《最好用的文字与公式编辑器，这套数学笔记神器送给你》，原文链接[点击这儿](https://www.jiqizhixin.com/articles/2019-04-06-2)。

在平时写博客或者写论文的时候，经常需要花些时间的就是嵌入数学公式。其实用 LaTex 表达式写数学公式还是挺麻烦的，至少一般人做不到手写速度。但是我们有 Mathpix Snip 啊，只要截个图，公式会自动转化为 LaTex 表达式，我们只需要简单地修改修改就行了。

Mathpix Snip 的设计初衷是帮助人们在通过手机或电脑输入数学公式时节省时间。为此，Mathpix 研发了一款免费 APP——Snip 来自动化这一繁琐过程。

- **Mathpix Snip**：https://mathpix.com/
- **软件下载链接**：[点击这儿](http://yunpan.chinadream.org//tools/mathpix_snipping_tool_setup.exe)

<!-- more -->

## 看看名人的评价

![Aodf4H.png](https://s2.ax1x.com/2019/04/09/Aodf4H.png)

爱因斯坦棺材板快盖不住了，当然这是开玩笑的，哈哈哈哈哈哈哈。

## 官网使用教程

### 第一步-通过输入键盘快捷键

| 按键                              | 说明                                                     |
| --------------------------------- | -------------------------------------------------------- |
| Ctrl + ⌘ + M on Mac               | ![AowVPJ.jpg](https://s2.ax1x.com/2019/04/09/AowVPJ.jpg) |
| Ctrl + Alt + M on Windows & Linux | ![AowA54.jpg](https://s2.ax1x.com/2019/04/09/AowA54.jpg) |

### 第二步-在屏幕截图框中捕获所需的输入

**只需点击并拖拽**

![Aowe2R.gif](https://s2.ax1x.com/2019/04/09/Aowe2R.gif)



### 第三步-轻松编辑您的LaTeX

**从任务栏中选择所需的格式，进行编辑**

![Aowmx1.gif](https://s2.ax1x.com/2019/04/09/Aowmx1.gif)

### 第四步-将LaTeX粘贴到任何兼容的编辑器中

**它已经被复制到你的剪贴板了。**

![AowZG9.png](https://s2.ax1x.com/2019/04/09/AowZG9.png)

## 测试用例

由于懒得找电子版的公式了，直接手写一个公式，看能否识别出来。这里写了一个关于机器学习中梯度下降的公式。

### 手写公式

![AoBtDP.jpg](https://s2.ax1x.com/2019/04/09/AoBtDP.jpg)

### 在屏幕截图框中捕获所需的输入

![AoB4C4.png](https://s2.ax1x.com/2019/04/09/AoB4C4.png)

### 粘贴结果到Typora

最终得到如下公式，可见
$$
\frac { 1 } { m } \sum _ { j = 1 } ^ { m } \left[ h _ { \theta } \left( x ^ { ( i ) } \right) - y ^ { ( i ) } \right] x _ { j } ^ { ( i ) }
$$
识别率还是挺高的，当然手写的话需要写得很**工整**才行。

**自此，该软件功能介绍完毕，后面是调用这个功能的API，如无需要可以不用继续看后面的内容，反正后面的API我调用了返回的是证书错误，估计APP_KEY有点问题。**

## 万能的API

为了方便使用，该公司还研发了一个 API（MathpixOCR），可以帮助开发者将 Mathpix 的功能集成到自己的应用程序。

你向 MathpixOCR 发送一张图片，它就会告诉你其中的数学公式。就这么简单。该 API 会返回 LaTex 以及图片元数据，你可以在你的应用中使用它。

### 示例

如下展示了如何简单调用 API 处理单张图像：

![AoDHzj.jpg](https://s2.ax1x.com/2019/04/10/AoDHzj.jpg)

调用API如下例：

```python
#!/usr/bin/env python
import sys
import base64
import requests
import json

# 将文件路径放在此处
file_path = 'limit.jpg'
image_uri = "data:image/jpg;base64," + base64.b64encode(open(file_path, "rb").read())
r = requests.post("https://api.mathpix.com/v3/latex",
    data=json.dumps({'url': image_uri}),
    headers={"app_id": "trial", "app_key": "34f1a4cea0eaca8540c95908b4dc84ab",
            "Content-type": "application/json"})
print(json.dumps(json.loads(r.text), indent=4, sort_keys=True))
```

```
curl -X POST https://api.mathpix.com/v3/latex \
    -H 'app_id: trial' \
    -H 'app_key: 34f1a4cea0eaca8540c95908b4dc84ab' \
    -H 'Content-Type: application/json' \
    --data '{ "url": "data:image/jpeg;base64,'$(base64 -i limit.jpg)'" }'
```

API返回的JSON结构如下所示：

```json
{
    "detection_list": [],
    "detection_map": {
        "contains_chart": 0,
        "contains_diagram": 0,
        "contains_geometry": 0,
        "contains_graph": 0,
        "contains_table": 0,
        "is_inverted": 0,
        "is_not_math": 0,
        "is_printed": 0
    },
    "error": "",
    "latex": "\\lim _ { x \\rightarrow 3} ( \\frac { x ^ { 2} + 9} { x - 3} )",
    "latex_confidence": 0.86757309488734,
    "position": {
        "height": 273,
        "top_left_x": 57,
        "top_left_y": 14,
        "width": 605
    }
}
```

### 请求参数

API希望一个请求必须含有两个请求头（app_id和app_key），以及一个含有 *url* 字段的JSON请求体，用于指定图片。
此字段是一个字符串，要么包含一个公共资源引用，用于检索图片内容，要么包含一个base64的图片编码数据url。

JSON请求体可包含一个可选的 `region` 对象，用以下属性来指定需要处理的图片像素：`top_left_x`、`top_left_y`、
`width`和`height`，都采用像素坐标（整数值）。
例如 `'region': {'top_left_x': 0, 'top_left_y': 0, 'width': 10, 'height': 10}`
指定图片左上方 10 x 10 的区域。

JSON请求体还可能包含一个名为 *formats* 的可选字段，它是一个对象，以指定想要在结果中展示的格式。

*latex* 字段（如果存在），可设置如下值：“raw”（结果为未修改的OCR输出）、“default” （结果为删除了多余空格的OCR输出），或"simplified"（结果为除去空格、操作快捷键，且在适当的情况下分为列表）。

*mathml* 字段（如果存在且设为true），表示服务器应为JSON结果添加一个 *mathml* 字段，它是一个含有可识别数学MathML标记的字符串。

*wolfram* 字段（如果存在且设为true），表示服务器应为JSON结果添加一个 *wolfram* 字段，它是一个与Wolfram Alpha引擎兼容的字符串。

在不兼容的情况下，服务器将替代添加一个 *mathml_error* 或 *wolfram_error* 字段。

*ocr* 字段（如果存在），标示应该将内容解析到什么程度。默认值为 `["math"]`。如果用户也想在文本中阅读，则应传入 `ocr=["math", "text"]`。文本也会返回在 `\text` 中，如此一来，Latex输出就可以总是被相应的Latex编译器渲染出来，如Mathjax或Katex。

| 参数         | 描述                                                  |
| :----------- | :---------------------------------------------------- |
| app_id       | 应用程序标识符（例如`mymathsolver`）                  |
| app_key      | 应用程序密钥                                          |
| content_type | application/json                                      |
| url          | 字符串                                                |
| region?      | {top_left_x, top_left_y, width, height}               |
| formats?     | {latex?: string, mathml?: boolean, wolfram?: boolean} |
| ocr?         | [“math”?, “text”?]                                    |

### 接收结果

服务器会返回解析图片后的结果，除非请求中包含一个 `callback` 对象，在此情况下，返回结果仅包含一个`session_id` 字段，同时服务器异步发送结果到 `callback.post` 所指定的url。异步发送包括 `result` 对象以及一个包含 `session_id` 的 `reply` 对象。

应用程序可以通过在请求中指定 `callback.reply` 的值来覆盖`session_id` 的返回值。服务器将在返回结果和异步发送中使用此值。

如需在发送中提供额外的数据，可以通过指定 `callback.body` 来实现。 服务器会将此值传递给发送数据。如果提供了 `callback.headers`，那么服务器将在发送的头部中发送此内容。

你可以通过访问 <https://requestb.in/> 来创建一个回调URL进行测试。 测试异步API的示例代码如右侧所示。

要测试异步API，请通过访问 <https://requestb.in/> 来将下列的回调URL替换成你自己的。

```python
#!/usr/bin/env python
import sys
import base64
import requests
import json

file_path = "integral.jpg"
image_uri = "data:image/jpeg;base64," + base64.b64encode(open(file_path, "rb").read())
r = requests.post("https://api.mathpix.com/v3/latex",
    data=json.dumps({'url': image_uri, 'callback': {'post': 'http://requestb.in/10z3g561', 'reply': file_path}}),
	headers={"app_id": "trial", "app_key": "34f1a4cea0eaca8540c95908b4dc84ab",
		"Content-type": "application/json"})
print(r.text)
```

```
curl -X POST https://api.mathpix.com/v3/latex \
    -H 'app_id: trial' \
    -H 'app_key: 34f1a4cea0eaca8540c95908b4dc84ab' \
    -H 'Content-Type: application/json' \
    --data '{ "url": "data:image/jpeg;base64,'$(base64 -i integral.jpg)'","callback":{"post": "http://requestb.in/10z3g561", "reply": "integral.jpg"}}'
```

下列JSON通过POST请求发送给回调URL：

```json
{
  "reply": "integral.jpg",
  "result": {
    "detection_list": [
      "is_printed"
    ],
    "detection_map": {
      "contains_chart": 0,
      "contains_diagram": 0,
      "contains_geometry": 0,
      "contains_graph": 0,
      "contains_table": 0,
      "is_inverted": 0,
      "is_not_math": 0,
      "is_printed": 1
    },
    "error": "",
    "latex": "\\int \\frac { 4 x } { \\sqrt { x ^ { 2 } + 1 } } d x",
    "latex_confidence": 0.99817252453161,
    "latex_list": [],
    "position": {
      "height": 215,
      "top_left_x": 57,
      "top_left_y": 0,
      "width": 605
    }
  }
}
```

### 置信值

API在 `latex_confidence` 字段中返回对latex的置信值。建议你弃用低于某一特定置信值的结果。你可以为你的应用调整该阈值。我们在iOS和Android的演示应用程序中采用的是20%的置信值。

### 检测类型

API在JSON字段 `detection_map` 中定义了多个检测类型。该字段描述如下：

| 检测              | 定义                                           |
| :---------------- | :--------------------------------------------- |
| contains_chart    | 包含离散数据集的可视化显示。                   |
| contains_table    | 包含离散数据集的表格显示。                     |
| contains_diagram  | 包含一个图表。                                 |
| contains_graph    | 包含一个1D、2D或3D的图形。 graph.              |
| contains_geometry | 包含图表、表格或图形。                         |
| is_printed        | 图片上是印刷的数学公式，而不是手写的数学公式。 |
| is_not_math       | 没有检测到有效的方程式。                       |

API为每一个检测类别返回置信度。

### 检测坐标

`position` 字段包含所求方程式的边界框，以图片的像素坐标系为基准。请注意，我们将代数方程和多线性方程的公式视为单个方程。如果多个方程被很多文本隔开，我们将只返回第一个方程。

## 处理一批图片

批处理请求如下所示：

```
curl -X POST https://api.mathpix.com/v3/batch \
    -H "app_id: trial" \
    -H "app_key: 34f1a4cea0eaca8540c95908b4dc84ab" \
    -H "Content-Type: application/json" \
    --data '{ "urls": {"inverted": "https://raw.githubusercontent.com/Mathpix/api-examples/master/images/inverted.jpg", "algebra": "https://raw.githubusercontent.com/Mathpix/api-examples/master/images/algebra.jpg"},"callback":{"post": "http://requestb.in/sr1x3lsr"}}'
```

```python
import requests
import json

base_url = 'https://raw.githubusercontent.com/Mathpix/api-examples/master/images/'

data = {
	'urls': {
		'algebra': base_url + 'algebra.jpg',
		'inverted': base_url + 'inverted.jpg'
	},
	'callback': {
		'post': 'http://requestb.in/1ftatkr1'
	}
}

r = requests.post(
	"https://api.mathpix.com/v3/batch", data=json.dumps(data),
	headers={
		'app_id': 'trial',
		'app_key': '34f1a4cea0eaca8540c95908b4dc84ab',
		'content-type': 'application/json'
	},
	timeout=30
)
reply = json.loads(r.text)
assert reply.has_key('batch_id')
```

返回结果如下：

```json
{
  "reply": {
    "batch_id": "11"
  }, 
  "result": {
    "algebra": {
      "detection_list": [], 
      "detection_map": {
        "contains_chart": 0, 
        "contains_diagram": 0, 
        "contains_geometry": 0, 
        "contains_graph": 0, 
        "contains_table": 0, 
        "is_inverted": 0, 
        "is_not_math": 0, 
        "is_printed": 0
      }, 
      "error": "", 
      "latex": "1 2 + 5 x - 8 = 1 2 x - 1 0 ", 
      "latex_confidence": 0.99640350138238, 
      "latex_list": [], 
      "position": {
        "height": 208, 
        "top_left_x": 0, 
        "top_left_y": 0, 
        "width": 1380
      }
    }, 
    "inverted": {
      "detection_list": [
        "is_inverted", 
        "is_printed"
      ], 
      "detection_map": {
        "contains_chart": 0, 
        "contains_diagram": 0, 
        "contains_geometry": 0, 
        "contains_graph": 0, 
        "contains_table": 0, 
        "is_inverted": 1, 
        "is_not_math": 0, 
        "is_printed": 1
      }, 
      "error": "", 
      "latex": "x ^ { 2 } + y ^ { 2 } = 9 ", 
      "latex_confidence": 0.99982263230866, 
      "latex_list": [], 
      "position": {
        "height": 170, 
        "top_left_x": 48, 
        "top_left_y": 85, 
        "width": 544
      }
    }
  }
}
```

Mathpix API支持在另一个接口（`/v3/batch`）使用一个POST请求处理多张图片。请求包括 `urls` ，用来为每个图片指定键值对，以及 `callback` ，用来指定将结果返回到哪里。键值对可以是一个字符串或一个包含url和附加特定参数的对象，如 `region` 和 `formats` 。
响应结果包含一个唯一的 `batch_id` 值，在处理完所有图片后，服务器发送一个消息到 `callback.post` ，包含带每张图片结果的键值对的 `result` 以及包含带 `batch_id` 的 `reply`。

应用程序可以在请求中指定 `callback.reply` 的值，来定义响应和结果中除了 `batch_id` 的字段。但是，服务器生成的 `batch_id` 不能被覆盖。

如果想在发送的内容中提供额外的数据，可以指定 `callback.body` 值。服务器将在发送中传递此值。如果提供了 `callback.headers` ，那么服务器将在发送的头部中包含此内容。

要检查批请求的状态，可以使用 GET `/v3/batch/:id` ，此处的 `:id` 是返回的 `batch_id`。请注意，GET请求必须包含相同的 `app_id` 和 `app_key` 头部作为请求数据。响应正文是JSON数据，包含了该批url键（字符串数组）、当前结果（键值对对象）和传递给原始批请求的回调对象。当批处理完成后，服务器仅在有限的时间内保留批处理的结果（目前为一周，但未来可能会改变）。以下链接包含GET API的示例：<https://github.com/Mathpix/api-examples/blob/master/python/batch.py>