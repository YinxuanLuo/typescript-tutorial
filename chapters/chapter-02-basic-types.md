# 第二章：TypeScript基础类型

## 📋 目录

- [2.1 原始数据类型](#21-原始数据类型)
- [2.2 数组类型](#22-数组类型)
- [2.3 元组类型](#23-元组类型)
- [2.4 枚举类型](#24-枚举类型)
- [2.5 any与unknown](#25-any与unknown)
- [2.6 void与never](#26-void与never)
- [2.7 类型推断](#27-类型推断)
- [2.8 类型断言](#28-类型断言)

---

## 2.1 原始数据类型

### 2.1.1 概述

TypeScript支持与JavaScript相同的7种原始数据类型：

```
┌─────────────────────────────────────────────────────────────────────┐
│                      JavaScript/TypeScript 原始类型                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐                 │
│   │ string  │  │ number  │  │ boolean │  │ null    │                 │
│   └─────────┘  └─────────┘  └─────────┘  └─────────┘                 │
│                                                                     │
│   ┌─────────┐  ┌─────────┐  ┌─────────┐                              │
│   │ undefined│  │ symbol │  │ bigint  │                              │
│   └─────────┘  └─────────┘  └─────────┘                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.1.2 字符串类型 (string)

```typescript
// 字符串声明方式
let name1: string = "张三";           // 双引号
let name2: string = '李四';           // 单引号
let name3: string = `王五`;           // 反引号（模板字符串）

// 模板字符串（可以嵌入变量）
let age: number = 25;
let greeting: string = `Hello, my name is ${name1}, I'm ${age} years old.`;
// Hello, my name is 张三, I'm 25 years old.

// 字符串常用操作
let str: string = "TypeScript";
console.log(str.length);           // 10
console.log(str.toUpperCase());    // TYPESCRIPT
console.log(str.substring(0, 4));  // Type
```

### 2.1.3 数字类型 (number)

```typescript
// 整数和浮点数都使用number类型
let integer: number = 42;          // 整数
let decimal: number = 3.14;         // 浮点数
let negative: number = -10;         // 负数

// 不同进制的表示
let binary: number = 0b1010;       // 二进制：10
let octal: number = 0o12;           // 八进制：10
let hex: number = 0xA;              // 十六进制：10
let decimal2: number = 1_000_000;  // 下划线分隔：1000000

// 特殊值
let infinity: number = Infinity;
let negInfinity: number = -Infinity;
let nan: number = NaN;              // Not a Number

// 数学运算
let sum: number = 10 + 5;          // 15
let product: number = 4 * 3;        // 12
let division: number = 10 / 3;      // 3.333...
let remainder: number = 10 % 3;      // 1
```

### 2.1.4 布尔类型 (boolean)

```typescript
let isActive: boolean = true;
let isCompleted: boolean = false;

// 布尔运算
let andResult: boolean = true && false;   // false
let orResult: boolean = true || false;     // true
let notResult: boolean = !true;            // false

// 条件判断
let age: number = 18;
let isAdult: boolean = age >= 18;           // true
```

### 2.1.5 null和undefined

```typescript
// null：表示刻意为空
let empty: null = null;

// undefined：表示未定义
let notDefined: undefined = undefined;

// 在严格模式下，null和undefined不能赋值给其他类型
let name: string = "张三";
// name = null;  // Error in strict mode

// 如果需要，可以启用严格null检查
let name2: string | null = null;  // 使用联合类型
```

### 2.1.6 symbol

```typescript
// symbol：表示唯一的标识符
let sym1: symbol = Symbol("key");
let sym2: symbol = Symbol("key");

console.log(sym1 === sym2);  // false - 每个symbol都是唯一的

// 常用作对象属性的键
const obj: { [sym1]: string } = {
  [sym1]: "value"
};
```

### 2.1.7 bigint

```typescript
// bigint：表示大于2^53-1的整数
let hugeNumber: bigint = 9007199254740991n;
let bigger: bigint = BigInt(9007199254740991);

// 运算
let sum: bigint = 100n + 200n;       // 300n
let product: bigint = 3n * 4n;       // 12n

// 注意：number和bigint不能混合运算
// let wrong: bigint = 100n + 1;  // Error
```

---

## 2.2 数组类型

### 2.2.1 数组声明方式

```typescript
// 方式一：类型[]
let arr1: number[] = [1, 2, 3, 4, 5];
let arr2: string[] = ["a", "b", "c"];
let arr3: boolean[] = [true, false, true];

// 方式二：Array<类型>
let arr4: Array<number> = [1, 2, 3];
let arr5: Array<string> = ["x", "y", "z"];

// 只读数组（不能修改）
let readonlyArr: ReadonlyArray<number> = [1, 2, 3];
// readonlyArr.push(4);  // Error: Property 'push' does not exist
// readonlyArr[0] = 10;  // Error: Cannot assign to '0'
```

### 2.2.2 多维数组

```typescript
// 二维数组
let matrix1: number[][] = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

// 三维数组
let matrix2: number[][][] = [
  [[1, 2], [3, 4]],
  [[5, 6], [7, 8]]
];

// 访问元素
console.log(matrix1[0][0]);  // 1
console.log(matrix1[1][2]);  // 6
```

### 2.2.3 数组常用方法

```typescript
let nums: number[] = [1, 2, 3, 4, 5];

// 遍历
nums.forEach(num => console.log(num));

// map - 映射
let doubled: number[] = nums.map(num => num * 2);  // [2, 4, 6, 8, 10]

// filter - 过滤
let evens: number[] = nums.filter(num => num % 2 === 0);  // [2, 4]

// reduce - 汇总
let sum: number = nums.reduce((acc, num) => acc + num, 0);  // 15

// find - 查找
let found: number | undefined = nums.find(num => num > 3);  // 4

// some/every - 判断
let hasEven: boolean = nums.some(num => num % 2 === 0);     // true
let allPositive: boolean = nums.every(num => num > 0);     // true

// sort - 排序
let sorted: number[] = [...nums].sort((a, b) => a - b);    // [1, 2, 3, 4, 5]
```

### 2.2.4 数组类型推断

```typescript
// TypeScript会自动推断数组类型
let autoInferred = [1, 2, 3];
// 推断为：number[]

let mixed = [1, "2", true];
// 推断为：(number | string | boolean)[]

// 如果数组只包含同类型，建议明确声明
let numbers: number[] = [1, 2, 3];  // 明确声明
```

---

## 2.3 元组类型

### 2.3.1 什么是元组？

```
┌─────────────────────────────────────────────────────────────────────┐
│                         数组 vs 元组                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   数组 Array - 同类型的有序集合                                      │
│   ┌────┬────┬────┬────┐                                            │
│   │ 1  │ 2  │ 3  │ 4  │  → number[]                                │
│   └────┴────┴────┴────┘                                            │
│                                                                     │
│   元组 Tuple - 不同类型的有序集合（固定长度）                         │
│   ┌────┬────┬────┐                                                  │
│   │"zs"│ 25 │true│  → [string, number, boolean]                     │
│   └────┴────┴────┘                                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.3.2 元组基本用法

```typescript
// 声明元组
let person: [string, number, boolean] = ["张三", 25, true];

// 访问元素
console.log(person[0]);  // 张三
console.log(person[1]);  // 25
console.log(person[2]);  // true

// 修改元素
person[0] = "李四";
// person[0] = 123;  // Error: Type 'number' is not assignable to type 'string'

// 解构赋值
const [name, age, isStudent] = person;
console.log(name);  // 李四
```

### 2.3.3 可选元组元素

```typescript
// 可选元素（必须放在必需元素之后）
let optionalTuple: [string, number?, boolean?] = ["张三"];
// 相当于 [string, number | undefined, boolean | undefined]

optionalTuple = ["张三", 25];
optionalTuple = ["张三", 25, true];

// 使用?.可选链
optionalTuple = ["张三", undefined, true];
```

### 2.3.4 命名元组

```typescript
// 为元素命名，提高可读性
let person: [name: string, age: number, isStudent: boolean] = ["张三", 25, true];

// 访问时可以按名称访问（实际还是按索引）
console.log(person.name);    // Error: 'name' does not exist on type
console.log(person[0]);      // 张三

// 更好的方式是使用接口或对象
```

### 2.3.5 实际应用场景

```typescript
// 1. 函数返回多个值
function getUserInfo(): [string, number, string] {
  return ["张三", 25, "北京"];
}

// 2. 字典/映射表
let dictionary: [string, any][] = [
  ["name", "张三"],
  ["age", 25],
  ["active", true]
];

// 3. RGB颜色
type RGB = [number, number, number];
const red: RGB = [255, 0, 0];
const green: RGB = [0, 255, 0];
const blue: RGB = [0, 0, 255];

// 4. 坐标点
type Point = [x: number, y: number];
const origin: Point = [0, 0];
const pointA: Point = [3, 4];
```

---

## 2.4 枚举类型

### 2.4.1 数字枚举

```typescript
// 数字枚举 - 默认从0开始
enum Direction {
  Up,      // 0
  Down,    // 1
  Left,    // 2
  Right    // 3
}

// 使用
let dir: Direction = Direction.Up;
console.log(dir);           // 0
console.log(Direction[0]);  // "Up" - 反向映射

// 自定义起始值
enum Status {
  Pending = 1,
  Active = 2,
  Completed = 4,
  Failed = 8
}

// 计算值
enum Color {
  Red = 1,
  Green = 2,
  Blue = 4
}
```

### 2.4.2 字符串枚举

```typescript
// 字符串枚举 - 每个成员必须初始化
enum Message {
  Success = "成功",
  Error = "错误",
  Warning = "警告",
  Info = "信息"
}

// 使用
let msg: Message = Message.Success;
console.log(msg);  // "成功"

// 字符串枚举没有反向映射
// console.log(Message["成功"]);  // Error
```

### 2.4.3 异构枚举（混合）

```typescript
// 混合数字和字符串（不推荐）
enum Hybrid {
  No = 0,
  Yes = "YES"
}

// 很少使用，了解即可
```

### 2.4.4 常量枚举

```typescript
// const enum 在编译时会被内联
const enum HttpStatus {
  OK = 200,
  NotFound = 404,
  ServerError = 500
}

// 编译后：let code = 200; （直接内联值，节省代码）
let code: number = HttpStatus.OK;

// 普通枚举编译后：
// var HttpStatus;
// (function (HttpStatus) {
//     HttpStatus[HttpStatus["OK"] = 200] = "OK";
//     ...
// })(HttpStatus || (HttpStatus = {}));
```

### 2.4.5 枚举实战应用

```typescript
// 1. 状态管理
enum OrderStatus {
  Pending = "PENDING",
  Paid = "PAID",
  Shipped = "SHIPPED",
  Delivered = "DELIVERED",
  Cancelled = "CANCELLED"
}

function getStatusText(status: OrderStatus): string {
  const statusMap = {
    [OrderStatus.Pending]: "等待支付",
    [OrderStatus.Paid]: "已支付",
    [OrderStatus.Shipped]: "已发货",
    [OrderStatus.Delivered]: "已送达",
    [OrderStatus.Cancelled]: "已取消"
  };
  return statusMap[status];
}

// 2. 权限控制
enum Permission {
  Read = 1 << 0,    // 1
  Write = 1 << 1,   // 2
  Execute = 1 << 2  // 4
}

function hasPermission(userPerms: number, perm: Permission): boolean {
  return (userPerms & perm) === perm;
}

let userPerms = Permission.Read | Permission.Write;
console.log(hasPermission(userPerms, Permission.Read));    // true
console.log(hasPermission(userPerms, Permission.Execute));   // false
```

---

## 2.5 any与unknown

### 2.5.1 any类型

```typescript
// any - 任意类型，绕过类型检查（尽量少用）
let value: any = "hello";
value = 123;
value = true;
value = { name: "张三" };

// 调用任意方法都不会报错
value.foo();      // 不报错
value.bar.xyz;    // 不报错

// 场景：处理不确定类型的数据
function processData(data: any): void {
  console.log(data);
  // 可以自由处理
}
```

### 2.5.2 unknown类型

```typescript
// unknown - 未知类型，更安全
let value: unknown = "hello";
// value = 123;
// value.foo();  // Error: Object is of type 'unknown'

// 使用前必须进行类型检查
if (typeof value === "string") {
  console.log(value.toUpperCase());  // 安全，只有在确定是string时才调用
}

// 或者使用类型守卫
function isString(val: unknown): val is string {
  return typeof val === "string";
}
```

### 2.5.3 any vs unknown 对比

```
┌─────────────────────────────────────────────────────────────────────┐
│                        any vs unknown                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   any                              unknown                          │
│   ┌─────────────────────┐          ┌─────────────────────┐         │
│   │ • 无类型检查        │          │ • 有类型检查         │         │
│   │ • 可赋值给任何类型  │          │ • 只能赋值给any     │         │
│   │ • 可访问任何属性    │          │ • 需类型检查后才能  │         │
│   │ • 危险⚠️            │          │   访问属性           │         │
│   └─────────────────────┘          │ • 安全✅             │         │
│                                    └─────────────────────┘         │
│                                                                     │
│   建议：优先使用unknown，any作为最后兜底                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

```typescript
// 示例对比
function processAny(value: any) {
  value.foo();  // 不报错 - 危险！
}

function processUnknown(value: unknown) {
  // value.foo();  // Error!
  
  if (typeof value === "object" && value !== null && "foo" in value) {
    (value as { foo: () => void }).foo();
  }
}
```

---

## 2.6 void与never

### 2.6.1 void类型

```typescript
// void - 表示没有返回值（用于函数）
function logMessage(message: string): void {
  console.log(message);
  // 没有return或return undefined
}

// 注意：返回undefined和void有区别
function returnUndefined(): void {
  return undefined;  // 合法
}

// void vs undefined
let voidVal: void = undefined;  // 合法
let undefVal: undefined = voidVal;  // 合法（void返回undefined）

// 变量很少用void，通常是函数返回值
let useless: void;  // 合法，但用处不大
```

### 2.6.2 never类型

```typescript
// never - 表示永不返回
// 1. 函数总是抛出异常
function throwError(message: string): never {
  throw new Error(message);
}

// 2. 函数无限循环
function infiniteLoop(): never {
  while (true) {
    // 永远不会结束
  }
}

// 3. 条件分支穷尽检查
type Shape = { kind: "circle"; radius: number } | { kind: "rectangle"; width: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    default:
      // shape: never - 穷尽检查
      const _exhaustive: never = shape;
      throw new Error("Unknown shape");
  }
}

// never赋值
let neverVal: never;
// neverVal = 1;           // Error
// neverVal = "string";   // Error
// neverVal = undefined;  // Error

// never可以赋值给任何类型
let anyVal: any = neverVal;  // 合法
```

### 2.6.3 void vs never 对比

```
┌─────────────────────────────────────────────────────────────────────┐
│                        void vs never                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   void                              never                           │
│   ┌─────────────────────┐          ┌─────────────────────┐         │
│   │ 函数没有返回值       │          │ 函数永不返回         │         │
│   │ 可能返回undefined   │          │ 抛出异常/无限循环    │         │
│   │ 是返回值类型        │          │ 是返回值类型         │         │
│   └─────────────────────┘          └─────────────────────┘         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.7 类型推断

### 2.7.1 自动类型推断

```typescript
// TypeScript会自动推断变量类型
let x = 3;          // 推断为 number
let y = "hello";    // 推断为 string
let z = true;       // 推断为 boolean

// 数组
let nums = [1, 2, 3];      // number[]
let mixed = [1, "2"];     // (string | number)[]

// 对象
let obj = { name: "张三", age: 25 };
// 推断为 { name: string; age: number }

// 函数返回值
function add(a: number, b: number) {
  return a + b;  // 推断返回值为 number
}
```

### 2.7.2 类型推断规则

```typescript
// 1. 从右向左推断
let name = "张三";  // string

// 2. 从使用位置推断（ contextual typing）
window.onload = function() {
  // 从window.onload类型推断参数为Event
};

// 3. 最佳通用类型推断
let arr = [1, "2", true];
// 推断为 (number | string | boolean)[]

// 4. 链式调用推断
let len = "hello".length;  // number
```

### 2.7.3 何时需要显式声明类型？

```typescript
// 1. 变量声明时未初始化
let value: string;  // 必须声明类型
value = "hello";

// 2. 函数参数
function greet(name: string): string {
  return `Hello, ${name}`;
}

// 3. 返回类型需要明确时
function parseJSON(json: string): Record<string, any> {
  return JSON.parse(json);
}

// 4. 复杂类型
let complex: Map<string, { name: string; age: number }[]> = new Map();
```

---

## 2.8 类型断言

### 2.8.1 基本语法

```typescript
// 方式一：as语法（推荐）
let value: unknown = "hello world";
let len: number = (value as string).length;

// 方式二：尖括号语法
let len2: number = (<string>value).length;

// 注意：在JSX中只能使用as语法
```

### 2.8.2 常见使用场景

```typescript
// 1. DOM元素类型转换
const input = document.getElementById("username") as HTMLInputElement;
input.value = "张三";

// 2. 处理联合类型
type StringOrNumber = string | number;
function processValue(val: StringOrNumber) {
  if (typeof val === "string") {
    console.log(val.toUpperCase());  // TS知道是string
  } else {
    console.log(val.toFixed(2));     // TS知道是number
  }
}

// 3. 断言为更具体的类型
interface Person {
  name: string;
  age: number;
}

interface Employee extends Person {
  employeeId: string;
}

function greet(person: Person) {
  // 可能是Employee
  const emp = person as Employee;
  console.log(emp.employeeId);
}
```

### 2.8.3 非空断言

```typescript
// 1. 使用!进行非空断言
let name: string | null = null;
// console.log(name.length);  // Error: 'name' is possibly 'null'

name = "张三";
console.log(name!.length);  // 3 - 断言name不为null

// 2. 可选链 + 非空断言
interface Config {
  db?: {
    host: string;
    port: number;
  };
}

const config: Config = {};
const host = config.db!.host;  // 假设已知db存在

// 3. 更好的方式：使用可选链
const host2 = config.db?.host ?? "localhost";
```

### 2.8.4 类型守卫

```typescript
// instanceof类型守卫
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function speak(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}

// typeof类型守卫
function padLeft(value: string | number, padding: string | number) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + value;  // value被当作string
  }
  return padding + value;
}

// 自定义类型守卫
function isString(val: unknown): val is string {
  return typeof val === "string";
}

function process(val: unknown) {
  if (isString(val)) {
    console.log(val.toUpperCase());  // val被当作string
  }
}
```

---

## 📊 基础类型速查表

```
┌──────────────┬──────────────────────────────────────────────────────┐
│ 类型         │ 示例                                                  │
├──────────────┼──────────────────────────────────────────────────────┤
│ string       │ "hello", 'world', `template`                          │
│ number       │ 42, 3.14, 0xFF, 0b1010                               │
│ boolean      │ true, false                                          │
│ null         │ null                                                  │
│ undefined    │ undefined                                              │
│ symbol       │ Symbol("id")                                          │
│ bigint       │ 42n                                                   │
│ array        │ number[], Array<number>                              │
│ tuple        │ [string, number, boolean]                             │
│ enum         │ enum Direction { Up, Down }                          │
│ any          │ 任意类型                                               │
│ unknown      │ 未知类型（安全版any）                                  │
│ void         │ 函数无返回值                                           │
│ never        │ 函数永不返回                                           │
└──────────────┴──────────────────────────────────────────────────────┘
```

---

## 💪 实战练习

### 练习1：基础类型
```typescript
// 声明以下变量
// 1. 一个名为username的字符串，值为"admin"
// 2. 一个名为age的数字，值为30
// 3. 一个名为isActive的布尔，值为true
// 4. 一个名为tags的字符串数组，值为["admin", "user", "guest"]

// 你的代码：
```

### 练习2：元组使用
```typescript
// 创建一个表示HTTP请求的元组
// [method: string, url: string, statusCode: number]
// 包含：POST, "/api/users", 201

// 你的代码：
```

### 练习3：枚举应用
```typescript
// 创建一个OrderStatus枚举
// 包含：Pending(待处理), Processing(处理中), Completed(完成), Failed(失败)
// 写一个函数，根据状态返回中文描述

// 你的代码：
```

---

## ⏭️ 下一步

前往 [第三章：类型系统深入](./chapter-03-type-system.md) 学习联合类型、交叉类型等高级概念

---

> 💡 **思考题**：
> 1. 为什么推荐使用unknown而不是any？
> 2. 什么情况下应该使用never类型？
> 3. 类型断言和非空断言有什么区别？