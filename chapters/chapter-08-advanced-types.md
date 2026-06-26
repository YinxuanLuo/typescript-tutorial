# 第八章：类型守卫与高级类型

## 📋 目录

- [8.1 类型收缩](#81-类型收缩)
- [8.2 穷尽性检查](#82-穷尽性检查)
- [8.3 高级类型技巧](#83-高级类型技巧)
- [8.4 类型安全设计模式](#84-类型安全设计模式)
- [8.5 类型断言与类型守卫](#85-类型断言与类型守卫)
- [8.6 类型编程实战](#86-类型编程实战)

---

## 8.1 类型收缩

### 8.1.1 类型收缩原理

```typescript
// 类型收缩（Type Narrowing）：从宽类型到窄类型的转变
// TypeScript通过控制流分析自动收缩类型

function process(value: string | number | null) {
  // 在这里value的类型是 string | number | null
  
  if (value !== null) {
    // 在这里value的类型收缩为 string | number
    // 可以访问string和number共有的方法
    
    value.toString();  // OK
    
    if (typeof value === "string") {
      // 收缩为string
      value.toUpperCase();  // OK
    } else {
      // 收缩为number
      value.toFixed(2);  // OK
    }
  }
}
```

### 8.1.2 收缩机制图解

```
┌─────────────────────────────────────────────────────────────────────┐
│                         类型收缩流程                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   string | number | null                                            │
│          │                                                         │
│          ├── typeof === "string"  →  string                        │
│          ├── typeof === "number"  →  number                        │
│          ├── === null           →  null                             │
│          └── !== null           →  string | number                  │
│                                                                     │
│   if (x !== null) {                                                │
│       // x: string | number                                         │
│       if (typeof x === "string") {                                 │
│           // x: string                                             │
│       } else {                                                      │
│           // x: number                                             │
│       }                                                             │
│   }                                                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 8.1.3 各种收缩技术

```typescript
// 1. typeof 收缩
function foo(x: string | number) {
  if (typeof x === "string") {
    x.toUpperCase();  // string
  } else {
    x.toFixed(2);     // number
  }
}

// 2. instanceof 收缩
class Dog { bark() {} }
class Cat { meow() {} }

function speak(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();  // Dog
  } else {
    animal.meow();  // Cat
  }
}

// 3. in 操作符收缩
interface Car { drive(): void; }
interface Boat { sail(): void; }

function operate(vehicle: Car | Boat) {
  if ("drive" in vehicle) {
    vehicle.drive();  // Car
  } else {
    vehicle.sail();   // Boat
  }
}

// 4. 字面量类型收缩
type Status = "pending" | "success" | "error";

function handleStatus(status: Status) {
  if (status === "pending") {
    // status: "pending"
  } else if (status === "success") {
    // status: "success"
  } else {
    // status: "error"
  }
}
```

---

## 8.2 穷尽性检查

### 8.2.1 never类型与穷尽检查

```typescript
// never类型表示永不存在的值
// 用于穷尽性检查：确保所有可能情况都被处理

type Shape = 
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "rectangle"; width: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "rectangle":
      return shape.width * shape.height;
    default:
      // 穷尽性检查：如果Shape添加新类型，编译器会报错
      const _exhaustive: never = shape;
      throw new Error(`Unknown shape: ${JSON.stringify(_exhaustive)}`);
  }
}
```

### 8.2.2 穷尽检查的好处

```typescript
// 当添加新类型时，穷尽检查会提示需要更新所有switch/if
type Color = "red" | "green" | "blue" | "yellow";

function getColorCode(color: Color): string {
  switch (color) {
    case "red": return "#FF0000";
    case "green": return "#00FF00";
    case "blue": return "#0000FF";
    // case "yellow": 忘记处理！
    default:
      const _exhaustive: never = color;
      // Error: Type '"yellow"' is not assignable to type 'never'
      throw new Error(`Unhandled color: ${_exhaustive}`);
  }
}
```

### 8.2.3 真实项目应用

```typescript
// API状态处理
type ApiState<T> = 
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

function renderState<T>(state: ApiState<T>) {
  switch (state.status) {
    case "idle":
      return null;
    case "loading":
      return <Spinner />;
    case "success":
      return <DataView data={state.data} />;
    case "error":
      return <ErrorView error={state.error} />;
    default:
      const _exhaustive: never = state;
      throw new Error(`Unknown state: ${_exhaustive}`);
  }
}

// Reducer中的穷尽检查
type Action =
  | { type: "INCREMENT" }
  | { type: "DECREMENT" }
  | { type: "RESET" };

function reducer(count: number, action: Action): number {
  switch (action.type) {
    case "INCREMENT":
      return count + 1;
    case "DECREMENT":
      return count - 1;
    case "RESET":
      return 0;
    default:
      const _exhaustive: never = action;
      throw new Error(`Unknown action: ${_exhaustive}`);
  }
}
```

---

## 8.3 高级类型技巧

### 8.3.1 递归类型

```typescript
// 递归类型：类型引用自身
// JSON类型定义
type JSONPrimitive = string | number | boolean | null;
type JSONValue = JSONPrimitive | JSONValue[] | { [key: string]: JSONValue };
type JSONObject = { [key: string]: JSONValue };

// 树形结构
interface TreeNode<T> {
  value: T;
  left?: TreeNode<T>;
  right?: TreeNode<T>;
}

// 深度嵌套对象
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

// 递归Promise
type DeepAwaited<T> = 
  T extends Promise<infer U>
    ? DeepAwaited<U>
    : T;
```

### 8.3.2 分布式类型

```typescript
// 条件类型在联合类型上的分布式行为
type ToArray<T> = T extends any ? T[] : never;

type Result = ToArray<string | number>;
// 相当于：ToArray<string> | ToArray<number>
// 结果：string[] | number[]

// 避免分布式：包裹在元组中
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type Result2 = ToArrayNonDist<string | number>;
// 结果：(string | number)[]

// 应用：过滤联合类型
type Exclude<T, U> = T extends U ? never : T;

type A = "a" | "b" | "c";
type B = Exclude<A, "a">;  // "b" | "c"

// 提取联合类型
type Extract<T, U> = T extends U ? T : never;

type C = Extract<"a" | "b" | "c", "a" | "c">;  // "a" | "c"
```

### 8.3.3 模板字面量进阶

```typescript
// 模板字面量类型
type EventName = `on${Capitalize<string>}`;

const validEvent: EventName = "onClick";   // OK
// const invalidEvent: EventName = "click";  // Error

// 路径类型
type ApiPath = `/api/${string}`;

function fetchApi(path: ApiPath) {
  return fetch(path);
}

fetchApi("/api/users");      // OK
fetchApi("/api/posts/123");  // OK
// fetchApi("/other");        // Error

// 组合多个模板
type CSSProperty = 
  | `${"margin" | "padding"}-${"top" | "right" | "bottom" | "left"}`
  | `${"margin" | "padding"}: ${number}px`;

const prop1: CSSProperty = "margin-top";
const prop2: CSSProperty = "padding-left";
```

### 8.3.4 映射类型进阶

```typescript
// 使用as重映射键
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type Person = { name: string; age: number };
type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }

// 过滤键
type RemoveOptional<T> = {
  [K in keyof T as {} extends Pick<T, K> ? K : never]: T[K];
};

interface User {
  id: string;
  name?: string;
  age?: number;
}
type RequiredUser = RemoveOptional<User>;  // { name?: string; age?: number }

// 条件映射
type Promises<T> = {
  [K in keyof T as T[K] extends Promise<any> ? K : never]: T[K];
};

type Mixed = { a: Promise<string>; b: string; c: Promise<number> };
type OnlyPromises = Promises<Mixed>;  // { a: Promise<string>; c: Promise<number> }
```

---

## 8.4 类型安全设计模式

### 8.4.1 标签联合模式

```typescript
// 标签联合（Tagged Union / 可辨识联合）
interface Loading {
  type: "loading";
}

interface Success<T> {
  type: "success";
  data: T;
}

interface Error {
  type: "error";
  message: string;
}

type AsyncState<T> = Loading | Success<T> | Error;

// 处理函数
function render<T>(state: AsyncState<T>): string {
  switch (state.type) {
    case "loading":
      return "Loading...";
    case "success":
      return `Data: ${JSON.stringify(state.data)}`;
    case "error":
      return `Error: ${state.message}`;
  }
}
```

### 8.4.2 Builder模式

```typescript
// 类型安全的Builder模式
class QueryBuilder<T extends Record<string, any> = {}> {
  private query: T = {} as T;
  
  select<K extends string>(
    ...fields: K[]
  ): QueryBuilder<{ [P in K]: any } & T> {
    return this as any;
  }
  
  where<K extends string, V>(
    field: K,
    value: V
  ): QueryBuilder<T & { [P in K]: V }> {
    return this as any;
  }
  
  orderBy(
    field: keyof T,
    direction?: "asc" | "desc"
  ): QueryBuilder<T> {
    return this as any;
  }
  
  build(): T {
    return this.query;
  }
}

// 使用 - 链式调用，类型安全
const query = new QueryBuilder()
  .select("id", "name", "email")
  .where("age", 18)
  .where("status", "active")
  .orderBy("createdAt", "desc")
  .build();

// query: { id: any; name: any; email: any; age: number; status: string; }
```

### 8.4.3 Result类型

```typescript
// 类似Rust的Result类型
type Result<T, E = Error> = 
  | { success: true; value: T }
  | { success: false; error: E };

// 工厂函数
function success<T>(value: T): Result<T, never> {
  return { success: true, value };
}

function failure<E>(error: E): Result<never, E> {
  return { success: false, error };
}

// 链式处理
type User = { id: number; name: string };

function parseUser(json: string): Result<User, SyntaxError> {
  try {
    const data = JSON.parse(json);
    return success(data as User);
  } catch (e) {
    return failure(e as SyntaxError);
  }
}

function validateUser(user: User): Result<User, string> {
  if (!user.name) {
    return failure("Name is required");
  }
  return success(user);
}

function handleUser(json: string): void {
  const userResult = parseUser(json);
  
  if (!userResult.success) {
    console.error("Parse error:", userResult.error);
    return;
  }
  
  const validationResult = validateUser(userResult.value);
  
  if (!validationResult.success) {
    console.error("Validation error:", validationResult.error);
    return;
  }
  
  console.log("User:", validationResult.value);
}
```

### 8.4.4 Option类型

```typescript
// 类似Haskell/Scala的Option/Maybe类型
type Option<T> = Some<T> | None;

interface Some<T> {
  readonly _tag: "Some";
  readonly value: T;
}

interface None {
  readonly _tag: "None";
}

const some = <T>(value: T): Option<T> => ({ _tag: "Some", value });
const none = <T>(): Option<T> => ({ _tag: "None" });

// 工具函数
function isSome<T>(opt: Option<T>): opt is Some<T> {
  return opt._tag === "Some";
}

function isNone<T>(opt: Option<T>): opt is None {
  return opt._tag === "None";
}

function map<T, U>(opt: Option<T>, fn: (value: T) => U): Option<U> {
  return isSome(opt) ? some(fn(opt.value)) : none();
}

function flatMap<T, U>(opt: Option<T>, fn: (value: T) => Option<U>): Option<U> {
  return isSome(opt) ? fn(opt.value) : none();
}

function getOrElse<T>(opt: Option<T>, defaultValue: T): T {
  return isSome(opt) ? opt.value : defaultValue;
}

// 使用
interface User {
  name: string;
  address?: { city: string };
}

function getCity(user: User): Option<string> {
  return some(user.address?.city ?? "Unknown");
}

const user: User = { name: "张三" };
const city = getCity(user);

if (isSome(city)) {
  console.log(city.value);
} else {
  console.log("No city");
}

// 更简洁的使用
console.log(getOrElse(getCity(user), "Unknown"));
```

---

## 8.5 类型断言与类型守卫

### 8.5.1 类型守卫

```typescript
// 类型守卫函数
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isNumber(value: unknown): value is number {
  return typeof value === "number";
}

function isNonNull<T>(value: T): value is T & {} {
  return value !== null && value !== undefined;
}

// 使用
function process(value: unknown) {
  if (isString(value)) {
    value.toUpperCase();  // string
  } else if (isNumber(value)) {
    value.toFixed(2);     // number
  }
}
```

### 8.5.2 自定义守卫

```typescript
// 接口守卫
interface Fish {
  swim(): void;
  hasGills: boolean;
}

interface Bird {
  fly(): void;
  hasWings: boolean;
}

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function isBird(pet: Fish | Bird): pet is Bird {
  return (pet as Bird).fly !== undefined;
}

// 对象守卫
function hasProperty<T, K extends string>(
  obj: T,
  key: K
): obj is T & { [P in K]: unknown } {
  return key in (obj as object);
}

const data: Record<string, unknown> = { name: "张三", age: 25 };

if (hasProperty(data, "name")) {
  console.log(data.name);  // string
}

// 数组守卫
function isArrayOf<T>(
  value: unknown,
  isT: (item: unknown) => item is T
): value is T[] {
  return Array.isArray(value) && value.every(isT);
}

const mixed = [1, "2", 3];
const strings = mixed.filter((x): x is string => typeof x === "string");
```

### 8.5.3 断言函数

```typescript
// 断言函数 - 条件不满足时抛出错误
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error(`Expected string, got ${typeof value}`);
  }
}

