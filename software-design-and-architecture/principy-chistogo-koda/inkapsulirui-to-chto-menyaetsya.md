# Инкапсулируй то, что меняется

**Encapsulate What Varies** – это один из ключевых принципов, сформулированных в книге «Head First Design Patterns» (и в классических паттернах GoF), который гласит:

> «Выделяйте (инкапсулируйте) те аспекты системы, которые подвержены изменениям, чтобы изменение этих аспектов не затрагивало остальной код».

Другими словами, если в вашей системе есть место, где логика часто меняется или от неё зависит множество факторов, лучше **отделить** эту логику от остальной части кода и взаимодействовать с ней через «контракт» (интерфейс, абстрактный класс). Тогда модификации не будут «проливаться» по всей системе.

***

### Зачем это нужно

1. **Ослабление связности**
   * Изменения в одной «обособленной» части кода (алгоритме, поведении) не вынуждают переписывать всё приложение.
2. **Легче расширять**
   * Можно добавить новую логику (например, новую стратегию, новый алгоритм) без ломки других частей.
3. **Упрощённое тестирование**
   * При чётком контракте (интерфейсе) вы изолированно тестируете разные варианты поведения.
4. **Повышение гибкости**
   * Иногда можно менять «варианты» поведения на лету (паттерны Strategy/State), не меняя остальной код.

***

### Пример: Утки и полёты (Strategy)

#### Ситуация

У нас есть разные виды «уток» (Duck). Все они должны уметь «крякать» и «летать». Но не все утки летают одинаково: некоторые не летают вовсе (резиновая утка), некоторые летают с маханием крыльев, а, например, «ракетная утка» может летать на реактивном двигателе.

#### Если не инкапсулировать (плохой вариант)

```typescript
class Duck {
  public quack(): void {
    console.log("Кря!");
  }
  
  // Все утки, по умолчанию, летают
  public fly(): void {
    console.log("Лечу...");
  }
}

class RubberDuck extends Duck {
  // Переопределяем fly() и quack() (резиновая не летает, пищит)
  public fly(): void {
    console.log("Не могу летать");
  }
  public quack(): void {
    console.log("Пи-пи!");
  }
}
```

* Мы полагаемся на **наследование**, и при каждом новом типе надо переопределять `fly()` и `quack()` заново, нередко «отвергая» логику родителя.
* Если меняется механика полёта (например, другой алгоритм — реактивный двигатель), приходится лезть в класс или плодить новых наследников.

#### «Encapsulate What Varies» (хороший вариант)

**Выделяем «настраиваемое поведение» (летать, крякать) в отдельные интерфейсы**

```typescript
// Интерфейсы (или абстрактные классы)
interface FlyBehavior {
  fly(): void;
}
interface QuackBehavior {
  quack(): void;
}

// Конкретные реализации (варианты) полёта
class FlyWithWings implements FlyBehavior {
  public fly(): void {
    console.log("Лечу, взмахивая крыльями!");
  }
}
class FlyNoWay implements FlyBehavior {
  public fly(): void {
    console.log("Не могу летать!");
  }
}
class FlyRocketPowered implements FlyBehavior {
  public fly(): void {
    console.log("Лечу на реактивном двигателе!");
  }
}

// Конкретные реализации (варианты) кряка
class QuackNormal implements QuackBehavior {
  public quack(): void {
    console.log("Кря-кря!");
  }
}
class QuackSqueak implements QuackBehavior {
  public quack(): void {
    console.log("Пи-пи!");
  }
}
class QuackMute implements QuackBehavior {
  public quack(): void {
    console.log("...");
  }
}

// Duck использует композицию, храня ссылки на интерфейсы.
class Duck {
  private flyBehavior: FlyBehavior;
  private quackBehavior: QuackBehavior;

  constructor(flyBehavior: FlyBehavior, quackBehavior: QuackBehavior) {
    this.flyBehavior = flyBehavior;
    this.quackBehavior = quackBehavior;
  }

  public performFly(): void {
    this.flyBehavior.fly();
  }

  public performQuack(): void {
    this.quackBehavior.quack();
  }

  // Если нужно, можно «под горячую руку» заменить поведение
  public setFlyBehavior(fb: FlyBehavior): void {
    this.flyBehavior = fb;
  }
  public setQuackBehavior(qb: QuackBehavior): void {
    this.quackBehavior = qb;
  }
}
```

**Использование**

```typescript
function main() {
  // «Обычная» утка — летает крыльями, крякает нормально
  const mallard = new Duck(new FlyWithWings(), new QuackNormal());
  mallard.performFly();    // "Лечу, взмахивая крыльями!"
  mallard.performQuack();  // "Кря-кря!"

  // «Резиновая» утка — не летает, пищит
  const rubber = new Duck(new FlyNoWay(), new QuackSqueak());
  rubber.performFly();     // "Не могу летать!"
  rubber.performQuack();   // "Пи-пи!"

  // «Супер» утка — можно динамически сменить полёт на реактивный
  rubber.setFlyBehavior(new FlyRocketPowered());
  rubber.performFly();     // "Лечу на реактивном двигателе!"
}

main();
```

Теперь все изменяющиеся части (способы полёта, способы кряка) **инкапсулированы** в отдельных классах, и `Duck` **композирует** их. Если в будущем нужно добавить «летать по-космически», мы создаём новый класс, имплементирующий `FlyBehavior`, и подключаем его утке.

***

### Ключевые моменты

* «Encapsulate What Varies» тесно связан с паттерном **Strategy** (и другими паттернами GoF), где мы заменяем набор `if-else` на «семейство» алгоритмов, оформленных как объекты.
* Принцип можно применять **повсюду**: когда видите, что «этот блок кода постоянно меняется» или «у нас есть несколько вариантов логики», вынесите эту часть в отдельный модуль/класс/интерфейс. Остальная система будет взаимодействовать с ним через абстракцию.
* Это повышает **гибкость**, **тестируемость** и уменьшает «эффект каскадных изменений» (когда из-за одной мелочи приходится менять код в нескольких местах).

***

#### Вывод

* **Encapsulate What Varies** – один из основных принципов гибкого дизайна (справедливо как для ООП, так и для функционального подхода).
* При его соблюдении вы снижаете риски, связанные с изменением/расширением логики, так как «переменная часть» живёт в одном месте, под абстрактным интерфейсом.
* Пример с «Утками» и паттерном **Strategy** хорошо демонстрирует, как можно выделить «вариативные аспекты» (полёт, кряканье) и спрятать их за интерфейсами, чтобы менять/расширять без ломки остального кода.
