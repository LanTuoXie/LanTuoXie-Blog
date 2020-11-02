# lib.es5.d.ts 文件

在当前文件有些方法是很有用的。

## `NaN` 和 `Infinity`

```ts
declare var NaN: number;
declare var Infinity: number;
```

## PropertyKey PropertyDescriptor

```ts
declare type PropertyKey = string | number | symbol;

interface PropertyDescriptor {
    configurable?: boolean;
    enumerable?: boolean;
    value?: any;
    writable?: boolean;
    get?(): any;
    set?(v: any): void;
}

interface PropertyDescriptorMap {
    [s: string]: PropertyDescriptor;
}
```

## typescript中一些有用的类型方法

Partial:

```ts
/**
 * Make all properties in T optional
 * 映射T所有的属性和类型
 */
type Partial<T> = {
    [P in keyof T]?: T[P];
};
```

Required:

```ts
/**
 * Make all properties in T required
 * 映射的同时且每个属性都是必须的
 */
type Required<T> = {
    [P in keyof T]-?: T[P];
};
```

Readonly:

```ts
/**
 * Make all properties in T readonly
 * 将所有属性映射为 readonly
 */
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

Pick:

```ts
/**
 * From T, pick a set of properties whose keys are in the union K
 * 选择交集的一部分去 映射
 */
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```

Record:

```ts
/**
 * Construct a type with a set of properties K of type T
 * K作为键名 T统一作为类型
 */
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

Exclude:

```ts
/**
 * Exclude from T those types that are assignable to U
 * 
 */
type Exclude<T, U> = T extends U ? never : T;
```

Extract:

```ts
/**
 * Extract from T those types that are assignable to U
 */
type Extract<T, U> = T extends U ? T : never;
```

Omit:

```ts
/**
 * Construct a type with the properties of T except for those in type K.
 */
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

NonNullable:

```ts
/**
 * Exclude null and undefined from T
 */
type NonNullable<T> = T extends null | undefined ? never : T;
```

Parameters:

```ts
/**
 * Obtain the parameters of a function type in a tuple
 */
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;
```

ConstructorParameters:

```ts
/**
 * Obtain the parameters of a constructor function type in a tuple
 */
type ConstructorParameters<T extends new (...args: any) => any> = T extends new (...args: infer P) => any ? P : never;
```

ReturnType:

```ts
/**
 * Obtain the return type of a function type
 */
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

InstanceType:

```ts
/**
 * Obtain the return type of a constructor function type
 */
type InstanceType<T extends new (...args: any) => any> = T extends new (...args: any) => infer R ? R : any;
```
