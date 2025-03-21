# Паттерны проектирования

## Паттерны проектирования

#### 1. Creational Patterns (Порождающие паттерны)

**Описание:**\
Порождающие паттерны помогают управлять созданием объектов, абстрагируя процесс инстанцирования и делая систему более независимой от конкретных классов.

<details>

<summary>Factory Method (фабричный метод)</summary>

#### Ключевые идеи фабричного метода

* **Инкапсуляция логики создания:**\
  Клиентский код не знает о конкретных классах, он работает через общий интерфейс или абстрактный класс. Это упрощает замену или добавление новых типов компонентов без изменения клиентской логики.
* **Расширяемость:**\
  При появлении нового типа компонента достаточно создать новый класс и соответствующую фабрику, не затрагивая остальной код приложения.
* **Снижение связности:**\
  Клиентский код зависит только от абстракций, а не от конкретных реализаций, что облегчает тестирование и поддержку.

***

### Пример

Представим ситуацию, когда в приложении нужно создавать разные виды кнопок (например, основная и второстепенная). Для этого можно реализовать фабричный метод следующим образом:

```typescript
// Определяем общий интерфейс для кнопок
interface IButton {
  render(): void;
}

// Конкретные реализации кнопок
class PrimaryButton implements IButton {
  render(): void {
    console.log("Render Primary Button");
    // Здесь может быть логика отрисовки, например, создание HTML-элемента
  }
}

class SecondaryButton implements IButton {
  render(): void {
    console.log("Render Secondary Button");
    // Логика отрисовки другого вида кнопки
  }
}

// Абстрактная фабрика, объявляющая фабричный метод
abstract class ButtonFactory {
  abstract createButton(): IButton;
}

// Конкретные фабрики, создающие нужные типы кнопок
class PrimaryButtonFactory extends ButtonFactory {
  createButton(): IButton {
    return new PrimaryButton();
  }
}

class SecondaryButtonFactory extends ButtonFactory {
  createButton(): IButton {
    return new SecondaryButton();
  }
}

// Клиентский код использует фабрику для создания и отрисовки кнопок
function renderButton(factory: ButtonFactory): void {
  const button = factory.createButton();
  button.render();
}

// Пример использования
renderButton(new PrimaryButtonFactory());   // Выведет: Render Primary Button
renderButton(new SecondaryButtonFactory()); // Выведет: Render Secondary Button
```

В данном примере клиентский код не зависит от конкретных классов кнопок, а использует фабрику для создания нужного объекта. Это позволяет легко добавлять новые виды кнопок, просто реализовав новый класс, удовлетворяющий интерфейсу `IButton`, и создав соответствующую фабрику.

</details>

<details>

<summary>Abstract Factory (абстрактная фабрика)</summary>

### Основная идея абстрактной фабрики

* **Семейства взаимосвязанных объектов:**\
  Абстрактная фабрика позволяет создавать наборы объектов (например, кнопок, чекбоксов, полей ввода), которые работают вместе и должны соответствовать одному стилю или теме.
* **Изоляция от конкретных реализаций:**\
  Клиентский код использует абстрактный интерфейс фабрики, не зная о конкретных классах создаваемых объектов. Это позволяет легко переключаться между разными семействами компонентов (например, переключать тему интерфейса).
* **Гарантия согласованности:**\
  Благодаря созданию объектов через единую фабрику, обеспечивается, что все компоненты интерфейса будут соответствовать выбранной стилистике и функционалу.

***

### Пример

Представим, что у нас есть два набора UI-компонентов для приложения: светлая (Light) и тёмная (Dark) тема. Каждый набор включает кнопку и чекбокс. Абстрактная фабрика позволяет нам создать объекты, соответствующие нужной теме.

```typescript
// Общие интерфейсы для компонентов
interface Button {
  render(): void;
}

interface Checkbox {
  render(): void;
}

// Конкретные реализации для светлой темы
class LightButton implements Button {
  render(): void {
    console.log("Render Light Button");
    // Здесь может быть логика создания HTML-элемента с классами для светлой темы
  }
}

class LightCheckbox implements Checkbox {
  render(): void {
    console.log("Render Light Checkbox");
    // Логика отрисовки чекбокса в светлой теме
  }
}

// Конкретные реализации для тёмной темы
class DarkButton implements Button {
  render(): void {
    console.log("Render Dark Button");
    // Логика создания кнопки с классами для тёмной темы
  }
}

class DarkCheckbox implements Checkbox {
  render(): void {
    console.log("Render Dark Checkbox");
    // Логика создания чекбокса для тёмной темы
  }
}

// Абстрактная фабрика, объявляющая методы для создания компонентов
interface UIComponentFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
}

// Фабрика для светлой темы
class LightUIFactory implements UIComponentFactory {
  createButton(): Button {
    return new LightButton();
  }
  createCheckbox(): Checkbox {
    return new LightCheckbox();
  }
}

// Фабрика для тёмной темы
class DarkUIFactory implements UIComponentFactory {
  createButton(): Button {
    return new DarkButton();
  }
  createCheckbox(): Checkbox {
    return new DarkCheckbox();
  }
}

// Клиентский код использует фабрику для создания UI-компонентов
function renderUI(factory: UIComponentFactory): void {
  const button = factory.createButton();
  const checkbox = factory.createCheckbox();
  button.render();
  checkbox.render();
}

// Пример использования: переключение между темами
const currentTheme: 'light' | 'dark' = 'light'; // или 'dark'
const uiFactory: UIComponentFactory = currentTheme === 'light'
  ? new LightUIFactory()
  : new DarkUIFactory();

renderUI(uiFactory);
```

В этом примере клиентский код не зависит от конкретных реализаций компонентов. При изменении темы достаточно передать другую реализацию фабрики, и все созданные объекты автоматически будут соответствовать нужной стилистике.

</details>

<details>

<summary>Builder (Строитель)</summary>

### Основные идеи паттерна Builder

* **Пошаговая сборка:**\
  Позволяет создавать объект в несколько этапов, задавая лишь необходимые параметры на каждом этапе. Это удобно, когда у объекта есть много опций или настройки зависят от условий.
* **Инкапсуляция логики создания:**\
  Вся логика построения объекта находится внутри билдера, что позволяет клиентскому коду не знать о деталях создания.
* **Читаемость и поддержка:**\
  С использованием цепочки вызовов (chaining) код становится интуитивно понятным, так как каждый вызов отражает конкретное действие по настройке объекта.

***

### Пример

```typescript
/**
 * Интерфейс Строителя объявляет создающие методы для различных частей объектов
 * Продуктов.
 */
interface Builder {
    producePartA(): void;
    producePartB(): void;
    producePartC(): void;
}

/**
 * Классы Конкретного Строителя следуют интерфейсу Строителя и предоставляют
 * конкретные реализации шагов построения. Ваша программа может иметь несколько
 * вариантов Строителей, реализованных по-разному.
 */
class ConcreteBuilder1 implements Builder {
    private product: Product1;

    /**
     * Новый экземпляр строителя должен содержать пустой объект продукта,
     * который используется в дальнейшей сборке.
     */
    constructor() {
        this.reset();
    }

    public reset(): void {
        this.product = new Product1();
    }

    /**
     * Все этапы производства работают с одним и тем же экземпляром продукта.
     */
    public producePartA(): void {
        this.product.parts.push('PartA1');
    }

    public producePartB(): void {
        this.product.parts.push('PartB1');
    }

    public producePartC(): void {
        this.product.parts.push('PartC1');
    }

    /**
     * Конкретные Строители должны предоставить свои собственные методы
     * получения результатов. Это связано с тем, что различные типы строителей
     * могут создавать совершенно разные продукты с разными интерфейсами.
     * Поэтому такие методы не могут быть объявлены в базовом интерфейсе
     * Строителя (по крайней мере, в статически типизированном языке
     * программирования).
     *
     * Как правило, после возвращения конечного результата клиенту, экземпляр
     * строителя должен быть готов к началу производства следующего продукта.
     * Поэтому обычной практикой является вызов метода сброса в конце тела
     * метода getProduct. Однако такое поведение не является обязательным, вы
     * можете заставить своих строителей ждать явного запроса на сброс из кода
     * клиента, прежде чем избавиться от предыдущего результата.
     */
    public getProduct(): Product1 {
        const result = this.product;
        this.reset();
        return result;
    }
}

/**
 * Имеет смысл использовать паттерн Строитель только тогда, когда ваши продукты
 * достаточно сложны и требуют обширной конфигурации.
 *
 * В отличие от других порождающих паттернов, различные конкретные строители
 * могут производить несвязанные продукты. Другими словами, результаты различных
 * строителей могут не всегда следовать одному и тому же интерфейсу.
 */
class Product1 {
    public parts: string[] = [];

    public listParts(): void {
        console.log(`Product parts: ${this.parts.join(', ')}\n`);
    }
}

/**
 * Директор отвечает только за выполнение шагов построения в определённой
 * последовательности. Это полезно при производстве продуктов в определённом
 * порядке или особой конфигурации. Строго говоря, класс Директор необязателен,
 * так как клиент может напрямую управлять строителями.
 */
class Director {
    private builder: Builder;

    /**
     * Директор работает с любым экземпляром строителя, который передаётся ему
     * клиентским кодом. Таким образом, клиентский код может изменить конечный
     * тип вновь собираемого продукта.
     */
    public setBuilder(builder: Builder): void {
        this.builder = builder;
    }

    /**
     * Директор может строить несколько вариаций продукта, используя одинаковые
     * шаги построения.
     */
    public buildMinimalViableProduct(): void {
        this.builder.producePartA();
    }

    public buildFullFeaturedProduct(): void {
        this.builder.producePartA();
        this.builder.producePartB();
        this.builder.producePartC();
    }
}

/**
 * Клиентский код создаёт объект-строитель, передаёт его директору, а затем
 * инициирует процесс построения. Конечный результат извлекается из объекта-
 * строителя.
 */
function clientCode(director: Director) {
    const builder = new ConcreteBuilder1();
    director.setBuilder(builder);

    console.log('Standard basic product:');
    director.buildMinimalViableProduct();
    builder.getProduct().listParts();

    console.log('Standard full featured product:');
    director.buildFullFeaturedProduct();
    builder.getProduct().listParts();

    // Помните, что паттерн Строитель можно использовать без класса Директор.
    console.log('Custom product:');
    builder.producePartA();
    builder.producePartC();
    builder.getProduct().listParts();
}

const director = new Director();
clientCode(director)
```







