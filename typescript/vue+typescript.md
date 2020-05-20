# vue typescript note

记录 vue + typescript 的笔记。

## 配置文件 tsconfig.json

如果一个目录下存在一个 `tsconfig.json` 文件，那么它意味着这个目录是 `TypeScript` 项目的根目录。

- 不带任何输入文件的情况下调用 `tsc`，编译器会从当前目录开始去查找 `tsconfig.json`文件，逐级向上搜索父目录。
- 不带任何输入文件的情况下调用 `tsc`，且使用命令行参数 `--project（或-p）`指定一个包含 `tsconfig.json`文件的目录。

`tsconfig.json` 每个属性解析

- `compilerOptions`：编译选项
    + `types 或 typeRoots`：可以禁止引入 `node_modules` 下面的 `@types`。
    + [其他属性](https://www.tslang.cn/docs/handbook/compiler-options.html)
- `files`：指定一个包含相对或绝对文件路径的列表。(需要编译那些文件)
- `includes 或者 exclude`：指定一个文件glob匹配模式列表。（需要编译那些文件）
- `extends`：用于继承别的 `tsconfig.json`
- `compileOnSave`：在保存 `.ts` 文件的时候重新编译。

## vue typescript 特别库

- `vue-class-component`： vue组件的装饰器。
- `vue-property-decorator`： vue组件属性装饰器。

## 基础类型

- 布尔值： `let isDone: boolean = false;`
- 数字： `const num: number = 123;`
- 字符串： `const str: string = 'abc';`
- 数组： `const arr: number[] = [1, 2, 3];`
- 元组： `const `
- 枚举： `enum { Red = 1, Green, Blue }`
- Any： `let notSure: any = 4;`
- void： `const fn = (): void => ()`
- Null： `let u: null = null;`
- Undefined: `let un: undefined = undefined;`
- Never： `const error = (): never => throw new Error('error')`never类型是那些总是会抛出异常或根本就不会有返回值;never类型是任何类型的子类型
- Object
- 类型断言<>：`(<string>str).length`。类型断言好比其它语言里的类型转换;或者说确保你的执行上下文是这个类型，因为你要查看的属性或者调用的方法要明确是这个数据类型才特有的。比如数组你才特有的方法或者属性，字符串你才特有的方法或者属性。

## 接口 interface

- 可选属性 `?`
- 只读属性: `readonly`
- 函数类型： `(a: string, b: number): void;`
- 可索引类型： `[x: number]: any;`
- 类类型: `constructor(h: number, m: number) {}`，`implements关键字`
- 继承接口： `extends`

## 类 Class

- 类
- 继承
- 公共-私有-保护 `public` `private` `protected`
- readonly修饰符
- 存取器 get set
- 静态属性 static
- 抽象类 abstract
- 把类当作接口使用

## 函数 function

- 函数类型
- 推断类型-可选参数和默认值
- 剩余参数
- this和箭头函数
- 重载

## 泛型 <T>

- 泛型变量<T>
- 泛型类型<T, K>
- 泛型类
- 泛型约束

## 枚举

- 数字枚举
- 字符串枚举
- 异构枚举
- 计算的和常量成员
- 联合枚举与枚举成员类型

## 泛型的高级类型

- 交叉类型：将多个类型合并为一个类型
- 联合类型： string | number
- 类型别名 `type`
- 索引类型和字符串索引签名 `keyof`
- `[P in keyof T]?: T[P]`

## Symbol

- `Symbol.hasInstance`
- `Symbol.isConcatSpreadable`
- `Symbol.iterator`
- `Symbol.match`
- `Symbol.replace`
- `Symbol.search`
- `Symbol.species`
- `Symbol.split`
- `Symbol.toPrimitive`
- `Symbol.toStringTag`
- `Symbol.unscopables`

## 命名空间 namespace 和 模块

- declare 定义的类型只会用于编译时的检查，编译结果中会被删除。
- `declare module moduleName`

## 注意类型导入顺序

类型导入是有顺序的。一般编辑器会提示。

## interface 扩展 ， 第三方库扩展， 原生属性扩展 ，类的属性扩展

一般默认情况下你要扩展一个interface可以这样子

```ts
interface demo {
    a: string;
}

interface demo {
    b: string;
}

// 最终结果是 
interface demo {
    a: string;
    b: string;
}
```

如果是第三方库

```ts
declare module 'vuex' {
    interface SomeInterface {
        // 你要扩展的属性
    }
}
```

如果是原生的，默认在 `typescript/lib` 中的。例如如何扩展原生DOM元素 `HTMLElement`

```ts
declare global {
    interface HTMLElement {
        // 你需要扩展的属性
    }
}
```

类加`interface`实现扩展

```ts
class Foo {
  x: number;
}
// ... elsewhere ...
interface Foo {
  y: number;
}
let a: Foo = ...;
console.log(a.x + a.y); // OK
```

类加 `namespace + export` 实现扩展

```ts
class C {
}
// ... elsewhere ...
namespace C {
  export let x: number;
}
let y = C.x; // OK
```

`区分值和类型`

```ts
// X 是值
namespace X {
  // Y 是值
  export interface Y { }
  // Z 是值
  export class Z { }
}

// ... elsewhere ...
namespace X {
  // 值：Y 类型：number
  export var Y: number;
  // 值 Z 和 C
  export namespace Z {
    export class C { }
  }
}

// 这里定义X的类型
type X = string;
```

## 声明文件

- 介绍
- 结构
- 举例
- 规范
- 深入
- 模板
- 发布
- 使用

识别库的类型： `全局库` 那些存在 `window` 下的，或者说命名空间是 `window` 的。`模块库`是只能工作在模块记载器的环境下，需要用到模块化系统。

可以声明的：

- `全局变量`： `declare var foo: number;`
- `全局函数`： `declare function greet(greeting: stirng): void;`
- `带属性的对象`： `declare namespace myLib{ let numberDemo: number }`
- `函数重载`： `declare function getWidget(n: number): Widget`

组织类型：

使用命名空间组织类型。
```ts
declare namespace GreetingLib {
    interface LogOptions {
        verbose?: boolean;
    }
    interface AlertOptions {
        modal: boolean;
        title?: string;
        color?: string;
    }
}
```

你也可以在一个声明中创建嵌套的命名空间：
```ts
declare namespace GreetingLib.Options {
    // Refer to via GreetingLib.Options.Log
    interface Log {
        verbose?: boolean;
    }
    interface Alert {
        modal: boolean;
        title?: string;
        color?: string;
    }
}
```

定义类: 使用declare class描述一个类或像类一样的对象
```ts
declare class Greeter {
    constructor(greeting: string);

    greeting: string;
    showGreeting(): void;
}
```

核心概念 - `类型` - `值`

`类型`：下面每个声明形式都会创建一个新的类型名称。

- 类型别名声明 `type sn = number | string`
- 接口声明 `interface I { x: number[]; }`
- 类声明 `class C {}`
- 枚举声明 `enum E { A, B, C }`
- 指定某个类型的 `import` 声明

`值`：值是运行时名字，可以在表达式里引用。比如 `let x =5;` 中的 `x` 就是值

- `let` `const` `var` 声明
- 包含值 `namespace` `module`声明
- `enum` 声明
- `class` 声明
- 指向值的 `import` 声明
- `function` 声明

`命名空间`：类型可以存在于命名空间里。

## typescript 的 类型系统

全局的命名空间 `global`。局部的 `namespace` 或者库的 `module`。整个环境在 `global`中，那么在整个程序中的优先级是如何的？

优先级：最大的当然是 `global`。
然后 `global` 中包含很多 `namespace`，以及在 `global中定义的一些类型`

`global`中定义的一些类型有：

- 上面提到的 `declare系列`
- `type`以及 `interface`，在 `ts类型系统` 中，这两个是最基本的，最常用的类型单元。
- 还有 `declare module`，这个是第三方库的。
- `declare functino` `declare class` `declare namespace` `declare 变量` 也是`ts类型系统`中很常用的基本类型单元。

由上所得结果：

`ts类型系统`由这些部分组成:

- `global`
- `declare module`
- `declare namespace`
- `declare class`
- `declare function`
- `declare 变量` `declare const` `declare let` `declare var`
- `type` 以及 `interface`

以上是 `ts类型系统` 的整体架构。其中 `type` 和 `interface`的组合以及扩展另谈。还有泛型的操作对应于 `functor`，可以做一些类型的转变(map :: T => T)。

`typing` 和 `types`：一般用于发布。以及查看一个 `npm` 包的结构。

## type 和 interface 的区别

```ts
interface Person {
    name: string;
    age: number;
    address: string;
}

// 注意： 类型不可以通过索引的方式查看类型，因为它就是一个类型
// 但是 interface 就可以 比如 Person['name'] 可以拿到 name的类型string
// 重点注意： keyof 拿到的是一个类型 或者 联合类型 不可以通过索引的方式查找联合类型的某个类型，但是可以通过 in 操作符遍历联合类型的独个类型
type keys = keyof Person; // 'name' | 'age' | 'address'

// type可以当当用类型表示，也可以用值来表示，如果是值在进行类型比较的时候就是`===`比较，比较值的同时也比较类型
type str = '123';
type num = 123; // str === num 为 false

// type是类型 没有命名空间的概念 而 interface有，interface可以作为命名空间使用
type person = { name: string, age: number, address: string } // 不可以 person.name
type person = { name: string, age: 12, address: string } // 匹配 age的时候一定要是2才匹配，且是全等 ===

// 混合方式不同 交叉操作：&  联合操作：|
type a = { a: 1 };
type b = { b: 2};
type c = a & b; // { a: 1, b: 2 }
type d = '123' | 123 // 只匹配 '123' 或者 123
type e = string | number // 可以匹配 字符串类型和数字类型

interface d { a: 1 }
interface d { b: 2 }

// interface d 等价于
namespace d {
    export const a = 1;
    export const b = 2;
}

// 拥有命名空间的概念 可以使用 索引 d.a d.b 查找类型
// 除了interface拥有命名空间 class也有
type a = d.a;
type b = d.b;
```

类型映射：
```ts
interface map {
    // Person是interface(拥有命名空间的概念)可以通过索引的方式查找类型
    [p in keyof Person]: Person[p];
}
```

## typescript 特殊的 操作符 extends in keyof typeof 三元运算符: infer

- `keyof`：类似`Object.keys`，不过是返回数据类型是 `type`, 注意不可以`keyof` `值`,可以是`interface` 和 `class`。
- `in`：用于遍历 `type 联合类型`
- `typeof`：拿 `值` 的 `类型`
- `extends`：继承,这里要注意继承的右边是`class`还是`interface` 还是 `联合类型` 还是`单个类型`
- `T extends U ? X : Y`：如果`T`包含的类型是`U`包含的类型的 '子集'，那么取结果`X`，否则取结果`Y`。比如`type NonNullable<T> = T extends null | undefined ? never : T;`；T如果是`null 或 undefined`那么取`never`。
- `infer`：表示待推断

## 引用

- [typescript高级技巧](https://www.baidu.com/link?url=Q18VVGx-DzVzWvL2Fru8N4BX0uxxEHEvMeCuI8bmeYGlyQ8N7IDX22FlJsuP2-szJq0naxmQNSSYe5aWvyBRQKK7ksdMIissi3qtg6SD4oS&wd=&eqid=89be340600007cb5000000065eba6310)
