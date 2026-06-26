# 第一章：TypeScript简介与环境搭建

## 📋 目录

- [1.1 TypeScript前世今生](#11-typescript前世今生)
- [1.2 TypeScript核心概念](#12-typescript核心概念)
- [1.3 环境搭建详解](#13-环境搭建详解)
- [1.4 开发工具配置](#14-开发工具配置)
- [1.5 第一个TypeScript程序](#15-第一个typescript程序)

---

## 1.1 TypeScript前世今生

### 1.1.1 历史背景

```
时间线：
┌─────────────────────────────────────────────────────────────────────┐
│  2012年  │  Microsoft发布TypeScript 0.8                              │
│     ↓    │                                                           │
│  2014年  │  TypeScript 1.0发布，IDE支持增强                            │
│     ↓    │                                                           │
│  2015年  │  Angular 2宣布使用TypeScript，生态开始爆发                   │
│     ↓    │                                                           │
│  2016年  │  TypeScript 2.0发布，重要特性：非空类型、控制流分析           │
│     ↓    │                                                           │
│  2018年  │  TypeScript 3.0发布，参数元组、泛型约束增强                   │
│     ↓    │                                                           │
│  2020年  │  TypeScript 4.0发布，变参元组、标记元组                      │
│     ↓    │                                                           │
│  2022年  │  TypeScript 4.9发布satisfies运算符                          │
│     ↓    │                                                           │
│  2024年  │  TypeScript 5.x持续迭代                                    │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.1.2 为什么要用TypeScript？

```
┌────────────────────────────────────────────────────────────────────┐
│                      TypeScript 核心价值                              │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐         │
│   │   可靠性    │     │   可维护性  │     │   开发效率  │         │
│   └─────────────┘     └─────────────┘     └─────────────┘         │
│         ↓                   ↓                   ↓                  │
│   • 类型检查        • 代码文档化        • 智能代码提示              │
│   • 编译时发现错误   • 易于重构         • 自动补全                 │
│   • 减少调试时间     • 清晰的数据结构    • 快速定位API              │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 1.1.3 TypeScript工作原理

```
┌─────────────────────────────────────────────────────────────────────┐
│                        TypeScript 编译流程                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌────────────┐      ┌────────────┐      ┌────────────┐           │
│   │  .ts文件   │ ---> │  tsc编译   │ ---> │  .js文件   │           │
│   │  源代码    │      │   检查     │      │  可执行    │           │
│   └────────────┘      └────────────┘      └────────────┘           │
│                             ↓                                       │
│                      ┌────────────┐                                 │
│                      │ 类型检查   │                                 │
│                      │ 错误报告   │                                 │
│                      └────────────┘                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                        TypeScript 类型检查                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   编译时检查（TypeScript）          运行时检查（JavaScript）          │
│   ┌────────────────────┐          ┌────────────────────┐            │
│   │ let name: string;  │          │ let name = "John"; │            │
│   │ name = 123; ❌     │          │ name = 123; ✅     │            │
│   │ // 编译错误！       │          │ // 运行成功...     │            │
│   └────────────────────┘          └────────────────────┘            │
│                                                                     │
│   ✓ 提前发现问题                          ✗ 运行后才发现问题         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1.2 TypeScript核心概念

### 1.2.1 什么是TypeScript？

```typescript
// TypeScript是JavaScript的超集
// 包含三部分：语法 + 类型系统 + 编译转换

// 1. 语法扩展 - 添加了类型注解等语法
let message: string = "Hello";

// 2. 类型系统 - 编译时进行类型检查
// 3. 编译转换 - 将TS编译成JS
```

### 1.2.2 TypeScript与JavaScript的关系

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                        JavaScript (ES5+)                            │
│                             ↓                                       │
│              ┌───────────────────────────────┐                     │
│              │     TypeScript (ES6+)         │                     │
│              │  ┌─────────────────────────┐  │                     │
│              │  │ • 类型注解              │  │                     │
│              │  │ • 接口                 │  │                     │
│              │  │ • 泛型                 │  │                     │
│              │  │ • 装饰器               │  │                     │
│              │  │ • 命名空间             │  │                     │
│              │  │ • 模块系统             │  │                     │
│              │  │ • 枚举类型             │  │                     │
│              │  └─────────────────────────┘  │                     │
│              └───────────────────────────────┘                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2.3 TypeScript编译选项示例

```json
// tsconfig.json 核心配置
{
  "compilerOptions": {
    // 编译目标版本
    "target": "ES2020",
    
    // 模块系统
    "module": "commonjs",
    
    // 输出目录
    "outDir": "./dist",
    
    // 源码目录
    "rootDir": "./src",
    
    // 严格模式
    "strict": true,
    
    // 是否生成声明文件
    "declaration": true,
    
    // 是否生成sourceMap
    "sourceMap": true
  },
  // 包含的文件
  "include": ["src/**/*"],
  // 排除的文件
  "exclude": ["node_modules", "dist"]
}
```

---

## 1.3 环境搭建详解

### 1.3.1 Node.js安装

```bash
# Windows
# 1. 访问 https://nodejs.org
# 2. 下载LTS版本（长期支持版）
# 3. 运行安装程序，一路Next

# macOS
brew install node

# Linux (Ubuntu)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 1.3.2 TypeScript安装

```bash
# 全局安装（命令行使用）
npm install -g typescript

# 验证安装
tsc --version
# 输出类似：Version 5.3.3

# 本地安装（项目使用）
npm install --save-dev typescript

# 使用npx运行（无需全局安装）
npx tsc --version
```

### 1.3.3 tsconfig.json 详解

```json
{
  // ==== 基本选项 ====
  "compilerOptions": {
    // 编译目标：es3, es5, es6, es2015, es2016, es2017, es2018, es2019, es2020, es2021, esnext
    "target": "ES2020",
    
    // 模块系统：none, commonjs, amd, es6, es2015, es2020, es2022, esnext, system, umd
    "module": "commonjs",
    
    // 严格模式总开关
    "strict": true,
    
    // ==== 输出选项 ====
    "outDir": "./dist",           // 输出目录
    "rootDir": "./src",          // 源码目录
    "outFile": "./bundle.js",    // 合并输出文件
    
    // ==== 文件相关 ====
    "include": ["src/**/*"],     // 包含的文件
    "exclude": ["node_modules"],  // 排除的文件
    "files": ["src/index.ts"],   // 明确指定文件
    
    // ==== 严格检查 ====
    "strictNullChecks": true,     // 严格的null和undefined检查
    "strictPropertyInitialization": true,  // 严格的属性初始化检查
    "noImplicitAny": true,        // 不允许隐式any类型
    "noImplicitThis": true,       // 不允许this隐式any
    
    // ==== 代码质量 ====
    "noUnusedLocals": true,       // 检查未使用的局部变量
    "noUnusedParameters": true,   // 检查未使用的参数
    "noImplicitReturns": true,    // 检查不完整的返回路径
    
    // ==== 模块解析 ====
    "baseUrl": "./",              // 基础路径
    "paths": {                    // 路径别名
      "@/*": ["src/*"]
    },
    
    // ==== 实验性功能 ====
    "experimentalDecorators": true,  // 启用装饰器
    "emitDecoratorMetadata": true    // 发射装饰器元数据
  }
}
```

### 1.3.4 快速创建项目模板

```bash
# 方法一：手动创建
mkdir my-ts-project
cd my-ts-project
npm init -y
npm install -D typescript @types/node
npx tsc --init

# 方法二：使用模板（推荐）
# 推荐使用官方tsconfig模板或IDE创建

# 方法三：使用express+ts模板
npx express-ts-generator my-project
```

---

## 1.4 开发工具配置

### 1.4.1 VS Code配置

```json
// .vscode/settings.json
{
  // TypeScript配置
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.suggest.autoImports": true,
  "typescript.tsdk": "node_modules/typescript/lib",
  
  // 格式化配置
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  
  // 保存时自动格式化
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  
  // Tab大小
  "editor.tabSize": 2,
  "editor.insertSpaces": true
}
```

### 1.4.2 ESLint配置

```bash
# 安装ESLint
npm install -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin

# .eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended'
  ],
  rules: {
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/explicit-function-return-type': 'off'
  }
};
```

### 1.4.3 Prettier配置

```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "arrowParens": "avoid"
}
```

---

## 1.5 第一个TypeScript程序

### 1.5.1 完整示例

```typescript
// src/index.ts

// 定义接口
interface User {
  name: string;
  age: number;
  email?: string;  // 可选属性
}

// 定义函数
function greet(user: User): string {
  return `Hello, ${user.name}! You are ${user.age} years old.`;
}

// 创建用户对象
const user: User = {
  name: "张三",
  age: 25
};

// 调用函数
const message = greet(user);
console.log(message);
```

### 1.5.2 编译与运行

```bash
# 编译单个文件
npx tsc src/index.ts

# 编译整个项目（根据tsconfig.json）
npx tsc

# 监视模式（文件变化自动编译）
npx tsc --watch

# 运行编译后的JS文件
node dist/index.js
```

### 1.5.3 常见编译错误解读

```typescript
// 错误1：类型不匹配
let name: string = 123;
// Error: Type 'number' is not assignable to type 'string'

// 错误2：缺少必需属性
interface Config { url: string; }
const cfg: Config = {};
// Error: Property 'url' is missing

// 错误3：不能为null
let data: string = null;
// Error: Type 'null' is not assignable to type 'string'

// 错误4：数组类型错误
let nums: number[] = [1, 2, "3"];
// Error: Argument of type 'string' is not assignable to parameter of type 'number'
```

### 1.5.4 使用ts-node直接运行

```bash
# 安装ts-node
npm install -D ts-node

# 直接运行TS文件（无需手动编译）
npx ts-node src/index.ts

# 或者使用tsx（更快）
npm install -D tsx
npx tsx src/index.ts
```

---

## 📊 环境验证检查清单

```
┌─────────────────────────────────────────────────────────────────┐
│                    TypeScript 环境检查清单                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  □ Node.js已安装                                                 │
│    命令：node -v                                                 │
│                                                                 │
│  □ npm已安装                                                     │
│    命令：npm -v                                                  │
│                                                                 │
│  □ TypeScript已安装                                              │
│    命令：tsc --version                                           │
│                                                                 │
│  □ tsconfig.json已创建                                           │
│    文件位置：项目根目录                                          │
│                                                                 │
│  □ VS Code已安装                                                 │
│    推荐插件：TypeScript Importer, TSLint, Prettier               │
│                                                                 │
│  □ 第一个TS文件已成功编译                                         │
│    验证：node dist/index.js 正常运行                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⏭️ 下一步

前往 [第二章：TypeScript基础类型](./chapter-02-basic-types.md) 学习TypeScript的核心数据类型

---

> 💡 **思考题**：
> 1. TypeScript为什么能帮我们提前发现错误？
> 2. 严格模式(strict)和普通模式有什么区别？