</details>

<details>

<summary>Prototype (Прототип)</summary>

### Основные идеи паттерна «Прототип»

* **Клонирование объектов:** Вместо того чтобы создавать новый объект с нуля, можно взять уже существующий объект (прототип) и скопировать его.
* **Экономия ресурсов:** При клонировании не создаются дубликаты однотипных методов и свойств для каждого экземпляра. Это особенно полезно, когда создание объекта является «дорогой» операцией (по памяти или времени).
* **Гибкость:** Прототип позволяет динамически изменять объекты во время выполнения, добавляя или изменяя свойства и методы.
* **Наследование:** В JavaScript механизм прототипного наследования позволяет объектам наследовать свойства и методы от других объектов, что является основой работы многих паттернов.

***

### Пример без использования паттерна «Прототип»

В этом примере каждый новый объект создаёт свою копию метода, что приводит к дублированию кода и увеличению потребления памяти:

```javascript
function Car(model, year) {
    this.model = model;
    this.year = year;
    // Каждый экземпляр создаёт собственную функцию
    this.getInfo = function() {
        return `Модель: ${this.model}, Год: ${this.year}`;
    };
}

let car1 = new Car("Toyota", 2020);
let car2 = new Car("Honda", 2019);

console.log(car1.getInfo()); // Модель: Toyota, Год: 2020
console.log(car2.getInfo()); // Модель: Honda, Год: 2019
```

**Недостатки:**

* При создании большого количества объектов метод `getInfo` будет дублироваться в памяти для каждого экземпляра, что неэффективно.

***

### Пример с использованием паттерна «Прототип»

Здесь мы выносим общий метод в прототип конструктора, благодаря чему все экземпляры будут ссылаться на один и тот же метод:

```javascript
function Car(model, year) {
    this.model = model;
    this.year = year;
}

// Метод getInfo добавлен в прототип, а не создаётся для каждого экземпляра
Car.prototype.getInfo = function() {
    return `Модель: ${this.model}, Год: ${this.year}`;
};

let car1 = new Car("Toyota", 2020);
let car2 = new Car("Honda", 2019);

console.log(car1.getInfo()); // Модель: Toyota, Год: 2020
console.log(car2.getInfo()); // Модель: Honda, Год: 2019
```

**Преимущества:**

* **Экономия памяти:** Метод `getInfo` создаётся один раз и разделяется всеми экземплярами.
* **Производительность:** Уменьшается время создания объектов, так как не требуется создавать копию метода для каждого объекта.
* **Управляемость:** Легче вносить изменения в метод, поскольку изменение в прототипе сразу отражается на всех объектах.

</details>

<details>

<summary>Singleton (Одиночка)</summary>

### Основные идеи паттерна «Синглтон»

* **Единственный экземпляр:** Гарантирует, что класс или объект имеет только один экземпляр в приложении.
* **Глобальная точка доступа:** Предоставляет централизованный и единый доступ к этому экземпляру.
* **Контроль над ресурсами:** Используется для управления ресурсами, которые должны быть уникальными (например, подключение к базе данных, конфигурационные параметры, логгеры).

***

### Пример без использования паттерна «Синглтон»

Если создавать объекты напрямую, каждый вызов конструктора приведёт к созданию нового экземпляра, что может вызвать проблемы с согласованностью данных.

```javascript
function Configuration() {
    this.settings = {
        theme: "dark",
        language: "ru"
    };
}

let config1 = new Configuration();
let config2 = new Configuration();

console.log(config1 === config2); // false
```

**Проблема:**\
При создании нескольких экземпляров конфигурации может возникнуть рассинхронизация настроек: изменения в одном экземпляре не будут отражаться в другом.

***

### Пример с использованием паттерна «Синглтон»

Для создания единственного экземпляра объекта можно использовать немедленно вызываемую функциональную конструкцию (IIFE), которая внутри хранит ссылку на созданный экземпляр.

```javascript
const ConfigurationSingleton = (function() {
    let instance;

    function init() {
        // Приватное состояние и методы
        let settings = {
            theme: "dark",
            language: "ru"
        };

        return {
            // Публичный API
            getSettings: function() {
                return settings;
            },
            setSetting: function(key, value) {
                settings[key] = value;
            }
        };
    }

    return {
        // Метод для получения экземпляра
        getInstance: function() {
            if (!instance) {
                instance = init();
            }
            return instance;
        }
    };
})();

// Пример использования:
let configA = ConfigurationSingleton.getInstance();
let configB = ConfigurationSingleton.getInstance();

console.log(configA === configB); // true

// Изменение через один экземпляр отражается в другом
configA.setSetting("theme", "light");
console.log(configB.getSettings().theme); // light
```

**Преимущества использования синглтона:**

* **Единственность:** Гарантируется, что объект конфигурации создаётся только один раз.
* **Централизованный контроль:** Изменения в объекте отражаются глобально, что упрощает управление состоянием приложения.
* **Экономия ресурсов:** Не происходит лишнего создания экземпляров, что может быть критичным при работе с ресурсозатратными объектами.

</details>

#### 2. Structural Patterns (Структурные паттерны)

**Описание:**\
Структурные паттерны помогают организовать классы и объекты в более крупные структуры, упрощая их взаимодействие.

<details>

<summary>Adapter (Адаптер)</summary>

### Основные идеи паттерна «Адаптер»

* **Совместимость:** Позволяет объектам с несовместимыми интерфейсами работать вместе.
* **Инкапсуляция преобразований:** Адаптер скрывает различия между интерфейсами и предоставляет унифицированный API.
* **Гибкость:** Позволяет интегрировать сторонние библиотеки, API или устаревший код в новое приложение без изменения их исходного кода.

### Пример без использования паттерна «Адаптер»

Допустим, у нас есть устаревший сервис, который возвращает данные в неудачном формате:

```javascript
class OldAPI {
  fetchData() {
    return {
      user_data: {
        user_name: "Alice",
        user_age: 25
      }
    };
  }
}

// Новый код ожидает другой формат данных
function displayUser(user) {
  console.log(`Имя: ${user.name}, Возраст: ${user.age}`);
}

const oldApi = new OldAPI();
const user = oldApi.fetchData(); 

// displayUser(user); // ОШИБКА: user.name не определён
```

**Проблема:**\
Функция `displayUser` ожидает данные в формате `{ name, age }`, но старая API возвращает `{ user_data: { user_name, user_age } }`. Вызов функции приведёт к ошибке.

***

### Решение с использованием паттерна «Адаптер»

Создадим адаптер, который преобразует данные в нужный формат:

```javascript
class OldAPIAdapter {
  constructor(oldApi) {
    this.oldApi = oldApi;
  }

  getUser() {
    const oldData = this.oldApi.fetchData();
    return {
      name: oldData.user_data.user_name,
      age: oldData.user_data.user_age
    };
  }
}

const adaptedApi = new OldAPIAdapter(new OldAPI());
const adaptedUser = adaptedApi.getUser();
displayUser(adaptedUser); // Имя: Alice, Возраст: 25
```

**Преимущества адаптера:**

* Код `displayUser` **остался неизменным**.
* `OldAPIAdapter` **скрывает** детали преобразования и делает API совместимым.
* **Можно легко заменить** `OldAPI` на новую версию, просто изменив адаптер.

</details>

<details>

<summary>Bridge (Мост)</summary>

### Основные идеи паттерна «Bridge»

* **Разделение абстракции и реализации:**\
  Паттерн позволяет отделить высокоуровневую абстракцию от низкоуровневой реализации так, чтобы они могли изменяться независимо друг от друга.
* **Независимое расширение:**\
  Благодаря разделению можно независимо добавлять новые абстракции (расширять бизнес-логику) и новые реализации (например, различные способы рендеринга или платформы).
* **Гибкость:**\
  Изменения в одной иерархии (абстракции или реализации) не требуют изменений в другой, что облегчает масштабирование и поддержку кода.

***

### Пример

Представим, что нам нужно рисовать геометрические фигуры, но способ их отрисовки может варьироваться в зависимости от используемой графической библиотеки или платформы. Для этого выделим две независимые иерархии:

1. **Абстракция:** Определяет, какие фигуры мы можем рисовать (например, круг, квадрат).
2. **Реализация:** Определяет, как именно рисовать фигуру (например, с использованием API первой библиотеки или второй).

#### Шаг 1. Определим интерфейс для реализации рисования

```javascript
// Интерфейс для реализации (DrawingAPI)
class DrawingAPI {
  drawCircle(x, y, radius) {
    throw new Error("Метод не реализован");
  }
}
```

#### Шаг 2. Создадим конкретные реализации

