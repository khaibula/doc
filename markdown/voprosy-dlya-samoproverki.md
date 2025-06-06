# Вопросы для самопроверки

<details>

<summary>Что такое объединение Union? Что такое пересечение intersection?</summary>

Union позволяет переменной, параметру или значению принадлежать **одному из нескольких типов**.

* **Синтаксис**: `Тип1 | Тип2 | ...`
*   **Пример**:

    ```typescript
    type ID = string | number; // ID может быть строкой или числом
    let userId: ID = "abc123"; 
    userId = 42; // тоже допустимо
    ```
* **Зачем использовать?**
  * Когда значение может быть разных типов (например, ID, который может быть строкой или числом).
  * Для работы с функциями, принимающими гибкие параметры.
* **Особенности**:
  * TypeScript разрешает только операции, допустимые для **всех типов** в Union.
  *   Чтобы использовать специфичные методы/свойства, нужно **сузить тип** (например, через `typeof` или `type guards`):\


      ```typescript
      function printId(id: ID) {
          if (typeof id === "string") {
          console.log(id.toUpperCase()); // OK, так как id здесь строка
        } else {
          console.log(id.toFixed(2)); // OK, id здесь число
        }
      }
      ```

Intersection объединяет несколько типов в **один**, который включает **все свойства** исходных типов.

* **Синтаксис**: `Тип1 & Тип2 & ...`
*   **Пример**:

    ```typescript
    type Person = { name: string };
    type Employee = { id: number; role: string };

    type EmployeePerson = Person & Employee; 
    // EmployeePerson = { name: string, id: number, role: string }

    const alice: EmployeePerson = {
      name: "Alice",
      id: 123,
      role: "Developer",
    };
    ```
* **Зачем использовать?**
  * Для комбинирования интерфейсов/типов (например, расширение объектов).
  * В миксинах (mixins) или для создания комплексных типов.
* **Особенности**:
  *   Если свойства с одинаковыми именами имеют **разные типы**, возникнет ошибка:

      ```typescript
      type A = { x: number };
      type B = { x: string };
      type C = A & B; // Ошибка: свойство 'x' имеет тип 'never' (number & string)
      ```
  * Работает с интерфейсами, типами объектов и примитивами (но для примитивов результат — `never`, так как `string & number` невозможно).

</details>



<details>

<summary>Что такое type? Что такое interface? Чем они отличаются?</summary>

#### **Что такое `type`?**

**`Type`** (псевдоним типа) — это способ создать **новое имя** для существующего типа или объединить/комбинировать типы.

* Может описывать примитивы, объекты, массивы, union-типы, intersection-типы и т.д.
*   Примеры:

    ```typescript
    type Age = number; // Псевдоним для числа
    type ID = string | number; // Union-тип
    type User = { name: string; age: number }; // Объект
    type Coordinates = [number, number]; // Кортеж
    ```

**Что такое `interface`?**

**`Interface`** — это способ описать **форму объекта**, включая его свойства, методы и сигнатуры.

* Используется для объектов, классов, функций.
* Поддерживает **наследование** (`extends`) и **слияние** (повторное объявление интерфейса добавляет свойства).
*   Пример:

    ```typescript
    interface User {
      name: string;
      age: number;
      greet(): void;
    }
    ```

#### **Ключевые различия**

| **Критерий**               | **`type`**                                   | **`interface`**                                       |
| -------------------------- | -------------------------------------------- | ----------------------------------------------------- |
| **Область применения**     | Любые типы (примитивы, union, intersection). | Только объекты/классы (не может описывать примитивы). |
| **Наследование**           | Через intersection (`&`).                    | Через `extends`.                                      |
| **Расширение**             | Нельзя расширить после объявления.           | Можно дополнять (declaration merging).                |
| **Имплементация в классе** | Да (но редко используется).                  | Да (через `implements`).                              |
| **Слияние (merging)**      | Нет.                                         | Да (повторное объявление добавляет свойства).         |
| **Пример сложных типов**   | `type A = string \| number;`                 | Не поддерживает union/intersection напрямую.          |

</details>



<details>

<summary>Что такое type assertion (утверждение типа) и как его выполнить?</summary>

*   **Type assertion** — явное указание типа, когда TS не может его вывести.

    ```typescript
    let value: any = "Hello";
    let str = value as string; // Утверждение типа через as.
    let num = <number>value;   // Альтернативный синтаксис (не работает в JSX).
    ```
* **Используйте осторожно**: Убедитесь, что данные соответствуют типу, чтобы избежать ошибок времени выполнения.

</details>



<details>

<summary>Что такое сужение типов (Type Narrowing)</summary>

Сужение типов — это процесс, при котором TypeScript определяет более конкретный тип значения на основе условий в коде. Это позволяет писать безопасный и понятный код, учитывая все возможные варианты типов. Рассмотрим основные методы сужения.

Оператор `typeof` возвращает строку, указывающую тип значения. TypeScript использует это для сужения типа внутри блока условия.

```typescript
function printValue(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase()); // Тип value: string
  } else {
    console.log(value.toFixed(2));   // Тип value: number
  }
}
```

**Проверки с помощью `instanceof`**

Проверяет, является ли объект экземпляром класса.