function process(value: unknown) {
  assertIsString(value);  // 断言后value是string
  value.toUpperCase();    // OK
}

// 带条件的断言
function assertIsDefined<T>(
  value: T,
  message: string = "Value must be defined"
): asserts value is NonNullable<T> {
  if (value === null || value === undefined) {
    throw new Error(message);
  }
}

function getName(user: { name?: string }): string {
  assertIsDefined(user.name, "User name is required");
  return user.name;  // string - 非空断言
}
```

### 8.5.4 可辨识联合类型守卫

```typescript
// 可辨识联合是最强的类型守卫
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

type Shape = Circle | Square | Rectangle;

// in 操作符
function isCircle(shape: Shape): shape is Circle {
  return "radius" in shape && shape.kind === "circle";
}

// kind 属性
function getArea(shape: Shape): number {
  // switch是最常用的方式
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "rectangle":
      return shape.width * shape.height;
  }
}
```

---

## 8.6 类型编程实战

### 8.6.1 完整工具类型实现

```typescript
// 1. DeepPartial
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

// 2. DeepRequired
type DeepRequired<T> = {
  [K in keyof T]-?: T[K] extends object ? DeepRequired<T[K]> : T[K];
};

// 3. DeepReadonly
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

// 4. DeepNonNullable
type DeepNonNullable<T> = {
  [K in keyof T]: T[K] extends object ? DeepNonNullable<T[K]> : NonNullable<T[K]>;
};