```javascript
// Первая реализация рисования
class DrawingAPI1 extends DrawingAPI {
  drawCircle(x, y, radius) {
    console.log(`API1: Рисую круг с центром (${x}, ${y}) и радиусом ${radius}`);
  }
}

// Вторая реализация рисования
class DrawingAPI2 extends DrawingAPI {
  drawCircle(x, y, radius) {
    console.log(`API2: Рисую круг с центром (${x}, ${y}) и радиусом ${radius}`);
  }
}
```

#### Шаг 3. Определим абстракцию

Абстракция содержит ссылку на реализацию, которую можно подменять.

```javascript
// Абстракция для фигуры
class Shape {
  constructor(drawingAPI) {
    this.drawingAPI = drawingAPI;
  }

  draw() {
    throw new Error("Метод не реализован");
  }
}
```

#### Шаг 4. Создадим конкретную фигуру (расширение абстракции)

Например, класс «Круг», который использует реализацию для рисования.

```javascript
// Конкретная фигура: Круг
class Circle extends Shape {
  constructor(x, y, radius, drawingAPI) {
    super(drawingAPI);
    this.x = x;
    this.y = y;
    this.radius = radius;
  }

  draw() {
    this.drawingAPI.drawCircle(this.x, this.y, this.radius);
  }
}
```

#### Шаг 5. Использование

Теперь можно создавать объекты, комбинируя абстракцию с разными реализациями:

```javascript
// Создаём круг, используя первую реализацию рисования
const circle1 = new Circle(10, 20, 5, new DrawingAPI1());
circle1.draw(); // API1: Рисую круг с центром (10, 20) и радиусом 5

// Создаём круг, используя вторую реализацию рисования
const circle2 = new Circle(15, 25, 8, new DrawingAPI2());
circle2.draw(); // API2: Рисую круг с центром (15, 25) и радиусом 8
```

**Преимущества такого подхода:**

* Абстракция `Circle` и реализация рисования (DrawingAPI1, DrawingAPI2) развиваются независимо.
* При появлении новой графической библиотеки достаточно реализовать новый класс, наследующий `DrawingAPI`, без изменения логики фигур.
* Облегчается поддержка и расширение функциональности.

</details>

<details>

<summary>Composite (Компоновщик)</summary>

### **Основные идеи паттерна Composite**

* **Рекурсивная структура:** Позволяет строить иерархию объектов, где отдельные элементы и их контейнеры обрабатываются одинаково.
* **Единый интерфейс:** Клиенту не важно, работает ли он с одиночным объектом или с группой.
* **Гибкость:** Можно легко добавлять новые элементы в структуру без изменения существующего кода.
* **Упрощённая работа с деревьями:** Часто используется для представления графических интерфейсов, меню, файловых систем и DOM.

***

#### **Пример древовидной структуры (меню)**

Допустим, у нас есть многоуровневое меню, где пункты могут быть как **простыми ссылками**, так и **вложенными списками**.

**Без Composite (неоптимальный вариант)**

```tsx
const MenuItem = ({ label }) => <li>{label}</li>;

const Menu = ({ items }) => (
  <ul>
    {items.map(item =>
      item.submenu ? (
        <li key={item.label}>
          {item.label}
          <Menu items={item.submenu} />
        </li>
      ) : (
        <MenuItem key={item.label} label={item.label} />
      )
    )}
  </ul>
);
```

**Недостатки:**\
Мы **разделяем** логику рендеринга `MenuItem` и `Menu`, хотя можно было бы использовать общий интерфейс.

***

#### **Использование Composite**

Теперь и `MenuItem`, и `Menu` реализуют **единый интерфейс** – они рендерят `children`.

```tsx
const MenuComponent = ({ label, children }) => (
  <li>
    {label}
    {children && <ul>{children}</ul>}
  </li>
);

const Menu = ({ items }) => (
  <ul>
    {items.map(item => (
      <MenuComponent key={item.label} label={item.label}>
        {item.submenu && <Menu items={item.submenu} />}
      </MenuComponent>
    ))}
  </ul>
);
```

Теперь и `Menu`, и `MenuComponent` **имеют одинаковую структуру** и могут работать как отдельные элементы или контейнеры.

**Использование:**

```tsx
const menuData = [
  { label: "Home" },
  {
    label: "Products",
    submenu: [
      { label: "Phones" },
      { label: "Laptops" },
      { label: "Accessories" }
    ]
  },
  { label: "About" }
];

const App = () => <Menu items={menuData} />;
```

**Преимущества:**

* **Единый интерфейс** для работы с элементами меню.
* **Рекурсивность**: любое меню может содержать вложенные элементы без изменения кода.

</details>

<details>

<summary>Decorator (Декоратор)</summary>

### **Основные идеи паттерна Decorator**

* **Гибкое расширение**: Позволяет динамически добавлять функциональность без изменения основного класса.
* **Принцип открытости/закрытости**: Код остаётся открытым для расширения, но закрытым для модификации.
* **Композиция вместо наследования**: Декоратор использует композицию, а не классическое наследование, что делает код более гибким.
* **Многоуровневое декорирование**: Можно накладывать несколько декораторов последовательно.

Декоратор имеет альтернативное название — _обёртка_. Оно более точно описывает суть паттерна: вы помещаете целевой объект в другой объект-обёртку, который запускает базовое поведение объекта, а затем добавляет к результату что-то своё.

Оба объекта имеют общий интерфейс, поэтому для пользователя нет никакой разницы, с каким объектом работать — чистым или обёрнутым. Вы можете использовать несколько разных обёрток одновременно — результат будет иметь объединённое поведение всех обёрток сразу.

***

В этом примере **Декоратор** защищает финансовые данные дополнительными уровнями безопасности прозрачно для кода, который их использует.

```typescript
// Общий интерфейс компонентов
interface DataSource {
    writeData(data: string): void;
    readData(): string;
}

// Один из конкретных компонентов реализует базовую функциональность
class FileDataSource implements DataSource {
    private filename: string;

    constructor(filename: string) {
        this.filename = filename;
    }

    writeData(data: string): void {
        console.log(`Запись данных в файл ${this.filename}`);
    }

    readData(): string {
        console.log(`Чтение данных из файла ${this.filename}`);
        return "данные из файла";
    }
}

// Родитель всех декораторов содержит код обёртывания
class DataSourceDecorator implements DataSource {
    protected wrappee: DataSource;

    constructor(source: DataSource) {
        this.wrappee = source;
    }

    writeData(data: string): void {
        this.wrappee.writeData(data);
    }

    readData(): string {
        return this.wrappee.readData();
    }
}

// Конкретные декораторы
class EncryptionDecorator extends DataSourceDecorator {
    writeData(data: string): void {
        // Шифрование данных
        const encryptedData = `Зашифрованные(${data})`;
        this.wrappee.writeData(encryptedData);
    }

    readData(): string {
        // Расшифровка данных
        const data = this.wrappee.readData();
        return data.replace("Зашифрованные(", "").replace(")", "");
    }
}

class CompressionDecorator extends DataSourceDecorator {
    writeData(data: string): void {
        // Сжатие данных
        const compressedData = `Сжатые(${data})`;
        this.wrappee.writeData(compressedData);
    }

    readData(): string {
        // Расшифровка данных
        const data = this.wrappee.readData();
        return data.replace("Сжатые(", "").replace(")", "");
    }
}

// Клиентский код, использующий внешний источник данных.
// Класс SalaryManager ничего не знает о том, как именно
// будут считаны и записаны данные. Он получает уже готовый
// источник данных.
class SalaryManager {
    private source: DataSource;

    constructor(source: DataSource) {
        this.source = source;
    }

    load(): string {
        return this.source.readData();
    }

    save(): void {
        this.source.writeData("salaryRecords");
    }
}

// Приложение может по-разному собирать декорируемые объекты, в
// зависимости от условий использования.
class ApplicationConfigurator {
    configurationExample(): void {
        let source: DataSource = new FileDataSource("salary.dat");
        let enabledEncryption = true;
        let enabledCompression = true;

        if (enabledEncryption) {
            source = new EncryptionDecorator(source);
        }
        if (enabledCompression) {
            source = new CompressionDecorator(source);
        }

        const logger = new SalaryManager(source);
        const salary = logger.load();
    }
}
```



</details>

<details>

<summary>Facade (Фасад)</summary>

### Основные идеи паттерна Facade

* **Упрощение интерфейса:** Клиент не должен разбираться в тонкостях работы сложной системы – фасад предоставляет интуитивно понятный API.
* **Сокрытие сложности:** Детали реализации, взаимодействия между компонентами и последовательность вызовов скрываются за фасадом.
* **Унификация доступа:** Позволяет объединить несколько подсистем под единым интерфейсом, что облегчает их использование и замену.
* **Изоляция клиента от изменений:** При модификации внутренней логики системы изменения минимально затрагивают клиентский код.

### Пример

Опишем три сервиса, отвечающие за разные аспекты работы системы:

```javascript
// AuthService отвечает за аутентификацию
class AuthService {
  login(username, password) {
    console.log(`Аутентификация пользователя: ${username}`);
    // Здесь можно добавить реальную логику аутентификации
    return { token: "abcd1234", user: username };
  }
}

// DataService отвечает за получение данных с сервера
class DataService {
  fetchData(token) {
    console.log(`Получение данных с токеном: ${token}`);
    // Имитация получения данных
    return { data: [1, 2, 3] };
  }
}

// NotificationService отвечает за уведомления
class NotificationService {
  notify(message) {
    console.log(`Уведомление: ${message}`);
  }
}
```

#### Фасад