```typescript
class Dog { bark() {} }
class Cat { meow() {} }

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark(); // Тип animal: Dog
  } else {
    animal.meow(); // Тип animal: Cat
  }
}
```

**Проверка истинности (Truthiness)**

Убирает «пустые» значения (`null`, `undefined`, `0`, `""`).

```typescript
function printLength(str?: string) {
  if (str) {
    console.log(str.length); // Тип str: string (не undefined или "")
  } else {
    console.log("Строка пуста");
  }
}
```

**Проверка наличия свойства (`in`)**

Определяет, есть ли свойство у объекта.

```typescript
interface Dog { bark(): void }
interface Cat { meow(): void }

function interact(pet: Dog | Cat) {
  if ("bark" in pet) {
    pet.bark(); // Тип pet: Dog
  } else {
    pet.meow(); // Тип pet: Cat
  }
}
```

**Пользовательские защитники типов (Type Guards)**

Функции, возвращающие предикат типа (`arg is Type`).

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function process(value: unknown) {
  if (isString(value)) {
    console.log(value.toUpperCase()); // Тип value: string
  }
}
```

</details>



<details>

<summary>Как работают функции в typescript?</summary>

**Типизация параметров и возвращаемого значения**

* **Параметры**: Указывайте типы для каждого аргумента.
* **Возвращаемое значение**: Тип после списка параметров.

```typescript
function add(a: number, b: number): number {
  return a + b;
}
```

**Перегрузка функций (Function Overloads)**

Определите несколько сигнатур для одной функции:

```typescript
// Сигнатуры перегрузки
function formatInput(input: string): string;
function formatInput(input: number): number;

// Реализация
function formatInput(input: string | number): string | number {
  if (typeof input === "string") return input.trim();
  return input.toFixed(2);
}
```

**Дженерики в функциях**

Создавайте универсальные функции для работы с разными типами:

```typescript
function identity<T>(arg: T): T {
  return arg;
}
identity<string>("Hello"); // Явное указание типа
identity(42); // Тип number выведен автоматически
```

**`void` vs `never`**

* **`void`**: Функция не возвращает значение.
* **`never`**: Функция никогда не завершается (например, выбрасывает ошибку).

```typescript
function logMessage(message: string): void {
  console.log(message);
}

