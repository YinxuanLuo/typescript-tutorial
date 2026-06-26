# 第九章：TypeScript编译配置

## 📋 目录

- [9.1 tsconfig.json基础](#91-tsconfigjson基础)
- [9.2 compilerOptions详解](#92-compileroptions详解)
- [9.3 项目引用和构建](#93-项目引用和构建)
- [9.4 严格模式](#94-严格模式)
- [9.5 JSX配置](#95-jsx配置)
- [9.6 声明文件配置](#96-声明文件配置)

---

## 9.1 tsconfig.json基础

### 9.1.1 基本配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### 9.1.2 配置继承

```json
// tsconfig.base.json - 基础配置
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node"
  }
}

// tsconfig.json - 继承并扩展
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist"
  },
  "include": ["src"]
}

// tsconfig.test.json - 测试配置
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "target": "ES2019",
    "types": ["jest", "node"]
  },
  "include": ["tests"]
}
```

### 9.1.3 配置查找规则

```
┌─────────────────────────────────────────────────────────────────────┐
│                     TypeScript配置查找顺序                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   1. 命令行 --project 参数指定的文件                                 │
│   2. 当前目录的 tsconfig.json                                        │
│   3. 向上查找父目录的 tsconfig.json                                  │
│   4. 无配置文件使用默认配置                                          │
│                                                                     │
│   注意：tsconfig.json 会在以下目录查找：                             │
│   • 包含 tsconfig.json 的目录                                       │
│   • 父目录（逐级向上）                                              │
│   • 直到找到或到达根目录                                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 9.2 compilerOptions详解

### 9.2.1 输出选项

```json
{
  "compilerOptions": {
    // 编译目标：es3, es5, es6, es2015, es2016, ..., es2022, esnext
    "target": "ES2020",
    
    // 模块系统：none, commonjs, amd, es6, es2015, es2020, es2022, esnext, system, umd
    "module": "commonjs",
    
    // 输出目录
    "outDir": "./dist",
    
    // 源码目录
    "rootDir": "./src",
    
    // 合并输出到单个文件（与outDir互斥）
    "outFile": "./bundle.js",
    
    // 是否生成声明文件
    "declaration": true,
    
    // 声明文件输出目录
    "declarationDir": "./types",
    
    // 是否生成sourceMap
    "sourceMap": true,
    
    // 是否生成 .d.ts.map
    "declarationMap": true,
    
    // 是否移除注释
    "removeComments": false,
    
    // 不输出文件，只做类型检查
    "noEmit": false,
    
    // 导入帮助函数的方式：classic 或 automatic
    "importHelpers": true
  }
}
```

### 9.2.2 严格检查选项

```json
{
  "compilerOptions": {
    // 严格模式总开关
    "strict": true,
    
    // 所有严格检查的总开关，启用后相当于：
    // strictNullChecks, strictPropertyInitialization,
    // noImplicitAny, noImplicitThis, alwaysStrict,
    // strictBindCallApply, strictFunctionTypes
    
    // 严格的null和undefined检查
    "strictNullChecks": true,
    
    // 严格的类属性初始化检查
    "strictPropertyInitialization": true,
    
    // 不允许隐式any类型
    "noImplicitAny": true,
    
    // 不允许this隐式any
    "noImplicitThis": true,
    
    // 使用严格的strict模式处理每个文件
    "alwaysStrict": true,
    
    // 严格的bind/call/apply检查
    "strictBindCallApply": true,
    
    // 严格的函数类型检查
    "strictFunctionTypes": true,
    
    // 解构时严格检查属性
    "noPropertyAccessFromIndexSignature": true
  }
}
```

### 9.2.3 代码质量选项

```json
{
  "compilerOptions": {
    // 检查未使用的局部变量
    "noUnusedLocals": true,
    
    // 检查未使用的参数
    "noUnusedParameters": true,
    
    // 检查不完整的返回语句
    "noImplicitReturns": true,
    
    // 检查switch语句的fall-through
    "noFallthroughCasesInSwitch": true,
    
    // 检查无法到达的代码
    "allowUnreachableCode": false,
    
    // 检查标记为undefined的标签
    "noUnusedLocals": true,
    
    // 报告未使用的导入
    "noUnusedParameters": true
  }
}
```

### 9.2.4 模块解析选项

```json
{
  "compilerOptions": {
    // 模块解析策略：node (classic)
    "moduleResolution": "node",
    
    // 基础路径
    "baseUrl": "./src",
    
    // 路径映射
    "paths": {
      "@/*": ["./*"],
      "@components/*": ["components/*"],
      "@utils/*": ["utils/*"]
    },
    
    // 类型根目录
    "typeRoots": ["./node_modules/@types", "./types"],
    
    // 自动引入的类型包
    "types": ["node", "jest"],
    
    // 允许导入json模块
    "resolveJsonModule": true,
    
    // 允许导入.css, .png等非模块
    "allowSyntheticDefaultImports": true,
    
    // esModuleInterop
    "esModuleInterop": true
  }
}
```

---

## 9.3 项目引用和构建

### 9.3.1 项目引用

```json
// tsconfig.json - 引用者
{
  "references": [
    { "path": "./utils" },
    { "path": "./components" }
  ]
}
```

```json
// utils/tsconfig.json
{
  "compilerOptions": {
    "composite": true,      // 启用项目引用
    "declaration": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*"]
}
```

```json
// components/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*"],
  "references": [
    { "path": "../utils" }
  ]
}
```

### 9.3.2 构建模式

```bash
# 启用构建模式（需要composite: true）
tsc --build

# 或者
npx tsc -b

# 增量构建（只重新编译更改的文件）
tsc -b --verbose

# 清理构建输出
tsc -b --clean

# 强制完整构建
tsc -b --force
```

### 9.3.3 monorepo配置示例

```
my-monorepo/
├── package.json
├── tsconfig.base.json
│
├── packages/
│   ├── shared/
│   │   ├── tsconfig.json
│   │   └── src/
│   │
│   ├── utils/
│   │   ├── tsconfig.json
│   │   └── src/
│   │
│   └── app/
│       ├── tsconfig.json
│       └── src/
│
└── tsconfig.json (根配置)
```

```json
// tsconfig.base.json
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "composite": true
  }
}

// packages/shared/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist"
  },
  "include": ["src/**/*"]
}

// packages/app/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist"
  },
  "include": ["src/**/*"],
  "references": [
    { "path": "../shared" },
    { "path": "../utils" }
  ]
}
```

---

## 9.4 严格模式

### 9.4.1 严格模式详解

```typescript
// strict: true 启用以下所有检查

