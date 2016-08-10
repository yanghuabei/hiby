---
title: babel5-to-babel6
date: 2016-08-10
tags: [es6, babel6, javascript, 构建]
categories: 开发
---

## 为什么要升级？
+ 性能提升：据说compile速度提升20%（但测试后发现，速度慢了20%+）。
+ 可配置的插件：更强的灵活性，以及更简单的插件API。
+ 更简洁的配置。

## 重大变化
+ 不再提供babel包
	+ 命令行工具由babel-cli包提供
	+ node api由babel-core提供
	+ polyfill由babel-polyfill提供
+ .babelrc配置变化
	+ 移除“blacklist”、“whitelist”、“optional”、“nonStandard”和“modules”等
	+ 增加`plugins`：转码逻辑通过一系列插件实现，默认不使用任何插件
	+ 增加`presets`配置，提供一些预设的插件集合
	+ external-helper选项由插件实现

## 升级过程中遇到问题

### babel官方未提供decorator transfrom插件
可以使用非官方插件：transform-decorators-legacy。

### babel提供的es2015插件集默认包含transform-es2015-modules-commonjs插件
SSP系统需要将es6模块转成AMD模块，需要在plugins中指定使用transform-es2015-modules-amd.

### babel提供的AMD模块转码插件默认开启strict mode
关闭strict mode的方法：

+ 不使用`preset-es2015`，需要**自定义preset**或者在**plugins**中引入所有需要的插件;
+ 插件中做如下配置：

```javascript { .theme-peacock }
{
	"plugins": [
		["transform-es2015-modules-amd", {"strict": false}]
	]
}
```

 <font color="red">Notes：如果不提供es2015相关插件的preset，`preset-stage-x`也不能使用</font>

### es6代码中有require时会导致转码失败
es6代码中出现`require`时，babel5能正确转码，但babel6会报如下错误：

	Property object of MemberExpression expected node to be of a type ["Expression"] but instead got null.

解决方法：使用`import`。

### 使用babel-core提供的api转码时，如果以对象形式传入配置，转码后AMD模块会有两层`define`
解决方法：传入配置中需要增加`babelrc=false`。
可能的原因：babel初始化配置时，如果传入配置中无`babelrc=false`，会从项目目录中找**.babelrc**文件，然后将传入配置与**.babelrc**配置合并，最终对代码进行两次AMD transfrom。

### ES6 module to AMD的转码逻辑变更
babel6提供的AMD转码插件对`export default`的转码逻辑做了修改，详情如下：
针对如下代码：
```
export default {
    foo: 1
};
```

使用babel5转码后：
```javascript { .theme-peacock }
define(["exports", "module"], function (exports, module) {
    module.exports = {
        foo: 1
    };;
});
```

使用babel6转码后：
```javascript { .theme-peacock }
define(["exports"], function (exports) {
    Object.defineProperty(exports, "__esModule", {
        value: true
    });
    exports.default = {
        foo: 1
    };
});
```

在AMD模块依赖es6模块的场景下，AMD模块代码会不能运行。

解决方法：
方法1. 项目代码全部使用es6;
方法2. 修改AMD模块中使用es6模块的方式;
方法3. 定义一个模块转码插件，实现babel5的转码逻辑（不是个好办法）。


```javascript { .theme-peacock }
// foo.js
const foo = {baz: 42, bar: false}
export default foo
// bar.js
import {baz} from './foo'
```

### polyfill和babel-external-helper
在babel-polyfill包中。
babel-cli提供babel-external-helpers命令，生成helpers代码。
