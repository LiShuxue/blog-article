## noImplicitAny 和 strictNullChecks

启用 noImplicitAny 选项后，当类型被隐式推断为 any 时，会抛出一个错误。

启用 strictNullChecks 选项会让我们更明确的处理 null 和 undefined，也会让我们免于忧虑是否忘记处理 null 和 undefined 。

## 常见类型

string，number，boolean，number[] (或者 `Array<number>`)

声明变量的时候：你可以选择性的添加一个类型注解，显式指定变量的类型，不过大部分时候，这不是必须的。

声明函数的时候：需要在每个参数后面添加一个类型注解。函数的返回值类型可以不声明，会自动推断。

## 对象类型 和 联合类型

```js
type a = {
  x: number,
  y: number,
};

type b = string | number;
```

## 字面量类型

字符串或者数字等字面量，也可以用来表示类型。表示只能是这些字面量中的一个。

```js
type A = 'left' | 'right' | 'center';
type B = 1 | 0 | -1;
```

## 类型中使用模板

我们可以在字面量类型声明时使用模板字符串。

```js
type World = "world";

type Greeting = `hello ${World}`;
// type Greeting = "hello world"
```

如果模板字面量里的多个变量都是联合类型，结果会交叉相乘。

```js
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
// type AllLocaleIDs = "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"
```

## never 类型

表示一个不可能存在的状态。意味着你可以在 switch 语句中使用 never 来做一个穷尽检查。

## 类型别名 type 和 接口 interface

类型别名和接口非常相似，大部分时候，你可以任意选择使用。接口的几乎所有特性都可以在 type 中使用，两者最关键的差别在于类型别名本身无法添加新的属性，而接口是可以扩展的。

## 类型断言

可以使用 as 或者 `<>`，在 jsx 中使用 as

```js
let a = something as A;

let b = <A>something;
```

## 非空断言操作符（后缀 !）

在任意表达式后面写上 `!` ，这是一个有效的类型断言，表示它的值不可能是 null 或者 undefined

## & 交叉类型

用于合并已经存在的对象类型，如果成员不一致，取成员的并集，如果某个成员一致，这个成员取他们类型的交集。

```js
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
type ColorfulCircle = Colorful & Circle;
// color: string; radius: number;

type a = string & number; // 即为 string 类型又为 number，即为 never 类型

type b = string & keyof Type; // keyof 操作符会返回 string | number | symbol 类型，所以最终结果是string
```

## 类型收窄

一个参数可能是多种类型，我们通过 if/else 将每个分支中的参数限定为某种类型。这种类型推导为更精确类型的过程，我们称之为收窄 (narrowing)。 在编辑器中，鼠标移上去我们是可以观察到类型的改变的。

- 可以使用 js 的 typeof 操作符，判断某个类型（只能判断基本类型和函数，数组对象和 null 都是 object），将参数限定为某一种类型

- 可以先判断 null 和 undefiend，将类型限定为真值。

- js 的 in 操作符可以判断一个对象是否有对应的属性名。

- instanceof 判断为某一种类型。

## 函数表达式

用来作为函数的注解。

void 表示一个函数并不会返回任何值，当函数并没有任何返回值，或者返回不了明确的值的时候，就应该用这种类型。

```js
type typeF = (s: string) => void;
```

## 函数调用签名 和 函数构造签名

`type typeF = (s: string) => void` 可以用来作为函数的类型，也可以定义在 interface 或者 type 中，就叫函数调用签名。

函数调用签名类似于函数表达式，中间用冒号而不是箭头。

函数构造签名类似于调用签名，但是需要加 new。

```js
// typeF 声明的函数既可以被调用，也可以被用来new
type typeF = {
  (s: string): void, // 调用签名
  new(s: string): SomeObject, // 构造签名
};
```

## 泛型函数

当函数的输出类型依赖函数的输入类型，或者两个输入的类型以某种形式相互关联。需要在函数的签名里声明一个泛型参数 T，比如下面的 name 后面，可以有多个。

如果一个类型参数仅仅出现在一个地方，强烈建议你重新考虑是否真的需要它。

```js
fucntion name<T> (arr: T[]): T {

}
```

## 泛型约束

在声明泛型的时候，使用 extends 来约束泛型必须是某种类型的，或者包含某些属性的。

```js
fucntion name<T extends { length: number }> (a: T, b: T) {

}

function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}
```

## 泛型接口，泛型类

泛型也可以用在接口和类上，或者类型别名 type 上。