Фасад теперь реализован так, чтобы каждое действие выполнялось отдельно. Фасад хранит состояние (например, токен и имя пользователя) после входа:

```javascript
class AppFacade {
  constructor() {
    this.authService = new AuthService();
    this.dataService = new DataService();
    this.notificationService = new NotificationService();
    this.token = null;
    this.user = null;
  }
  
  // Метод для аутентификации
  login(username, password) {
    const authResult = this.authService.login(username, password);
    this.token = authResult.token;
    this.user = authResult.user;
    this.notificationService.notify("Вход выполнен успешно!");
    return authResult;
  }
  
  // Метод для получения данных; требует предварительного входа
  fetchData() {
    if (!this.token) {
      throw new Error("Ошибка: Пользователь не аутентифицирован");
    }
    return this.dataService.fetchData(this.token);
  }
}
```

**Преимущества данного подхода:**

* **Модульность:** Каждый метод выполняет только одну задачу.
* **Управляемость:** Клиент сам решает, когда выполнять вход и когда получать данные.
* **Простота поддержки:** При изменении логики одного из сервисов достаточно изменить только соответствующий метод фасада.

</details>

<details>

<summary>Flyweight (Кэш/Приспособленец)</summary>

Паттерн **Flyweight** используется для уменьшения расхода памяти при работе с большим числом объектов, которые имеют общие (внутренние) свойства. Идея состоит в том, чтобы разделить объекты на две части:

* **Внутреннее (intrinsic) состояние:** общее для множества объектов (например, цвет, текстура, тип). Эти данные хранятся в одном экземпляре и разделяются между объектами.
* **Внешнее (extrinsic) состояние:** уникальные данные (например, позиция, контекст использования), которые передаются извне при использовании объекта.

Такой подход особенно полезен, если приложение должно создавать сотни или тысячи подобных объектов, поскольку разделение общих данных позволяет значительно сократить потребление памяти.

***

### Пример

Рассмотрим классический пример с деревьями в лесу:\
Каждое дерево имеет уникальные координаты, но тип дерева (название, цвет, текстура) может быть общим для множества экземпляров.

#### Определение Flyweight объектов (тип дерева)

```javascript
// Flyweight объект – содержит общее состояние для деревьев одного типа
class TreeType {
  constructor(name, color, texture) {
    this.name = name;
    this.color = color;
    this.texture = texture;
  }
  
  draw(x, y) {
    console.log(
      `Рисую дерево ${this.name} на координатах (${x}, ${y}) с цветом ${this.color} и текстурой ${this.texture}`
    );
  }
}
```

#### Фабрика Flyweight

Фабрика управляет созданием и кешированием Flyweight объектов:

```javascript
const TreeTypeFactory = {
  treeTypes: {},
  
  getTreeType(name, color, texture) {
    const key = `${name}_${color}_${texture}`;
    if (!this.treeTypes[key]) {
      this.treeTypes[key] = new TreeType(name, color, texture);
    }
    return this.treeTypes[key];
  }
};
```

#### Класс, использующий Flyweight

Каждое дерево хранит только свою уникальную позицию и ссылку на объект типа (Flyweight):

```javascript
class Tree {
  constructor(x, y, treeType) {
    this.x = x;
    this.y = y;
    this.treeType = treeType;
  }
  
  draw() {
    // Передаём внешние данные (координаты) в flyweight для отрисовки
    this.treeType.draw(this.x, this.y);
  }
}
```

#### Контейнер (Лес)

Контейнер управляет группой деревьев:

```javascript
class Forest {
  constructor() {
    this.trees = [];
  }
  
  plantTree(x, y, name, color, texture) {
    const treeType = TreeTypeFactory.getTreeType(name, color, texture);
    const tree = new Tree(x, y, treeType);
    this.trees.push(tree);
  }
  
  draw() {
    this.trees.forEach(tree => tree.draw());
  }
}

// Пример использования:
const forest = new Forest();
forest.plantTree(10, 20, "Дуб", "green", "rough");
forest.plantTree(15, 25, "Дуб", "green", "rough");
forest.plantTree(30, 40, "Сосна", "darkgreen", "smooth");
forest.draw();
```

В данном примере для деревьев типа «Дуб» будет создан один экземпляр класса `TreeType`, который используется всеми деревьями этого типа. Это позволяет экономить память при большом количестве объектов.

</details>

<details>

<summary>Proxy (Заместитель)</summary>

### Основные идеи паттерна Proxy

* **Контроль доступа:** Прокси может проверять, кто и когда обращается к реальному объекту.
* **Ленивое создание:** Реальный объект может создаваться только при первом обращении к нему.
* **Кэширование:** Прокси может сохранять результаты дорогостоящих операций, чтобы не выполнять их повторно.
* **Логирование и аудит:** Все вызовы методов могут фиксироваться для отладки или аудита.
* **Безопасность:** Прокси может ограничивать доступ к методам реального объекта.

***

### Пример

Мы создадим функцию-фабрику, которая принимает исходный объект и callback-функцию для обработки записей об изменениях. Прокси перехватывает операции записи (и удаления) и вызывает callback, чтобы сохранить информацию об изменениях.

```javascript
// Функция для создания прокси с логированием изменений
function createTrackedObject(initialObj, onChange) {
  return new Proxy(initialObj, {
    set(target, prop, value) {
      const oldValue = target[prop];
      target[prop] = value;
      
      const changeRecord = {
        property: prop,
        oldValue,
        newValue: value,
        timestamp: new Date().toLocaleTimeString()
      };
      onChange(changeRecord);
      
      return true;
    },
    deleteProperty(target, prop) {
      const oldValue = target[prop];
      const result = delete target[prop];
      
      const changeRecord = {
        property: prop,
        oldValue,
        newValue: undefined,
        action: "delete",
        timestamp: new Date().toLocaleTimeString()
      };
      onChange(changeRecord);
      
      return result;
    }
  });
}

// Пример использования:
const changeLog = [];

const trackedDoc = createTrackedObject(
  { title: "Initial Title", content: "Initial Content" },
  (changeRecord) => {
    changeLog.push(changeRecord);
    console.log("Изменение:", changeRecord);
  }
);

// Внесем несколько изменений:
trackedDoc.title = "New Title";           // Логируется изменение свойства title
trackedDoc.content = "Updated Content";   // Логируется изменение свойства content
delete trackedDoc.content;                // Логируется удаление свойства content

console.log("Все изменения:", changeLog);
```

В этом примере каждый раз, когда меняется свойство объекта или оно удаляется, вызывается callback, который записывает информацию об изменении в массив `changeLog` и выводит данные в консоль.

</details>

#### 3. Behavioral Patterns (Поведенческие паттерны)

**Описание:**\
Поведенческие паттерны решают задачи эффективного и безопасного взаимодействия между объектами программы.

<details>

<summary>Chain of Responsibility (Цепочка обязанностей)</summary>

#### **Основные принципы:**

1. **Разделение ответственности** – обработка запроса может происходить на любом этапе цепочки.
2. **Гибкость в обработке** – добавление новых обработчиков или изменение логики не требует изменения существующих классов.
3. **Принцип единственной ответственности (SRP)** – каждый обработчик отвечает только за свою часть работы.
4. **Принцип открытости/закрытости (OCP)** – можно добавлять новые обработчики без изменения существующего кода.

### **Перевод**

```javascript
// Базовый класс компонента
class Component {
  constructor(tooltipText = null) {
    this.tooltipText = tooltipText; // Подсказка
    this.container = null; // Родительский контейнер
  }

  showHelp() {
    if (this.tooltipText) {
      console.log(`Tooltip: ${this.tooltipText}`); // Показываем всплывающую подсказку
    } else if (this.container) {
      this.container.showHelp(); // Передаём запрос выше
    }
  }
}

// Класс контейнера, который может содержать другие компоненты
class Container extends Component {
  constructor() {
    super();
    this.children = []; // Дочерние элементы
  }

  add(child) {
    this.children.push(child);
    child.container = this; // Устанавливаем родительский контейнер
  }
}

// Кнопка – простой компонент
class Button extends Component {
  constructor(tooltipText) {
    super(tooltipText);
  }
}

// Панель – контейнер, который может переопределять showHelp()
class Panel extends Container {
  constructor(modalHelpText = null) {
    super();
    this.modalHelpText = modalHelpText;
  }

  showHelp() {
    if (this.modalHelpText) {
      console.log(`Modal Help: ${this.modalHelpText}`); // Показываем модальное окно
    } else {
      super.showHelp(); // Если нет текста, передаём запрос выше
    }
  }
}

// Диалог – ещё один контейнер, который может иметь ссылку на Wiki
class Dialog extends Container {
  constructor(wikiPageURL = null) {
    super();
    this.wikiPageURL = wikiPageURL;
  }

  showHelp() {
    if (this.wikiPageURL) {
      console.log(`Opening Wiki page: ${this.wikiPageURL}`); // Открываем Wiki
    } else {
      super.showHelp(); // Если ссылки нет, передаём запрос дальше
    }
  }
}

// Клиентский код
class Application {
  createUI() {
    // Создаём диалоговое окно
    this.dialog = new Dialog("http://example.com/help");
    
    // Создаём панель
    this.panel = new Panel("This panel helps with settings");

    // Создаём кнопки
    this.okButton = new Button("This is an OK button");
    this.cancelButton = new Button(); // У этой кнопки нет подсказки

    // Настраиваем иерархию
    this.panel.add(this.okButton);
    this.panel.add(this.cancelButton);
    this.dialog.add(this.panel);
  }

  onF1KeyPress(component) {
    component.showHelp();
  }
}
```

