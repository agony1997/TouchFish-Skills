---
name: frontend_typescript
description: "TypeScript 型別系統專家。專精於型別定義、泛型、進階型別、介面、型別推論、裝飾器、工具型別。關鍵字: typescript, type, interface, generic, 型別, 泛型, 介面, utility types"
---

# TypeScript Expert

You are a TypeScript Expert specializing in type systems, generics, advanced types, and type-safe JavaScript development.

## Overview

TypeScript is a typed superset of JavaScript that adds static type checking. It helps catch errors early, improves IDE support, and makes code more maintainable.

## When to use this skill

Activate this skill when users:
- Work with TypeScript (關鍵字: "typescript", "ts", "type", "型別")
- Define types (關鍵字: "interface", "type", "介面", "型別定義")
- Use generics (關鍵字: "generic", "泛型", "T extends")
- Need advanced types (關鍵字: "utility types", "mapped types", "conditional types")

## Core Concepts

### 1. Basic Types

```typescript
// Primitive types
let name: string = "John";
let age: number = 30;
let isActive: boolean = true;
let nothing: null = null;
let undef: undefined = undefined;

// Arrays
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ["a", "b"];

// Tuples
let tuple: [string, number] = ["John", 30];

// Enum
enum Color {
  Red,
  Green,
  Blue
}
let color: Color = Color.Red;

// Any (avoid when possible)
let anything: any = "can be anything";

// Unknown (safer than any)
let value: unknown = "string";
if (typeof value === "string") {
  console.log(value.toUpperCase());
}

// Void
function logMessage(message: string): void {
  console.log(message);
}

// Never (function never returns)
function throwError(message: string): never {
  throw new Error(message);
}
```

### 2. Interfaces and Types

```typescript
// Interface
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;           // Optional
  readonly created: Date; // Readonly
}

// Type alias
type UserId = number | string;

type Point = {
  x: number;
  y: number;
};

// Union types
type Status = "pending" | "approved" | "rejected";

// Intersection types
type Employee = User & {
  department: string;
  salary: number;
};

// Function types
type AddFunction = (a: number, b: number) => number;

const add: AddFunction = (a, b) => a + b;
```

### 3. Generics

```typescript
// Generic function
function identity<T>(arg: T): T {
  return arg;
}

const num = identity<number>(42);
const str = identity("hello"); // Type inference

// Generic interface
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

const userResponse: ApiResponse<User> = {
  data: { id: 1, name: "John", email: "john@example.com", created: new Date() },
  status: 200,
  message: "Success"
};

// Generic class
class DataStore<T> {
  private data: T[] = [];

  add(item: T): void {
    this.data.push(item);
  }

  get(index: number): T | undefined {
    return this.data[index];
  }

  getAll(): T[] {
    return this.data;
  }
}

const numberStore = new DataStore<number>();
numberStore.add(1);
numberStore.add(2);

// Generic constraints
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

logLength("hello");      // OK
logLength([1, 2, 3]);   // OK
logLength({ length: 5 }); // OK
// logLength(123);       // Error: number doesn't have length
```

### 4. Advanced Types

```typescript
// Mapped types
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

// Conditional types
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false

// Extract/Exclude
type T1 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T2 = Exclude<"a" | "b" | "c", "a" | "f">;  // "b" | "c"

// ReturnType
function getUser() {
  return { id: 1, name: "John" };
}

type UserReturnType = ReturnType<typeof getUser>;
// { id: number; name: string; }

// Parameters
type GetUserParams = Parameters<typeof getUser>;
```

### 5. Utility Types

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Partial - all properties optional
type PartialUser = Partial<User>;

// Required - all properties required
type RequiredUser = Required<User>;

// Readonly - all properties readonly
type ReadonlyUser = Readonly<User>;

// Pick - select properties
type UserPreview = Pick<User, "id" | "name">;

// Omit - exclude properties
type UserWithoutPassword = Omit<User, "password">;

// Record
type UserRoles = Record<string, string[]>;
const roles: UserRoles = {
  admin: ["read", "write", "delete"],
  user: ["read"]
};

// NonNullable
type NullableString = string | null | undefined;
type NonNullString = NonNullable<NullableString>; // string
```

### 6. Type Guards

```typescript
// typeof type guard
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + value;
  }
  return padding + value;
}

// instanceof type guard
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}

// Custom type guard
interface Bird {
  fly(): void;
}

interface Fish {
  swim(): void;
}

function isBird(pet: Bird | Fish): pet is Bird {
  return (pet as Bird).fly !== undefined;
}

function move(pet: Bird | Fish) {
  if (isBird(pet)) {
    pet.fly();
  } else {
    pet.swim();
  }
}
```

### 7. Decorators

```typescript
// Class decorator
function Component(target: Function) {
  target.prototype.selector = "app-component";
}

@Component
class MyComponent {}

// Method decorator
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    return originalMethod.apply(this, args);
  };

  return descriptor;
}

class Calculator {
  @Log
  add(a: number, b: number): number {
    return a + b;
  }
}

// Property decorator
function Required(target: any, propertyKey: string) {
  let value: any;

  const getter = () => value;
  const setter = (newValue: any) => {
    if (!newValue) {
      throw new Error(`${propertyKey} is required`);
    }
    value = newValue;
  };

  Object.defineProperty(target, propertyKey, {
    get: getter,
    set: setter
  });
}

class User {
  @Required
  email: string;
}
```

### 8. Async/Await with Types

```typescript
async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  return data as User;
}

async function getUsers(): Promise<User[]> {
  try {
    const users = await fetchUser(1);
    return [users];
  } catch (error) {
    console.error(error);
    return [];
  }
}
```

## Best Practices

### 1. Prefer Interfaces for Objects
```typescript
// Good
interface User {
  id: number;
  name: string;
}

// Also good for unions/primitives
type Status = "active" | "inactive";
```

### 2. Use Type Inference
```typescript
// Let TypeScript infer
const numbers = [1, 2, 3]; // number[]
const user = { name: "John", age: 30 }; // { name: string; age: number }

// Explicit when needed
const items: number[] = [];
```

### 3. Avoid Any
```typescript
// Bad
function process(data: any) { }

// Good
function process<T>(data: T) { }
// or
function process(data: unknown) { }
```

### 4. Use Strict Mode
```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true
  }
}
```

### 5. Type vs Interface
```typescript
// Use interface for objects that can be extended
interface User {
  name: string;
}

interface Employee extends User {
  salary: number;
}

// Use type for unions, intersections, utilities
type ID = string | number;
type Coordinates = [number, number];
```

## Quick Reference

### Basic Syntax
```typescript
let name: string;
const age: number = 30;
function greet(name: string): string { return "Hello"; }
const arrow = (x: number): number => x * 2;
```

### Common Types
```typescript
string, number, boolean, null, undefined
any, unknown, never, void
Array<T>, T[], [string, number] (tuple)
{ key: string } (object)
(arg: T) => R (function)
```

### Type Operations
```typescript
T | U         // Union
T & U         // Intersection
T extends U   // Constraint
keyof T       // Keys of T
typeof x      // Type of x
T[K]          // Index access
```

### Utility Types
```typescript
Partial<T>, Required<T>, Readonly<T>
Pick<T, K>, Omit<T, K>
Record<K, T>, Exclude<T, U>, Extract<T, U>
NonNullable<T>, ReturnType<T>, Parameters<T>
```

---

**Remember:** TypeScript is about catching errors at compile time. Invest time in proper typing for better code quality and developer experience.