function throwError(message: string): never {
  throw new Error(message);
}
```

**Описание конструкций**

Функции JavaScript также могут быть вызваны с помощью `new`оператора. TypeScript называет их _конструкторами_ , поскольку они обычно создают новый объект. Вы можете написать _сигнатуру конструкции_ , добавив `new`ключевое слово перед сигнатурой вызова:

```typescript
type SomeConstructor = {
  new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

</details>



<details>

<summary>Что такое object? Чем он отличается от Object?</summary>

Тип `object` в TypeScript представляет **любое не-примитивное значение**. Это объекты, массивы, функции, классы и т.д., но не примитивы (строки, числа, `boolean`, `null`, `undefined`, `symbol`, `bigint`).

> `object`не является `Object`. **Всегда** используйте `object`!

**Отличие от `Object`, `{}`**:

*   `Object` и `{}` включают **все значения** (даже примитивы):

    ```typescript
    let a: {} = "Hello"; // OK (примитив оборачивается в объект)
    let b: Object = 42;  // OK
    let c: object = 42;  // Ошибка

    ```

</details>



<details>

<summary>Расширение интерфейса(extends) против пересечения(&#x26;). В чем разница?</summary>

Мы только что рассмотрели два способа объединения типов, которые похожи, но на самом деле немного отличаются. С интерфейсами мы могли использовать `extends`предложение для расширения из других типов, и мы могли сделать что-то похожее с пересечениями и назвать результат псевдонимом типа. Главное различие между ними заключается в том, как обрабатываются конфликты, и это различие обычно является одной из главных причин, по которой вы бы выбрали один из двух вариантов между интерфейсом и псевдонимом типа пересечения.

Если интерфейсы определены с одинаковым именем, TypeScript попытается объединить их, если свойства совместимы. Если свойства несовместимы (т. е. у них одинаковое имя свойства, но разные типы), TypeScript выдаст ошибку.

В случае типов пересечения свойства с разными типами будут автоматически объединены. Когда тип будет использоваться позже, TypeScript будет ожидать, что свойство будет удовлетворять обоим типам одновременно, что может привести к неожиданным результатам.

Например, следующий код выдаст ошибку, поскольку свойства несовместимы:

```typescript
interface Person {  name: string; }
interface Person {  name: number; }
```

Напротив, следующий код скомпилируется, но результатом его будет `never`тип:

```typescript
interface Person1 {
  name: string;
}
 
interface Person2 {
  name: number;
}
 
type Staff = Person1 & Person2
 
declare const staffer: Staff;
staffer.name;
         
(property) name: never
```

В этом случае Staff потребует, чтобы свойство name было и строкой, и числом, что приведет к свойству типа `never`.

</details>



<details>

<summary>Что такое index сигнатура в ts?</summary>

Index сигнатура в TypeScript позволяет описывать объекты с динамическими свойствами, когда заранее неизвестны имена свойств, но известен их тип. Это особенно полезно для работы с "словари" или "карты", где ключи могут быть произвольными строками или числами, а значения имеют определённый тип.

#### Основной синтаксис

Для объявления index сигнатуры используется следующий синтаксис:

```ts
interface StringDictionary {
  [key: string]: string;
}
```

Здесь:

* **`[key: string]`** — указывает, что объект может иметь любое количество свойств, ключ которых — строки.
* **`: string`** — тип значений для этих свойств.

Можно использовать и числовые индексы:

```ts
interface NumberDictionary {
  [index: number]: number;
}
```

Важно помнить, что в JavaScript числовые ключи автоматически приводятся к строкам. Поэтому в интерфейсе, если объявлена числовая сигнатура, должна быть совместимость со строковой сигнатурой, если они используются вместе.

#### Пример использования

```ts
interface UserProfile {
  name: string;
  age: number;
  [propName: string]: string | number;
}

const user: UserProfile = {
  name: "Alice",
  age: 30,
  city: "Moscow",  // Дополнительное свойство, соответствующее index сигнатуре
  country: "Russia"
};
```

Здесь:

* Свойства `name` и `age` строго типизированы.
* Благодаря index сигнатуре `[propName: string]: string | number;` можно добавлять дополнительные свойства, ключи которых являются строками, а значения — либо строками, либо числами.

</details>



<details>

<summary>Что такое кортежи? Как они работают?</summary>

Кортежи — это структуры данных, которые позволяют хранить фиксированное количество элементов, каждый из которых имеет свой тип. Они похожи на массивы, но с строгим контролем типов и длины.

Синтаксис:\
`let tupleName: [тип1, тип2, ..., типN] = [значение1, значение2, ..., значениеN];`

**Пример**:

```typescript
let user: [string, number, boolean] = ["Alice", 30, true];
```

**Именованные кортежи (TypeScript 4.0+)**

Можно задавать имена элементам для улучшения читаемости:

```typescript
type Point = [x: number, y: number];
const p: Point = [10, 20];
console.log(p.x); // Ошибка: имена существуют только для документации!
// Доступ по-прежнему через индекс: p[0], p[1].
```

Кортежи также могут иметь остаточные элементы, которые должны иметь тип массива/кортежа.

```typescript
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
```

</details>



<details>

<summary>Что такое дженерики? Как они устроены?</summary>

Дженерики — это инструмент для создания **переиспользуемых компонентов**, которые могут работать с разными типами данных, сохраняя при этом **строгую типизацию**. Они позволяют писать гибкий код, не теряя преимуществ проверки типов.

#### **Зачем нужны дженерики?**

Представьте функцию, которая возвращает переданный аргумент:

```typescript
function identity(arg: any): any {
  return arg;
}
```

* Проблема: Тип `any` отключает проверку типов. Мы не знаем, что вернет функция.
* Решение: Использовать дженерик, чтобы сохранить тип аргумента и возвращаемого значения.

**Ограничения дженериков (`extends`)**

Иногда нужно ограничить типы, которые можно использовать в дженерике.\
**Пример**: Функция, работающая с объектами, у которых есть свойство `length`.

```typescript
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): void {
  console.log(arg.length);
}

logLength("Hello"); // OK (у строки есть length)
logLength([1, 2]);  // OK (у массива есть length)
logLength(42);      // Ошибка: у числа нет length
```

**Значения по умолчанию для дженериков**

Можно указать тип по умолчанию, если параметр не задан:

```typescript
interface PaginatedResponse<T = any> {
  data: T[];
  page: number;
}

const response: PaginatedResponse<string> = {
  data: ["a", "b"],
  page: 1,
};
```

</details>



<details>

<summary>Что знаешь про индексированные типы доступа (Indexed Access Types)?</summary>

Индексированные типы позволяют получить тип свойства объекта **по ключу** или тип элемента массива/кортежа **по индексу**. Это полезно для динамического получения типов на основе ключей.

**Пример с массивами**

Для массивов используйте `[number]`, чтобы получить тип элемента:

```typescript
type StringArray = string[];
type ArrayElement = StringArray[number]; // string

// Пример с массивом объектов
interface Book {
  title: string;
  pages: number;
}

type Library = Book[];
type BookType = Library[number]; // Book
```

</details>



<details>

<summary>Что знаешь про условные типы?</summary>

Условные типы позволяют создавать **динамические типы**, которые зависят от других типов. Они работают по принципу тернарного оператора (`condition ? trueType : falseType`), но для типов

**Базовый синтаксис**

```typescript
T extends U ? X : Y
```

* **`T`** — проверяемый тип.
* **`U`** — тип, с которым сравнивается `T`.
* Если `T` совместим с `U`, то тип становится **`X`**, иначе — **`Y`**.

**Распределение условных типов (Distributive Conditional Types)**

Если `T` является объединением (union), условный тип применяется к каждому элементу отдельно:

```typescript
type ToArray<T> = T extends any ? T[] : never;

type NumbersOrStrings = number | string;
type Arr = ToArray<NumbersOrStrings>; // number[] | string[]
```

Нужно так же учитывать, чтобы пройтись по каждому элементу union, необходимо сопоставить его к какому либо типу, например `any`

Условные типы так же могут сокращать перегрузки функций\
Вместо

```typescript
interface IdLabel {
  id: number /* some fields */;
}
interface NameLabel {
  name: string /* other fields */;
}
 
function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw "unimplemented";
}
```

Получаем

```typescript
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
  
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  throw "unimplemented";
}
```

</details>



<details>

<summary>Что знаешь о ключевом слове infer?</summary>

ключевое слово **`infer`** — это мощные инструменты TypeScript для создания гибких и динамических типов

**Синтаксис:**

```typescript
T extends infer U ? U : never
```

**Где:**

* `U` — временный тип, выводимый из `T`.

**Примеры использования `infer`**

**a. Получение типа возвращаемого значения функции (`ReturnType`)**

```typescript
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() { return { name: "Alice" }; }
type User = MyReturnType<typeof getUser>; // { name: string }
```

**b. Извлечение типа параметров функции (`Parameters`)**

```typescript
type MyParameters<T> = T extends (...args: infer P) => any ? P : never;

function greet(name: string, age: number) {}
type Params = MyParameters<typeof greet>; // [string, number]
```

**c. Получение типа элемента массива**

```typescript
type ArrayItem<T> = T extends (infer Item)[] ? Item : never;

type Numbers = ArrayItem<number[]>; // number
```

**d. Работа с промисами**

```typescript
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type A = UnwrapPromise<Promise<string>>; // string
type B = UnwrapPromise<42>;              // 42
```

</details>



<details>

<summary>Дистрибутивные union типы, что такое? Как работает?</summary>

Дистрибутивные условные типы – это особенность TypeScript, согласно которой условный тип автоматически распределяется по всем членам union-типа. С теоретической точки зрения это можно рассматривать как операцию «map» над суммарными (union) типами, где каждое слагаемое обрабатывается отдельно.

### Как это работает

При объявлении условного типа в форме:

```ts
type Conditional<T> = T extends U ? X : Y;
```

если `T` является union-типом (например, `A | B | C`), то TypeScript интерпретирует это так, как если бы условие применялось к каждому компоненту по отдельности:

```ts
type Conditional<A | B | C> = (A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y);
```

Таким образом, результатом является union из результатов применения условного типа к каждому элементу исходного union.



С точки зрения теории типов, union-тип можно рассматривать как _сумму_ (sum type), где значение может принадлежать одному из нескольких вариантов. Дистрибутивное поведение условных типов похоже на сопоставление с образцом (pattern matching) для суммарных типов, когда функция (в данном случае условный тип) применяется к каждому возможному варианту.

Это позволяет писать более выразительные и обобщенные типы, так как поведение условного типа становится более предсказуемым:

* Если вам нужно проверить каждый вариант union-типа отдельно – дистрибуция помогает.
* Если же нужно проверить объединенный тип целиком, можно «обернуть» параметр, например, в кортеж, чтобы предотвратить дистрибуцию:

```ts
type CheckNumberNonDistributive<T> = [T] extends [number] ? "number" : "not number";

type NonDistributedResult = CheckNumberNonDistributive<number | string>;
// Здесь [number | string] не распределяется по компонентам,
// результат: "not number"
```

</details>



<details>

<summary>Что знаешь про сопоставленные типы или Mapped Types</summary>

Сопоставленные типы позволяют создавать **новые типы на основе существующих**, итерируясь по ключам и преобразуя их. Это мощный инструмент для модификации свойств объекта: изменения их типов, модификаторов (`readonly`, `?`) или даже переименования ключей.

**Базовый синтаксис**

```typescript
type NewType = {
  [K in KeyType]: ValueType;
};
```

* **`KeyType`** — набор ключей (обычно `keyof T`).
* **`ValueType`** — тип значения для каждого ключа (может зависеть от исходного типа).

Как можно менять названия ключей? (Key Remapping via `as)`

В TypeScript можно использовать ключевое слово `as` в mapped types для изменения имен ключей. Это делается с помощью синтаксиса:

```typescript
type NewType = {
  [K in OldType as NewKey]: OldType[K];
};
```

Здесь:

* `K` — это ключ из исходного типа `OldType`.
* `NewKey` — новое имя ключа, которое вы хотите использовать.
* `OldType[K]` — значение, связанное с ключом `K` в исходном типе.

**Переименование ключей**

Допустим, у вас есть тип `Person`, и вы хотите создать новый тип, где все ключи будут иметь префикс `user_`.

```typescript
type Person = {
  name: string;
  age: number;
  isActive: boolean;
};

type User = {
  [K in keyof Person as `user_${K}`]: Person[K];
};

// Результат:
// type User = {
//   user_name: string;
//   user_age: number;
//   user_isActive: boolean;
// };
```

</details>



<details>

<summary>Что такое типы литералов шаблонов (Template Literal Types) ?</summary>

Это мощный инструмент, который позволяет создавать сложные строковые типы на основе шаблонов. Они похожи на шаблонные строки в JavaScript (те, что с обратными кавычками `` `Hello, ${name}` ``), но работают на уровне типов

Типы литералов шаблонов используют синтаксис, похожий на JavaScript:

```typescript
type MyType = `Hello, ${string}`; // Пример типа
```

Здесь `${string}` — это "placeholder", который может быть заменен любым строковым типом, union-типом, или даже другим шаблонным литералом.

**Фиксированные форматы**

Допустим, вы хотите описать тип для CSS-единиц:

```typescript
type CSSUnit = `${number}px` | `${number}em` | `${number}%`;
const width: CSSUnit = "100px"; // OK
const height: CSSUnit = "20vh"; // Ошибка: 'vh' не входит в тип.
```

**Динамические ключи объектов**

Совмещая с **Key Remapping**, можно создавать объекты с ключами, подчиняющимися шаблону:

```typescript
type EventHandlers = {
  [K in `on${Capitalize<"click" | "hover">}`]: () => void;
};
// Результат:
// type EventHandlers = {
//   onClick: () => void;
//   onHover: () => void;
// };
```

**Утилиты для строк**

TypeScript предоставляет встроенные утилиты для работы со строками:

* `Uppercase<T>`, `Lowercase<T>`,
* `Capitalize<T>`, `Uncapitalize<T>`.

</details>



<details>

<summary>Расскажи про классы, что можно настраивать?</summary>

это синтаксический сахар над прототипным наследованием JavaScript, но с добавлением строгой типизации, модификаторов доступа и других возможностей

**Базовый синтаксис**

Классы в TypeScript объявляются так же, как в ES6+, но с добавлением типов:

```typescript
class Person {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  greet(): string {
    return `Hello, I'm ${this.name}!`;
  }
}

const alice = new Person("Alice", 30);
console.log(alice.greet()); // "Hello, I'm Alice!"
```

**Модификаторы доступа**

TypeScript добавляет контроль над видимостью свойств и методов:

* **`public`** (по умолчанию): доступно везде.
* **`private`**: доступно только внутри класса.
* **`protected`**: доступно внутри класса и его подклассов.
* **`readonly`**: свойство нельзя изменить после инициализации.

```typescript
class Animal {
  public name: string;
  private secret: string = "hidden";
  protected age: number;
  readonly species: string;

  constructor(name: string, age: number, species: string) {
    this.name = name;
    this.age = age;
    this.species = species;
  }
}

class Dog extends Animal {
  bark() {
    console.log(`Age: ${this.age}`); // OK (protected)
    // console.log(this.secret); // Ошибка: private
  }
}

const dog = new Dog("Rex", 5, "Canine");
// dog.species = "Cat"; // Ошибка: readonly
```

**Абстрактные классы**

Абстрактные классы не могут быть инстанциированы напрямую. Они задают "шаблон" для подклассов:

```typescript
abstract class Shape {
  abstract area(): number; // Абстрактный метод (без реализации)

  printArea(): string {
    return `Area: ${this.area()}`;
  }
}

class Circle extends Shape {
  constructor(public radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }
}

// const shape = new Shape(); // Ошибка: абстрактный класс
const circle = new Circle(5);
console.log(circle.printArea()); // "Area: 78.54..."
```

**Интерфейсы для классов**

Интерфейсы могут описывать структуру класса через ключевое слово `implements`:

```typescript
interface Drivable {
  speed: number;
  startEngine(): void;
  stopEngine(): void;
}

class Car implements Drivable {
  speed: number = 0;

  startEngine() {
    console.log("Engine started");
  }

  stopEngine() {
    console.log("Engine stopped");
  }
}
```

**Декораторы классов**

Декораторы — это функции, которые модифицируют поведение класса или его членов (экспериментальная фича, включается через `experimentalDecorators` в `tsconfig.json`):

```typescript
function LogClass(target: Function) {
  console.log(`Class ${target.name} created!`);
}

@LogClass
class User {
  constructor(public name: string) {}
}
// В консоли: "Class User created!"
```

</details>



<details>

<summary>Что такое Enums? Для чего они нужны?</summary>

Это специальный тип, который позволяет задавать именованные константы для набора числовых или строковых значений

**Числовые Enums (Numeric Enums)**

**По умолчанию** элементы получают автоинкрементные числовые значения, начиная с 0.\
Но можно задать начальное значение:

```typescript
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right, // 3
}