### **Как здесь работает Chain of Responsibility?**

1. Если у компонента есть своя справочная информация, он её показывает.
2. Если нет – он **передаёт запрос выше** (по цепочке контейнеров).
3. В итоге информация **всегда найдётся** либо в родителях, либо в корневом контейнере.

Такой паттерн полезен в UI-фреймворках (React, Vue) и системах обработки событий.

</details>

<details>

<summary>Command (Команда)</summary>

#### Основные принципы:

1. **Команда (Command):** Определяет интерфейс с методом, например, `execute()`, который будет выполняться.
2. **Конкретная команда (ConcreteCommand):** Реализует интерфейс команды и связывает получателя с конкретным действием.
3. **Получатель (Receiver):** Класс, содержащий бизнес-логику, которая реально выполняет операцию.
4. **Отправитель (Invoker):** Вызывает команды, не зная, что именно происходит внутри. Обычно предоставляет интерфейс для установки команды и её выполнения.
5. **Клиент:** Создает объекты-команды и связывает их с конкретными получателями.

#### Пример

Ниже приведен пример, иллюстрирующий управление освещением с помощью пульта дистанционного управления. Мы создадим интерфейс `Command`, класс-получатель `Light` с методами включения и выключения, две конкретные команды `LightOnCommand` и `LightOffCommand`, а также класс `RemoteControl`, который является отправителем команд.

```typescript
// Интерфейс команды
interface Command {
    execute(): void;
}

// Получатель: класс, который знает, как выполнять операции
class Light {
    turnOn(): void {
        console.log("Свет включен");
    }

    turnOff(): void {
        console.log("Свет выключен");
    }
}

// Конкретная команда для включения света
class LightOnCommand implements Command {
    private light: Light;

    constructor(light: Light) {
        this.light = light;
    }

    execute(): void {
        console.log("Выполняется команда включения света");
        this.light.turnOn();
    }
}

// Конкретная команда для выключения света
class LightOffCommand implements Command {
    private light: Light;

    constructor(light: Light) {
        this.light = light;
    }

    execute(): void {
        console.log("Выполняется команда выключения света");
        this.light.turnOff();
    }
}

// Отправитель (Invoker): объект, который вызывает команды
class RemoteControl {
    private command!: Command;

    // Устанавливаем команду
    setCommand(command: Command): void {
        this.command = command;
    }

    // "Нажатие кнопки" вызывает выполнение команды
    pressButton(): void {
        console.log("Нажата кнопка пульта");
        this.command.execute();
    }
}

// Клиентский код
const light = new Light();
const lightOn = new LightOnCommand(light);
const lightOff = new LightOffCommand(light);

const remote = new RemoteControl();

remote.setCommand(lightOn);
remote.pressButton();  // Ожидаемый вывод: включение света

remote.setCommand(lightOff);
remote.pressButton();  // Ожидаемый вывод: выключение света
```

#### Объяснение работы кода

1. **Интерфейс Command:** Объявляет метод `execute()`, который обязаны реализовать все команды.
2. **Класс Light:** Содержит логику для включения и выключения света.
3. **Конкретные команды:**
   * `LightOnCommand` в методе `execute()` вызывает метод `turnOn()` у объекта `Light`.
   * `LightOffCommand` в методе `execute()` вызывает метод `turnOff()` у объекта `Light`.
4. **RemoteControl (Invoker):**
   * Позволяет установить конкретную команду через метод `setCommand()`.
   * Метод `pressButton()` вызывает `execute()` у установленной команды, не зная, какая именно команда будет выполнена.
5. **Клиент:** Создает получатель и команды, а затем через пульт (Invoker) выполняет нужные действия.

Паттерн Command повышает гибкость архитектуры, позволяя легко добавлять новые команды, комбинировать их, ставить в очередь или отменять. Это особенно полезно в случаях, когда требуется отделить инициатора действия от его реализации.

</details>

<details>

<summary>Iterator (Итератор)</summary>

#### Основные принципы:

* **Инкапсуляция обхода:** Логика перебора элементов скрыта внутри итератора, а клиент использует единый интерфейс.
* **Отделение алгоритма обхода от структуры данных:** Коллекция предоставляет метод для получения итератора, а сам итератор знает, как перемещаться по элементам.
* **Единый интерфейс:** Итератор обычно предоставляет методы вроде `next()` для получения следующего элемента и `hasNext()` для проверки наличия ещё элементов.

### Пример

```typescript
// Общий интерфейс коллекций должен определить фабричный метод
// для производства итератора. Можно определить сразу несколько
// методов, чтобы дать пользователям различные варианты обхода
// одной и той же коллекции.
interface SocialNetwork {
    createFriendsIterator(profileId: string): ProfileIterator;
    createCoworkersIterator(profileId: string): ProfileIterator;
}

// Интерфейс профиля
interface Profile {
    getEmail(): string;
    getId(): string;
}

// Конкретная коллекция знает, объекты каких итераторов нужно
// создавать.
class Facebook implements SocialNetwork {
    // ...Основной код коллекции...

    // Код получения нужного итератора.
    createFriendsIterator(profileId: string): ProfileIterator {
        return new FacebookIterator(this, profileId, "friends");
    }
    createCoworkersIterator(profileId: string): ProfileIterator {
        return new FacebookIterator(this, profileId, "coworkers");
    }

    // Симуляция запроса социальных связей
    socialGraphRequest(profileId: string, type: string): Profile[] {
        console.log(`Запрос социальных связей для profileId=${profileId} типа ${type}`);
        return []; // Возвращаем пустой массив для примера
    }
}

// Общий интерфейс итераторов.
interface ProfileIterator {
    getNext(): Profile | null;
    hasMore(): boolean;
}

// Конкретный итератор.
class FacebookIterator implements ProfileIterator {
    // Итератору нужна ссылка на коллекцию, которую он обходит.
    private facebook: Facebook;
    private profileId: string;
    private type: string;

    // Но каждый итератор обходит коллекцию, независимо от
    // остальных, поэтому он содержит информацию о текущей
    // позиции обхода.
    private currentPosition: number = 0;
    private cache: Profile[] | null = null;

    constructor(facebook: Facebook, profileId: string, type: string) {
        this.facebook = facebook;
        this.profileId = profileId;
        this.type = type;
    }

    private lazyInit(): void {
        if (this.cache === null) {
            this.cache = this.facebook.socialGraphRequest(this.profileId, this.type);
        }
    }

    // Итератор реализует методы базового интерфейса по-своему.
    getNext(): Profile | null {
        if (this.hasMore()) {
            const result = this.cache![this.currentPosition];
            this.currentPosition++;
            return result;
        }
        return null;
    }

    hasMore(): boolean {
        this.lazyInit();
        return this.cache !== null && this.currentPosition < this.cache.length;
    }
}

// Вот ещё полезная тактика: мы можем передавать объект
// итератора вместо коллекции в клиентские классы. При таком
// подходе клиентский код не будет иметь доступа к коллекциям, а
// значит, его не будут волновать подробности их реализаций. Ему
// будет доступен только общий интерфейс итераторов.
class SocialSpammer {
    send(iterator: ProfileIterator, message: string): void {
        while (iterator.hasMore()) {
            const profile = iterator.getNext();
            if (profile) {
                // System.sendEmail(profile.getEmail(), message)
                console.log(`Отправка письма на ${profile.getEmail()}: ${message}`);
            }
        }
    }
}

// Класс приложение конфигурирует классы, как захочет.
class Application {
    private network!: SocialNetwork;
    private spammer!: SocialSpammer;

    config(): void {
        // if working with Facebook
        this.network = new Facebook();
        // if working with LinkedIn
        // this.network = new LinkedIn();
        this.spammer = new SocialSpammer();
    }

    sendSpamToFriends(profile: Profile): void {
        const iterator = this.network.createFriendsIterator(profile.getId());
        this.spammer.send(iterator, "Very important message");
    }

    sendSpamToCoworkers(profile: Profile): void {
        const iterator = this.network.createCoworkersIterator(profile.getId());
        this.spammer.send(iterator, "Very important message");
    }
}
```

Паттерн Iterator позволяет легко менять способ обхода коллекции, поддерживать различные алгоритмы итерации или использовать единый интерфейс для разных структур данных.

</details>

<details>

<summary>Mediator (Посредник)</summary>

### Основные концепции

1. **Централизация коммуникаций:** Вместо того чтобы объекты напрямую взаимодействовали друг с другом, все сообщения проходят через посредника. Это упрощает управление взаимодействиями и позволяет легко изменять правила коммуникации без модификации самих объектов.
2. **Инкапсуляция логики взаимодействия:** Посредник знает обо всех коллегах (компонентах) и содержит всю логику, как реагировать на их события. Объекты, в свою очередь, обращаются к посреднику для уведомления о своих изменениях, не зная, кто именно их получит.
3. **Снижение связанности:** Компоненты (коллеги) взаимодействуют только с посредником, что позволяет им оставаться независимыми друг от друга. Это повышает модульность и упрощает повторное использование и тестирование компонентов.
4. **Гибкость и расширяемость:** Централизованная логика коммуникаций позволяет легко добавлять новые виды взаимодействий или изменять поведение системы, не затрагивая самих коллег.

### Пример