```js
interface Box<Type> {
  contents: Type;
}

class GenericNumber<NumType> {
  zeroValue: NumType;
  add: (x: NumType, y: NumType) => NumType;
}

type Test<Type> {

}
```

## 函数重载

尽可能的使用联合类型替代重载。

## 任意属性和索引签名

对象类型中可以用 `[propName: string]` 声明任意属性。

定义数组时可以用 `[index: number]` 表示数组索引。定义在 interface 或者 type 中时，就叫索引签名。

```js
interface Person {
    name: string
    age?: number
    [propName: string]: any // 定义了一个任意属性，其值也是任意类型的
}
interface StringArray {
   [index: number]: string; // 这种形式就叫索引签名，类似前面的函数签名和构造签名
}
```

## 通过索引访问类型

通过索引访问某一个类型。对于对象类型，索引就是某个 key，对于数组类型，索引就是`[number]`

类似取对象中某个 key 的值一样，对一个对象类型的 key 取值，获取某个 key 的类型。可以结合联合类型或者 keyof 使用。

```js
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"];
// type Age = number

type I1 = Person["age" | "name"];
// type I1 = string | number

type I2 = Person[keyof Person];
// type I2 = string | number | boolean
```

作为索引的只能是类型，不能是值。所以下面是错误的，但换成类型别名可以。

```js
const key = 'age';
type Age = Person[key]; // 错误

type key = 'age';
type Age = Person[key]; // 正确
```

使用 typeof arr[number]获取数组元素的类型，对于普通数组，可以使用 as const 将其变成 readonly 的类型字面量。

先执行 typeof，获取类型，再通过[number]索引访问类型。

```js
const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];

type Person = typeof MyArray[number];

// type Person = {
//    name: string;
//    age: number;
// }

const APP = ['TaoBao', 'Tmall', 'Alipay'] as const;
type app = typeof APP[number];
// type app = "TaoBao" | "Tmall" | "Alipay"
```

## 映射类型

有的时候，一个类型需要基于另外一个类型，但是你又不想拷贝一份，这个时候可以考虑使用映射类型。

映射类型建立在索引签名的语法上。语法是 `[property in type]: type`

```js
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};

// 基于这个类型
type FeatureFlags = {
  darkMode: () => void;
  newUserProfile: () => void;
};

// 重新生成一个新的类型
type FeatureOptions = OptionsFlags<FeatureFlags>;
// type FeatureOptions = {
//    darkMode: boolean;
//    newUserProfile: boolean;
// }
```

在映射类型中使用 as 语句实现键名重新映射，语法是 `[Properties in type as NewKeyType]: Type[Properties]`

```js
type Getters<Type> = {
    // 使用了模板字面量的语法
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};

interface Person {
    name: string;
    age: number;
    location: string;
}

type LazyPerson = Getters<Person>;

// type LazyPerson = {
//    getName: () => string;
//    getAge: () => number;
//    getLocation: () => string;
// }
```

## 映射修饰符

在使用映射类型时，有两个额外的修饰符可能会用到，一个是 readonly，用于设置属性只读，一个是 ? ，用于设置属性可选。

你可以通过前缀 - 或者 + 删除或者添加这些修饰符，如果没有写前缀，相当于使用了 + 前缀。

```js
// 删除属性中的只读属性
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
  readonly id: string;
  readonly name: string;
};

type UnlockedAccount = CreateMutable<LockedAccount>;

// type UnlockedAccount = {
//    id: string;
//    name: string;
// }

// 删除属性中的可选属性
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};

type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};

type User = Concrete<MaybeUser>;
// type User = {
//    id: string;
//    name: string;
//    age: number;
// }
```

## 对象类型的属性

一个对象类型的属性，可能是 string, number, 或者 symbol，所以下面会报错。因为 T 是任意对象，所以 T 的 keys K 就可能是那三种。

```js
function useKey<T, K extends keyof T>(o: T, k: K) {
  var name: string = k;
  // Type 'string | number | symbol' is not assignable to type 'string'.
}
```

## typeof

js 中本来就有 typeof 关键字，用来获取某个变量的类型。也可以用在 ts 中。

```js
const person = { name: 'kevin', age: '18' };
type Kevin = typeof person;

// type Kevin = {
//   name: string;
//   age: string;
// }

function identity<Type>(arg: Type): Type {
  return arg;
}

type result = typeof identity;
// type result = <Type>(arg: Type) => Type
```

typeof 搭配 keyof 操作符用于获取属性名的联合字符串。

```js
var test = {
    x: 1,
    y: 1
}
type tt = keyof typeof test; // 正确 tt: 'x' | 'y'
```

## keyof 类型操作符

