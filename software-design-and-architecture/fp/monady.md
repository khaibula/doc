# Монады

**Монады** — это абстрактная структура из функционального программирования, которая позволяет элегантно описывать **вычисления** (в том числе «эффекты») в чистом (без побочных эффектов) стиле. Формально из категории теории: монада — это «моноид в категории эндофункторов», но на практике это означает, что монада предоставляет **способ** (1) «обернуть» значение или вычисление в контекст, и (2) последовательно «связывать» (bind, flatMap) эти вычисления, не нарушая чистоту.

***

### 1. Что «делают» монады

1. **Инкапсулируют контекст** (побочные эффекты, возможное отсутствие значения, ошибки, асинхронность и т. д.)
   * Вместо того, чтобы загрязнять основной код проверками (нет ли null?), логикой обработки ошибок, мы оборачиваем результат в «контейнер» (Maybe, Either, Promise) и работаем через «bind»/«map».
2. **Упрощают композицию**
   * Набор функций, которые возвращают значения в «монаде» (например, `Maybe<T>`), можно последовательно связывать, не пишя кучу if-else/try-catch.
   * Монада задаёт единый протокол, как «сцеплять» вычисления.
3. **Сохраняют чистоту** (чистые функции)
   * В «чистых» функциональных языках (Haskell) даже операции ввода-вывода оформлены как `IO` монада, что позволяет компилятору знать, где появляются эффекты, и всё равно держать основной код «чистым» (без мутирования глобального состояния).

***

### 2. Какие бывают (примеры)

#### 2.1. Maybe / Option

* **Maybe\<T>** (или **Option\<T>**) — либо «значение есть» (`Just(x)`), либо «значения нет» (`Nothing`).
* Позволяет избегать «null reference exception»: если `Nothing`, связка «пропускает» вычисления, а результат остаётся `Nothing`.

#### 2.2. Either / Result

* **Either\<L, R>** (или **Result\<T, E>**) — либо «значение R», либо «ошибка L».
* Удобно для «сигнализации» об ошибках, без бросков исключений (throw). При последовательной связи «ошибка» коротко замыкает цепочку.

#### 2.3. IO / Task / Effect

* В «чистом» языке вроде Haskell любая операция ввода-вывода (чтение консоли, запись в файл) представляется как **IO**-монада.
* В более «реактивных» окружениях (JavaScript) это может быть **Promise** (тоже монада: `then()` — аналог bind).
* Подход: «просто описываем эффект, но не выполняем тут же». Реальное исполнение контролируется средой — благодаря «do-notation» (в Haskell) или цепочке `then`/`await`.

#### 2.4. List / Stream

* **List** в функциональных языках часто рассматривается как монада, где «bind» — это «декартово произведение + map», позволяя строить генераторы комбинаций.
* Для `List<T>`, связывать означает «для каждого элемента — вернуть новый список, всё объединить».

#### 2.5. State / Reader / Writer

* Монады для более специфических случаев:
  * **State** — несёт «скрытое» состояние (s), передаёт его между вычислениями без «глобальной» мутации.
  * **Reader** — предоставляет «общий контекст» (конфигурацию) в цепочке вычислений.
  * **Writer** — собирает «лог» или «аккумулирует» данные по мере вычислений.

***

### 3. Как это выглядит: (Haskell-подобный) «bind»

Ключевой метод монады: **`bind (или flatMap) :: M a -> (a -> M b) -> M b`**.

* Из «контейнера со значением типа a» и функции, которая из `a` возвращает «контейнер со значением b», мы получаем «контейнер со значением b».

#### Пример: Maybe

```hs
-- Maybe a = Just a | Nothing

-- bind :: Maybe a -> (a -> Maybe b) -> Maybe b
bind (Just x) f = f x
bind Nothing  _ = Nothing
```

* Если есть `Just x`, применяем функцию `f x`. Если `Nothing`, результат тоже `Nothing`.

#### Пример: Promise (JS)

```js
// "then" — это bind для Promise
const p = fetch("url")
  .then(response => response.json())
  .then(data => { /* ... */ })
  .catch(err => { /* ... */ });
```

* Каждый `.then()` принимает «предыдущее значение», возвращает новый Promise. Если в какой-то момент ошибка, переходим к `.catch`.

***

### 4. Почему их называют «монадами»

* Термин «монада» (monad) пришёл из **категориальной теории** (category theory). Там **монада** — это функтор с двумя операциями (`unit` и `bind`), удовлетворя нескольким аксиомам (тождественность, ассоциативность).
* Простыми словами:
  1. Есть контейнер (эндофунктор `M`), который «оборачивает» значение типа `A` в `M<A>`.
  2. Есть «unit» (или `return`): `A -> M<A>` (как «конструирование» контейнера).
  3. Есть «bind» (`flatMap`): `M<A> -> (A -> M<B>) -> M<B>`, цепляющее вычисления.
* «Монада» — более общее понятие, чем конкретный «Maybe» или «Promise», это паттерн: **что-то** (функтор) + метод «bind» + законы.

***

### Итог

* **Монады** — это «паттерн» (или «типовой» механизм) для **последовательной** композиции вычислений, где каждое вычисление возвращает результат в «контейнере» (контекст).
* Они используются, чтобы «обернуть» разные аспекты: отсутствие значения, ошибки, асинхронность, ввод-вывод, состояние.
* Ключ: есть метод `bind` (или `flatMap`), который упрощает работу с результатом внутри контейнера без раскрытия деталей.
* Название «монада» исторически из математики (категориальная теория). Это звучит сложно, но на практике — это очень удобная концепция, упрощающая чистый FP и в целом «оборачивающая» эффекты в предсказуемую, компонуемую форму.