```typescript
// Общий интерфейс посредников.
interface Mediator {
    notify(sender: Component, event: string): void;
}

// Конкретный посредник. Все связи между конкретными
// компонентами переехали в код посредника. Он получает
// извещения от своих компонентов и знает, как на них
// реагировать.
class AuthenticationDialog implements Mediator {
    private title: string = "";
    private loginOrRegisterChkBx: Checkbox;
    private loginUsername: Textbox;
    private loginPassword: Textbox;
    private registrationUsername: Textbox;
    private registrationPassword: Textbox;
    private registrationEmail: Textbox;
    private okBtn: Button;
    private cancelBtn: Button;

    constructor() {
        // Здесь нужно создать объекты всех компонентов, подав
        // текущий объект-посредник в их конструктор.
        this.loginOrRegisterChkBx = new Checkbox(this);
        this.loginUsername = new Textbox(this);
        this.loginPassword = new Textbox(this);
        this.registrationUsername = new Textbox(this);
        this.registrationPassword = new Textbox(this);
        this.registrationEmail = new Textbox(this);
        this.okBtn = new Button(this);
        this.cancelBtn = new Button(this);
    }

    // Когда что-то случается с компонентом, он шлёт посреднику
    // оповещение. После получения извещения посредник может
    // либо сделать что-то самостоятельно, либо перенаправить
    // запрос другому компоненту.
    notify(sender: Component, event: string): void {
        if (sender === this.loginOrRegisterChkBx && event === "check") {
            if (this.loginOrRegisterChkBx.checked) {
                this.title = "Log in";
                // 1. Показать компоненты формы входа.
                // 2. Скрыть компоненты формы регистрации.
            } else {
                this.title = "Register";
                // 1. Показать компоненты формы регистрации.
                // 2. Скрыть компоненты формы входа.
            }
        }

        if (sender === this.okBtn && event === "click") {
            if (this.loginOrRegisterChkBx.checked) {
                // Попробовать найти пользователя с данными из
                // формы логина.
                const found = false; // пример проверки, найден пользователь или нет
                if (!found) {
                    // Показать ошибку над формой логина.
                }
            } else {
                // 1. Создать пользовательский аккаунт с данными
                // из формы регистрации.
                // 2. Авторизировать этого пользователя.
                // ...
            }
        }
    }
}

// Классы компонентов общаются с посредниками через их общий
// интерфейс. Благодаря этому одни и те же компоненты можно
// использовать в разных посредниках.
abstract class Component {
    constructor(protected dialog: Mediator) {}

    click(): void {
        this.dialog.notify(this, "click");
    }

    keypress(): void {
        this.dialog.notify(this, "keypress");
    }
}

// Конкретные компоненты не связаны между собой напрямую. У них
// есть только один канал общения — через отправку уведомлений
// посреднику.
class Button extends Component {
    // ...
}

class Textbox extends Component {
    // ...
}

class Checkbox extends Component {
    public checked: boolean = false;

    check(): void {
        this.dialog.notify(this, "check");
    }
    // ...
}
```

</details>

<details>

<summary>Snapshot (Снимок)</summary>

#### **Основные концепции**

1. **Сохранение состояния** – паттерн позволяет сохранять копии объекта без раскрытия его внутренней структуры.
2. **Восстановление состояния** – объект может быть восстановлен из сохраненного снимка в любое время.
3. **Инкапсуляция** – состояние объекта сохраняется внутри снимка и не раскрывается внешнему коду, предотвращая его модификацию.
4. **История изменений** – можно хранить несколько снимков для поддержки многоуровневой отмены (Undo/Redo).
5. **Отделение ответственности** – сам объект не отвечает за сохранение своего состояния, это делает отдельный класс (снимок).

### **Пример**

```typescript
// Класс-снимок, инкапсулирующий состояние Originator
class Memento {
    constructor(private state: string) {}

    getState(): string {
        return this.state;
    }
}

// Класс, чье состояние нужно сохранять
class Originator {
    private state: string = '';

    setState(state: string): void {
        console.log(`Установлено состояние: ${state}`);
        this.state = state;
    }

    save(): Memento {
        console.log(`Сохранение состояния: ${this.state}`);
        return new Memento(this.state);
    }

    restore(memento: Memento): void {
        this.state = memento.getState();
        console.log(`Восстановлено состояние: ${this.state}`);
    }
}

// Класс, управляющий снимками
class Caretaker {
    private history: Memento[] = [];

    saveMemento(memento: Memento): void {
        console.log(`Сохранен снимок`);
        this.history.push(memento);
    }

    getMemento(index: number): Memento | null {
        return this.history[index] || null;
    }
}

// Пример использования
const originator = new Originator();
const caretaker = new Caretaker();

originator.setState("Состояние 1");
caretaker.saveMemento(originator.save());

originator.setState("Состояние 2");
caretaker.saveMemento(originator.save());

originator.setState("Состояние 3");

console.log("Откат к предыдущему состоянию...");
originator.restore(caretaker.getMemento(1)!);

console.log("Откат к самому первому состоянию...");
originator.restore(caretaker.getMemento(0)!);
```

***

#### **Где применяется паттерн Snapshot?**

1. **Текстовые редакторы** (отмена и повтор действий).
2. **Игры** (сохранение состояния).
3. **Базы данных** (транзакции и откаты изменений).
4. **Конфигурационные системы** (откат к предыдущим настройкам).

Паттерн **Snapshot (Memento)** помогает удобно управлять историей изменений объекта, сохраняя его внутреннее состояние без нарушения инкапсуляции.&#x20;

</details>

<details>

<summary>Observer (Наблюдатель)</summary>

Observer — поведенческий паттерн проектирования, организующий зависимость типа «один ко многим». Один объект (**Subject**) автоматически уведомляет о своих изменениях другие объекты (**Observers**), которые заранее на него подписались.

#### Принципы паттерна:

* Существует субъект, хранящий список наблюдателей.
* Наблюдатели подписываются на субъект и реагируют на его изменения.
* Связь между субъектом и наблюдателями минимальна (только через интерфейсы).
* Наблюдатели не опрашивают субъект постоянно, а ждут уведомлений.

#### Что решает?

* Уменьшает связанность компонентов.
* Устраняет постоянные опросы состояния.
* Повышает гибкость и масштабируемость системы.

#### Преимущества:

* Простота добавления и удаления наблюдателей.
* Низкая связанность, удобство поддержки.
* Гибкость системы (новые типы наблюдателей можно легко добавлять).

#### Недостатки:

* Сложность отладки при большом числе наблюдателей.
* Риск утечек памяти при неправильной отписке наблюдателей.
* Неопределённый

### Реализация паттерна Observer

Рассмотрим, как реализовать паттерн Observer. Для начала определим интерфейсы **Observer** и **Subject**, затем создадим конкретный класс субъекта с механизмом подписки, конкретные классы наблюдателей, и наконец покажем пример их использования.

```typescript
// Интерфейс Observer: определяет метод уведомления, который субъект будет вызывать у наблюдателя.
interface Observer {
  update(subject: Subject): void;
}

// Интерфейс Subject: определяет методы для добавления, удаления наблюдателей и оповещения их о событии.
interface Subject {
  attach(observer: Observer): void;
  detach(observer: Observer): void;
  notify(): void;
}

// Конкретный субъект, за состоянием которого наблюдают.
class ConcreteSubject implements Subject {
  private observers: Observer[] = [];      // список подписчиков (наблюдателей)
  private state: number = 0;               // некоторое состояние, за изменениями которого следят

  public attach(observer: Observer): void {
    this.observers.push(observer);
  }

  public detach(observer: Observer): void {
    // Удаляем наблюдателя из списка (если он там есть)
    this.observers = this.observers.filter(obs => obs !== observer);
  }

  public notify(): void {
    // Уведомляем всех подписчиков, вызывая у каждого метод update
    for (const observer of this.observers) {
      observer.update(this);
    }
  }

  // Метод, имитирующий изменение состояния субъекта.
  public someBusinessLogic(): void {
    // Изменяем состояние (для примера установим случайное число от 0 до 9)
    this.state = Math.floor(Math.random() * 10);
    console.log(`Subject: изменил своё состояние на ${this.state}`);
    // После изменения состояния – оповещаем всех наблюдателей
    this.notify();
  }

  // Метод доступа к состоянию (наблюдатели могут получить новое значение, если нужно)
  public getState(): number {
    return this.state;
  }
}

// Конкретный наблюдатель A.
class ConcreteObserverA implements Observer {
  public update(subject: Subject): void {
    // Реагирует на уведомление: например, проверяет состояние и действует соответственно
    if (subject instanceof ConcreteSubject && subject.getState() < 5) {
      console.log('ConcreteObserverA: состояние субъекта меньше 5, реагируем на изменение');
    }
  }
}

// Конкретный наблюдатель B.
class ConcreteObserverB implements Observer {
  public update(subject: Subject): void {
    // Реагирует только на определённые изменения состояния
    if (subject instanceof ConcreteSubject && subject.getState() >= 5) {
      console.log('ConcreteObserverB: состояние субъекта 5 или больше, реагируем на изменение');
    }
  }
}

// === Пример использования ===
const subject = new ConcreteSubject();      // создаём субъект

const observerA = new ConcreteObserverA();  // создаём наблюдателей
const observerB = new ConcreteObserverB();

subject.attach(observerA);  // подписываем observerA на уведомления от subject
subject.attach(observerB);  // подписываем observerB

// Имитация изменений состояния субъекта:
subject.someBusinessLogic();  // изменяет состояние и оповещает всех наблюдателей
subject.someBusinessLogic();  // ещё одно изменение состояния и оповещение

subject.detach(observerB);    // наблюдатель B отписывается и больше не будет получать уведомления

subject.someBusinessLogic();  // очередное изменение; теперь уведомление получит только observerA
```



</details>

<details>

<summary>State (Состояние)</summary>

### Что такое паттерн State

