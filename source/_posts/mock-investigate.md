---
title: 前端Mock工具调研
date: 2014-11-24
tags: [前端, mock, 前后端同步开发, 接口模拟]
categories: 开发
---

前后端分离的开发实践中，mock是一个非常重要的工具。由于前后端开发很难做到同步，前端需要对未完成的接口的响应进行mock；而后端在前端没有完成的情况下，需要构造请求，测试接口的正确性。本文对现有的接口mock工具做了调研，分析其优劣，在最后给出了mock平台的功能设想。

## 现有的解决方案

对github上七个比较流行的前端mock工具做了调研，从工具的接入方式、接口的配置、mock数据的构造、mock数据的管理、支持的数据类型、真实响应的模拟等方面对其进行了分析总结。

### [Jquery-mockjax](https://github.com/jakerella/jquery-mockjax)

#### 功能

1. 接入方式：拦截jquery api发出的ajax请求

2. 接口的配置
    通过接口`$.mockjax(/* Object */ options)`配置请求与mock数据的映射，支持的请求配置方式有：

    - 通配符
    ```
      $.mockjax({url: "/data/*"})
    ```
    - 正则匹配
    ```
      $.mockjax({url: /^\/data\/(quote|tweet)$/i})
    ```
    - 字符串
    ```
      $.mockjax({url: "/data/1"})
    ```
    - 根据请求参数动态生成mock配置
    ```
      // 通过参数settings可以获取请求url、headers等
      $.mockjax(function(settings){})
    ```

3. mock响应数据的构造方式

    mockjax的所有响应数据需要通过以下三种方式指定，数据需要手工创建。

	- 通过`mockjax`接口的参数`responseText/responseXML`指定
	- 通过参数`proxy`定向到json文件
	- 通过参数`response`，动态构造或指定数据

4. mock数据管理

	- 写死在mock配置中
	- 通过json文件组织，通过参数proxy被使用

5. 支持的数据类型：text, html, json, jsonp, script, xml

6. 真实响应的模拟
	- 模拟网络延时
	- 指定响应码
	- 指定响应头、支持自定义响应头
	- 模拟服务端超时

7. 其他功能
	- 通过接口`$.mockjaxSetting`全局配置
	- 响应完成后的钩子
	- 移除指定mock配置

#### 优点
	1. 支持几乎所有的主流浏览器
	2. 支持的数据类型较多
	3. 可以根据请求参数动态生成不同的响应
	4. 配置简单，容易上手
	5. 支持自定义响应头
	6. 提供`onAfter{Cxxx}`钩子

#### 缺点
	1. 所有的mock数据都要手工创建
	2. 必须与jquery结合使用，使用场景受到严重限制
	3. mock相关的代码侵入项目代码，需要在打包时移除

### [Pretender](https://github.com/trek/pretender)

#### 功能
1. 接入方式

	替换`XMLHttpRequest`对象，拦截所有请求，定向到预定义的pretend服务

2. 接口的配置
	通过方法`get`, `put`, `post`, `delete`, `patch`, `head`配置path与响应的对应关系，path支持以下两种形式：

	```
	this.get('/api/songs/12', function (request) {});

	this.get('/api/songs/:song_id', function (request) {});
	```

3. mock响应数据的构造方式

	通过回调指定，响应数据需要手工创建。

4. mock数据管理：硬编码到代码里。

5. 支持的数据类型：json, text, html
6. 其他功能

	+ 提供mock响应完成后钩子
	+ 支持响应回调中获取query参数
	+ 记录处理和未处理的请求数

#### 优点
	1. 不依赖项目所使用的js库
	2. 配置简单，容易上手

#### 缺点
	1. path配置比较麻烦，不支持正则和通配符
	2. 没有提供从文件获取mock数据的支持
	3. 不能模拟异常响应，不能模拟真实网络环境

### [Mockjs](https://github.com/nuysoft/Mock)

#### 功能
1. 接入方式：拦截ajax请求，内置支持jQuery, Zepto, KISSY。
2. 接口的配置：url字符串或正则表达式，请求类型
3. mock响应数据的构造方式
	- 根据html模板生成模拟数据
	- 根据数据模板生成自动生成模拟数据