// 1. strictNullChecks
function getUser(id: string): User | null {
  const user = db.find(id);
  return user;  // 可能返回null
}

const user = getUser("123");
// user.name  // Error: 'user' is possibly 'null'
if (user !== null) {
  console.log(user.name);  // OK
}

// 2. noImplicitAny
// let value = 123;  // Error: 需要显式类型
let value: number = 123;  // OK

// 3. strictPropertyInitialization
class User {
  name: string;  // Error: 未初始化
  // name: string = "";  // OK
}

// 4. noImplicitThis
function greet() {
  // console.log(this.name);  // Error: 'this' implicitly has type 'any'
}
```

### 9.4.2 迁移到严格模式

```typescript
// 分步骤迁移

// 步骤1: 启用noImplicitAny
// 逐步为所有变量添加类型注解

// 步骤2: 启用strictNullChecks
// 使用可选链、null检查、?.操作符

function processUser(user: User | null) {
  if (user) {
    console.log(user.name);  // OK，在null检查后
    console.log(user.address?.city);  // OK，可选链
  }
}

// 步骤3: 处理strictPropertyInitialization
class User {
  name: string;
  email: string;
  
  constructor(name: string, email: string) {
    this.name = name;
    this.email = email;
  }
}

// 步骤4: 启用所有严格检查
// 逐步修复剩余问题
```

---

## 9.5 JSX配置

### 9.5.1 JSX选项

```json
{
  "compilerOptions": {
    // JSX处理模式
    "jsx": "react-jsx",  // react | react-jsx | react-native | preserve
    
    // JSX工厂函数
    "jsxFactory": "h",
    
    // JSX Fragment工厂
    "jsxFragmentFactory": "Fragment",
    
    // JSX输出目录（用于preserve模式）
    "outDir": "./dist"
  }
}
```

| 选项 | 说明 |
|------|------|
| `react` | React.createElement方式，需要编译 |
| `react-jsx` | 新的JSX转换，不需要React导入 |
| `react-native` | 保留JSX，输出到文件 |
| `preserve` | 保留JSX，输出到文件 |

### 9.5.2 JSX类型

```typescript
// React 17+ 的JSX类型
import { JSX } from "react";