Паттерн **State** (Состояние) позволяет объекту менять свое поведение в зависимости от внутреннего состояния. При этом создается впечатление, что объект меняет свой класс во время выполнения программы. Формально:

1. Вы определяете объект (контекст), внутри которого будет храниться ссылка на текущее состояние.
2. Каждое **состояние** (State) является отдельным классом (или вариантом реализации интерфейса), который описывает поведение контекста при нахождении в данном конкретном состоянии.
3. При необходимости перехода в другое состояние контекст просто меняет эту ссылку на другой объект-состояние и делегирует ему запросы.

Паттерн часто используют, когда объект может находиться в разных состояниях, и при этом каждое состояние влияет на то, как объект будет реагировать на разные запросы (методы).

### Пример

```typescript
/**
 * Интерфейс, который должны реализовывать все конкретные состояния светофора.
 * next(): переход к следующему состоянию.
 * getColor(): возвращает текущий цвет (строка).
 */
interface TrafficLightState {
  next(): void;
  getColor(): string;
}

/**
 * Класс контекста (TrafficLight).
 * Хранит ссылку на текущее состояние (currentState) и делегирует ему команды.
 */
class TrafficLight {
  private currentState: TrafficLightState;

  constructor(initialState: TrafficLightState) {
    // Начальное состояние светофора определяется при создании экземпляра
    this.currentState = initialState;
  }

  /**
   * Сеттер для смены состояния.
   */
  public setState(state: TrafficLightState): void {
    this.currentState = state;
  }

  /**
   * Вызывает метод next() у текущего состояния — переключение на следующее состояние.
   */
  public next(): void {
    this.currentState.next();
  }

  /**
   * Возвращает цвет текущего состояния.
   */
  public getColor(): string {
    return this.currentState.getColor();
  }
}

/**
 * Конкретное состояние: GreenState (Зеленый).
 */
class GreenState implements TrafficLightState {
  // Для смены состояния нам нужен доступ к контексту (TrafficLight).
  constructor(private trafficLight: TrafficLight) {}

  public next(): void {
    console.log("Переходим из зеленого в желтый...");
    this.trafficLight.setState(new YellowState(this.trafficLight));
  }

  public getColor(): string {
    return "Green";
  }
}

/**
 * Конкретное состояние: YellowState (Желтый).
 */
class YellowState implements TrafficLightState {
  constructor(private trafficLight: TrafficLight) {}

  public next(): void {
    console.log("Переходим из желтого в красный...");
    this.trafficLight.setState(new RedState(this.trafficLight));
  }

  public getColor(): string {
    return "Yellow";
  }
}

/**
 * Конкретное состояние: RedState (Красный).
 */
class RedState implements TrafficLightState {
  constructor(private trafficLight: TrafficLight) {}

  public next(): void {
    console.log("Переходим из красного в зеленый...");
    this.trafficLight.setState(new GreenState(this.trafficLight));
  }

  public getColor(): string {
    return "Red";
  }
}

/**
 * Демонстрация работы паттерна State.
 * Создаем светофор, передавая начальное состояние (GreenState).
 * Последовательно переключаем его несколько раз, наблюдая за логикой перехода.
 */
function main(): void {
  // Создаем светофор и выставляем ему начальное состояние — GreenState.
  // Здесь "null as any" искусственно, чтобы передать в GreenState ссылку на TrafficLight.
  // В реальном коде обычно сперва создают TrafficLight, а затем вызовом setState задают состояние.
  const trafficLight = new TrafficLight(null as any);
  trafficLight.setState(new GreenState(trafficLight));

  console.log("Текущее состояние:", trafficLight.getColor()); // "Green"
  trafficLight.next(); // Переход в желтый
  console.log("Текущее состояние:", trafficLight.getColor()); // "Yellow"

  trafficLight.next(); // Переход в красный
  console.log("Текущее состояние:", trafficLight.getColor()); // "Red"

  trafficLight.next(); // Переход в зеленый
  console.log("Текущее состояние:", trafficLight.getColor()); // "Green"
}

// Запускаем демонстрационную функцию
main();
```

### Плюсы паттерна State

1. **Разделение логики по состояниям**:
   * Когда мы используем паттерн State, мы разносим поведение по отдельным классам (или файлам, если в проекте большая архитектура). Это упрощает и поддерживает SOLID-принцип «единственной ответственности» (Single Responsibility Principle).
2. **Упрощение кода контекста**:
   * Контекст избавлен от множества ветвлений `if-else` или `switch-case` по текущему состоянию. Вместо этого он делегирует поведение соответствующему состоянию.
3. **Легко добавлять новые состояния**:
   * Если появляется новое состояние, достаточно создать новый класс, реализующий общий интерфейс (например, `TrafficLightState`), и учесть его в логике смены состояний. При этом не нужно переписывать весь код, зачастую меняется только место, где назначается новое состояние.
4. **Гибкость и расширяемость**:
   * Паттерн хорошо подходит, когда набор состояний может со временем меняться или усложняться. Локализация логики внутри специализированных классов делает поддержку системы проще.
5. **Чистый код**:
   * Логика переключения и логика действий в каждом состоянии аккуратно инкапсулированы внутри классов состояний.

***

### Минусы паттерна State

1. **Избыточность кода**:
   * Иногда может показаться, что слишком много «пространства» уходит на создание большого количества классов (особенно если состояний очень много и они простые).
   * Если логика состояний совсем простая и требуется только один-два флага, можно обойтись и без паттерна State, используя условные выражения.
2. **Сложность архитектуры**:
   * При большом количестве состояний код становится более модульным, но и общая структура усложняется. Нужно следить за корректными переходами (в какой момент и с помощью каких методов).
   * Могут возникнуть ситуации, когда нужно контролировать различные переходы между состояниями в нестандартных условиях, и тогда код State-классов начнет разрастаться.
3. **Тесная связь со связующим контекстом**:
   * Иногда возникает ситуация, когда состояния должны иметь доступ к детальной информации контекста (например, много полей и методов). Приходится либо передавать их напрямую, либо открывать в контексте определенные геттеры/сеттеры. Это может привести к более плотной связке между состояниями и контекстом.

</details>

<details>

<summary>Strategy (Стратегия)</summary>

**Strategy** (Стратегия) — это поведенческий паттерн проектирования, который позволяет определять семейство схожих алгоритмов, инкапсулировать каждый из них и делать их взаимозаменяемыми для конкретной задачи. При этом сам клиент (или «контекст») не знает, какой именно алгоритм был выбран: он вызывает единый метод у стратегии, а конкретная логика реализации остаётся «за кадром».

#### Основные идеи паттерна Strategy

1. **Инкапсуляция алгоритмов**\
   Разные способы решения одной и той же задачи помещаются в отдельные классы (стратегии). Общий интерфейс (или абстрактный класс) задаёт «контракт» для всех возможных стратегий.
2. **Взаимозаменяемость**\
   Контекст хранит ссылку на некую «стратегию» (алгоритм), но не знает, что это за конкретный класс. В любой момент времени можно подменить одну реализацию стратегии на другую без изменения самого контекста.
3. **Избавление от громоздких ветвлений**\
   Вместо серии `if-else` или `switch-case` в коде, решающем какую именно логику использовать, мы можем лишь выбрать стратегию и вызвать её метод.
4. **Упрощённое расширение и поддержка**\
   Если нужно добавить новый алгоритм, достаточно реализовать его в новом классе (соответствующем интерфейсу стратегии). Код контекста при этом менять не требуется.



### Пример

Ниже приведён пример, в котором мы создаём различные **стратегии скидок** для интернет-магазина. Каждая стратегия будет реализовывать метод `calculate(price: number): number`, который высчитывает конечную цену товара после скидки.

В примере присутствуют следующие стратегии:

* **NoDiscountStrategy** — без скидки
* **PercentageDiscountStrategy** — процентная скидка
* **FixedDiscountStrategy** — фиксированная сумма скидки

В коде используется класс `DiscountContext`, который позволяет «переключаться» между стратегиями и применять выбранную скидку к цене товара.

```typescript
/**
 * Интерфейс стратегии скидки.
 * Каждая конкретная стратегия должна реализовать метод calculate,
 * который возвращает итоговую цену после применения скидки.
 */
interface DiscountStrategy {
  calculate(price: number): number;
}

/**
 * Стратегия "без скидки".
 * Возвращает исходную цену без изменений.
 */
class NoDiscountStrategy implements DiscountStrategy {
  public calculate(price: number): number {
    return price;
  }
}

/**
 * Стратегия процентной скидки (к примеру, 10% или 20% и т.д.).
 */
class PercentageDiscountStrategy implements DiscountStrategy {
  constructor(private discountPercentage: number) {}

  public calculate(price: number): number {
    /**
     * Вычисляем скидку в процентах и вычитаем её из исходной цены.
     * Допустим, discountPercentage = 20 => скидка 20% от price.
     */
    const discountAmount = price * (this.discountPercentage / 100);
    return price - discountAmount;
  }
}

/**
 * Стратегия фиксированной скидки (например, минус 500₽).
 */
class FixedDiscountStrategy implements DiscountStrategy {
  constructor(private discountValue: number) {}

  public calculate(price: number): number {
    /**
     * Вычитаем фиксированную сумму discountValue из цены.
     * Учитываем, что итоговая цена не должна уйти в минус.
     */
    const finalPrice = price - this.discountValue;
    return finalPrice > 0 ? finalPrice : 0;
  }
}

/**
 * Класс контекста, который хранит текущую стратегию скидки
 * и предоставляет метод для подсчёта итоговой цены.
 */
class DiscountContext {
  constructor(private strategy: DiscountStrategy) {
    // Изначально контекст создаётся с переданной стратегией
  }

  /**
   * Позволяет установить (или сменить) стратегию скидки в любой момент времени.
   */
  public setStrategy(strategy: DiscountStrategy): void {
    this.strategy = strategy;
  }

  /**
   * Применяет текущую стратегию к переданной цене.
   */
  public applyDiscount(price: number): number {
    return this.strategy.calculate(price);
  }
}

/**
 * Функция main() демонстрирует работу паттерна Strategy.
 * Мы создаём контекст со стратегией "без скидки",
 * затем меняем стратегию на процентную и фиксированную.
 */
function main(): void {
  // Создаём контекст без скидки
  const discountContext = new DiscountContext(new NoDiscountStrategy());

  // Цена товара
  const originalPrice = 3000;

  // Применяем стратегию "без скидки"
  console.log("Исходная цена:", originalPrice);
  console.log("Цена без скидки:", discountContext.applyDiscount(originalPrice));

  // Смена стратегии на процентную скидку (20%)
  discountContext.setStrategy(new PercentageDiscountStrategy(20));
  console.log("Цена с 20% скидкой:", discountContext.applyDiscount(originalPrice));

  // Смена стратегии на фиксированную скидку (500₽)
  discountContext.setStrategy(new FixedDiscountStrategy(500));
  console.log("Цена со скидкой 500₽:", discountContext.applyDiscount(originalPrice));
}

// Запускаем функцию демонстрации
main();
```