enum StatusCode {
  NotFound = 404,
  BadRequest = 400,
  Success = 200,
}
```

**Строковые Enums (String Enums)**

Каждому элементу явно присваивается строка:

```typescript
enum Color {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE",
}
```

**Как Enums компилируются в JavaScript?**

TypeScript генерирует объект, который связывает имена со значениями (и наоборот):

```typescript
// Исходный TS-код:
enum Direction { Up, Down }

// Скомпилированный JS:
var Direction;
(function (Direction) {
  Direction[Direction["Up"] = 0] = "Up";
  Direction[Direction["Down"] = 1] = "Down";
})(Direction || {});
```

Это позволяет получать **ключи по значениям**:

```typescript
console.log(Direction[0]); // "Up"
```

#### **Const Enums**

Если добавить ключевое слово `const`, компилятор заменит обращения к enum на их значения (без генерации объекта):

```typescript
const enum FastDirection {
  Up,
  Down,
}

console.log(FastDirection.Up); // В JS: console.log(0 /* Up */);
```

**Плюсы**: Уменьшение размера кода.\
**Минусы**: Невозможно получить ключи по значениям.

**Проблемы Enums**

* **Генерируемый код**: Обычные Enums создают объекты в JS, что увеличивает размер бандла.
* **Дублирование**: Для строковых Enums значения часто повторяют имена (`Red = "RED"`).
* **Tree-shaking**: Bundler'ам сложно удалить неиспользуемые Enums.

**Альтернативы Enums**

**Union Types + as const**

```typescript
const Direction = {
  Up: 0,
  Down: 1,
} as const;