对一个对象类型使用 keyof 操作符，会返回该对象属性名组成的一个字符串或者数字字面量的联合。

keyof 后面跟的是类型，或者是直接的 js 对象，因为这种 js 对象会被推断出类型。但不能是一个对象的引用。

```js
type Point = { x: number; y: number };
type P = keyof Point; // 正确 tt: 'x' | 'y'

var test = {
    x: 1,
    y: 1
}
type tt = keyof test; // 错误
type tt = keyof {x:1, y:1} // 正确 tt: 'x' | 'y'
```

如果后面非要跟一个对象的引用，需要使用 typeof 先获取对象的 type。

```js
var test = {
    x: 1,
    y: 1
}
type tt = keyof typeof test; // 正确 tt: 'x' | 'y'
```

## as const

as const 可以把对象或数组转为一个只读的字面量类型。

```js
const APP = ['TaoBao', 'Tmall', 'Alipay'] as const;
// const APP: readonly ["TaoBao", "Tmall", "Alipay"]

const req = {
    url: "https://example.com",
    method: "GET"
} as const;

// const req: {
//     readonly url: "https://example.com";
//     readonly method: "GET";
// }
```

## 内置字符操作类型

Uppercase 把每个字符转为大写形式

```js
type Greeting = 'Hello, world';
type ShoutyGreeting = Uppercase<Greeting>;
// type ShoutyGreeting = "HELLO, WORLD"
```

Lowercase 把每个字符转为小写形式

```js
type Greeting = 'Hello, world';
type QuietGreeting = Lowercase<Greeting>;
// type QuietGreeting = "hello, world"
```

Capitalize 把字符串的第一个字符转为大写形式

```js
type LowercaseGreeting = 'hello, world';
type Greeting = Capitalize<LowercaseGreeting>;
// type Greeting = "Hello, world"
```

## 内置类型工具

### `Partial<Type>`

用于构造一个 Type 下面的所有属性都设置为可选的类型

### `Required<Type>`

用于构造一个 Type 下面的所有属性全都设置为必填的类型

### `Readonly<Type>`

用于构造一个 Type 下面的所有属性全都设置为只读的类型

### `Record<Keys, Type>`

用于构造一个对象类型，它所有的键都是 Keys 类型，它所有的值都是 Type 类型。

```js
interface CatInfo {
  age: number;
  breed: string;
}

type CatName = 'miffy' | 'boris' | 'mordred';

const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: 'Persian' },
  boris: { age: 5, breed: 'Maine Coon' },
  mordred: { age: 16, breed: 'British Shorthair' },
};
```

### `Omit<Type, Keys>`

用于构造一个类型，它是从 Type 类型里面忽略（移除）了一些属性 Keys，用于对象类型或 interface

### `Pick<Type, Keys>`

用于构造一个类型，它是从 Type 类型里面选择了一些属性 Keys，用于对象类型或 interface

### `Exclude<UnionType, ExcludedMembers>`

用于构造一个类型，它是从 UnionType 联合类型里面排除了所有可以赋给 ExcludedMembers 的类型。

### `Extract<Type, Union>`

用于构造一个类型，它是从 Type 类型里面提取了所有可以赋给 Union 的类型。

### `ReturnType<Type>`

用于构造一个含有 Type 函数的返回值的类型。

```js
declare function f1(): { a: number; b: string };

type T0 = ReturnType<() => string>;
// type T0 = string

type T1 = ReturnType<(s: string) => void>;
// type T1 = void

type T2 = ReturnType<<T>() => T>;
// type T2 = unknown

type T3 = ReturnType<<T extends U, U extends number[]>() => T>;
// type T3 = number[]

type T4 = ReturnType<typeof f1>;
```

## 命名空间

命名空间，可以作为一个模块来维护一些类型声明或者接口，类等，内部可以用 export 导出，外部可以通过 xxx.yyy 使用。

命名空间可以被分割成多个文件，写在不同的文件里面，但是他们仍然是同一个命名空间。但是需要在 B 文件中添加 A 的引用`/// <reference path="A.ts" />`

现在都用 ES6 的模块替换了，不再使用命名空间了。

```js
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }
}

let validators: { [s: string]: Validation.StringValidator; } = {};
```

## 三斜线指令

`/// <reference path="..." />` 指令是三斜线指令中最常见的一种，它用于声明文件间的依赖。

## declare

declare 关键字用于声明一些已经存在的 JavaScript 代码或标识符的类型或值，以便在代码中使用它们。

```js
declare function myFunc(arg1: string, arg2: number): boolean;
declare const MY_VAR: string;
```