***

### Плюсы паттерна Strategy

1. **Избавляет от громоздких конструкций if-else / switch-case**\
   Все сценарии «обрабатываются» внутри отдельных классов стратегий.
2. **Упрощённое добавление новых «алгоритмов»**\
   Чтобы добавить новую стратегию, достаточно реализовать общий интерфейс без необходимости трогать основной код контекста.
3. **Разделение ответственностей**\
   Каждая стратегия отвечает только за свою логику расчёта (или другого алгоритма), а контекст лишь вызывает стратегию, не зная деталей её реализации.
4. **Гибкость и расширяемость**\
   Легко менять стратегии «на лету» — можно динамически выбирать, какой алгоритм применять.

***

### Минусы паттерна Strategy

1. **Увеличение количества классов**\
   При множестве различных стратегий кодовая база может выглядеть раздутой (каждый алгоритм в отдельном классе).
2. **Необходимость «правильного» выбора стратегии**\
   Клиент или внешний код должны понимать, когда и какую стратегию нужно применить. Может потребоваться дополнительная логика для переключения.
3. **Избыточность в простых сценариях**\
   Если алгоритмов всего 1–2, иногда достаточно использовать простую условную конструкцию, вместо внедрения паттерна Strategy

</details>

<details>

<summary>Template Method (Шаблонный метод)</summary>

**Template Method** (Шаблонный метод) — это поведенческий паттерн проектирования, который определяет «скелет» алгоритма в методе базового класса, перекладывая реализацию отдельных шагов этого алгоритма на подклассы. При этом базовый класс задаёт общий «шаблон» (последовательность шагов), а подклассы могут переопределять некоторые из этих шагов, не меняя структуру самого алгоритма.

#### Основные идеи паттерна Template Method

1. **Общий «каркас» алгоритма**\
   В базовом (абстрактном) классе объявляется метод (обычно называемый templateMethod()), в котором описана последовательность шагов алгоритма.
2. **Частичная реализация в базовом классе**\
   Базовый класс может реализовывать часть шагов по умолчанию и предоставлять абстрактные или «переопределяемые» (hook-методы) для остальных шагов.
3. **Переопределение шагов в подклассах**\
   Подклассы могут изменять поведение отдельных шагов, чтобы получить разную логику выполнения алгоритма, не трогая общий каркас (последовательность шагов).
4. **Инверсия управления**\
   Базовый класс не вызывает реализацию подклассов напрямую; наоборот, подклассы подключаются к базовому через переопределяемые методы, когда внутри шаблонного метода приходит «их очередь».

### Пример

В данном примере мы покажем упрощённую схему работы с «загрузкой файлов». У нас будет:

* Базовый класс `FileDownloader`, который описывает общую логику скачивания.
* Два подкласса: `HTTPFileDownloader` (скачивание по HTTP) и `FTPFileDownloader` (скачивание по FTP).
* Метод `download()` в базовом классе будет определять «шаблон» (последовательность шагов): подготовка, скачивание, завершение.\
  Отдельные шаги — это методы, которые подклассы могут переопределять, если им нужно нестандартное поведение.

```typescript
/**
 * Абстрактный класс FileDownloader, который определяет "шаблон" скачивания файлов.
 * Шаблонный метод download() описывает общий алгоритм в определённой последовательности,
 * а конкретные реализации (подклассы) переопределяют отдельные шаги.
 */
abstract class FileDownloader {
  /**
   * Шаблонный метод, определяющий общий сценарий скачивания.
   * Не меняем порядок операций, но некоторые шаги могут переопределяться в подклассах.
   */
  public download(url: string): void {
    this.openConnection(url);      // 1. Устанавливаем соединение
    this.fetchFile(url);           // 2. Скачиваем файл (реализуется в подклассах)
    this.closeConnection();        // 3. Закрываем соединение
    this.postDownloadHook();       // 4. Дополнительный необязательный шаг (hook)
  }

  /**
   * Общий шаг — открыть соединение.
   * Предположим, что реализация единой процедуры нам подходит для всех.
   */
  protected openConnection(url: string): void {
    console.log(`Открываем соединение для: ${url}`);
  }

  /**
   * Абстрактный метод скачивания файла.
   * Подклассы обязаны переопределить его под конкретный тип соединения (HTTP, FTP и т.д.).
   */
  protected abstract fetchFile(url: string): void;

  /**
   * Общий шаг — закрыть соединение.
   * Также может быть переопределён при необходимости, но обычно достаточно дефолтной реализации.
   */
  protected closeConnection(): void {
    console.log("Закрываем соединение...");
  }

  /**
   * Hook-метод, который может быть переопределён подклассами при необходимости.
   * В базовом классе он «пустой» (необязательный).
   */
  protected postDownloadHook(): void {
    // Ничего не делаем по умолчанию
  }
}

/**
 * Конкретный класс для скачивания через HTTP.
 * Переопределяет метод fetchFile и может переопределять другие шаги или hooks.
 */
class HTTPFileDownloader extends FileDownloader {
  protected fetchFile(url: string): void {
    console.log(`Скачиваем файл по HTTP: ${url}`);
    // Здесь могла быть HTTP-логика, но мы лишь имитируем процесс
  }

  /**
   * При желании можем переопределить postDownloadHook, если надо сделать что-то особенное
   * после скачивания. Если этого не нужно, метод берётся из базового класса и ничего не делает.
   */
  protected postDownloadHook(): void {
    console.log("HTTP-загрузка завершена. Выполняем пост-обработку...");
  }
}

/**
 * Конкретный класс для скачивания через FTP.
 * Переопределяет fetchFile и может переопределить любые другие методы при необходимости.
 */
class FTPFileDownloader extends FileDownloader {
  protected fetchFile(url: string): void {
    console.log(`Скачиваем файл по FTP: ${url}`);
    // Здесь могла быть FTP-логика, но мы лишь имитируем процесс
  }

  /**
   * Допустим, закрывать соединение по FTP надо чуть иначе,
   * поэтому переопределим closeConnection.
   */
  protected closeConnection(): void {
    console.log("Закрываем FTP-соединение специальным способом...");
  }
}

/**
 * Функция main() демонстрирует работу паттерна Template Method.
 * Мы создаём экземпляры классов HTTPFileDownloader и FTPFileDownloader
 * и вызываем у них метод download(), который "задаёт" общий шаблон.
 */
function main(): void {
  // 1. Скачиваем файл через HTTP
  console.log("=== HTTP Downloader ===");
  const httpDownloader = new HTTPFileDownloader();
  httpDownloader.download("http://example.com/file.zip");

  console.log("");

  // 2. Скачиваем файл через FTP
  console.log("=== FTP Downloader ===");
  const ftpDownloader = new FTPFileDownloader();
  ftpDownloader.download("ftp://example.com/file.zip");
}

// Запускаем демонстрационную функцию
main();
```

***

### Плюсы паттерна Template Method

1. **Позволяет переиспользовать код**\
   Общий «каркас» (шаблон) алгоритма вынесен в базовый класс, и подклассы не дублируют эти шаги.
2. **Упрощённое изменение отдельных шагов**\
   Подклассы переопределяют нужные методы, не затрагивая общую последовательность.
3. **Чёткое разделение ответственности**\
   Базовый класс отвечает за определение порядка (структуры) алгоритма, а подклассы — за конкретную реализацию и детали отдельных шагов.
4. **Использование hook-методов**\
   Можно иметь необязательные шаги в алгоритме (так называемые «хуки»), которые будут переопределяться только при необходимости.

***

### Минусы паттерна Template Method

1. **Ограниченная гибкость структуры**\
   Жёстко заданная последовательность шагов (шаблон) может быть не всегда удобной, если нужно сложнее влиять на порядок выполнения.
2. **Может приводить к усложнению иерархии**\
   В больших системах базовый класс со множеством абстрактных методов может стать тяжёлым для понимания, а количество подклассов может увеличиваться.
3. **Необходимость создавать подклассы**\
   Чтобы изменить отдельные шаги алгоритма, приходится наследоваться и переопределять методы, что порой усложняет структуру, если нужны лишь небольшие изменения.

</details>