// 5. Mutable
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};

// 6. RequiredKeys / OptionalKeys
type RequiredKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? never : K;
}[keyof T];

type OptionalKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? K : never;
}[keyof T];

// 7. FunctionKeys
type FunctionKeys<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];

// 8. PromiseType
type Awaited<T> = 
  T extends null | undefined ? T :
  T extends object & { then(onfulfilled: infer F, ...args: any): any } ?
    F extends (value: infer V, ...args: any) => any ? Awaited<V> : never :
    T;
```

### 8.6.2 实际项目工具类型

```typescript
// 表单类型
type FormState<T> = {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
  isSubmitting: boolean;
  isValid: boolean;
};

// API响应类型
type ApiResponse<T, E = string> = 
  | { success: true; data: T; error: never }
  | { success: false; data: never; error: E };

// 分页类型
type Paginated<T> = {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
  hasMore: boolean;
};

// 事件类型
type EventHandlers<T extends Record<string, any>> = {
  [K in keyof T as `on${Capitalize<string & K>}`]?: (event: T[K]) => void;
};

// CSS Properties类型
type CSSProperties = Record<string, string | number>;

// Route params类型
type RouteParams<Path extends string> = 
  Path extends `${string}:${infer Param}/${infer Rest}` 
    ? Param | RouteParams<`/${Rest}`>
    : Path extends `${string}:${infer Param}`
      ? Param
      : never;