function MyComponent(): JSX.Element {
  return <div>Hello</div>;
}

// 泛型组件类型
function GenericComponent<T>(props: { items: T[] }): JSX.Element {
  return (
    <ul>
      {props.items.map(item => (
        <li key={item.toString()}>{item}</li>
      ))}
    </ul>
  );
}

// Vue 3的JSX类型
import { VNode } from "vue";

function render(): VNode {
  return <div>Hello</div>;
}
```

---

## 9.6 声明文件配置

### 9.6.1 声明文件选项

```json
{
  "compilerOptions": {
    // 生成声明文件
    "declaration": true,
    
    // 生成声明映射文件
    "declarationMap": true,
    
    // 声明文件输出目录
    "declarationDir": "./types",
    
    // 生成 .d.ts 时同时生成 sourceMap
    "declarationSourceMap": true,
    
    // 仅生成声明文件，不生成js
    "emitDeclarationOnly": true,
    
    // 最大声明文件数量（防止过多文件）
    "maxNodeModuleJsDepth": 0
  }
}
```

### 9.6.2 生成声明文件示例

```bash
# 编译并生成声明文件
tsc --declaration --emitDeclarationOnly

# 输出
# dist/
#   ├── index.js
#   ├── index.d.ts
#   └── index.d.ts.map
```

---

## 📊 配置速查表

```
┌────────────────────────────────┬─────────────────────────────────────┐
│ 选项                           │ 说明                                │
├────────────────────────────────┼─────────────────────────────────────┤
│ target                         │ 编译目标版本                         │
│ module                         │ 模块系统                            │
│ strict                         │ 严格模式                            │
│ outDir                         │ 输出目录                            │
│ rootDir                        │ 源码目录                            │
│ include/exclude                │ 包含/排除文件                        │
│ references                     │ 项目引用                            │
│ extends                        │ 配置继承                            │
│ jsx                            │ JSX处理模式                         │
│ declaration                    │ 生成声明文件                        │
│ sourceMap                      │ 生成sourcemap                      │
│ moduleResolution              │ 模块解析策略                        │
│ baseUrl/paths                  │ 路径别名配置                        │
│ noEmit                         │ 仅类型检查                          │
│ esModuleInterop                │ ES模块互操作                        │
│ allowSyntheticDefaultImports   │ 允许默认导入                        │
│ skipLibCheck                   │ 跳过库检查                          │
└────────────────────────────────┴─────────────────────────────────────┘
```

---

## ⏭️ 下一步

前往 [第十章：实际项目最佳实践](./chapter-10-best-practices.md) 学习项目实战经验

---

> 💡 **思考题**：
> 1. tsconfig.json的extends字段有什么作用？
> 2. 什么是strict模式？它包含哪些检查？
> 3. 项目引用（references）有什么优势？