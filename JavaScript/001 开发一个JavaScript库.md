# 开发一个JavaScript库

前言：因为工作需要制作一个Js的音视频库组件，flvJs并不能满足正常的需求，自己也没有做过相关的开发，所以需要学习一下子，嘿嘿。

## 1. 代码规范

Js规范从检索的内容来看，大概两种开发规范比较普遍：`commonjs`和`es6`

1. `commonjs`是node的模块加载规范，一般用这种规范写node程序
2. `es6`是`ECMAScript2015`定义的模块加载规范，但是貌似都不是很支持，但是可以推荐使用`es6`规范来写代码，再由工具转为其他原生Js能够运行的格式。

## 2. 构建工具选择

构建工具我决定采用`webpack`，`webpack`可以将Js打包时导出`and/commonjs/umd`规范。

## 3. 脚手架

决定采用`npm`

## 4.JavaScript组件化写法

### 4.1 入门级写法

入门写法就是完完全全的全局函数，全局变量的写法。

```javascript
$(function(){
    var input = $('#input');

    function getSize(s){
        return s.length;
    }
    //...
})
```

### 4.2 作用域隔离

使用单个变量模拟命名空间。

```javascript
var test = {
    input: null,
    // 初始化
    init: function(config){
        this.input = $(config.id);
        this.bind();
        return this;
    },
    // 绑定
    bind: function(){

    },
    getSize: function(){
        return this.input.val().length;
    },
    // 渲染元素
    render: function(){

    }
}

$(function(){
    // 再dom ready后调用
    test.init({id: '#input'}).render();
})
```