type Direction = typeof Direction[keyof typeof Direction]; // 0 | 1
```

**Плюсы**: Нет генерируемого кода, лучше tree-shaking.\
**Минусы**: Нет обратного маппинга (ключ → значение).

**Типы для строковых литералов**

```typescript
type Color = "RED" | "GREEN" | "BLUE";
```

</details>



<details>

<summary>Что такое пространство имен(Namespace)?</summary>

это способ организации кода в логические группы, чтобы избежать конфликтов имен и улучшить структуру проекта

**Основной синтаксис**

Пространства имен объявляются с помощью ключевого слова `namespace`:

```typescript
namespace MyUtils {
  export function log(message: string) {
    console.log(message);
  }

  export const PI = 3.14;
}

// Использование:
MyUtils.log("Hello!"); // "Hello!"
console.log(MyUtils.PI); // 3.14
```

* **`export`** делает элементы доступными вне неймспейса.
* Без `export` элементы будут приватными внутри пространства.

Пространства имен можно вкладывать друг в друга:

```typescript
namespace Geometry {
  export namespace Area {
    export function circle(radius: number) {
      return Math.PI * radius ** 2;
    }
  }
}

// Использование:
Geometry.Area.circle(5); // ~78.54
```

#### **Слияние Namespace (Declaration Merging)**

TypeScript позволяет объявлять одно и то же пространство имен в разных местах. Они автоматически объединяются:

```typescript
namespace MyApp {
  export interface User {
    name: string;
  }
}

