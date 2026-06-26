# 第四章：接口与类型别名

## 📋 目录

- [4.1 接口基础](#41-接口基础)
- [4.2 接口特性详解](#42-接口特性详解)
- [4.3 类型别名](#43-类型别名)
- [4.4 接口vs类型别名](#44-接口vs类型别名)
- [4.5 索引签名](#45-索引签名)
- [4.6 接口继承](#46-接口继承)
- [4.7 混合类型](#47-混合类型)
- [4.8 实际项目应用](#48-实际项目应用)

---

## 4.1 接口基础

### 4.1.1 定义接口

```typescript
// 使用interface关键字定义接口
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;           // 可选属性
  readonly createdAt: Date;  // 只读属性
}

// 使用接口
const user: User = {
  id: 1,
  name: "张三",
  email: "zhangsan@example.com",
  createdAt: new Date()
};

// 只读属性不能修改
// user.createdAt = new Date();  // Error: Cannot assign to 'createdAt' because it is a read-only property
```

### 4.1.2 接口语法图解

```
┌─────────────────────────────────────────────────────────────────────┐
│                         接口定义结构                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   interface 接口名称 {                                               │
│       属性名: 类型;           ← 普通属性                              │
│       属性名?: 类型;          ← 可选属性（?）                         │
│       readonly 属性名: 类型;  ← 只读属性（readonly）                 │
│       方法名(): 返回类型;     ← 方法签名                              │
│       (参数: 类型): 返回类型; ← 调用签名                              │
│   }                                                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.1.3 接口属性类型

```typescript
// 1. 普通属性
interface Point {
  x: number;
  y: number;
}

// 2. 可选属性
interface Config {
  host?: string;
  port?: number;
  timeout?: number;
}

// 3. 只读属性
interface Response {
  readonly status: number;
  readonly message: string;
}

// 4. 任意属性
interface Dictionary {
  [key: string]: any;
}

// 5. 方法
interface Logger {
  log(message: string): void;
  error(message: string, code?: number): void;
}
```

---

## 4.2 接口特性详解

### 4.2.1 可选属性

```typescript
// 可选属性：属性名后加?
interface User {
  id: number;
  name: string;
  email?: string;   // 可选
  phone?: string;   // 可选
}

// 使用
const user1: User = {
  id: 1,
  name: "张三"
};

const user2: User = {
  id: 2,
  name: "李四",
  email: "lisi@example.com"
};

// 访问可选属性
console.log(user1.email?.toLowerCase());  // undefined（安全调用）
```

### 4.2.2 只读属性

```typescript
// readonly修饰符：创建后不能修改
interface Product {
  readonly id: string;
  name: string;
  price: number;
}

const product: Product = {
  id: "P001",
  name: "iPhone",
  price: 999
};

// product.id = "P002";  // Error: Cannot assign to 'id' because it is a read-only property

// 注意：如果是对象内部的可变属性，仍然可以修改
interface Config {
  readonly settings: { url: string };
}

const config: Config = {
  settings: { url: "http://example.com" }
};

config.settings.url = "http://new.example.com";  // OK：内部属性可变
// config.settings = { url: "..." };  // Error：整个对象不可赋值
```

### 4.2.3 方法签名

```typescript
// 接口中定义方法
interface Calculatable {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
}

class Calculator implements Calculatable {
  add(a: number, b: number): number {
    return a + b;
  }
  
  subtract(a: number, b: number): number {
    return a - b;
  }
}

// 可选方法
interface Greeter {
  greet(): void;
  farewell?: () => void;  // 可选方法
}

// 方法的this类型
interface TypedGreeter {
  name: string;
  greet(this: TypedGreeter): string;
}

const greeter: TypedGreeter = {
  name: "张三",
  greet() {
    return `Hello, I'm ${this.name}`;
  }
};
```

### 4.2.4 函数类型接口

```typescript
// 方式一：调用签名
interface AddFn {
  (a: number, b: number): number;
}

const add: AddFn = (x, y) => x + y;

// 方式二：类型别名
type AddFunction = (a: number, b: number) => number;

// 方式三：泛型函数接口
interface Mapper<T, U> {
  (item: T): U;
}

const toString: Mapper<number, string> = (item) => String(item);
const toDouble: Mapper<number, number> = (item) => item * 2;
```

### 4.2.5 构造函数签名

```typescript
// 使用new关键字定义构造函数
interface PersonConstructor {
  new (name: string): Person;
}

interface Person {
  name: string;
  greet(): string;
}

// 使用构造函数签名创建实例
function createPerson(ctor: PersonConstructor, name: string): Person {
  return new ctor(name);
}

class Student implements Person {
  constructor(public name: string) {}
  
  greet(): string {
    return `Hi, I'm ${this.name}`;
  }
}

const student = createPerson(Student, "张三");
console.log(student.greet());  // "Hi, I'm 张三"
```

---

## 4.3 类型别名

### 4.3.1 基本用法

```typescript
// 使用type关键字定义类型别名
type Point = {
  x: number;
  y: number;
};

// 使用类型别名
const point: Point = { x: 1, y: 2 };

// 别名也可以用于基本类型
type ID = string | number;
type UserId = ID;

let id: UserId = "user123";
id = 456;  // 也合法
```

### 4.3.2 类型别名的优势

```typescript
// 1. 给复杂类型起一个简洁的名字
type Callback = (error: Error | null, result: string | null) => void;

// 2. 联合类型
type Result = SuccessResult | ErrorResult;

// 3. 元组
type Coordinates = [number, number, number];

// 4. 映射类型
type Stringify<T> = {
  [K in keyof T]: string;
};

// 5. 条件类型
type IsArray<T> = T extends any[] ? true : false;
```

### 4.3.3 类型别名与泛型

```typescript
// 带泛型的类型别名
type Pair<T, U> = {
  first: T;
  second: U;
};

type StringNumberPair = Pair<string, number>;
// { first: string; second: number }

// 泛型约束
type ExtractArrayType<T extends any[]> = T[number];

type Num = ExtractArrayType<number[]>;  // number
type Str = ExtractArrayType<string[]>;  // string
```

### 4.3.4 类型别名的自引用

```typescript
// 类型别名可以自引用
type TreeNode<T> = {
  value: T;
  left?: TreeNode<T>;
  right?: TreeNode<T>;
};

type LinkedList<T> = {
  value: T;
  next?: LinkedList<T>;
};

// 使用
const tree: TreeNode<number> = {
  value: 1,
  left: {
    value: 2,
    left: { value: 4 },
    right: { value: 5 }
  },
  right: {
    value: 3
  }
};

const list: LinkedList<string> = {
  value: "a",
  next: {
    value: "b",
    next: {
      value: "c"
    }
  }
};
```

---

## 4.4 接口vs类型别名

### 4.4.1 核心区别

```
┌─────────────────────────────────────────────────────────────────────┐
│                        接口 vs 类型别名                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────────────────┐    ┌─────────────────────────┐         │
│   │        接口            │    │       类型别名          │         │
│   ├─────────────────────────┤    ├─────────────────────────┤         │
│   │ • 可以被类实现          │    │ • 不能被类实现          │         │
│   │ • 可以被继承/扩展       │    │ • 可以使用交叉类型      │         │
│   │ • 有声明合并特性        │    │ • 支持联合/交叉类型    │         │
│   │ • 只能定义对象结构      │    │ • 可定义任何类型        │         │
│   │ • 更适合公开API        │    │ • 更灵活                │         │
│   └─────────────────────────┘    └─────────────────────────┘         │
│                                                                     │
│   何时使用接口：定义类要实现的契约公开API结构                          │
│   何时使用类型别名：联合类型、映射类型、元组等复杂类型                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.4.2 代码对比

```typescript
// 接口定义
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// 类型别名定义
type Animal2 = {
  name: string;
};

type Dog2 = Animal2 & {
  breed: string;
};

// 接口可以被类实现
class Pet implements Animal {
  name: string;
}

// 类型别名不能被类实现（但可以用交叉类型模拟）
```

### 4.4.3 声明合并

```typescript
// 接口支持声明合并
interface User {
  name: string;
}

interface User {
  age: number;
}

// 合并后等价于：
// interface User {
//   name: string;
//   age: number;
// }

// 类型别名不支持声明合并
type User2 = {
  name: string;
};

// type User2 = { age: number };  // Error: Duplicate identifier 'User2'
```

### 4.4.4 实际选择建议

```typescript
// 推荐使用接口的场景
// 1. 定义类的结构
interface Serializable {
  serialize(): string;
}

class Session implements Serializable {
  serialize() {
    return JSON.stringify(this);
  }
}

// 2. 定义公开API的结构
interface ApiResponse {
  code: number;
  message: string;
  data: any;
}

// 3. 需要声明合并时
interface Window {
  analytics: Analytics;
}

// 推荐使用类型别名的场景
// 1. 联合类型
type Result = Success | Error | Loading;

// 2. 元组
type Point = [number, number];

// 3. 映射类型
type Readonly<T> = { readonly [K in keyof T]: T[K] };

// 4. 条件类型
type Flatten<T> = T extends any[] ? T[number] : T;

// 5. 函数类型
type Callback = (data: string) => void;
```

---

## 4.5 索引签名

### 4.5.1 字符串索引签名

```typescript
// [key: string]: 类型 - 表示可以有任意多个字符串键
interface StringDictionary {
  [key: string]: string;
}

const dict: StringDictionary = {
  hello: "world",
  foo: "bar",
  // 可以添加任意字符串键
  arbitrary: "value"
};

// 访问
console.log(dict["hello"]);  // "world"
```

### 4.5.2 数字索引签名

```typescript
// [key: number]: 类型 - 用于数组/类数组
interface NumberArray {
  [index: number]: string;
}

const arr: NumberArray = ["a", "b", "c"];
console.log(arr[0]);  // "a"

// 数组默认实现了number索引签名
const nativeArr: string[] = ["x", "y", "z"];
// 等价于
const nativeArr2: {
  [index: number]: string;
  length: number;
  pop(): string | undefined;
  push(...items: string[]): number;
  // ...
} = ["x", "y", "z"];
```

### 4.5.3 混合索引签名

```typescript
// 同时支持字符串和数字索引
interface Mixed {
  [key: string]: string | number;
  [key: number]: string;  // 必须是string的子类型
}

const mixed: Mixed = {
  name: "张三",
  0: "zero",        // 数字键
  1: "one",         // 数字键
  age: 25           // 字符串键
};
```

### 4.5.4 索引签名与属性类型

```typescript
// 索引签名与其他属性共存
interface Config {
  // 必须属性
  apiUrl: string;
  apiKey: string;
  
  // 任意额外属性
  [key: string]: string | number | boolean;
}

const config: Config = {
  apiUrl: "https://api.example.com",
  apiKey: "secret123",
  timeout: 5000,        // 额外属性
  retries: 3,           // 额外属性
  debug: true           // 额外属性
};
```

### 4.5.5 实际应用

```typescript
// 1. 动态用户属性
interface UserData {
  id: string;
  name: string;
  [key: string]: any;  // 允许动态属性
}

const user: UserData = {
  id: "u001",
  name: "张三",
  customField: "自定义值",
  metadata: { created: "2024-01-01" }
};

// 2. 数据库记录
interface DbRecord {
  _id: string;
  _rev: string;
  [field: string]: any;  // 其他字段任意
}

// 3. CSS样式对象
interface CSSProperties {
  [property: string]: string | number;
}

const styles: CSSProperties = {
  color: "red",
  fontSize: 16,
  marginTop: "10px",
  padding: 20
};
```

---

## 4.6 接口继承

### 4.6.1 单接口继承

```typescript
// 使用extends关键字继承
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

const dog: Dog = {
  name: "Buddy",
  breed: "Golden Retriever"
};
```

### 4.6.2 多接口继承

```typescript
// 可以继承多个接口
interface Serializable {
  serialize(): string;
}

interface Loggable {
  log(): void;
}

interface Persistable extends Serializable, Loggable {
  save(): void;
}

class DataManager implements Persistable {
  serialize(): string {
    return JSON.stringify(this);
  }
  
  log(): void {
    console.log("Logging...");
  }
  
  save(): void {
    console.log("Saving...");
  }
}
```

### 4.6.3 继承中的类型覆盖

```typescript
interface Base {
  id: string | number;
}

interface Derived extends Base {
  id: string;  // 收窄类型
}

interface Base2 {
  config: {
    timeout: number;
  };
}

interface Derived2 extends Base2 {
  config: {
    timeout: number;
    retries: number;
  };
}
```

### 4.6.4 接口继承与类型别名的交叉类型

```typescript
// 类型别名使用交叉类型实现继承
type Animal = {
  name: string;
};

type Dog = Animal & {
  breed: string;
};

// 同样可以多类型交叉
type Walkable = { canWalk: true };
type Swimmable = { canSwim: true };
type Amphibious = Walkable & Swimmable;

// interface也可以用交叉类型模拟
interface AnimalInterface {
  name: string;
}

type DogType = AnimalInterface & {
  breed: string;
};
```

---

## 4.7 混合类型

### 4.7.1 函数对象类型

```typescript
// 混合类型：同时具有属性和可调用特性
interface Counter {
  (): number;           // 可调用
  count: number;         // 有属性
  reset(): void;         // 有方法
}

function createCounter(): Counter {
  const counter = (() => counter.count) as Counter;
  counter.count = 0;
  counter.reset = () => { counter.count = 0; };
  return counter;
}

const counter = createCounter();
console.log(counter());      // 0
counter.count = 10;
console.log(counter());      // 10
counter.reset();
console.log(counter());     // 0
```

### 4.7.2 jQuery风格的混合类型

```typescript
// 类似jQuery的API
interface JQuery {
  (selector: string): JQueryElement;
  html(content: string): JQueryElement;
  addClass(className: string): JQueryElement;
  removeClass(className: string): JQueryElement;
  length: number;
}

// 实现
function $(): JQuery {
  const elements: Element[] = [];
  
  const jq = function(selector: string) {
    elements.push(...document.querySelectorAll(selector));
    return jq;
  } as JQuery;
  
  jq.html = (content: string) => {
    elements.forEach(el => el.innerHTML = content);
    return jq;
  };
  
  // ... 其他方法
  
  jq.length = elements.length;
  
  return jq;
}

// 使用
$("div").html("Hello").addClass("container");
```

### 4.7.3 Promise的混合类型

```typescript
// Promise构造函数是经典的混合类型
interface PromiseConstructor {
  new <T>(
    executor: (
      resolve: (value: T) => void,
      reject: (reason?: any) => void
    ) => void
  ): Promise<T>;
}

// 函数式混合
interface FunctionWithName {
  (): any;
  name: string;
}

function namedFunction(fn: () => any, name: string): FunctionWithName {
  const named = fn as FunctionWithName;
  named.name = name;
  return named;
}
```

---

## 4.8 实际项目应用

### 4.8.1 API接口定义

```typescript
// 请求类型
interface ApiRequest<T = any> {
  url: string;
  method: "GET" | "POST" | "PUT" | "DELETE" | "PATCH";
  params?: T;
  headers?: Record<string, string>;
}

// 响应类型
interface ApiResponse<T = any> {
  code: number;
  message: string;
  data: T;
  timestamp: number;
}

// 分页请求
interface PaginatedRequest {
  page: number;
  pageSize: number;
  sortBy?: string;
  sortOrder?: "asc" | "desc";
}

// 分页响应
interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
}

// 使用示例
interface User {
  id: number;
  name: string;
  email: string;
}

type GetUsersRequest = PaginatedRequest;
type GetUsersResponse = PaginatedResponse<User>;

const request: ApiRequest<GetUsersRequest> = {
  url: "/api/users",
  method: "GET",
  params: { page: 1, pageSize: 20 }
};
```

### 4.8.2 组件Props定义

```typescript
// React/Vue组件Props类型
interface ButtonProps {
  variant: "primary" | "secondary" | "danger";
  size?: "small" | "medium" | "large";
  disabled?: boolean;
  onClick?: (event: MouseEvent) => void;
  children: React.ReactNode;
  icon?: React.ReactElement;
  fullWidth?: boolean;
}

// 表单组件
interface FormFieldProps {
  name: string;
  label?: string;
  type?: "text" | "email" | "password" | "number";
  value: string | number;
  onChange: (value: string) => void;
  error?: string;
  required?: boolean;
  placeholder?: string;
  disabled?: boolean;
}
```

### 4.8.3 状态管理类型

```typescript
// Redux风格Action
interface Action<T = any> {
  type: string;
  payload?: T;
}

interface AsyncAction<T = any> extends Action<T> {
  meta?: {
    requestId: string;
    timestamp: number;
  };
}

// 用户状态
interface UserState {
  id: string | null;
  name: string;
  email: string;
  avatar?: string;
  status: "idle" | "loading" | "success" | "error";
  error?: string;
}

// Action类型
type UserAction =
  | { type: "SET_USER"; payload: User }
  | { type: "UPDATE_USER"; payload: Partial<User> }
  | { type: "SET_LOADING"; payload: boolean }
  | { type: "SET_ERROR"; payload: string }
  | { type: "LOGOUT" };

// Reducer
function userReducer(state: UserState, action: UserAction): UserState {
  switch (action.type) {
    case "SET_USER":
      return { ...state, ...action.payload, status: "success" };
    case "UPDATE_USER":
      return { ...state, ...action.payload };
    case "SET_LOADING":
      return { ...state, status: "loading" };
    case "SET_ERROR":
      return { ...state, status: "error", error: action.payload };
    case "LOGOUT":
      return { id: null, name: "", email: "", status: "idle" };
    default:
      return state;
  }
}
```

### 4.8.4 数据库模型类型

```typescript
// ORM风格模型
interface BaseModel {
  id: string;
  createdAt: Date;
  updatedAt: Date;
}

interface UserModel extends BaseModel {
  email: string;
  username: string;
  passwordHash: string;
  profile?: UserProfile;
  roles: string[];
  isActive: boolean;
}

interface UserProfile {
  firstName: string;
  lastName: string;
  phone?: string;
  avatar?: string;
  bio?: string;
}

interface PostModel extends BaseModel {
  authorId: string;
  title: string;
  content: string;
  tags: string[];
  status: "draft" | "published" | "archived";
  viewCount: number;
  likeCount: number;
  publishedAt?: Date;
}

// Repository接口
interface Repository<T extends BaseModel> {
  findById(id: string): Promise<T | null>;
  findAll(filter?: Partial<T>): Promise<T[]>;
  create(data: Omit<T, "id" | "createdAt" | "updatedAt">): Promise<T>;
  update(id: string, data: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

interface UserRepository extends Repository<UserModel> {
  findByEmail(email: string): Promise<UserModel | null>;
  findByUsername(username: string): Promise<UserModel | null>;
}
```

---

## 📊 接口vs类型别名速查表

```
┌───────────────────┬────────────────────────┬────────────────────────┐
│ 特性              │ 接口                    │ 类型别名                │
├───────────────────┼────────────────────────┼────────────────────────┤
│ 定义对象结构      │ ✅                      │ ✅                      │
│ 定义基本类型      │ ❌                      │ ✅                      │
│ 定义联合类型      │ ❌                      │ ✅                      │
│ 定义元组          │ ❌                      │ ✅                      │
│ 可被类实现        │ ✅                      │ ❌                      │
│ 支持继承(extends) │ ✅                      │ ✅ (通过交叉类型&)      │
│ 支持声明合并      │ ✅                      │ ❌                      │
│ 支持泛型          │ ✅                      │ ✅                      │
│ 可调用            │ ✅ (调用签名)           │ ✅ (函数类型)           │
│ 可构造            │ ✅ (new签名)            │ ⚠️ (有限支持)           │
└───────────────────┴────────────────────────┴────────────────────────┘
```

---

## 💪 实战练习

### 练习1：定义用户系统接口
```typescript
// 定义一个用户系统的完整接口
// 包含：
// 1. User - 基本用户信息
// 2. Admin - 管理员（继承User，增加权限列表）
// 3. UserRepository - 用户仓库接口（CRUD方法）
// 4. UserService - 用户服务接口（登录、注册、修改密码）

// 你的代码：
```

### 练习2：实现混合类型
```typescript
// 创建一个工厂函数，返回一个同时具有：
// 1. 可作为函数调用，返回累计调用次数
// 2. 有reset()方法重置计数
// 3. 有count属性获取当前计数
// 4. 有version属性

// 你的代码：
```

---

## ⏭️ 下一步

前往 [第五章：函数与泛型](./chapter-05-functions-generics.md) 学习函数类型和泛型编程

---

> 💡 **思考题**：
> 1. 什么时候应该用接口而不是类型别名？
> 2. 什么是声明合并？它有什么实际应用？
> 3. 索引签名有什么用途？使用时应注意什么？