type Params = RouteParams<"/users/:userId/posts/:postId">;
// "userId" | "postId"
```

### 8.6.3 验证库类型

```typescript
// 验证规则类型
type ValidationRule<T> = (value: T) => boolean | string;

interface Validator<T> {
  rules: Partial<Record<keyof T, ValidationRule<T[keyof T]>[]>>;
}

// 创建验证器
function createValidator<T>(rules: Validator<T>["rules"]): Validator<T> {
  return { rules };
}

// 验证函数
function validate<T>(data: T, validator: Validator<T>): Partial<Record<keyof T, string>> {
  const errors: Partial<Record<keyof T, string>> = {};
  
  for (const key in validator.rules) {
    const rules = validator.rules[key]!;
    const value = data[key];
    
    for (const rule of rules) {
      const result = rule(value);
      if (result !== true) {
        errors[key] = typeof result === "string" ? result : "Validation failed";
        break;
      }
    }
  }
  
  return errors;
}

// 使用
interface User {
  name: string;
  email: string;
  age: number;
}

const validator = createValidator<User>({
  name: [
    v => v.length >= 2 || "Name must be at least 2 characters",
    v => v.length <= 50 || "Name must be at most 50 characters"
  ],
  email: [
    v => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v) || "Invalid email address"
  ],
  age: [
    v => v >= 18 || "Must be at least 18 years old"
  ]
});

const errors = validate({
  name: "张",
  email: "invalid",
  age: 15
}, validator);
// { name: "Name must be at least 2 characters", email: "Invalid email address", age: "Must be at least 18 years old" }
```

### 8.6.4 类型安全的EventEmitter

```typescript
// 类型安全的EventEmitter
type EventMap = Record<string, any[]>;

