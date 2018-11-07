---
title: Node 中用 ESLint 验证代码
date: 2018-11-07 22:48:19
tags: ESLint 
---

# Node 中用 ESLint 验证代码

### 前言

ESLint 我们一般用在项目工程里，用来监督我们的代码格式、风格等；

另外也能用来检验一段代码字符串是不是符合规则。

```javascript
//可以对 code 这个字符串进行验证，例如我们不允许使用 location.href
let code = 'location.href="zhengxiaoer.cn";'; 
```

使用场景：当这段代码串是用户提供的时候时，我们可以静态的进行一些验证（当然，由于 JavaScript 太灵活，这样的验证完全限制不了用户，能绕过的方法太多）。

### 怎么用 [官方文档](https://eslint.bootcss.com/docs/4.0.0/developer-guide/nodejs-api/)

1、先安装 `ESLint` ， `npm install -s eslint` 或 `yarn add eslint`

2、代码

```javascript
const Linter = require("eslint").Linter;
// 检查的配置
const CONFIG = {
  "parser": "babel-eslint",
  "env": {
    "browser": true,
    "es6": true
  },
  // rules 参考官方文档 https://eslint.bootcss.com/docs/rules/
  rules: {
    'no-eval': 2,//不能使用 eval 
    'no-with': 2,//不能使用 with 
    'no-delete-var': 2,//不能删除变量 delete a; 
  }
};

// 等待验证的代码
let code = 'eval("console.log(1+1)")';
// 初始化验证对象
let linter = new Linter();
// 开始验证
// messages 为返回的验证结果，数据结构可以查看文档 https://eslint.bootcss.com/docs/4.0.0/developer-guide/nodejs-api/
let messages = linter.verify(code, CONFIG);
```



### 定制自己的规则

1、新建 myrule.js， [官方文档](https://eslint.bootcss.com/docs/developer-guide/working-with-rules/) ，一开始可能不知道怎么下手写代码，可以参考官方的验证规则源代码，进行模仿

假如实现对所有调用 location.href 的都不通过：

```javascript
"use strict";
module.exports = {
    meta: {     
    },   
    create(context) { 
        // 返回需要对 AST 进行判断
        //这个网址可以快速查看代码生成的 AST https://astexplorer.net
        return {  
            // 对所有调用 location.href 的都不通过
            MemberExpression: (node)=>{
                let objName = node.object.name;
                let propertyName = node.property.name;
                if((/^location$/).test(objName) && (/^href/).test(propertyName)) {
                    // 返回错误数据
                    context.report({
                        node,
                        message: "不能使用 {{objName}}.{{propertyName}}",
                        data: {
                            objName,
                            propertyName
                        }
                    });
                }
            }
        };   
    } 
};
```

2、eslint 绑定我们的验证

```javascript
const Linter = require("eslint").Linter;
// 检查的配置
const CONFIG = {
  "parser": "babel-eslint",
  "env": {
    "browser": true,
    "es6": true
  },
  // rules 参考官方文档 https://eslint.bootcss.com/docs/rules/
  rules: {
    'no-eval': 2,//不能使用 eval 
    'no-with': 2,//不能使用 with 
    'no-delete-var': 2,//不能删除变量 delete a; 
    
    'myrule':2, //使用我们的规则
  }
};

// 等待验证的代码
let code = 'eval("console.log(1+1)")';
// 初始化验证对象
let linter = new Linter();
// 映入规则
linter.defineRules({
	"myrule": require('./myrule'), // 引入js 文件
});
// 开始验证
let messages = linter.verify(code, CONFIG);
```



### 结语

ESLint 还能结合各类编辑器 vscode [插件](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) , WebStorm [插件](https://plugins.jetbrains.com/plugin/7494-eslint) ,可以结合自己的需要，发挥想象力。[官方介绍](https://eslint.bootcss.com/docs/user-guide/integrations/)