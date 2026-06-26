# 第七章：模块与命名空间

## 📋 目录

- [7.1 模块系统基础](#71-模块系统基础)
- [7.2 导入导出详解](#72-导入导出详解)
- [7.3 模块解析](#73-模块解析)
- [7.4 命名空间](#74-命名空间)
- [7.5 声明文件](#75-声明文件)
- [7.6 模块与类型](#76-模块与类型)
- [7.7 实际项目结构](#77-实际项目结构)

---

## 7.1 模块系统基础

### 7.1.1 ES模块 vs CommonJS

```typescript
// ES模块 (ESM) - 使用 import/export
// 文件名: greeting.ts

export const greeting = "Hello";
export function sayHello(name: string): string {
  return `${greeting}, ${name}!`;
}

export class User {
  constructor(public name: string) {}
}

// 导入
import { greeting, sayHello, User } from "./greeting";

// CommonJS (CJS) - 使用 module.exports/require
// 文件名: math.js

exports.add = function(a: number, b: number) {
  return a + b;
};

exports.multiply = function(a: number, b: number) {
  return a * b;
};

// 导入
const { add, multiply } = require("./math");
```

### 7.1.2 TypeScript模块配置

```json
// tsconfig.json
{
  "compilerOptions": {
    // 模块系统
    "module": "commonjs",        // 输出模块格式
    "moduleResolution": "node",  // 解析策略：node/classic
    
    // ES模块特性
    "esModuleInterop": true,     // 让CommonJS模块像ES模块一样工作
    "allowSyntheticDefaultImports": true,  // 允许从模块默认导入
    "forceConsistentCasingInFileNames": true,  // 文件名大小写一致
    
    // 路径配置
    "baseUrl": "./src",
    "paths": {
      "@/*": ["./*"],
      "@components/*": ["./components/*"]
    }
  }
}
```

### 7.1.3 模块输出选项

```typescript
// 编译目标不同，输出的模块格式也不同

// module: "commonjs" - Node.js环境
// 输出:
// exports.Math = void 0;
var Math = /** @class */ (function () {
//   function Math() {}
//   Math.prototype.add = function (a, b) { return a + b; };
//   return Math;
// }());
// exports.Math = Math;

// module: "ES6"/"ES2015" - 浏览器原生模块
// 输出:
// export class Math {}

// module: "AMD" - 异步模块定义
// module: "UMD" - 通用模块定义
// module: "System" - SystemJS
```

---

## 7.2 导入导出详解

### 7.2.1 命名导出

```typescript
// 方式一：直接在声明前导出
export const PI = 3.14159;
export function add(a: number, b: number): number {
  return a + b;
}
export class Calculator {}

// 方式二：先定义，再统一导出
const E = 2.71828;
function multiply(a: number, b: number): number {
  return a * b;
}

export { E, multiply };
```

### 7.2.2 默认导出

```typescript
// 默认导出 - 每个模块只能有一个
// greet.ts
export default function greet(name: string): string {
  return `Hello, ${name}!`;
}

// 可以同时有默认导出和命名导出
export default function defaultFunc() {}
export const namedExport = "value";

// 导入默认导出
import greet from "./greet";
// 或者
import { default as defaultFunc } from "./greet";
```

### 7.2.3 导入语法

```typescript
// 命名导入
import { add, multiply, Calculator } from "./math";

// 重命名导入
import { add as sum, Calculator as Calc } from "./math";

// 导入所有命名导出
import * as MathUtils from "./math";

// 默认导入
import greet from "./greet";

// 混合导入
import defaultExport, { named1, named2 } from "./module";

// 仅导入副作用（执行模块但不导入任何内容）
import "./polyfills";
```

### 7.2.4 重新导出

```typescript
// 重新导出（不导入，直接导出）
export { add, multiply } from "./math";
export { Calculator } from "./math";
export { add as sum } from "./math";

// 重新导出所有
export * from "./math";

// 重新导出并重命名
export { default } from "./defaultExport";
export { default as DefaultClass } from "./someModule";
```

### 7.2.5 动态导入

```typescript
// 动态导入 - 返回Promise
async function loadModule() {
  const math = await import("./math");
  const result = math.add(1, 2);
}

// 条件导入
async function loadFeature(flag: boolean) {
  if (flag) {
    const { featureA } = await import("./features/featureA");
    featureA.init();
  } else {
    const { featureB } = await import("./features/featureB");
    featureB.init();
  }
}

// 按需加载 - 代码分割
const handleClick = () => {
  import("./heavy").then(({ HeavyClass }) => {
    new HeavyClass();
  });
};
```

---

## 7.3 模块解析

### 7.3.1 模块解析策略

```typescript
// Node解析策略 (moduleResolution: "node")
// 查找顺序：相对导入 -> ./, ../ | 非相对导入 -> node_modules

// 假设在 src/components/Button.ts 中
import { User } from "./User";           // src/components/User.ts
import { Utils } from "../utils";        // src/utils.ts
import { React } from "react";           // node_modules/react
import { Component } from "./Component"; // src/components/Component/index.ts 或 .d.ts

// Classic解析策略 (moduleResolution: "classic")
// 更多用于旧项目和声明文件
```

### 7.3.2 路径别名配置

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@/*": ["./*"],
      "@components/*": ["components/*"],
      "@utils/*": ["utils/*"],
      "@types/*": ["types/*"]
    }
  }
}

// 使用
import { Button } from "@components/Button";
import { formatDate } from "@utils/date";
import { User } from "@/types/user";

// 配合webpack配置
// webpack.config.js
// resolve: {
//   alias: {
//     '@': path.resolve(__dirname, 'src'),
//     '@components': path.resolve(__dirname, 'src/components')
//   }
// }
```

### 7.3.3 扩展名处理

```typescript
// TypeScript会自动解析以下扩展名：
// 1. .ts, .tsx
// 2. .d.ts
// 3. .js, .jsx (在allowJs为true时)
// 4. .json (在resolveJsonModule为true时)

// 目录导入 (index文件)
import { Button } from "./components/Button";
// 实际上导入 ./components/Button/index.ts 或 ./components/Button/index.js

// package.json main字段
import Utils from "utils";
// 解析为 node_modules/utils/package.json 中的 main 字段指向的文件
```

---

## 7.4 命名空间

### 7.4.1 命名空间定义

```typescript
// 命名空间 - 将相关代码组织在一起，避免全局污染
// 旧式组织方式，ES6模块出现后逐渐淘汰

// Validation.ts
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }
  
  const lettersRegexp = /^[A-Za-z]+$/;
  const numberRegexp = /^[0-9]+$/;
  
  export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string): boolean {
      return lettersRegexp.test(s);
    }
  }
  
  export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string): boolean {
      return s.length === 5 && numberRegexp.test(s);
    }
  }
}
```

### 7.4.2 命名空间使用

```typescript
// 使用命名空间
/// <reference path="Validation.ts" />

let validators: Validation.StringValidator[] = [];

validators.push(new Validation.LettersOnlyValidator());
validators.push(new Validation.ZipCodeValidator());

// 别名
import lettersValidator = Validation.LettersOnlyValidator;
const lv = new lettersValidator();
```

### 7.4.3 多文件命名空间

```typescript
// 文件1: Animal.ts
namespace Animal {
  export interface Animal {
    name: string;
    speak(): void;
  }
}

// 文件2: Dog.ts
/// <reference path="Animal.ts" />
namespace Animal {
  export class Dog implements Animal {
    constructor(public name: string) {}
    speak(): void {
      console.log(`${this.name} says: Woof!`);
    }
  }
}

// 文件3: Cat.ts
/// <reference path="Animal.ts" />
namespace Animal {
  export class Cat implements Animal {
    constructor(public name: string) {}
    speak(): void {
      console.log(`${this.name} says: Meow!`);
    }
  }
}

// 使用
/// <reference path="Animal.ts" />
/// <reference path="Dog.ts" />
/// <reference path="Cat.ts" />

const dog = new Animal.Dog("Buddy");
const cat = new Animal.Cat("Whiskers");
dog.speak();  // Buddy says: Woof!
cat.speak();  // Whiskers says: Meow!
```

### 7.4.4 模块 vs 命名空间

```
┌─────────────────────────────────────────────────────────────────────┐
│                        模块 vs 命名空间                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   命名空间 (Namespace)           模块 (Module)                       │
│   ┌─────────────────────┐       ┌─────────────────────┐            │
│   │ • 全局作用域组织代码 │       │ • ES6标准           │            │
│   │ • 用///reference     │       │ • 显式导入导出      │            │
│   │ • 可能产生全局污染   │       │ • 天然隔离          │            │
│   │ • 旧代码兼容        │       │ • 推荐方式          │            │
│   │ • 适合工具函数      │       │ • 适合大型应用      │            │
│   └─────────────────────┘       └─────────────────────┘            │
│                                                                     │
│   推荐：优先使用ES模块，命名空间仅用于旧代码或全局扩展                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7.5 声明文件

### 7.5.1 .d.ts声明文件

```typescript
// 声明文件用于为JavaScript库提供类型信息
// mylib.d.ts

// 声明模块
declare module "mylib" {
  export function doSomething(): void;
  export const VERSION: string;
  export class MyClass {
    constructor(name: string);
    greet(): string;
  }
}

// 声明全局变量
declare const GLOBAL_CONFIG: {
  apiUrl: string;
  timeout: number;
};

// 声明全局函数
declare function globalFunction(param: string): number;

// 声明全局类型
declare type GlobalCallback = (data: any) => void;
```

### 7.5.2 内置声明文件

```typescript
// TypeScript内置了很多声明文件
// lib.dom.d.ts - DOM API
// lib.es2015.d.ts - ES6 API
// lib.node.d.ts - Node.js API (@types/node)

// 直接使用，不需要导入
document.getElementById("app");
window.location.href;
console.log("message");

// 数组方法
[1, 2, 3].map(x => x * 2);
Promise.resolve(42);
```

### 7.5.3 第三方库声明

```typescript
// 使用@types/*包获取社区声明文件
// npm install --save-dev @types/lodash

// 使用
import _ from "lodash";
_.chunk([1, 2, 3, 4], 2);  // [[1, 2], [3, 4]]

// 如果没有声明文件，可以自己声明
// types/custom/index.d.ts
declare module "custom-lib" {
  export interface Config {
    debug: boolean;
  }
  
  export function init(config: Config): void;
  export const version: string;
}
```

### 7.5.4 扩展已有声明

```typescript
// 扩展第三方库的类型
// 例如：扩展lodash
import { LoDashStatic } from "lodash";

declare module "lodash" {
  interface LoDashStatic {
    // 添加自定义方法
    myCustomMethod(): string;
  }
}

// 使用
import _ from "lodash";
_.myCustomMethod();

// 扩展window
interface Window {
  ga: (command: string, ...args: any[]) => void;
  analytics: Analytics;
}

// 扩展express
declare global {
  namespace Express {
    interface Request {
      userId?: string;
      headers: {
        authorization?: string;
      };
    }
  }
}
```

---

## 7.6 模块与类型

### 7.6.1 使用import type

```typescript
// 只导入类型，不导入实际值
import type { User, Order } from "./types";
import type { Config } from "./config";

// 好处：
// 1. 编译后完全不包含这些导入，减小bundle
// 2. 明确区分类型导入和值导入
// 3. 在const assertion场景下特别有用

// 常量断言
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000
} as const;

type Config = typeof config;

// 在类型代码中使用
import type { Component } from "react";

function render<T extends Component>(component: T): JSX.Element {
  return component.render();
}
```

### 7.6.2 类型专有导入导出

```typescript
// export type 专有导出
// types.ts
export type ID = string | number;
export interface User {
  id: ID;
  name: string;
}

export type { User as UserType };

// import type 专有导入
import type { ID, User } from "./types";

// 重新导出类型
export type { User } from "./user";
```

### 7.6.3 全局类型声明

```typescript
// global.d.ts - 全局类型声明

// 全局变量
declare const API_BASE_URL: string;

// 全局函数
declare function trackEvent(event: string, data?: object): void;

// 全局接口
interface Window {
  analytics: Analytics;
}

// 全局命名空间
declare namespace GLOBAL {
  const VERSION: string;
  function log(message: string): void;
}

// 全局枚举
declare enum LogLevel {
  DEBUG = 0,
  INFO = 1,
  WARN = 2,
  ERROR = 3
}
```

---

## 7.7 实际项目结构

### 7.7.1 标准项目结构

```
my-project/
├── src/
│   ├── index.ts           # 入口文件
│   ├── App.ts             # 主应用
│   │
│   ├── components/        # 组件
│   │   ├── Button/
│   │   │   ├── index.ts
│   │   │   ├── Button.tsx
│   │   │   └── Button.test.tsx
│   │   └── Input/
│   │
│   ├── pages/             # 页面
│   │   ├── Home/
│   │   └── About/
│   │
│   ├── hooks/             # 自定义hooks
│   │   ├── useAuth.ts
│   │   └── useAsync.ts
│   │
│   ├── utils/            # 工具函数
│   │   ├── format.ts
│   │   └── validate.ts
│   │
│   ├── services/         # API服务
│   │   ├── api.ts
│   │   └── user.service.ts
│   │
│   ├── stores/           # 状态管理
│   │   └── user.store.ts
│   │
│   ├── types/            # 类型定义
│   │   ├── index.ts
│   │   ├── user.ts
│   │   └── api.ts
│   │
│   └── constants/        # 常量
│       └── index.ts
│
├── tests/                # 测试文件
│   ├── unit/
│   └── integration/
│
├── dist/                 # 编译输出
├── node_modules/
├── package.json
├── tsconfig.json
├── jest.config.js
└── README.md
```

### 7.7.2 Barrel文件模式

```typescript
// src/components/index.ts - Barrel文件
// 统一导出所有组件，简化导入路径

export { Button } from "./Button";
export { Input } from "./Input";
export { Select } from "./Select";
export { Modal } from "./Modal";
// ...

// 使用 - 只需一个导入
import { Button, Input, Modal } from "@/components";

// 而不是
import { Button } from "@/components/Button";
import { Input } from "@/components/Input";
import { Modal } from "@/components/Modal";
```

### 7.7.3 命名导出约定

```typescript
// 建议：使用命名导出而非默认导出（React生态例外）

// types/user.ts
export type UserRole = "admin" | "user" | "guest";

export interface User {
  id: string;
  name: string;
  email: string;
  role: UserRole;
  createdAt: Date;
}

export interface CreateUserDTO {
  name: string;
  email: string;
  password: string;
}

export interface UpdateUserDTO {
  name?: string;
  email?: string;
}

// services/user.service.ts
import type { User, CreateUserDTO, UpdateUserDTO } from "@/types/user";

export class UserService {
  async getUser(id: string): Promise<User> {
    // 实现
  }
  
  async createUser(dto: CreateUserDTO): Promise<User> {
    // 实现
  }
  
  async updateUser(id: string, dto: UpdateUserDTO): Promise<User> {
    // 实现
  }
}

export const userService = new UserService();
```

### 7.7.4 循环依赖处理

```typescript
// 循环依赖示例
// a.ts -> b.ts -> a.ts

// a.ts
import { b } from "./b";

export class A {
  value = 1;
  
  getFromB() {
    // b可能还未完全初始化
    return b?.value;  // 使用可选链防护
  }
}

export const a = new A();

// b.ts
import { a } from "./a";

export class B {
  value = 2;
  
  getFromA() {
    return a?.value ?? 0;
  }
}

export const b = new B();

// 更好的设计：避免循环依赖
// 使用依赖注入或重构设计
```

### 7.7.5 模块使用示例

```typescript
// src/types/user.ts
export interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
  status: "active" | "inactive" | "banned";
}

export interface UserProfile extends User {
  bio?: string;
  phone?: string;
  address?: Address;
}

export interface Address {
  city: string;
  street: string;
  zipCode: string;
}

// src/services/user.service.ts
import type { User, UserProfile, CreateUserDTO } from "@/types/user";

export class UserService {
  async getUserById(id: string): Promise<User | null> {
    // 实现
  }
  
  async updateProfile(id: string, data: Partial<UserProfile>): Promise<UserProfile> {
    // 实现
  }
}

export const userService = new UserService();

// src/hooks/useUser.ts
import { userService } from "@/services/user.service";
import type { User } from "@/types/user";
import { ref } from "vue";

export function useUser(id: string) {
  const user = ref<User | null>(null);
  const loading = ref(false);
  const error = ref<string | null>(null);
  
  async function fetchUser() {
    loading.value = true;
    try {
      user.value = await userService.getUserById(id);
    } catch (e) {
      error.value = (e as Error).message;
    } finally {
      loading.value = false;
    }
  }
  
  return { user, loading, error, fetchUser };
}
```

---

## 📊 模块速查表

```
┌─────────────────────────────────┬────────────────────────────────────┐
│ 语法                            │ 说明                               │
├─────────────────────────────────┼────────────────────────────────────┤
│ export const x = 1             │ 命名导出                           │
│ export function fn() {}        │ 命名导出函数                       │
│ export class C {}              │ 命名导出类                         │
│ export { x, y }                │ 命名导出解构                       │
│ export { x as y }              │ 重命名导出                         │
│ export default fn              │ 默认导出                           │
│ export * from "./module"       │ 重新导出所有                       │
│ export type { T }              │ 类型专有导出                       │
├─────────────────────────────────┼────────────────────────────────────┤
│ import { x } from "./mod"      │ 命名导入                           │
│ import x from "./mod"          │ 默认导入                           │
│ import * as ns from "./mod"    │ 命名空间导入                       │
│ import { x as y } from "./mod"  │ 重命名导入                         │
│ import type { T } from "./mod" │ 类型专有导入（仅类型）              │
│ import "./mod"                 │ 副作用导入                         │
│ export type { T } from "./mod" │ 类型专有重新导出                   │
└─────────────────────────────────┴────────────────────────────────────┘
```

---

## 💪 实战练习

### 练习1：重构为模块化结构
```typescript
// 将以下代码重构为模块化结构
// 包含：types.ts, utils.ts, main.ts

// 类型
interface User {
  id: number;
  name: string;
  email: string;
}

interface Product {
  id: number;
  name: string;
  price: number;
}

// 工具函数
function validateEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

function formatCurrency(amount: number): string {
  return `¥${amount.toFixed(2)}`;
}

// 主逻辑
const users: User[] = [];
const products: Product[] = [];

function addUser(user: User) {
  if (validateEmail(user.email)) {
    users.push(user);
  }
}

function addProduct(product: Product) {
  products.push(product);
}

export { users, products, addUser, addProduct };
```

### 练习2：创建声明文件
```typescript
// 为以下JavaScript库创建TypeScript声明文件
// mylib.js

function createStore(initialState) {
  let state = initialState;
  const listeners = [];
  
  return {
    getState: () => state,
    dispatch: (action) => {
      state = { ...state, ...action };
      listeners.forEach(fn => fn(state));
    },
    subscribe: (fn) => {
      listeners.push(fn);
      return () => {
        const index = listeners.indexOf(fn);
        if (index > -1) listeners.splice(index, 1);
      };
    }
  };
}

// 你的代码：mylib.d.ts
```

---

## ⏭️ 下一步

前往 [第八章：类型守卫与高级类型](./chapter-08-advanced-types.md) 深入学习高级类型技巧

---

> 💡 **思考题**：
> 1. ES模块和CommonJS模块有什么区别？
> 2. 什么是Barrel文件模式？有什么优缺点？
> 3. 如何扩展第三方库的声明文件？