class TypedEventEmitter<Events extends EventMap> {
  private listeners = new Map<keyof Events, Set<Function>>();
  
  on<K extends keyof Events>(
    event: K,
    listener: (...args: Events[K]) => void
  ): this {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(listener);
    return this;
  }
  
  off<K extends keyof Events>(
    event: K,
    listener: (...args: Events[K]) => void
  ): this {
    this.listeners.get(event)?.delete(listener);
    return this;
  }
  
  emit<K extends keyof Events>(event: K, ...args: Events[K]): void {
    this.listeners.get(event)?.forEach(listener => listener(...args));
  }
  
  once<K extends keyof Events>(
    event: K,
    listener: (...args: Events[K]) => void
  ): this {
    const wrapper = (...args: Events[K]) => {
      this.off(event, wrapper);
      listener(...args);
    };
    return this.on(event, wrapper);
  }
}

// 使用
interface AppEvents {
  userLoggedIn: [userId: string, timestamp: Date];
  userLoggedOut: [userId: string];
  error: [error: Error, context: string];
  dataLoaded: [data: any[]];
}

const emitter = new TypedEventEmitter<AppEvents>();

emitter.on("userLoggedIn", (userId, timestamp) => {
  console.log(`${userId} logged in at ${timestamp}`);
});

emitter.on("error", (error, context) => {
  console.error(`Error in ${context}:`, error);
});

// 编译时会检查参数类型
// emitter.emit("userLoggedIn", "user123");  // Error: Expected 2 arguments
emitter.emit("userLoggedIn", "user123", new Date());  // OK
```

---

## 📊 高级类型速查表

```
┌─────────────────────────────────┬────────────────────────────────────┐
│ 类型                            │ 说明                               │
├─────────────────────────────────┼────────────────────────────────────┤
│ T extends U ? X : Y             │ 条件类型                           │
│ infer R                         │ 类型推断                           │
│ keyof T                         │ 键名联合类型                       │
│ T[K]                            │ 索引访问                           │
│ [K in keyof T]                 │ 映射类型                           │
│ [K as NewKey in keyof T]        │ 键重映射                           │
│ `template ${T}`                │ 模板字面量                         │
│ never                           │ 空类型，穷尽检查                   │
│ T | U                           │ 联合类型                           │
│ T & U                           │ 交叉类型                           │
│ Partial<T>                      │ 可选                               │
│ Required<T>                     │ 必需                               │
│ Readonly<T>                     │ 只读                               │
│ Pick<T, K>                     │ 选择                               │
│ Omit<T, K>                     │ 排除                               │
│ Exclude<T, U>                   │ 排除类型                           │
│ Extract<T, U>                   │ 提取类型                           │
│ NonNullable<T>                  │ 非空                               │
│ ReturnType<T>                  │ 返回类型                           │
│ Parameters<T>                  │ 参数类型                           │
│ InstanceType<T>                 │ 实例类型                           │
└─────────────────────────────────┴────────────────────────────────────┘
```

---

## 💪 实战练习

### 练习1：实现类型安全的reduce
```typescript
// 实现一个类型安全的reduce函数
// 接收数组，累加器函数，返回类型应正确推断

// 你的代码：
function reduce<T, U>(
  array: T[],
  reducer: (acc: U, current: T, index: number) => U,
  initial: U
): U {
  // 实现这里
}

// 期望：
// const result = reduce([1, 2, 3], (sum, n) => sum + n, 0);
// result类型应为 number
```

### 练习2：实现Optional工具类型
```typescript
// 实现以下工具类型：

// 1. Optional<T, K> - 使指定属性可选
type Optional<T, K extends keyof T> = /* 实现这里 */

// 2.至少有一个必需属性
type AtLeastOne<T, Keys extends keyof T = keyof T> = /* 实现这里 */

// 使用示例：
type A = Optional<{ name: string; age: number }, "age">;
// { name: string; age?: number }

type B = AtLeastOne<{ a?: string; b?: string; c?: string }>;
// 至少有一个属性是必需的
```

---

## ⏭️ 下一步

前往 [第九章：TypeScript编译配置](./chapter-09-configuration.md) 学习tsconfig.json和各种编译选项

---

> 💡 **思考题**：
> 1. 什么是类型收缩？有哪些收缩技术？
> 2. never类型在穷尽性检查中起什么作用？
> 3. 如何实现一个类型安全的EventEmitter？