namespace MyApp {
  export function getUser(): User {
    return { name: "Alice" };
  }
}

// Использование:
const user: MyApp.User = MyApp.getUser();
```

Это полезно для расширения типов или добавления функциональности (например, в декларациях библиотек).

**Лучшие практики**

* **Избегайте неймспейсов в новых проектах**: Используйте модули (`import/export`).
* **Не смешивайте модули и неймспейсы**: Это приводит к путанице.
*   **Используйте `declare namespace` в декларациях**: Для описания типов сторонних библиотек.

    ```typescript
    // Пример декларации типов
    declare namespace React {
      interface Component { /* ... */ }
    }
    ```

</details>



<details>

<summary>Что такое файлы деклараций(declaration files) d.ts. Для чего они нужны?</summary>

Это специальные файлы, которые содержат только информацию о типах. Они нужны, чтобы TypeScript понимал структуру существующего JavaScript-кода (например, сторонних библиотек) и обеспечивал проверку типов

**Что такое файлы деклараций?**

Это файлы с расширением `.d.ts`, которые описывают:

* Типы переменных, функций, классов.
* Структуру модулей.
* Глобальные объекты (например, `window` или `process`).

**Пример:**

```typescript
// myLib.d.ts
declare function greet(name: string): void;
declare const version: string;
```

Здесь TypeScript узнает, что есть функция `greet` и константа `version`, даже если их реализация написана на JavaScript.

**Зачем они нужны?**

* **Интеграция с JavaScript-библиотеками**: Например, для jQuery или Lodash.
* **Типизация легаси-кода**: Если проект частично написан на JS.
* **Описание глобальных переменных**: Например, переменных, добавленных в `window`.
* **Публикация библиотек**: Чтобы пользователи вашего TS-кода видели типы.

**Глобальные vs модульные декларации**

*   **Глобальные**: Описывают типы, доступные везде (например, через `<script>` в браузере).

    ```typescript
    // globals.d.ts
    declare const APP_VERSION: string;
    ```
*   **Модульные**: Описывают структуру конкретного модуля.

    ```typescript
    // модуль "math-utils.d.ts"
    declare module "math-utils" {
        export function sum(a: number, b: number): number;
    }
    ```

**Объявление переменных и функций**

```typescript
declare const PI: number;
declare function calculateArea(radius: number): number;
```

**Описание классов**

```typescript
declare class User {
    constructor(name: string);
    getName(): string;
}
```

**Модули (для сторонних библиотек)**

```typescript
declare module "lodash" {
    export function shuffle<T>(array: T[]): T[];
}
```

**Глобальные расширения**

```typescript
declare global {
    interface Window {
        myCustomProp: string;
    }
}
```



**Пример: Типизация jQuery**

Допустим, у вас есть JS-код, подключающий jQuery через `<script>`. Чтобы TypeScript его понимал:

```typescript
// jquery.d.ts
declare interface JQuery {
    modal(command: "open" | "close"): void;
}

