# 第十章：实际项目最佳实践

## 📋 目录

- [10.1 项目结构设计](#101-项目结构设计)
- [10.2 类型设计原则](#102-类型设计原则)
- [10.3 代码组织规范](#103-代码组织规范)
- [10.4 与框架集成](#104-与框架集成)
- [10.5 测试最佳实践](#105-测试最佳实践)
- [10.6 性能优化](#106-性能优化)

---

## 10.1 项目结构设计

### 10.1.1 标准项目结构

```
my-app/
├── src/
│   ├── index.ts                    # 应用入口
│   ├── App.tsx                    # 根组件
│   │
│   ├── assets/                    # 静态资源
│   │   ├── images/
│   │   └── styles/
│   │
│   ├── components/                # 通用组件
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.css
│   │   │   ├── Button.test.tsx
│   │   │   └── index.ts
│   │   └── index.ts
│   │
│   ├── features/                  # 功能模块（DDD）
│   │   ├── auth/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   ├── services/
│   │   │   ├── types/
│   │   │   └── auth.ts
│   │   │
│   │   └── users/
│   │       ├── components/
│   │       ├── hooks/
│   │       ├── services/
│   │       └── types/
│   │
│   ├── hooks/                    # 共享hooks
│   │   ├── useAuth.ts
│   │   ├── useFetch.ts
│   │   └── index.ts
│   │
│   ├── services/                 # API服务
│   │   ├── api.ts               # axios配置
│   │   ├── auth.service.ts
│   │   └── user.service.ts
│   │
│   ├── stores/                   # 状态管理
│   │   ├── user.store.ts
│   │   └── index.ts
│   │
│   ├── types/                    # 全局类型
│   │   ├── api.ts               # API类型
│   │   ├── user.ts              # 用户类型
│   │   └── index.ts
│   │
│   ├── utils/                    # 工具函数
│   │   ├── format.ts
│   │   ├── validation.ts
│   │   └── index.ts
│   │
│   └── constants/                # 常量
│       └── index.ts
│
├── tests/                        # 测试
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── docs/                         # 文档
├── package.json
├── tsconfig.json
└── README.md
```

### 10.1.2 Barrel导出模式

```typescript
// src/components/index.ts
export { Button } from "./Button";
export { Input } from "./Input";
export { Modal } from "./Modal";
export { Select } from "./Select";

// src/types/index.ts
export type { User, UserRole } from "./user";
export type { ApiResponse, PaginatedResponse } from "./api";
export type { ValidationRule, Validator } from "./validation";

// src/hooks/index.ts
export { useAuth } from "./useAuth";
export { useAsync } from "./useAsync";
export { useForm } from "./useForm";
```

### 10.1.3 Feature-Based结构

```typescript
// 按功能组织，每个功能有完整的自包含结构
// features/auth/

// features/auth/types/
export interface LoginCredentials {
  email: string;
  password: string;
}

export interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  loading: boolean;
}

// features/auth/hooks/
export function useAuth() {
  const { user, login, logout, isAuthenticated } = useAuthStore();
  return { user, login, logout, isAuthenticated };
}

// features/auth/components/
export { LoginForm } from "./LoginForm";
export { AuthProvider } from "./AuthProvider";

// features/auth/services/
export class AuthService {
  async login(credentials: LoginCredentials): Promise<AuthResponse> {
    // 实现
  }
}

// features/auth/index.ts
export * from "./types";
export * from "./hooks";
export * from "./components";
export { AuthService } from "./services";
```

---

## 10.2 类型设计原则

### 10.2.1 类型分层设计

```typescript
// 1. Domain层 - 核心业务类型
// types/domain/
interface User {
  id: string;
  name: string;
  email: Email;
  createdAt: Date;
  updatedAt: Date;
}

interface Order {
  id: string;
  userId: string;
  items: OrderItem[];
  status: OrderStatus;
  totalAmount: Money;
  createdAt: Date;
}

// 2. Application层 - 应用特定类型
// types/application/
interface CreateOrderDTO {
  userId: string;
  items: { productId: string; quantity: number }[];
}

interface UpdateUserDTO {
  name?: string;
  email?: Email;
}

interface OrderFilter {
  status?: OrderStatus;
  userId?: string;
  dateFrom?: Date;
  dateTo?: Date;
}

// 3. Infrastructure层 - 外部系统类型
// types/infrastructure/
interface ApiResponse<T> {
  code: number;
  message: string;
  data: T;
  timestamp: number;
}

interface DatabaseRow {
  [key: string]: any;
}
```

### 10.2.2 避免any，使用unknown

```typescript
// ❌ 不推荐：使用any
function processData(data: any): any {
  return data.foo.bar;
}

// ✅ 推荐：使用unknown + 类型守卫
function processData(data: unknown): ProcessedData {
  if (!isValidData(data)) {
    throw new Error("Invalid data");
  }
  return transformData(data);
}

function isValidData(data: unknown): data is ValidData {
  return (
    typeof data === "object" &&
    data !== null &&
    "foo" in data &&
    typeof (data as any).foo === "object"
  );
}
```

### 10.2.3 使用Branded Types防止混淆

```typescript
// 防止类型混淆的branded types
type UserId = string & { readonly __brand: "UserId" };
type OrderId = string & { readonly __brand: "OrderId" };

function createUserId(id: string): UserId {
  return id as UserId;
}

function createOrderId(id: string): OrderId {
  return id as OrderId;
}

type Money = number & { readonly __brand: "Money" };
type Percentage = number & { readonly __brand: "Percentage" };

function createMoney(amount: number): Money {
  return amount as Money;
}

function createPercentage(value: number): Percentage {
  return (value % 100) as Percentage;
}

// 使用
const userId = createUserId("123");
const orderId = createOrderId("456");
const money = createMoney(100);
const percentage = createPercentage(50);

// 防止混淆
// userId === orderId  // compile error!
// money + percentage  // compile error!
```

### 10.2.4 善用const断言

```typescript
// const断言使类型更精确
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retryCount: 3
} as const;

// 类型为：
// {
//   readonly apiUrl: "https://api.example.com";
//   readonly timeout: 5;
//   readonly retryCount: 3;
// }

// 用于配置对象
const routes = ["/", "/about", "/contact"] as const;
type Route = typeof routes[number];  // "/" | "/about" | "/contact"

// 用于状态
const statusOptions = [
  { value: "pending", label: "待处理", color: "gray" },
  { value: "success", label: "成功", color: "green" },
  { value: "error", label: "错误", color: "red" }
] as const;

type StatusOption = typeof statusOptions[number];
type StatusValue = StatusOption["value"];
```

---

## 10.3 代码组织规范

### 10.3.1 类型就近原则

```typescript
// 类型定义应该离使用它的地方最近

// ✅ 推荐：组件相关类型放在组件文件内
import React, { useState } from "react";

interface ButtonProps {
  variant: "primary" | "secondary" | "danger";
  size?: "small" | "medium" | "large";
  children: React.ReactNode;
  onClick?: () => void;
}

export function Button({ variant, size = "medium", children, onClick }: ButtonProps) {
  return <button className={`btn btn-${variant} btn-${size}`}>{children}</button>;
}

// ❌ 不推荐：把所有类型放一个文件
// types/index.ts
// interface ButtonProps { ... }
```

### 10.3.2 类型命名规范

```typescript
// 1. 接口和类型别名
interface User { ... }           // 普通对象
type UserId = string;           // 简单类型alias
type UserList = User[];         // 集合类型
type UserPredicate = (user: User) => boolean;  // 函数类型

// 2. 带修饰的类型
type Optional<T> = T | undefined;
type Nullable<T> = T | null;
type AsyncResult<T, E = Error> = Promise<Result<T, E>>;

// 3. DTO和事件类型
type CreateUserDTO = { ... };
type UpdateUserDTO = { ... };
type UserCreatedEvent = { ... };
type UserDeletedEvent = { ... };

// 4. 状态类型
type UserState = { ... };
type AuthStatus = "idle" | "loading" | "authenticated" | "error";
```

### 10.3.3 导入排序规范

```typescript
// 1. React相关
import React, { useState, useEffect } from "react";

// 2. 第三方库
import { useRouter } from "next/router";
import { useForm } from "react-hook-form";
import { Button } from "@/components";

// 3. 内部模块
import { useAuth } from "@/hooks";
import { userService } from "@/services";
import { UserCard, UserList } from "./components";

// 4. 类型导入（使用显式type导入）
import type { User, CreateUserDTO } from "@/types";
import type { AxiosRequestConfig } from "axios";

// 5. 样式和资源
import "./UserPage.css";
import userIcon from "./user.png";
```

---

## 10.4 与框架集成

### 10.4.1 React + TypeScript

```typescript
// components/Button.tsx
import React from "react";

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "secondary" | "danger";
  size?: "small" | "medium" | "large";
  isLoading?: boolean;
  children: React.ReactNode;
}

export const Button: React.FC<ButtonProps> = ({
  variant = "primary",
  size = "medium",
  isLoading = false,
  children,
  disabled,
  ...props
}) => {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled || isLoading}
      {...props}
    >
      {isLoading ? "Loading..." : children}
    </button>
  );
};

// hooks/useUsers.ts
import { useState, useEffect } from "react";
import { userService } from "@/services";
import type { User } from "@/types";

export function useUsers() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    userService.getAll()
      .then(setUsers)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);

  return { users, loading, error };
}

// generic组件
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
}

export function List<T>({ items, renderItem }: ListProps<T>) {
  return <ul>{items.map(renderItem)}</ul>;
}
```

### 10.4.2 Vue 3 + TypeScript

```typescript
// components/UserCard.vue
<script setup lang="ts">
import { defineProps, computed } from "vue";

interface Props {
  user: {
    id: string;
    name: string;
    email: string;
  };
  showEmail?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  showEmail: false
});

const displayText = computed(() => 
  props.showEmail ? `${props.user.name} (${props.user.email})` : props.user.name
);
</script>

// composables/useAuth.ts
import { ref, computed } from "vue";

interface User {
  id: string;
  name: string;
}

const user = ref<User | null>(null);
const isAuthenticated = computed(() => user.value !== null);

export function useAuth() {
  const login = async (email: string, password: string) => {
    // 实现登录
  };

  const logout = () => {
    user.value = null;
  };

  return {
    user,
    isAuthenticated,
    login,
    logout
  };
}
```

### 10.4.3 Node.js + TypeScript

```typescript
// src/services/user.service.ts
import { UserRepository } from "../repositories";
import type { User, CreateUserDTO, UpdateUserDTO } from "../types";

export class UserService {
  constructor(private userRepo: UserRepository) {}

  async getUserById(id: string): Promise<User | null> {
    return this.userRepo.findById(id);
  }

  async createUser(dto: CreateUserDTO): Promise<User> {
    const existing = await this.userRepo.findByEmail(dto.email);
    if (existing) {
      throw new Error("User with this email already exists");
    }
    return this.userRepo.create(dto);
  }

  async updateUser(id: string, dto: UpdateUserDTO): Promise<User> {
    return this.userRepo.update(id, dto);
  }
}

// src/middleware/auth.ts
import { Request, Response, NextFunction } from "express";

export interface AuthRequest extends Request {
  userId?: string;
  userRole?: string;
}

export function authMiddleware(
  req: AuthRequest,
  res: Response,
  next: NextFunction
): void {
  const token = req.headers.authorization?.split(" ")[1];
  
  if (!token) {
    res.status(401).json({ error: "Unauthorized" });
    return;
  }

  try {
    const decoded = verifyToken(token);
    req.userId = decoded.userId;
    req.userRole = decoded.role;
    next();
  } catch {
    res.status(401).json({ error: "Invalid token" });
  }
}
```

---

## 10.5 测试最佳实践

### 10.5.1 类型化的测试

```typescript
// __tests__/user.service.test.ts
import { describe, it, expect, vi } from "vitest";
import { UserService } from "../../services/user.service";
import { UserRepository } from "../../repositories/user.repository";
import type { User, CreateUserDTO } from "../../types";

describe("UserService", () => {
  const mockRepository: UserRepository = {
    findById: vi.fn(),
    findByEmail: vi.fn(),
    findAll: vi.fn(),
    create: vi.fn(),
    update: vi.fn(),
    delete: vi.fn(),
  };

  const service = new UserService(mockRepository);

  describe("getUserById", () => {
    it("should return user when found", async () => {
      const mockUser: User = {
        id: "1",
        name: "张三",
        email: "zhangsan@example.com",
        createdAt: new Date(),
        updatedAt: new Date(),
      };

      vi.mocked(mockRepository.findById).mockResolvedValue(mockUser);

      const result = await service.getUserById("1");

      expect(result).toEqual(mockUser);
      expect(mockRepository.findById).toHaveBeenCalledWith("1");
    });

    it("should return null when user not found", async () => {
      vi.mocked(mockRepository.findById).mockResolvedValue(null);

      const result = await service.getUserById("nonexistent");

      expect(result).toBeNull();
    });
  });
});
```

### 10.5.2 类型守卫测试

```typescript
// utils/validation.test.ts
import { describe, it, expect } from "vitest";
import { isValidEmail, isNonEmptyString, isPositiveNumber } from "../validation";

describe("Validation", () => {
  describe("isValidEmail", () => {
    it("should return true for valid emails", () => {
      expect(isValidEmail("test@example.com")).toBe(true);
      expect(isValidEmail("user.name@domain.co.uk")).toBe(true);
    });

    it("should return false for invalid emails", () => {
      expect(isValidEmail("invalid")).toBe(false);
      expect(isValidEmail("missing@")).toBe(false);
      expect(isValidEmail("@domain.com")).toBe(false);
    });
  });

  describe("Type guards", () => {
    it("should narrow type correctly", () => {
      const value: string | number = "hello";

      if (isNonEmptyString(value)) {
        // TypeScript knows value is string here
        expect(value.length).toBeGreaterThan(0);
      }
    });
  });
});
```

---

## 10.6 性能优化

### 10.6.1 类型编译优化

```json
{
  "compilerOptions": {
    "skipLibCheck": true,           // 跳过库检查，加速编译
    "incremental": true,            // 增量编译
    "tsBuildInfoFile": ".tsbuildinfo"  // 构建缓存
  }
}
```

### 10.6.2 运行时类型优化

```typescript
// ❌ 不推荐：每次渲染都重新创建对象
function Component({ items }: { items: User[] }) {
  const userMap = new Map(items.map(u => [u.id, u]));  // 每次渲染重新创建
  
  return <div>{/* ... */}</div>;
}

// ✅ 推荐：使用useMemo缓存
function Component({ items }: { items: User[] }) {
  const userMap = useMemo(
    () => new Map(items.map(u => [u.id, u])),
    [items]
  );
  
  return <div>{/* ... */}</div>;
}

// ❌ 不推荐：频繁类型转换
function processData(data: any) {
  return (data as User).id;
}

// ✅ 推荐：减少不必要的类型转换
function processData(data: unknown) {
  if (isUser(data)) {
    return data.id;  // 类型已收缩为User
  }
  throw new Error("Invalid data");
}
```

### 10.6.3 类型体操优化

```typescript
// 避免过于复杂的类型计算
// 类型计算发生在编译时，但过于复杂的类型会影响编译速度

// ❌ 不推荐：深层递归类型
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object 
    ? DeepReadonly<T[K]> 
    : T[K];
};

// ✅ 替代：提供浅层版本和深层版本
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

// 或者限制递归深度
```

---

## 📊 项目实践清单

```
┌─────────────────────────────────────────────────────────────────────┐
│                     TypeScript项目检查清单                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  配置项                                                              │
│  □ tsconfig.json 配置完整                                            │
│  □ 启用了 strict 模式                                               │
│  □ 配置了路径别名 (@/*)                                             │
│  □ 配置了 skipLibCheck 优化编译                                      │
│                                                                     │
│  代码组织                                                            │
│  □ 合理使用模块和 Barrel 文件                                        │
│  □ 类型定义就近原则                                                 │
│  □ 接口和类型别名 命名规范                                           │
│  □ 统一使用命名导出                                                  │
│                                                                     │
│  类型设计                                                            │
│  □ 避免使用 any，使用 unknown 代替                                   │
│  □ 善用 const 断言                                                  │
│  □ 使用 branded types 防止类型混淆                                   │
│  □ 合理设计类型层次（domain/application/infrastructure）             │
│                                                                     │
│  测试                                                                │
│  □ 编写类型化的单元测试                                             │
│  □ 类型守卫有对应的测试                                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ⏭️ 下一步

前往 [附录：常见问题与解决方案](./appendix/FAQ.md) 查看FAQ

---

> 💡 **思考题**：
> 1. 你的项目中如何组织类型定义？
> 2. 如何避免使用any同时保持代码灵活性？
> 3. 什么是branded types？有什么实际应用？