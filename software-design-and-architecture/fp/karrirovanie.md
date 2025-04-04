# Каррирование

**Каррирование** (currying) — это приём в функциональном программировании, при котором функция с несколькими аргументами превращается в цепочку функций, каждая из которых принимает один (или меньшее количество) аргумент.

Иначе говоря, если у нас есть функция `f(a, b, c)`, то в стиле каррирования она может быть преобразована в `f(a)(b)(c)`:

* Сначала вызываем `f` с одним аргументом `a`, результатом является новая функция, которая ждёт следующий аргумент `b`.
* Затем результатом уже будет функция, ожидающая `c`, и наконец выдающая окончательный результат.

***

### 1. Зачем нужно каррирование

#### 1.1. Повторное использование и частичное применение

* Каррированную функцию легко частично применить (partial application), «зафиксировав» одни аргументы и отложив другие.
* Например, вы можете «задать» часть аргументов заранее и получить функцию, которая принимает оставшиеся аргументы.

#### 1.2. Удобная композиция функций

* При функциональном стиле, где функции часто передают результаты друг другу, «одноаргументность» упрощает создание композиций.
* Многие абстракции (например, в Haskell) рассчитывают на то, что любая функция формально принимают один аргумент и возвращает что-то (в том числе ещё функцию).

#### 1.3. Чистота и декларативность

* Каррирование позволяет разделять логику «где мы берём частичные аргументы» и «где применяем оставшиеся».
* Позволяет элегантно и более декларативно выражать цепочки вычислений.

***

### 2. Простой пример (JavaScript)

Допустим, у нас есть обычная функция, принимающая 2 аргумента:

```js
function add(a, b) {
  return a + b;
}
```

**Каррированная версия** выглядит так:

```js
function addCurry(a) {
  return function(b) {
    return a + b;
  };
}

// Использование:
console.log(addCurry(2)(3)); // 5
```

Теперь мы можем делать **частичное применение**:

```js
const add2 = addCurry(2); // «зафиксировали» a=2
console.log(add2(10));    // 12
console.log(add2(100));   // 102
```

#### Стрелочная запись для каррирования

```js
const addCurry = a => b => a + b;
```

Это «цепочка» лямбд: первая берёт `a`, возвращает функцию, которая берёт `b`.

***

### 3. Пример с несколькими аргументами

Скажем, функция `mul(a, b, c)` умножает три числа. В каррированном стиле:

```js
const mulCurry = a => b => c => a * b * c;
```

* Вызываем: `mulCurry(2)(3)(4) // 24`
*   Частично:

    ```js
    const mul2 = mulCurry(2);       // функция, которая ждёт (b => c => 2*b*c)
    const mul2and3 = mul2(3);       // функция, которая ждёт (c => 2*3*c)
    console.log(mul2and3(4));       // 24
    console.log(mul2and3(10));      // 60
    ```

***

### 4. Как каррирование реализовано в функциональных языках

* В **Haskell** функции формально принимают всегда один аргумент и возвращают либо значение, либо новую функцию. «Многоаргументная функция» — это сахар: `f a b c` трактуется как `((f a) b) c`.
* В **Scala**, **F#**, **OCaml** и других языках тоже есть поддержка каррирования на уровне синтаксиса или стандартных библиотек.
* В JS/TS обычно вручную пишут функцию высшего порядка или используют библиотеки (Ramda, lodash/fp) с поддержкой каррирования.

***

### 5. Каррирование vs. Частичное применение

* **Каррирование**: превращение функции `f(x, y, z)` → `f(x)(y)(z)`.
* **Частичное применение**: создание новой функции, где зафиксирована часть аргументов, а остальные ещё не заданы.

Обычно каррирование облегчает частичное применение, ведь в каррированном варианте функция уже «по умолчанию» принимает по одному аргументу.

***

### 6. Пример практической пользы

#### 6.1. Фильтр по заданному предикату

В JS можно написать:

```js
// filterBy :: (a -> Boolean) -> [a] -> [a]
const filterBy = pred => arr => arr.filter(pred);
```

* Это каррированная функция: сначала принимаем `pred`, возвращаем функцию, ждущую `arr`.
* Применение:

```js
const isEven = x => x % 2 === 0;
const filterEven = filterBy(isEven); // частично применили

console.log(filterEven([1,2,3,4])); // [2,4]
```

#### 6.2. Логгирование с «префиксом»

```js
const logger = prefix => msg => {
  console.log(`[${prefix}] ${msg}`);
};

const warn = logger("WARN");
warn("Low disk space"); // [WARN] Low disk space
```

***

### 7. Когда каррирование может быть избыточным

1. **Часто меняется сигнатура**
   * Если нам нужно динамически разное количество аргументов, каррирование может путать.
2. **Слишком сложная логика**
   * Множество вложенных вызовов может ухудшить читаемость, если разработчики не привыкли к функциональному стилю.
3. **Простые случаи**
   * Иногда обычная функция «f(a, b, c)» и её вызов `f(1,2,3)` достаточно наглядны.

Но в функциональном стиле (особенно в FP-библиотеках) каррирование часто является базовой техникой, дающей гибкость и лаконичность кода.

***

#### Итог

**Каррирование** — преобразование функции, принимающей несколько аргументов, в набор функций, каждая из которых берёт по одному аргументу (или меньше). Оно облегчает **частичное применение** и **композицию** функций, является фундаментальной идеей во многих функциональных языках. В JavaScript и других мультипарадигм. языках каррирование часто используется при написании более «функционального» кода и создании модульных, переиспользуемых утилит.