declare const $: (selector: string) => JQuery;
```

Теперь в TS-коде можно писать:

```typescript
$("#myModal").modal("open"); // Без ошибок типов!
```

**DefinitelyTyped и `@types`**

Большинство популярных JS-библиотек уже имеют декларации в репозитории [DefinitelyTyped](https://definitelytyped.org/). Установите их через npm:

```bash
npm install --save-dev @types/jquery
```

Теперь TypeScript автоматически подхватит типы для `jQuery`.

</details>



<details>

<summary>Доступы к значениями массива через [number]</summary>

```typescript
[1, 2, 3][number] // => union 1 | 2 | 3`
```

Механизм в TypeScript

1. **Кортеж как тип:**\
   Тип `[1, 2, 3]` представляет фиксированную последовательность, где каждый элемент имеет свой точный (литеральный) тип:
   * Элемент с индексом `0` имеет тип `1`.
   * Элемент с индексом `1` имеет тип `2`.
   * Элемент с индексом `2` имеет тип `3`.
2. **Индексный доступ (`T[K]`):**\
   В TypeScript, когда мы пишем тип в виде `T[K]`, где `T` – это тип объекта или кортежа, а `K` – тип ключа, компилятор возвращает тип свойства(й) для заданного ключа или набора ключей.
3. **Использование ключа `number`:**\
   В данном случае ключом является тип `number`. Для кортежа это означает, что мы хотим получить тип значений для всех возможных числовых индексов (0, 1, 2 и т.д.).\
   Поскольку для каждого индекса определён конкретный тип (в нашем случае литералы `1`, `2` и `3`), результатом индексного доступа становится объединение этих типов.
4.  **Результат – объединение (union) типов:**\
    Таким образом, `[1, 2, 3][number]` сводится к объединению типов элементов:

    * Для индекса `0` тип — `1`
    * Для индекса `1` тип — `2`
    * Для индекса `2` тип — `3`

    Результат: `1 | 2 | 3`.

</details>



<details>

<summary>Рекурсивное разложение массива на union</summary>

```typescript
type TupleToUnion<T extends any[]> = T extends [infer F, ...infer R] ? F | TupleToUnion<R> : never;
```

Пошаговое объяснение

1. **Ограничение типа параметра**\
   Типовой параметр `T` ограничен типом `any[]`, то есть функция принимает только массивы, чаще всего – кортежи (tuple).
2. **Условный тип и сопоставление с образцом**\
   Выражение `T extends [infer F, ...infer R]` проверяет, можно ли представить `T` как массив, состоящий из первого элемента (`F`) и оставшейся части (`R`).
   * Если сопоставление проходит успешно, `infer F` извлекает тип первого элемента, а `...infer R` – тип оставшихся элементов в виде массива.
   * Если же сопоставление не проходит (например, `T` – пустой массив), условный тип возвращает `never`.
3.  **Рекурсия и построение union-типа**\
    Если сопоставление успешно, тип определяется как:

    ```typescript
    F | TupleToUnion<R>
    ```

    То есть берётся тип первого элемента `F`, и к нему применяется рекурсивный вызов `TupleToUnion` для оставшейся части кортежа `R`.\
    Результатом рекурсии является объединение (union) всех элементов кортежа.

1) **База рекурсии**\
   Когда кортеж опустеет (то есть больше не будет удовлетворять условию `[infer F, ...infer R]`), сработает вторая часть условного типа, которая возвращает `never`. Это завершает рекурсивное вычисление.

</details>



<details>

<summary>Union в массив возможных вариантов</summary>

```typescript
type Permutation<T, Acc = T> = [T] extends [never]
	? []
	: Acc extends Acc
		? [Acc, ...Permutation<Exclude<T, Acc>>]
		: never;

  

type Test = Permutation<"A" | "B" | "C">; // ['A', 'B', 'C'] | ['A', 'C', 'B'] | ['B', 'A', 'C'] | ['B', 'C', 'A'] | ['C', 'A', 'B'] | ['C', 'B', 'A']
```

Как это работает:

1. Идет проверка на never, чтобы понять, что наш Union закончился
2. Мы создаем константу Acc, чтобы мы могли итерироваться по `union` с помощью конструкции `Acc extends Acc`, которая извлекает из `union` по одному элементу
3. В конечный результат `[Acc, ...Permutation<Exclude<T, Acc>>]` мы кладем `Acc`, и запускаем новый цикл в цикле с помощью `Permutation<Exclude<T, Acc>>`, исключая из передаваемого `T` текущий итерируемый элемент, после чего деструктурируем результат в изначальный массив. Если просматривать каждую итерацию, то это будет работать следующим образом: dsafdf

#### Начало рекурсии

**Шаг 1: Первая итерация — выбор `"A"`**

Сначала TypeScript начинает проверять объединение через конструкцию `Acc extends Acc`, что делает проверку для каждого элемента объединения отдельно. Первым берется `"A"`.

Теперь у нас:

* `Acc = "A"`
* `T = "A" | "B" | "C"`

Мы попадаем в условие: `[Acc, ...Permutation<Exclude<T, Acc>>]`

То есть, для `Acc = "A"` результатом будет массив, начинающийся с `"A"`, и нам нужно вычислить перестановки для оставшихся элементов. Чтобы это сделать, мы применяем: `Exclude<T, Acc> // Exclude<"A" | "B" | "C", "A"> = "B" | "C"`

Теперь мы вызываем `Permutation<"B" | "C">` для рекурсивной обработки оставшихся элементов.

**Шаг 2: Рекурсия с `"B" | "C"`**

