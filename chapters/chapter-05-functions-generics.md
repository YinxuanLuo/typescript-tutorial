# 第五章：函数与泛型

## 📋 目录

- [5.1 函数类型基础](#51-函数类型基础)
- [5.2 泛型入门](#52-泛型入门)
- [5.3 泛型约束](#53-泛型约束)
- [5.4 泛型类和接口](#54-泛型类和接口)
- [5.5 泛型工具类型](#55-泛型工具类型)
- [5.6 多泛型参数](#56-多泛型参数)
- [5.7 泛型条件与推断](#57-泛型条件与推断)
- [5.8 实际应用场景](#58-实际应用场景)

---

## 5.1 函数类型基础

### 5.1.1 函数声明与类型

```typescript
// 普通函数
function add(a: number, b: number): number {
  return a + b;
}

// 函数表达式
const add2: (a: number, b: number) => number = function(a, b) {
  return a + b;
};

// 箭头函数
const add3: (a: number, b: number) => number = (a, b) => a + b;

// 类型别名
type AddFn = (a: number, b: number) => number;
const add4: AddFn = (a, b) => a + b;
```

### 5.1.2 函数类型详解

```typescript
// 函数类型包含：
// 1. 参数类型
// 2. 返回值类型
// 3. this类型（可选）
// 4. 泛型（可选）

// 完整函数类型
interface Callback {
  (error: Error | null, result: string | null): void;
  metadata?: Record<string, string>;
}

// 使用
const callback: Callback = (err, result) => {
  if (err) {
    console.error(err.message);
  } else {
    console.log(result);
  }
};
```

### 5.1.3 可选参数

```typescript
// 可选参数：参数名后加?
function greet(name: string, greeting?: string): string {
  if (greeting) {
    return `${greeting}, ${name}!`;
  }
  return `Hello, ${name}!`;
}

greet("张三");           // "Hello, 张三!"
greet("张三", "Hi");    // "Hi, 张三!"

// 可选参数必须在必需参数之后
// 错误示例：
// function wrong(a?: number, b: number) {}  // Error

// 正确示例：
function correct(a: number, b?: number) {}
```

### 5.1.4 默认参数

```typescript
// 默认参数
function createUser(
  name: string,
  age: number = 18,
  role: string = "user"
): User {
  return { name, age, role };
}

// 使用
createUser("张三");           // { name: "张三", age: 18, role: "user" }
createUser("李四", 25);      // { name: "李四", age: 25, role: "user" }
createUser("王五", 30, "admin"); // { name: "王五", age: 30, role: "admin" }

// 默认参数可以是表达式
function getUrl(base: string, path: string = "/"): string {
  return `${base}${path}`;
}
```

### 5.1.5 剩余参数

```typescript
// rest参数
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, n) => acc + n, 0);
}

sum(1, 2, 3);        // 6
sum(1, 2, 3, 4, 5); // 15

// 多个参数
function printAll(names: string[], ...flags: boolean[]): void {
  names.forEach((name, i) => {
    console.log(`${name}: ${flags[i] ?? false}`);
  });
}

printAll(["a", "b", "c"], true, false, true);
```

### 5.1.6 this类型

```typescript
// 函数中的this类型
// 需要在函数参数中声明this的类型

interface UIElement {
  addClickListener(onClick: (this: void, event: MouseEvent) => void): void;
}

class Handler {
  info: string = "";
  
  // this类型为void，表示不依赖this
  handleClick(this: void, event: MouseEvent): void {
    console.log("Clicked!");
  }
}

const h = new Handler();
const ui: UIElement = {
  addClickListener(onClick) {
    onClick.call(void 0, new MouseEvent("click"));  // 手动绑定this
  }
};

ui.addClickListener(h.handleClick);  // 不会报错
```

### 5.1.7 函数重载

```typescript
// 函数重载：同一函数名，不同参数类型/个数
function reverse(str: string): string;           // 重载1
function reverse(arr: number[]): number[];      // 重载2
function reverse(arr: string[]): string[];      // 重载3
function reverse(arr: string | number[]): string | number[] {
  if (typeof arr === "string") {
    return arr.split("").reverse().join("");
  }
  return arr.slice().reverse();
}

// 使用
reverse("hello");        // "olleh"
reverse([1, 2, 3]);       // [3, 2, 1]
reverse(["a", "b", "c"]); // ["c", "b", "a"]

// 真实场景：查找元素
function find<T>(
  array: T[],
  predicate: (item: T) => boolean
): T | undefined;         // 重载1：返回单个或undefined

function find<T>(
  array: T[],
  predicate: (item: T) => boolean,
  defaultValue: T
): T;                    // 重载2：返回默认值类型

function find<T>(
  array: T[],
  predicate: (item: T) => boolean,
  defaultValue?: T
): T | undefined {
  const found = array.find(predicate);
  return found !== undefined ? found : defaultValue;
}

const nums = [1, 2, 3, 4, 5];
find(nums, n => n > 3);              // 4
find(nums, n => n > 10, -1);        // -1
```

---

## 5.2 泛型入门

### 5.2.1 什么是泛型？

```typescript
// 泛型：参数化类型，使代码能够适用于多种类型
// 避免重复代码，同时保持类型安全

// 不使用泛型：需要为每种类型编写单独函数
function identityNumber(arg: number): number {
  return arg;
}

function identityString(arg: string): string {
  return arg;
}

// 使用泛型：一个函数，适用于多种类型
function identity<T>(arg: T): T {
  return arg;
}

const num = identity<number>(42);      // 类型参数为number
const str = identity<string>("hello");  // 类型参数为string
const inferred = identity(42);          // 类型被推断为number
```

### 5.2.2 泛型函数

```
┌─────────────────────────────────────────────────────────────────────┐
│                         泛型函数图解                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   function identity<T>(arg: T): T                                   │
│          │                                                          │
│          ├── T 是类型参数（类型变量）                                 │
│          │                                                          │
│          └── arg: T 表示参数类型为T，返回类型也为T                    │
│                                                                     │
│   使用时：                                                           │
│   identity<number>(42)  → T = number                               │
│   identity<string>("x") → T = string                               │
│   identity(true)        → T = boolean (推断)                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.2.3 泛型变量

```typescript
// 使用泛型变量处理更复杂的类型
function loggingIdentity<T>(arg: T): T {
  console.log(arg.length);  // Error: T不一定有length属性
  return arg;
}

// 正确做法：使用泛型约束
function loggingIdentity<T>(arg: T[]): T[] {
  console.log(arg.length);  // OK: T[]肯定有length
  return arg;
}

// 或者
function loggingIdentity2<T extends { length: number }>(arg: T): T {
  console.log(arg.length);  // OK: 约束了必须有length
  return arg;
}
```

### 5.2.4 泛型类型

```typescript
// 泛型函数类型
interface GenericIdentityFn {
  <T>(arg: T): T;
}

// 或者
type GenericIdentity = <T>(arg: T) => T;

// 带类型参数的接口
interface GenericIdentityFn2<T> {
  (arg: T): T;
  defaultValue: T;
}

const stringIdentity: GenericIdentityFn2<string> = {
  (arg) => arg,
  defaultValue: "default"
};
```

### 5.2.5 泛型类

```typescript
// 泛型类
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
  
  constructor(zeroValue: T) {
    this.zeroValue = zeroValue;
    this.add = (x, y) => x + y;
  }
}

const numberGeneric = new GenericNumber<number>(0);
numberGeneric.add(1, 2);           // 3
// numberGeneric.add("a", "b");   // Error

const stringGeneric = new GenericNumber<string>("");
stringGeneric.add("hello", " world");  // "hello world"

// 注意：静态成员不能使用类类型参数
// class Wrong<T> {
//   static defaultValue: T;  // Error: 静态成员不能引用类类型参数
// }
```

---

## 5.3 泛型约束

### 5.3.1 基本约束

```typescript
// 使用extends关键字约束泛型
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);  // OK: 现在保证有length
  return arg;
}

loggingIdentity("hello");        // OK
loggingIdentity([1, 2, 3]);      // OK
loggingIdentity({ length: 10 });  // OK
// loggingIdentity(123);         // Error: number没有length
```

### 5.3.2 多重约束

```typescript
// 多个约束条件
interface Printable {
  print(): void;
}

interface Serializable {
  serialize(): string;
}

function process<T extends Printable & Serializable>(obj: T): void {
  obj.print();
  console.log(obj.serialize());
}

// 联合类型约束
function min<T extends Comparable<T>>(a: T, b: T): T {
  // Comparable需要实现比较逻辑
  return a.compareTo(b) < 0 ? a : b;
}
```

### 5.3.3 keyof约束

```typescript
// 使用keyof约束泛型
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = {
  name: "张三",
  age: 25,
  active: true
};

const name = getProperty(person, "name");    // string
const age = getProperty(person, "age");      // number
// getProperty(person, "invalid");           // Error: 不存在的属性

// 完整示例：设置属性
function setProperty<T, K extends keyof T>(
  obj: T,
  key: K,
  value: T[K]
): T[K] {
  obj[key] = value;
  return value;
}

setProperty(person, "name", "李四");    // OK
// setProperty(person, "name", 123);      // Error: 类型不匹配
```

### 5.3.4 泛型约束的实际应用

```typescript
// 1. 创建指定类型的数组
function createArray<T>(length: number, value: T): T[] {
  return Array(length).fill(value);
}

createArray(3, "x");     // ["x", "x", "x"]
createArray(3, 0);       // [0, 0, 0]

// 2. 从对象数组中提取属性
function pluck<T, K extends keyof T>(array: T[], key: K): T[K][] {
  return array.map(item => item[key]);
}

const users = [
  { name: "张三", age: 25 },
  { name: "李四", age: 30 }
];

const names = pluck(users, "name");   // string[]
const ages = pluck(users, "age");     // number[]

// 3. 不可变对象
type Immutable<T> = {
  readonly [K in keyof T]: T[K];
};

function freeze<T extends object>(obj: T): Immutable<T> {
  return Object.freeze(obj);
}

const frozen = freeze({ x: 1, y: 2 });
// frozen.x = 3;  // Error: Cannot assign to 'x'
```

---

## 5.4 泛型类和接口

### 5.4.1 泛型接口

```typescript
// 泛型接口
interface Pair<K, V> {
  key: K;
  value: V;
}

const pair1: Pair<string, number> = { key: "age", value: 25 };
const pair2: Pair<number, string> = { key: 1, value: "one" };

// 泛型接口与函数
interface SearchFunc {
  <T>(array: T[], predicate: (item: T) => boolean): boolean;
}

const search: SearchFunc = (array, predicate) => {
  return array.some(predicate);
};

// 带默认类型参数的泛型接口
interface Container<T = string> {
  value: T;
}

const c1: Container = { value: "default" };     // 默认string
const c2: Container<number> = { value: 42 };    // number
```

### 5.4.2 泛型类

```typescript
// 泛型类
class Stack<T> {
  private items: T[] = [];
  
  push(item: T): void {
    this.items.push(item);
  }
  
  pop(): T | undefined {
    return this.items.pop();
  }
  
  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }
  
  isEmpty(): boolean {
    return this.items.length === 0;
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
// numberStack.push("3");  // Error: Argument of type 'string' is not assignable

const stringStack = new Stack<string>();
stringStack.push("hello");
stringStack.push("world");
```

### 5.4.3 泛型枚举和命名空间

```typescript
// 泛型枚举（不常用）
// 枚举本身不能是泛型的，但可以包含泛型方法

// 泛型命名空间
namespace Utils {
  export function createPair<K, V>(k: K, v: V): Pair<K, V> {
    return { key: k, value: v };
  }
}

const pair = Utils.createPair("name", "张三");
```

### 5.4.4 泛型默认值

```typescript
// TypeScript 2.3+ 支持泛型默认值
interface Response<T = any> {
  code: number;
  data: T;
  message: string;
}

// 不指定时使用默认类型
const r1: Response = { code: 200, data: "ok", message: "" };
const r2: Response<number[]> = { code: 200, data: [1, 2, 3], message: "" };

// 默认类型与约束
interface ApiResponse<T = string, E = Error> {
  success: boolean;
  data: T;
  error?: E;
}

// 使用
type StringApi = ApiResponse;                    // T=string, E=Error
type NumberApi = ApiResponse<number>;           // T=number, E=Error
type CustomApi = ApiResponse<number, string>;   // T=number, E=string
```

---

## 5.5 泛型工具类型

### 5.5.1 内置泛型工具类型

```typescript
// Partial<T> - 将所有属性变为可选
interface User {
  id: number;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; }

// Required<T> - 将所有属性变为必需
type RequiredUser = Required<User>;

// Readonly<T> - 将所有属性变为只读
type ReadonlyUser = Readonly<User>;

// Pick<T, K> - 从T中选择属性K
type UserPreview = Pick<User, "id" | "name">;
// { id: number; name: string; }

// Omit<T, K> - 从T中排除属性K
type UserWithoutEmail = Omit<User, "email">;
// { id: number; name: string; }
```

### 5.5.2 Record<K, V>

```typescript
// Record创建键值对类型
type Role = "admin" | "user" | "guest";

type Permission = Record<Role, string[]>;
// {
//   admin: string[];
//   user: string[];
//   guest: string[];
// }

const permissions: Permission = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"]
};

// 复杂示例
type EntityId = string | number;
type EntityType = "user" | "post" | "comment";

type EntityMap = Record<EntityType, Record<EntityId, { name: string }>>;

const entities: EntityMap = {
  user: {
    "1": { name: "张三" },
    "2": { name: "李四" }
  },
  post: {
    "101": { name: "文章1" }
  },
  comment: {}
};
```

### 5.5.3 Exclude和Extract

```typescript
// Exclude<T, U> - 从T中排除可以赋值给U的类型
type T0 = Exclude<"a" | "b" | "c", "a">;         // "b" | "c"
type T1 = Exclude<string | number | boolean, string>;  // number | boolean
type T2 = Exclude<undefined | null, null>;        // undefined

// Extract<T, U> - 从T中提取可以赋值给U的类型
type T3 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T4 = Extract<string | number, string>;       // string
type T5 = Extract<(() => void) | Function, Function>;  // Function
```

### 5.5.4 NonNullable和Parameters

```typescript
// NonNullable<T> - 排除null和undefined
type T0 = NonNullable<string | null | undefined>;  // string
type T1 = NonNullable<number[] | null | undefined>;  // number[]

// Parameters<T> - 提取函数参数类型为元组
type T2 = Parameters<(x: number, y: string) => void>;  // [x: number, y: string]
type T3 = Parameters<typeof console.log>;  // any[] (console.log是泛函数)

// ConstructorParameters<T> - 提取构造函数参数
class Person {
  constructor(name: string, age: number) {}
}

type T4 = ConstructorParameters<typeof Person>;  // [name: string, age: number]
```

### 5.5.5 ReturnType和InstanceType

```typescript
// ReturnType<T> - 获取函数返回类型
type T0 = ReturnType<() => string>;           // string
type T1 = ReturnType<() => Promise<number>>;  // Promise<number>
type T2 = ReturnType<typeof Math.random>;     // number

// InstanceType<T> - 获取构造函数实例类型
class Person {
  name: string;
}

type T3 = InstanceType<typeof Person>;  // Person

// 实际应用：创建工厂函数的返回类型
function createInstance<T extends new (...args: any[]) => any>(
  Constructor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new Constructor(...args);
}

const person = createInstance(Person, "张三");
```

### 5.5.6 自定义泛型工具类型

```typescript
// DeepPartial - 深度Partial
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

interface Company {
  name: string;
  address: {
    city: string;
    street: string;
  };
}

type PartialCompany = DeepPartial<Company>;
// {
//   name?: string;
//   address?: {
//     city?: string;
//     street?: string;
//   };
// }

// DeepReadonly - 深度只读
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

// DeepRequired - 深度Required
type DeepRequired<T> = {
  [K in keyof T]-?: T[K] extends object ? DeepRequired<T[K]> : T[K];
};
```

---

## 5.6 多泛型参数

### 5.6.1 多参数泛型

```typescript
// 多个类型参数
function pair<K, V>(key: K, value: V): [K, V] {
  return [key, value];
}

const p1 = pair("name", "张三");    // [string, string]
const p2 = pair("age", 25);        // [string, number]

// 使用不同类型
function mixArray<T, U>(arr1: T[], arr2: U[]): (T | U)[] {
  return [...arr1, ...arr2];
}

mixArray([1, 2], ["a", "b"]);  // (number | string)[]
```

### 5.6.2 泛型元组

```typescript
// 泛型元组
function swap<T, U>(pair: [T, U]): [U, T] {
  return [pair[1], pair[0]];
}

const swapped = swap(["hello", 42]);  // [42, "hello"]

// 链式调用
class Result<T, E = Error> {
  constructor(
    public value?: T,
    public error?: E
  ) {}
  
  isOk(): this is { value: T } {
    return this.error === undefined;
  }
  
  isErr(): this is { error: E } {
    return this.error !== undefined;
  }
}

function divide(a: number, b: number): Result<number, string> {
  if (b === 0) {
    return new Result(undefined, "Division by zero");
  }
  return new Result(a / b);
}

const result = divide(10, 2);
if (result.isOk()) {
  console.log(result.value);  // 5
}
```

### 5.6.3 泛型与默认类型

```typescript
// 泛型参数可以有默认值
interface Response<T = any, E = any> {
  success: boolean;
  data?: T;
  error?: E;
}

// 使用
const r1: Response = { success: true, data: "ok" };
const r2: Response<number> = { success: true, data: 42 };
const r3: Response<number, string> = { success: false, error: "Not found" };

// 函数默认值
function merge<T extends object = {}, U extends object = {}>(
  target: T,
  source: U
): T & U {
  return { ...target, ...source };
}

merge({ a: 1 });           // { a: number }
merge({ a: 1 }, { b: 2 });  // { a: number; b: number }
```

---

## 5.7 泛型条件与推断

### 5.7.1 泛型条件类型

```typescript
// 泛型条件类型
type IsString<T> = T extends string ? true : false;

type A = IsString<"hello">;  // true
type B = IsString<123>;       // false

// 延迟分发
type ToArray<T> = T extends any ? T[] : never;

type C = ToArray<string | number>;  // string[] | number[]
type D = ToArray<never>;            // never

// 非分布式版本
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type E = ToArrayNonDist<string | number>;  // (string | number)[]
```

### 5.7.2 infer关键字

```typescript
// 使用infer推断类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

type R1 = ReturnType<() => string>;           // string
type R2 = ReturnType<() => Promise<number>>; // Promise<number>

// 推断数组元素类型
type ElementType<T> = T extends (infer E)[] ? E : T;

type E1 = ElementType<string[]>;  // string
type E2 = ElementType<number[]>;  // number

// 推断函数参数
type FirstArg<T> = T extends (first: infer F, ...rest: any[]) => any ? F : never;

type F1 = FirstArg<(x: string, y: number) => void>;  // string
```

### 5.7.3 递归泛型

```typescript
// 递归类型：Flatten
type Flatten<T> = T extends Array<infer U> ? Flatten<U> : T;

type F1 = Flatten<number[]>;           // number
type F2 = Flatten<string[][]>;         // string
type F3 = Flatten<number[][][]>;       // number

// 深层次属性类型
type DeepReadonly<T> = T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
  : T;

type ReadonlyPerson = DeepReadonly<{
  name: string;
  address: { city: string };
}>;
// {
//   readonly name: string;
//   readonly address: {
//     readonly city: string;
//   };
// }
```

### 5.7.4 分布式技巧

```typescript
// 过滤联合类型
type ExcludeString<T> = T extends string ? never : T;

type E1 = ExcludeString<string | number | boolean>;  // number | boolean

// 提取函数类型
type ExtractFunction<T> = T extends (...args: any[]) => any ? T : never;

type F1 = ExtractFunction<string | (() => void) | number>;  // () => void

// 条件类型的巧妙应用
type IsNever<T> = [T] extends [never] ? true : false;

type N1 = IsNever<never>;   // true
type N2 = IsNever<string>;  // false

// 注意：never是分布式条件类型的特殊情况
// 不能直接使用 T extends never ? true : false 来判断never
```

---

## 5.8 实际应用场景

### 5.8.1 API通用封装

```typescript
// 泛型API封装
async function fetchData<T>(
  url: string,
  options?: RequestInit
): Promise<ApiResponse<T>> {
  const response = await fetch(url, options);
  return response.json();
}

interface ApiResponse<T = any> {
  code: number;
  message: string;
  data: T;
}

interface User {
  id: number;
  name: string;
}

// 使用
const user = await fetchData<User>("/api/users/1");
console.log(user.data.name);  // 类型安全
```

### 5.8.2 通用工具函数

```typescript
// pipe函数
function pipe<A, B>(fn1: (a: A) => B): (a: A) => B;
function pipe<A, B, C>(fn1: (a: A) => B, fn2: (b: B) => C): (a: A) => C;
function pipe<A, B, C, D>(
  fn1: (a: A) => B,
  fn2: (b: B) => C,
  fn3: (c: C) => D
): (a: A) => D;
function pipe(...fns: Function[]): Function {
  return (x: any) => fns.reduce((v, f) => f(v), x);
}

const process = pipe(
  (s: string) => s.trim(),
  (s: string) => s.toLowerCase(),
  (s: string) => s.replace(/\s+/g, "-")
);

process("Hello   World");  // "hello-world"
```

### 5.8.3 React/Vue组件泛型

```typescript
// React Props with generics
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map(item => (
        <li key={keyExtractor(item)}>
          {renderItem(item)}
        </li>
      ))}
    </ul>
  );
}

// 使用
<List
  items={users}
  renderItem={user => <span>{user.name}</span>}
  keyExtractor={user => user.id}
/>
```

### 5.8.4 表单验证泛型

```typescript
// 泛型表单验证
interface ValidationRule<T> {
  required?: boolean;
  minLength?: number;
  maxLength?: number;
  pattern?: RegExp;
  custom?: (value: T) => boolean | string;
}

type Validator<T> = {
  [K in keyof T]?: ValidationRule<T[K]>;
};

interface FormState<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
}

function validate<T>(values: T, rules: Validator<T>): Partial<Record<keyof T, string>> {
  const errors: any = {};
  
  for (const key in rules) {
    const value = values[key];
    const rule = rules[key];
    
    if (rule?.required && !value) {
      errors[key] = "此字段为必填项";
    }
    // ... 更多验证
  }
  
  return errors;
}

// 使用
interface LoginForm {
  username: string;
  password: string;
}

const rules: Validator<LoginForm> = {
  username: { required: true, minLength: 3 },
  password: { required: true, minLength: 6 }
};

const errors = validate({ username: "ab", password: "123" }, rules);
// { username: "用户名至少3个字符", password: undefined }
```

---

## 📊 泛型速查表

```
┌─────────────────────────────────────┬────────────────────────────────┐
│ 语法                                │ 说明                           │
├─────────────────────────────────────┼────────────────────────────────┤
│ function identity<T>(arg: T): T    │ 泛型函数                       │
│ interface Pair<K, V>                │ 泛型接口                       │
│ class Container<T>                  │ 泛型类                         │
│ T extends U                         │ 泛型约束                       │
│ keyof T                             │ 获取类型的所有键               │
│ T[K]                                │ 索引访问类型                   │
│ infer R                             │ 类型推断                       │
│ T extends U ? X : Y                 │ 条件类型                       │
│ Partial<T>                          │ 所有属性可选                   │
│ Required<T>                         │ 所有属性必需                   │
│ Readonly<T>                         │ 所有属性只读                   │
│ Pick<T, K>                         │ 选择属性                       │
│ Omit<T, K>                         │ 排除属性                       │
│ Record<K, V>                        │ 键值对类型                     │
│ Exclude<T, U>                       │ 排除类型                       │
│ Extract<T, U>                       │ 提取类型                       │
│ NonNullable<T>                      │ 排除null/undefined             │
│ ReturnType<T>                       │ 函数返回类型                   │
│ Parameters<T>                       │ 函数参数类型                   │
│ InstanceType<T>                     │ 构造函数实例类型               │
└─────────────────────────────────────┴────────────────────────────────┘
```

---

## 💪 实战练习

### 练习1：实现通用pipe函数
```typescript
// 实现一个pipe函数，支持任意数量的函数链接
// pipe(f1, f2, f3)(initialValue) 等同于 f3(f2(f1(initialValue)))

// 你的代码：
function pipe(...fns) {
  // 实现这里
}
```

### 练习2：实现DeepMerge类型
```typescript
// 实现DeepMerge<T, U>，深度合并两个对象类型

interface A {
  x: { y: string; z: number };
}

interface B {
  x: { w: boolean };
  m: string;
}

// 期望结果：
// {
//   x: { y: string; z: number; w: boolean };
//   m: string;
// }

// 你的代码：
type DeepMerge<T, U> = /* 实现这里 */
```

### 练习3：实现Curry函数
```typescript
// 实现函数柯里化，将 (a, b, c) => d 转换为 a => b => c => d

// 期望：
// const curried = curry((a: number, b: number, c: number) => a + b + c);
// curried(1)(2)(3) // 6
// curried(1, 2)(3) // 6
// curried(1)(2, 3) // 6

// 你的代码：
function curry(fn: Function): any {
  // 实现这里
}
```

---

## ⏭️ 下一步

前往 [第六章：装饰器与元编程](./chapter-06-decorators.md) 学习装饰器的高级用法

---

> 💡 **思考题**：
> 1. 泛型和any有什么区别？为什么泛型更安全？
> 2. 什么是分布式条件类型？它有什么特性？
> 3. 如何实现一个DeepPartial工具类型？