4. mock数据管理

	mock数据根据数据模板动态生成，需要管理数据模板。

5. 支持的数据类型：json, text, html

#### 优点
	1. 给出了数据模板定义、数据占位符定义，提供了丰富的语义化的模拟数据生成器
	2. 只需定义一份数据模板，即可批量生成模拟数据

#### 缺点
	1. 数据模板定义没有复用机制
	2. mock请求、响应配置功能简陋
	3. 不支持真实网络环境的模拟
	4. 不支持异常响应的模拟

### [APItizer](https://github.com/retro/apitizer)

#### 功能
1. 接入方式：拦截ajax请求
2. 接口的配置
	- 调用`apitizer.addSchema('schemaName', schema)`时，apitizer会自动生成restful风格api的mock配置。
	- 可通过`apitizer.fixture('MOTHOD /URL', function(params) {})`进行配置

3. mock响应数据的构造方式
	- 根据预定义的json schema生成json数据
	- 可以定制数据生成规则，覆盖默认的生成规则
	- 可以通过回调，响应其他途径获取的数据

4. mock数据管理
	- addSchema后，调用schemaStore('schemaName', number)会自动生成指定条目的
数据存入数据库，默认使用数据库中存储的作为mock数据。
	- 可以响应随机生成的mock数据

5. 支持的数据类型：json

6. 真实响应的模拟
	- 模拟响应延迟
	- 模拟表单更新
	- 模拟列表排序、搜索
	- 模拟异常响应

7. 其他功能
	- 使用json schema验证后端接口的正确性
	- 支持json schema通过`$ref`复用

#### 优点
	1. 使用json schema自动生成数据，不需要手工创建
	2. 支持schema的复用
	3. 能够mock表单修改、列表排序和搜索
	4. 可以mock随机数据，也可以mock固定数据
	5. 对restful api提供原生支持

#### 缺点
	1. 未提供有效的json schema管理方式
	2. 对于非restful风格的api支持不是很好
	3. 依赖jquery

### [RAP](https://github.com/thx/RAP)

### 功能

1. 接入方式

	通过rap插件拦截ajax请求，支持KISSY、jquery, 将以下代码写在KISSY或jQuery js代码之后即可:

	```
	<script type="text/javascript" src="http://{{domainName}}/rap.plugin.js?projectId={{projectId}}&mode={{mode}}"></script>
	```

	其中：

	+ {{projectId}}为用户所编辑的接口在RAP中的项目ID
	+ {{mode}}为RAP路由的工作模式, 默认值为3。

	mode不同值的具体含义如下:

	+ 0 不拦截
	+ 1 拦截全部
	+ 2 黑名单中的项不拦截
	+ 3 仅拦截白名单中的项

2. 接口的配置
	- 根据接口定义自动生成mock服务
	- 支持黑名单、白名单配置

3. mock响应数据的构造方式
	- 根据接口中定义的数据模板自动生成模拟数据

4. mock数据管理
	- 接口定义时存储数据模板，不存储具体数据

5. 支持的数据类型：json

6. 其他功能
	- 提供按项目管理接口的功能
	- 提供接口版本管理的功能
	- 根据接口定义申城接口文档
	- 根据接口定义生成接口的json schema详情
	- 提供多项目见mock模板的复用
	- 支持restful api

#### 优点
	1. 根据接口定义自动生成mock服务
	2. 根据接口定义中的数据模板自动生成mock数据
	3. 由于mock数据是根据接口定义自动生成的，当接口变化时，不需要考虑mock数据修改的问题
	4. 黑白名单功能便于在真实后端与mock服务之间切换

#### 缺点
	1. 功能复杂，需要一定的学习成本
	2. 只能mock随机数据
	3. 接口定义和数据模板复用度低

### [Mocky](https://github.com/studiodev/Mocky)

待添加

### [Mockable]()

待添加


## 理想中的mock平台

	1. mock服务能够灵活配置，后端与mock服务之间能够无缝结合，快速切换
	2. 数据自动生成，可以是随机数据，也可以是固定数据，并且提供便利的切换方式
	3. 当请求参数异常时，能够返回正确的异常信息
	4. 当接口发生变化时，不需要修改原有的mock数据
	5. 能够mock反例数据，便于自测