Теперь у нас новая рекурсивная ветка с типом `Permutation<"B" | "C">`. Здесь процесс аналогичен:

* Сначала выбираем элемент `"B"`.
* Это дает нам результат `[Acc, ...Permutation<Exclude<T, Acc>>]`, где `Acc = "B"`, а оставшиеся элементы — это: `Exclude<"B" | "C", "B"> = "C"`

Теперь мы вызываем `Permutation<"C">` для дальнейшей рекурсии.

**Шаг 3: Рекурсия с `"C"`**

На следующем шаге мы уже имеем: `Permutation<"C">` В этом случае `Acc = "C"`, и остается вычислить перестановки для:

`Exclude<"C", "C"> = never`

Когда `T = never`, мы попадаем в базовый случай: `[T] extends [never] ? [] : ...` И возвращаем пустой массив `[]`, который завершает текущую ветку рекурсии.

**Шаг 4: Обратный путь**

Теперь мы начинаем возвращаться обратно, добавляя каждый элемент в результат.

* Для `Permutation<"C">` результат — `["C"]` (так как пустой массив плюс элемент `"C"`).
* Для `Permutation<"B" | "C">` с элементом `"B"` результатом будет `["B", "C"]`.
* В самой первой итерации с `"A"` и оставшимися `Permutation<"B" | "C">` результатом будет `["A", "B", "C"]`.

Таким образом, первая ветка рекурсии дала нам одну перестановку: `["A", "B", "C"]`.

#### Шаг 5: Вторая итерация с `"A"`

Теперь, после завершения первой ветки, TypeScript продолжает обработку второй возможной перестановки для `"A"`, но теперь с другим выбором:

* В месте, где `"B"` была первой, теперь выбирается `"C"`, а затем обрабатываются оставшиеся элементы аналогично, как мы сделали ранее.
* Это даст перестановку `["A", "C", "B"]`.

#### Что происходит дальше?

После того, как все перестановки для `"A"` вычислены, TypeScript продолжит с элементами `"B"` и `"C"` как первыми, рекурсивно генерируя все оставшиеся перестановки, включая:

* `["B", "A", "C"]`
* `["B", "C", "A"]`
* `["C", "A", "B"]`
* `["C", "B", "A"]`

</details>



<details>

<summary>Почему проверку на never нужно заворачивать в массив?</summary>

В TypeScript условные типы являются **дистрибутивными** по отношению к "голым" (naked) параметрам. Это означает, что если типовой параметр используется напрямую в условном типе, условие применяется к каждому члену union-типа по отдельности. Когда тип T равен `never`, он ведёт себя особым образом.

#### Проблема с дистрибутивностью и `never`

Рассмотрим такой тип:

```ts
tsКопировать кодtype IsNever<T> = T extends never ? true : false;
```

Если мы попытаемся проверить `IsNever<never>`, ожидаем, что результат будет `true`. Однако из-за дистрибутивного поведения условного типа происходит следующее:

* Когда T равен `never`, условный тип дистрибутивно применяется к каждому члену T. Но у `never` **нет членов** (оно представлено пустым union‑типом), поэтому результатом является **пустой union**, то есть итоговое значение становится `never`.

#### Решение: упаковка в кортеж

Чтобы избежать дистрибутивности, необходимо «упаковать» T в структуру, например, в кортеж. Тогда условный тип применяется к целому типу, а не к его потенциальным членам:

```ts
type IsNever<T> = [T] extends [never] ? true : false;
```

Теперь:

* Если T равен `never`, мы получаем проверку \[never] extends \[never], которая возвращает `true`.
* Если T не равен `never`, проверка не пройдет, и вернётся `false`.

</details>



<details>

<summary>Почему never это пустой union</summary>

В TypeScript тип **never** представляет собой тип, у которого **нет ни одного возможного значения**. Если рассматривать union-типы как объединение возможных вариантов, то можно сказать, что never — это union из «нулевого числа» вариантов, то есть пустой union.

#### Почему never считается пустым union

1. **Отсутствие значений:**\
   Тип never описывает ситуации, в которых значение никогда не появляется. Например, функция, которая всегда выбрасывает исключение или зацикливается, имеет тип never, так как она никогда не возвращает значение. Таким образом, если попытаться представить все возможные значения типа never, их окажется ноль.
2. **Union как совокупность вариантов:**\
   В теории типов union — это объединение множества типов, где каждое значение должно принадлежать хотя бы одному из этих типов. Если объединить 0 типов, то получается пустой набор вариантов. Поэтому never можно интерпретировать как union, состоящий из нуля типов.
3.  **Дистрибутивное поведение условных типов:**\
    Когда мы пишем условные типы, например:

    ```ts
    type IsNever<T> = T extends never ? true : false;
    ```

    и если T равно never, TypeScript пытается распределить never по его возможным членам. Но так как never не имеет ни одного элемента (оно пустое), распределение не происходит, и условие не срабатывает так, как ожидается. Именно поэтому часто используют упаковку в кортеж:

    ```ts
    type IsNever<T> = [T] extends [never] ? true : false;
    ```

    Эта упаковка предотвращает дистрибутивное поведение и позволяет корректно проверить, что T — это never.

</details>







