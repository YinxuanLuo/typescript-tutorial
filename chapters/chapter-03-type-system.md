# 第三章：类型系统深入

## 📋 目录

- [3.1 联合类型与交叉类型](#31-联合类型与交叉类型)
- [3.2 类型字面量](#32-类型字面量)
- [3.3 类型守卫与类型保护](#33-类型守卫与类型保护)
- [3.4 可辨识联合类型](#34-可辨识联合类型)
- [3.5 映射类型](#35-映射类型)
- [3.6 条件类型](#36-条件类型)
- [3.7 模板字面量类型](#37-模板字面量类型)
- [3.8 类型推断infer](#38-类型推断infer)

---

## 3.1 联合类型与交叉类型

### 3.1.1 联合类型 (Union Types)

```typescript
// 联合类型：多种类型之一
let value: string | number;
value = "hello";   // OK
value = 123;       // OK
// value = true;    // Error

// 函数参数使用联合类型
function printId(id: number | string) {
  console.log(`ID: ${id}`);
}

printId(123);        // OK
printId("abc123");   // OK

// 访问联合类型的属性（只能访问共有属性）
function getLength(input: string | number): number {
  // 只能访问两者共有的属性和方法
  // input.trim();        // Error: number没有trim方法
  return String(input).length;  // 转换后访问
}
```

### 3.1.2 交叉类型 (Intersection Types)

```typescript
// 交叉类型：多种类型的组合（AND关系）
interface Person {
  name: string;
  age: number;
}

interface Employee {
  employeeId: string;
  department: string;
}

// 交叉类型：既是Person又是Employee
type EmployeePerson = Person & Employee;

const emp: EmployeePerson = {
  name: "张三",
  age: 30,
  employeeId: "E001",
  department: "技术部"
};

// 交叉类型用于mixin模式
function extend<T, U>(first: T, second: U): T & U {
  return { ...first, ...second };
}

const extended = extend({ name: "张三" }, { age: 30 });
// extended: { name: string } & { age: number }
```

### 3.1.3 联合类型 vs 交叉类型 对比

```
┌─────────────────────────────────────────────────────────────────────┐
│                     联合类型 vs 交叉类型                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   联合类型 (A | B)              交叉类型 (A & B)                      │
│   ┌─────────────────┐          ┌─────────────────┐                  │
│   │  A 或者 B        │          │  既是A又是B     │                  │
│   │                 │          │                 │                  │
│   │  "或"的关系      │          │  "与"的关系     │                  │
│   └─────────────────┘          └─────────────────┘                  │
│                                                                     │
│   示例：                      示例：                                 │
│   string | number             Person & Serializable                │
│   可以是字符串或数字           同时是人和可序列化                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

```typescript
// 实际应用对比
type A = { a: string };
type B = { b: number };

// 联合类型
type Union = A | B;
const union1: Union = { a: "hello" };     // OK
const union2: Union = { b: 123 };         // OK
// const union3: Union = { a: "x", b: 1 }; // Error: 只能二选一

// 交叉类型
type Intersection = A & B;
const inter1: Intersection = { a: "hello", b: 123 };  // OK
// const inter2: Intersection = { a: "x" };           // Error: 必须同时包含
```

### 3.1.4 高级联合类型应用

```typescript
// 1. 字面量联合类型
type Status = "pending" | "success" | "error";
type Direction = "north" | "south" | "east" | "west";

// 2. 数字字面量联合类型
type HttpStatus = 200 | 201 | 400 | 401 | 403 | 404 | 500;

// 3. 接口+联合类型
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  side: number;
}

type Shape = Circle | Square;

function getArea(shape: Shape): number {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
  } else {
    return shape.side ** 2;
  }
}

// 4. 多重联合类型
type StringOrNumberArray = Array<string | number>;
type KeyValuePair = [string, string | number | boolean];
```

---

## 3.2 类型字面量

### 3.2.1 基本概念

```typescript
// 类型字面量：表示一个精确的类型
type T1 = "hello";           // 只能是字符串"hello"
type T2 = 42;                // 只能是数字42
type T3 = true;              // 只能是布尔true

// 字面量类型必须与具体值绑定
let literal: "hello" = "hello";
// literal = "world";  // Error: 不能赋值为其他字符串
```

### 3.2.2 对象字面量类型

```typescript
// 对象字面量类型
type Point = {
  x: number;
  y: number;
};

// 必须是完全匹配的结构
const p: Point = { x: 1, y: 2 };  // OK
// const p2: Point = { x: 1 };     // Error: 缺少y

// 嵌套字面量
type Config = {
  database: {
    host: string;
    port: number;
  };
  cache: {
    enabled: boolean;
  };
};
```

### 3.2.3 函数字面量类型

```typescript
// 函数类型字面量
type Handler = (event: MouseEvent) => void;

// 函数表达式
const clickHandler: Handler = (event) => {
  console.log(event.clientX, event.clientY);
};
```

### 3.2.4 字面量类型的实际应用

```typescript
// 1. 配置对象的类型
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

interface ApiConfig {
  method: HttpMethod;
  url: string;
  headers?: Record<string, string>;
}

const config: ApiConfig = {
  method: "GET",    // 只能是这四种之一
  url: "/api/users"
};

// 2. 状态机
type State = "idle" | "loading" | "success" | "error";

interface Transition {
  from: State;
  to: State;
}

const transition: Transition = {
  from: "idle",
  to: "loading"
};

// 3. 带属性的常量对象
const CONSTANTS = {
  MAX_RETRIES: 3,
  TIMEOUT: 5000,
  API_BASE: "https://api.example.com"
} as const;

type ConstantsType = typeof CONSTANTS;
// {
//   readonly MAX_RETRIES: 3;
//   readonly TIMEOUT: 5000;
//   readonly API_BASE: "https://api.example.com";
// }
```

---

## 3.3 类型守卫与类型保护

### 3.3.1 typeof类型守卫

```typescript
// typeof用于原始类型检查
function printValue(value: string | number | boolean) {
  if (typeof value === "string") {
    // value在这里是string类型
    console.log(value.toUpperCase());
  } else if (typeof value === "number") {
    // value在这里是number类型
    console.log(value.toFixed(2));
  } else {
    // value在这里是boolean类型
    console.log(value ? "Yes" : "No");
  }
}

// typeof的局限性：只能用于原始类型
// 无法用于class和interface
```

### 3.3.2 instanceof类型守卫

```typescript
// instanceof用于类检查
class Dog {
  bark() {
    console.log("Woof!");
  }
}

class Cat {
  meow() {
    console.log("Meow!");
  }
}

function speak(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else if (animal instanceof Cat) {
    animal.meow();
  }
}

// 实例
const dog = new Dog();
speak(dog);  // 输出: Woof!
```

### 3.3.3 自定义类型守卫函数

```typescript
// 类型谓词（type predicate）
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isNumber(value: unknown): value is number {
  return typeof value === "number";
}

function processValue(value: unknown) {
  if (isString(value)) {
    console.log(value.toUpperCase());  // value是string
  } else if (isNumber(value)) {
    console.log(value * 2);  // value是number
  }
}

// 对象类型守卫
interface Person {
  name: string;
  age: number;
}

interface Company {
  name: string;
  employeeCount: number;
}

function isPerson(obj: Person | Company): obj is Person {
  return "age" in obj;
}

function getName(entity: Person | Company): string {
  if (isPerson(entity)) {
    return entity.name;  // Person
  } else {
    return entity.name;  // Company
  }
}
```

### 3.3.4 in操作符类型守卫

```typescript
// in操作符用于检查属性是否存在
interface Car {
  drive(): void;
}

interface Boat {
  sail(): void;
}

function operate(vehicle: Car | Boat) {
  if ("drive" in vehicle) {
    vehicle.drive();  // Car
  } else {
    vehicle.sail();   // Boat
  }
}
```

### 3.3.5 可null类型守卫

```typescript
// 可选链 + 空值合并
interface User {
  name: string;
  address?: {
    city: string;
    street: string;
  };
}

const user: User = { name: "张三" };

// 安全访问可选属性
const city = user.address?.city ?? "未知";
console.log(city);  // "未知"

// 非空断言（慎用）
// console.log(user.address!.city);  // Runtime Error!

// 类型守卫函数
function hasAddress(user: User): user is User & { address: NonNullable<User["address"]> } {
  return user.address !== undefined;
}

if (hasAddress(user)) {
  console.log(user.address.city);  // 安全
}
```

---

## 3.4 可辨识联合类型

### 3.4.1 基本概念

```typescript
// 可辨识联合（Tagged Union / Sum Type）
// 通过一个共同的"辨识"属性区分union中的每个成员

interface Circle {
  kind: "circle";    // 辨识属性（字面量类型）
  radius: number;
}

interface Square {
  kind: "square";    // 辨识属性（字面量类型）
  side: number;
}

interface Rectangle {
  kind: "rectangle"; // 辨识属性（字面量类型）
  width: number;
  height: number;
}

// 组合成联合类型
type Shape = Circle | Square | Rectangle;
```

### 3.4.2 完整示例

```typescript
// 定义形状接口
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  side: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

interface Triangle {
  kind: "triangle";
  base: number;
  height: number;
}

// 可辨识联合
type Shape = Circle | Square | Rectangle | Triangle;

// 计算面积函数
function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default:
      // 穷尽性检查
      const _exhaustive: never = shape;
      throw new Error("Unknown shape type");
  }
}

// 使用
console.log(calculateArea({ kind: "circle", radius: 5 }));    // 78.54
console.log(calculateArea({ kind: "square", side: 4 }));     // 16
console.log(calculateArea({ kind: "rectangle", width: 3, height: 6 }));  // 18
```

### 3.4.3 可辨识联合的优势

```
┌─────────────────────────────────────────────────────────────────────┐
│                     可辨识联合类型优势                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   1. 类型安全                                                       │
│      • 编译器确保处理所有情况                                        │
│      • 穷尽性检查                                                  │
│                                                                     │
│   2. 代码清晰                                                       │
│      • 辨识属性明确标识类型                                         │
│      • 易于理解和维护                                               │
│                                                                     │
│   3. 模式匹配                                                       │
│      • switch/case自然处理各类型                                    │
│      • 每个分支类型收窄                                             │
│                                                                     │
│   4. 可扩展性                                                       │
│      • 添加新类型时，编译器提示需处理                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.4.4 实际应用：状态机

```typescript
// 异步操作状态
interface LoadingState {
  status: "loading";
}

interface SuccessState<T> {
  status: "success";
  data: T;
}

interface ErrorState {
  status: "error";
  message: string;
}

type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

// 使用状态机
function renderAsyncState<T>(state: AsyncState<T>) {
  switch (state.status) {
    case "loading":
      return `<div>加载中...</div>`;
    case "success":
      return `<div>数据: ${JSON.stringify(state.data)}</div>`;
    case "error":
      return `<div>错误: ${state.message}</div>`;
  }
}

// 使用
type User = { name: string; age: number };

const loadingState: AsyncState<User> = { status: "loading" };
const successState: AsyncState<User> = { 
  status: "success", 
  data: { name: "张三", age: 25 } 
};
const errorState: AsyncState<User> = { 
  status: "error", 
  message: "网络错误" 
};

console.log(renderAsyncState(successState));
```

---

## 3.5 映射类型

### 3.5.1 基本映射类型

```typescript
// 映射类型：从已有类型创建新类型
type Person = {
  name: string;
  age: number;
  email: string;
};

// 所有属性变为可选
type PartialPerson = Partial<Person>;
// { name?: string; age?: number; email?: string; }

// 所有属性变为必需
type RequiredPerson = Required<Person>;

// 所有属性变为只读
type ReadonlyPerson = Readonly<Person>;
// { readonly name: string; readonly age: number; readonly email: string; }

// 所有属性变为可选且只读
type ReadonlyPartialPerson = Readonly<Partial<Person>>;
```

### 3.5.2 内置映射类型

```typescript
// Partial<T> - 将所有属性变为可选
interface User {
  id: number;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// 所有属性变为 id?: number; name?: string; email?: string;

// Required<T> - 将所有可选属性变为必需
type StrictUser = Required<User>;

// Readonly<T> - 将所有属性变为只读
type FrozenUser = Readonly<User>;

// Pick<T, K> - 从T中选择属性K
type UserPreview = Pick<User, "id" | "name">;
// { id: number; name: string; }

// Omit<T, K> - 从T中移除属性K
type UserWithoutEmail = Omit<User, "email">;
// { id: number; name: string; }

// Record<K, V> - 创建键值对类型
type Role = "admin" | "user" | "guest";
type Permissions = Record<Role, string[]>;
// { admin: string[]; user: string[]; guest: string[]; }
```

### 3.5.3 自定义映射类型

```typescript
// 1. 将所有属性值变为另一种类型
type Stringify<T> = {
  [K in keyof T]: string;
};

interface Point {
  x: number;
  y: number;
}

type StringPoint = Stringify<Point>;
// { x: string; y: string; }

// 2. 将所有属性变为nullable
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

type NullablePoint = Nullable<Point>;
// { x: number | null; y: number | null; }

// 3. 将所有属性变为数组
type Arrayify<T> = {
  [K in keyof T]: T[K][];
};

type ArrayPoint = Arrayify<Point>;
// { x: number[]; y: number[]; }

// 4. 深度Partial
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

interface Config {
  db: {
    host: string;
    port: number;
  };
  cache: {
    enabled: boolean;
  };
}

type PartialConfig = DeepPartial<Config>;
// 所有嵌套属性都变为可选
```

### 3.5.4 映射类型与键重新映射

```typescript
// TypeScript 4.1+ 支持键重新映射

// 使用as子句重命名键
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// {
//   getName: () => string;
//   getAge: () => number;
// }

// 过滤属性
type RemoveKind<T> = {
  [K in keyof T as Exclude<K, "kind">]: T[K];
};

interface Shape {
  kind: "circle" | "square";
  radius?: number;
  side?: number;
}

type ShapeWithoutKind = RemoveKind<Shape>;
// { radius?: number; side?: number; }

// 条件映射
type Eventual<T> = {
  [K in keyof T as T[K] extends Promise<any> ? K : never]: T[K];
};

interface Api {
  users: Promise<User[]>;
  posts: Post[];
  comments: Promise<Comment[]>;
}

type AsyncProps = Eventual<Api>;
// { users: Promise<User[]>; comments: Promise<Comment[]>; }
```

---

## 3.6 条件类型

### 3.6.1 基本语法

```typescript
// 条件类型语法：T extends U ? X : Y
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;   // true
type B = IsString<number>;   // false
type C = IsString<"hello">;  // true (字符串字面量是string的子类型)
```

### 3.6.2 分布式条件类型

```typescript
// 当T是联合类型时，条件类型会分布式处理
type ToArray<T> = T extends any ? T[] : never;

type StringOrNumberArray = ToArray<string | number>;
// 相当于：ToArray<string> | ToArray<number>
// 结果：string[] | number[]

// 注意：需要包裹在元组中避免分布式
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type NonDist = ToArrayNonDist<string | number>;
// 结果：(string | number)[]
```

### 3.6.3 实用条件类型

```typescript
// 1. Extract - 从T中提取可以赋值给U的类型
type Extract<T, U> = T extends U ? T : never;

type A = string | number | boolean;
type B = Extract<A, string | number>;  // string | number

// 2. Exclude - 从T中排除可以赋值给U的类型
type Exclude<T, U> = T extends U ? never : T;

type C = Exclude<A, string>;  // number | boolean

// 3. NonNullable - 排除null和undefined
type NonNullable<T> = T extends null | undefined ? never : T;

type D = NonNullable<string | null | undefined>;  // string

// 4. ReturnType - 获取函数返回类型
type ReturnType<T extends (...args: any) => any> = 
  T extends (...args: any) => infer R ? R : never;

function greet(name: string): string {
  return `Hello, ${name}`;
}

type GreetReturn = ReturnType<typeof greet>;  // string

// 5. Parameters - 获取函数参数类型
type Parameters<T extends (...args: any) => any> = 
  T extends (...args: infer P) => any ? P : never;

type GreetParams = Parameters<typeof greet>;  // [name: string]
```

### 3.6.4 条件类型实战

```typescript
// 1. 根据条件选择类型
type MessageOf<T> = T extends { message: infer M } ? M : never;

interface Dog {
  message: string;
}

interface Cat {
  meow(): void;
}

type DogMessage = MessageOf<Dog>;    // string
type CatMessage = MessageOf<Cat>;    // never

// 2. 提取数组元素类型
type Unwrap<T> = T extends Array<infer E> ? E : T;

type A = Unwrap<string[]>;    // string
type B = Unwrap<number>;      // number

// 3. 深度只读
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

// 4. 延迟分发（避免分布式）
type IsUnion<T, U = T> = 
  [T] extends [never] 
    ? false 
    : T extends U 
      ? [U] extends [T] 
        ? false 
        : true 
      : false;

type C = IsUnion<string | number>;   // true
type D = IsUnion<string>;             // false
```

---

## 3.7 模板字面量类型

### 3.7.1 基本语法

```typescript
// 模板字面量类型：使用模板字符串语法创建新类型
type World = "world";
type Greeting = `hello ${World}`;  // "hello world"

// 可以组合多种类型
type PropEventSource<T> = {
  on<K extends string & keyof T>(
    eventName: `${K}Changed`,
    callback: (newValue: T[K]) => void
  ): void;
};
```

### 3.7.2 字符串操作类型

```typescript
// TypeScript内置的字符串操作类型
type Greeting = "Hello, World";

// Uppercase - 转大写
type Shout = Uppercase<Greeting>;  // "HELLO, WORLD"

// Lowercase - 转小写
type Whisper = Lowercase<Greeting>;  // "hello, world"

// Capitalize - 首字母大写
type Greet = Capitalize<Greeting>;  // "Hello, world"

// Uncapitalize - 首字母小写
type Uncap = Uncapitalize<Greeting>;  // "hello, World"
```

### 3.7.3 实战应用

```typescript
// 1. 事件名生成
interface Props {
  name: string;
  age: number;
  visible: boolean;
}

type PropEventSource<T> = {
  on<K extends string & keyof T>(
    eventName: `${K}Changed`,
    callback: (value: T[K]) => void
  ): void;
};

declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>;

const person = makeWatchedObject({
  name: "张三",
  age: 25,
  visible: true
});

person.on("nameChanged", (newName) => {
  console.log(`Name changed to: ${newName}`);
});

person.on("ageChanged", (newAge) => {
  console.log(`Age changed to: ${newAge}`);
});

// 2. CSS属性类型
type CSSProperty = 
  | "margin"
  | "padding"
  | "border"
  | "background";

type CSSDirection = "top" | "right" | "bottom" | "left";
type CSSSide = "top" | "bottom" | "left" | "right";
type CSSSize = "small" | "medium" | "large";

type CSSPropertyName = `${CSSProperty}-${CSSDirection}` | 
                       `${CSSProperty}-${CSSSide}` |
                       `${CSSProperty}-${CSSSize}`;
                       
// 生成的类型：
// "margin-top" | "margin-right" | "padding-left" | ...

// 3. API路由类型
type Method = "GET" | "POST" | "PUT" | "DELETE";
type ApiRoute = `/api/${string}/${Method}`;

function handleRoute(route: ApiRoute) {
  console.log(route);
}

handleRoute("/api/users/GET");   // OK
handleRoute("/api/posts/POST"); // OK
// handleRoute("/users/GET");    // Error: 不以/api开头
```

### 3.7.4 复杂模板字面量

```typescript
// 提取类型
type ExtractRouteParams<T extends string> = 
  T extends `${infer _Start}/${infer Param}/${infer _End}` 
    ? Param 
    : T extends `${infer _Start}/${infer Param}` 
      ? Param 
      : never;

type Route1 = ExtractRouteParams<"/api/users/:id">;  // "users/:id" (不完全匹配)
type Route2 = ExtractRouteParams<"/users/:id">;      // "users/:id"

// 更精确的提取
type ExtractParam<T extends string> =
  T extends `${infer _Prefix}:${infer Param}/${infer _Suffix}` 
    ? Param | ExtractParam<`/${infer Param}/${infer _Suffix}`>
    : T extends `${infer _Prefix}:${infer Param}`
      ? Param
      : never;

type Param = ExtractParam<"/users/:userId/posts/:postId">;
// "userId" | "postId"

// 生成setter类型
type Setters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

interface State {
  name: string;
  age: number;
}

type StateSetters = Setters<State>;
// {
//   setName: (value: string) => void;
//   setAge: (value: number) => void;
// }
```

---

## 3.8 类型推断infer

### 3.8.1 基本概念

```typescript
// infer关键字：在条件类型中推断类型
// 只能在条件类型的extends子句中使用

// 基本语法
type InferType<T> = T extends infer U ? U : never;

type A = InferType<string>;   // string
type B = InferType<number>;    // number
```

### 3.8.2 推断函数返回类型

```typescript
// 推断函数返回类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

function getUser() {
  return { id: 1, name: "张三" };
}

type UserReturn = ReturnType<typeof getUser>;
// { id: number; name: string; }
```

### 3.8.3 推断函数参数类型

```typescript
// 推断函数参数类型
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

function fetchData(url: string, options: RequestInit): Promise<Response> {
  return fetch(url, options);
}

type FetchParams = Parameters<typeof fetchData>;
// [url: string, options: RequestInit]

// 推断第一个参数
type FirstParam<T> = T extends (first: infer F, ...rest: any[]) => any ? F : never;

type URL = FirstParam<typeof fetchData>;  // string
```

### 3.8.4 推断数组/元组元素

```typescript
// 推断数组元素类型
type ElementType<T> = T extends (infer E)[] ? E : never;

type Num = ElementType<number[]>;   // number
type Str = ElementType<string[]>;   // string
type Mixed = ElementType<(string | number)[]>;  // string | number

// 推断元组类型
type Head<T extends any[]> = T extends [infer First, ...any[]] ? First : never;

type H1 = Head<[string, number, boolean]>;  // string
type H2 = Head<[1, 2, 3]>;                 // 1

type Tail<T extends any[]> = T extends [any, ...infer Rest] ? Rest : never;

type T1 = Tail<[string, number, boolean]>;  // [number, boolean]

// 推断最后一个元素
type Last<T extends any[]> = T extends [...any[], infer L] ? L : never;

type L1 = Last<[string, number, boolean]>;  // boolean
```

### 3.8.5 推断Promise内部类型

```typescript
// 推断Promise resolve的类型
type Awaited<T> = T extends Promise<infer U> ? U : T;

type A = Awaited<Promise<string>>;          // string
type B = Awaited<Promise<{ id: number }>>; // { id: number }
type C = Awaited<number>;                   // number（不是Promise，直接返回）

// 深层Awaited
type DeepAwaited<T> = 
  T extends Promise<infer U> 
    ? U extends Promise<any> 
      ? DeepAwaited<U> 
      : U 
    : T;

type Nested = DeepAwaited<Promise<Promise<string>>>;  // string
```

### 3.8.6 综合实例

```typescript
// 完整的工具类型实现
type UnwrapPromise<T> = 
  T extends Promise<infer U> 
    ? U extends object 
      ? { [K in keyof U]: UnwrapPromise<U[K]> }
      : U 
    : T;

interface User {
  id: number;
  friends: Promise<User[]>;
}

type Resolved = UnwrapPromise<{
  id: number;
  friends: User[];
}>;

// 实现一个简单的Flatten类型
type Flatten<T> = 
  T extends Array<infer U> 
    ? U extends object 
      ? Flatten<U> 
      : U 
    : T;

type F1 = Flatten<number[]>;        // number
type F2 = Flatten<string[][]>;      // string
type F3 = Flatten<{ a: number }>;  // { a: number }
```

---

## 📊 高级类型速查表

```
┌─────────────────────────────────┬────────────────────────────────────┐
│ 类型                            │ 说明                               │
├─────────────────────────────────┼────────────────────────────────────┤
│ T | U                           │ 联合类型（T或U）                   │
│ T & U                           │ 交叉类型（T和U）                   │
│ T extends U ? X : Y             │ 条件类型                           │
│ infer X                         │ 类型推断                           │
│ keyof T                         │ 键名联合类型                       │
│ T[K]                            │ 索引访问类型                       │
│ [K in keyof T]                  │ 映射类型                           │
│ [K as NewKey in keyof T]       │ 键重映射                           │
│ `template ${T}`                │ 模板字面量类型                     │
│ Exclude<T, U>                   │ 从T排除U                           │
│ Extract<T, U>                   │ 从T提取U                           │
│ NonNullable<T>                  │ 排除null和undefined                │
│ ReturnType<T>                   │ 函数返回类型                       │
│ Parameters<T>                   │ 函数参数类型                       │
│ InstanceType<T>                 │ 构造函数实例类型                   │
│ Partial<T>                      │ 所有属性可选                       │
│ Required<T>                     │ 所有属性必需                       │
│ Readonly<T>                     │ 所有属性只读                       │
│ Pick<T, K>                     │ 选择属性                           │
│ Omit<T, K>                     │ 排除属性                           │
│ Record<K, V>                    │ 键值对类型                         │
└─────────────────────────────────┴────────────────────────────────────┘
```

---

## 💪 实战练习

### 练习1：实现一个工具类型
```typescript
// 实现DeepPartial<T>，将所有嵌套属性变为可选

interface Company {
  name: string;
  address: {
    city: string;
    street: string;
  };
}

// 期望结果：
// {
//   name?: string;
//   address?: {
//     city?: string;
//     street?: string;
//   };
// }

// 你的代码：
type DeepPartial<T> = /* 实现这里 */
```

### 练习2：实现条件类型
```typescript
// 实现If<C, T, F>，如果C为true返回T，否则返回F

type A = If<true, "yes", "no">;  // "yes"
type B = If<false, "yes", "no">; // "no"

// 你的代码：
type If<C, T, F> = /* 实现这里 */
```

### 练习3：模板字面量应用
```typescript
// 创建一个EventMap类型，为对象每个属性生成change事件

interface State {
  name: string;
  age: number;
}

// 期望：
// {
//   nameChange: (newValue: string) => void;
//   ageChange: (newValue: number) => void;
// }

// 你的代码：
type EventMap<T> = /* 实现这里 */
```

---

## ⏭️ 下一步

前往 [第四章：接口与类型别名](./chapter-04-interfaces.md) 学习接口和类型别名的深入用法

---

> 💡 **思考题**：
> 1. 联合类型和交叉类型在什么场景下使用？
> 2. 什么是分布式条件类型？它有什么特性？
> 3. 映射类型中的as子句有什么作用？