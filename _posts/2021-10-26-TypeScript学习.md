---
layout:     post
title:      "TypeScript学习指南"
date:       2021-10-26 19:00:00
author:     "Mountain"
header-img: "img/post-bg-2015.jpg"
tags:
    - TypeScript
---

[github地址](https://github.com/sy-tech-start/TypeScript-Vue-Starter)

[官方中文文档](https://www.tslang.cn/index.html)

### 1、TypeScript是什么？

TS是javascript类型的超集，可以编译成纯javascript;  --- 静态语言
 ts是js的一种强类型约束，使用时的数据格式必须符合定义时声明的数据格式 --- 强类型

### 2、数据类型

##### 基础数据类型

- 布尔: boolean
- 字符串: string
- 数字: number
- 对象: object
- 数组: (number | string)[]
- 方法: void
- 枚举: enum
- symbol
- any
- never
- null
- undefined

##### 复杂数据类型

- 接口: Interface -- 一般是为复杂object类型提供结构类型检查
- 类: Class
- 交叉类型(&) : Female & Male
- 联合类型(|) : Female | Male
- 泛型(T): --- 一种类型约束，定义时不确定类型并确定约束，使用时确定类型

### 3、类型声明的方式

#### 声明:

```
type age = number

type GridConstructor = new (...args: any[]) => LitElement & GridInterface & BaseInterface;

interface GridInterface {
  grid: GridAttrs;
  gridProperties: GridProperties;
  gridOption: GridOption;
}

class Person {
    name: string = "张三";
    age: number = 18
    
    getName() {
        return this.name
    }
    setName() {
        this.name = "李四"
    }         
}
```

#### 使用:

```
const name: string = "张三"
const age: number = 18

const person = new Person();
console.log(person.name)
console.log(person.group) // error: Property 'group' does not exist on type Person
```

#### 声明的作用域

- 局部作用

  - 声明在文件内部
  - import的方式导入声明

- 全局作用

  - .d.ts文件， 并且使用 declare 关键字声明类型 --- 发现

    .ts文件也可以起到全局声明的作用

    - 使用*.d.ts文件而非 *.ts文件  因为tsc会对.ts文件编译 不会对.d.ts文件进行编译

  - 在全局声明文件中不可使用import "**" 的导入方式，会被降级为模块局部声明

  - 使用import() [参考](https://stackoverflow.com/questions/39040108/import-class-in-definition-file-d-ts)

- declare关键字

  - 不存在的时候需要使用declare声明(比如声明一些没有类型文件的第三方库的数据结构)
  - [When to use](https://dzone.com/articles/quick-tip-–-typescript-declare?utm_source=devglan)
  - [使用意图](https://stackoverflow.com/questions/43335962/purpose-of-declare-keyword-in-typescript)
  - declare global --- 在使用import的文件中进行全局声明

### 4、接口

一种声明对象内部具体数据结构的方式
 参考: interface.ts

### 5、类

- 类的修饰符
  - puclic --- (类内部)this. 、(子类内部)subClass.this. 、 (实例)intance. 几种方式都可访问
  - private --- 仅(类内部)this. 可以访问
  - protected --- 仅(类内部)this. 、(子类内部)subClass.this.可以访问
  - readonly --- 只读 不可修改
  - get --- 读取操作拦截器
  - set --- 赋值操作拦截器
  - static --- 仅可通过Class. 或者 (类内部)this. 可以访问
  - abstract --- 只能被继承 不能被实例化
- 接口与类
  - 类实现接口, 使用 implements 关键字
  - 接口描述了类的公共部分，而不是公共和私有两部分。 它不会帮你检查类是否具有某些私有成员。
  - 接口可以继承接口(可以同时继承多个接口)
  - 接口可以继承类 --- 遇到类(A)中带有private/protect修饰符的属性，该接口只能被类A或其子类实现
  - 混合类型?  ---同时作为函数与对象使用

### 6、模块与命名空间

#### 模块

- 导入、导出的写法与es6大致相同

#### 命名空间

- 防止类型/变量冲突

#### 第三方模块的类型声明

```
/// <reference path="node.d.ts"/>
import * as URL from "url";
let myUrl = URL.parse("http://www.typescriptlang.org");
// 简写
// 声明
declare module "hot-new-module";

// 使用
import x, {y} from "hot-new-module";
x(y);
```

### 7、装饰器

装饰器是实验特性，需要启用experimentalDecorators配置：
 [ES6中的装饰器](https://es6.ruanyifeng.com/#docs/decorator)
 文章推荐: [使用 TypeScript 装饰器装饰你的代码](https://juejin.cn/post/6844903876605280269)

### 8、Mixin

官方推荐的示例能够满足功能，但是需要在最终使用的类中声明一下所有的属性及其类型，太麻烦

使用装饰器?

### 9、ts编译与配置

##### 编译方式

- tsc
- 打包工具 + 插件
  - webpack + ts-loader
  - webpack + babel + 类型检查插件
  - rollup + typescript插件

##### 编译配置方式

[参考: 官方中文文档](https://www.tslang.cn/docs/handbook/compiler-options.html)

- 通过命令行参数配置
- 配置文件(tsconfig.json)
  - 从根目录开始逐层查找, 最里面一级的配置文件为准
  - 可以继承 --- 解决公共配置修改问题

### 10、遇到编辑器类型报错/编译报错怎么办?

- 自己解决
  - [多读字面意思](https://www.typescriptlang.org/docs/handbook/2/understanding-errors.html)
  - [错误信息列表](https://www.tslang.cn/docs/handbook/error.html)
- google / baidu
- discord --- chat tool
- github-issues
- stack overflow
- 不要使用any !important

### 11、问题

- 为什么在generic.ts文件中没有import 也能使用 interface.ts文件中的类型接口？  该接口未全局声明
  - 原因: 没有使用export / import 等关键字 默认全局声明

#### 文章推荐

- [TS 学习指南](https://juejin.cn/post/6872111128135073806)
- [Vue+Ts入门实践](https://juejin.cn/post/6844903865255477261#heading-25)