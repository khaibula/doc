# Запахи кода

### Раздувальщики

Раздувальщики представляют код, методы и классы, которые раздулись до таких больших размеров, что с ними стало невозможно эффективно работать. Все эти запахи зачастую не появляются сразу, а нарастают в процессе эволюции программы (особенно когда никто не пытается бороться с ними).

<details>

<summary>Logn Method (Длинный метод)</summary>

**Long Method** («слишком длинный метод») — один из распространённых антипаттернов в программировании. Он возникает, когда функция или метод обрастают чрезмерным количеством логики, становятся слишком большими и сложными для чтения и понимания.

#### Основные идеи и причины появления

1. **Сложность логики**\
   Разработчик может не разделять логику на более мелкие подзадачи. В результате появляется «гигантская» функция, в которой смешаны различные этапы вычислений.
2. **Несоблюдение принципов чистого кода**\
   Нарушается принцип единственной ответственности (Single Responsibility Principle): метод начинает выполнять слишком много разных задач, что делает его трудно сопровождать.
3. **Отсутствие рефакторинга**\
   Если долго не проводить рефакторинг и не выделять повторяющуюся логику, код постепенно растёт вширь и вниз, обрастая дополнительной функциональностью.
4. **Страх изменить рабочий код**\
   Иногда разработчики не решаются трогать существующий рабочий метод, опасаясь ломать «настроенную» функциональность, и продолжают расширять его «как есть».

***

### Пример

Ниже — упрощённый (но показательный) пример. Предположим, у нас есть метод, который должен:

* Прочитать данные (условно, из какого-то внешнего ресурса).
* Проанализировать их и выполнить несколько проверок.
* Применить разные правила валидации.
* Сохранить результат куда-то.
* Сформировать и отправить уведомление.

Все эти задачи помещены в один «монолитный» метод, что создаёт антипаттерн Long Method.

```typescript
class DataProcessor {
  public processData(): void {
    console.log("Начинаем обработку...");

    // 1. Чтение данных
    let data = this.fetchData();
    if (!data || data.length === 0) {
      console.log("Нет данных для обработки");
      return;
    }

    // 2. Начало анализа
    let isValid = true;
    for (let item of data) {
      // Проверка на нулевые значения
      if (item.value === 0) {
        isValid = false;
        console.log("Значение равно 0, пропускаем обработку");
        break;
      }
      // Проверка формата
      if (!this.validateFormat(item)) {
        isValid = false;
        console.log("Неверный формат, пропускаем обработку");
        break;
      }
      // Допустим, ещё несколько проверок...
    }

    // 3. Дополнительные правила валидации
    if (isValid) {
      // Несколько громоздких условий 
      // (в реальном коде может быть десятки строк логики)
      let specialFlag = false;
      for (let item of data) {
        if (item.type === "SPECIAL") {
          specialFlag = true;
          // Выполняем специальные действия...
          console.log("Обнаружен SPECIAL-тип, обрабатываем особым образом...");
        }
      }
      // Ещё несколько веток условий...
    }

    // 4. Сохранение результата
    if (isValid) {
      try {
        this.saveData(data);
        console.log("Результаты сохранены...");
      } catch (error) {
        console.error("Ошибка сохранения:", error);
        return;
      }
    } else {
      console.log("Данные признаны невалидными, пропускаем сохранение.");
    }

    // 5. Формирование и отправка уведомления
    if (isValid) {
      this.sendNotification(`Обработка завершена для ${data.length} элементов`);
    }
    console.log("Обработка завершена.");
  }

  // Вспомогательные методы
  private fetchData(): { value: number; type?: string }[] {
    // Имитируем получение данных
    return [
      { value: 100, type: "NORMAL" },
      { value: 200, type: "SPECIAL" },
    ];
  }

  private validateFormat(item: { value: number; type?: string }): boolean {
    // Примитивная проверка на пример
    return typeof item.value === "number";
  }

  private saveData(data: { value: number; type?: string }[]): void {
    // Имитируем сохранение
    // В реальном коде могла быть логика записи в БД или на диск
  }

  private sendNotification(message: string): void {
    // Имитируем отправку уведомления
    console.log(`Уведомление отправлено: ${message}`);
  }
}
```

* Этот метод **processData** уже сейчас выглядит перегруженным: он и читает данные, и валидирует, и сохраняет, и рассылает уведомления — всё в одном месте.
* При дальнейшем росте требований метод может разрастись в ещё более громоздкий «комбайн».

</details>

<details>

<summary>Primitive Obsession (Одержимость элементарными типами)</summary>

**Primitive Obsession** (одержимость примитивами) — это антипаттерн, когда разработчик использует простые типы данных (строки, числа, булевы значения) там, где логически напрашиваются более «сильные» типы или специализированные структуры. Вместо того чтобы создавать отдельные классы (или структуры) для моделирования сущностей, люди часто используют строки, числа и т. д. Это усложняет поддержку кода и может привести к ошибкам, связанных с неправильным использованием этих примитивов.

#### Основные идеи и причины появления

1. **Нежелание «избыточных» классов**\
   Порой кажется, что проще оперировать «примитивом», чем создавать несколько новых классов. Разработчики боятся «раздувать код», предпочитая «попроще».
2. **Лень или спешка**\
   Проще «запихнуть» всё в строку или число, чем продумывать отдельный класс со своими полями и методами.
3. **Наличие множества «магических» значений**\
   Логика, завязанная на строках, числах и т. д. (например, строки "ADMIN", "USER" вместо enum или специализированного типа), может увеличивать риск опечаток и рассеивания правил в разных местах кода.
4. **Отсутствие рефакторинга**\
   Разработчик со временем добавляет всё новые проверки, ветвления на эти примитивы. Когда логика растёт, «примитивы» уже не соответствуют сложности предметной области, но менять всё сразу бывает страшно.

***

#### Пример «как не надо»

Ниже пример с классом `User`, где:

* Роль пользователя хранится как строка (`"ADMIN"`, `"USER"`, и т.д.).
* Адрес — тоже строка с неявной структурой.

```typescript
class User {
  public name: string;
  public role: string;     // "ADMIN", "USER", "GUEST" и т.д.
  public address: string;  // "Город, Улица, Дом"

  constructor(name: string, role: string, address: string) {
    this.name = name;
    this.role = role;
    this.address = address;
  }

  public printUserInfo(): void {
    console.log(`Имя: ${this.name}`);

    if (this.role === "ADMIN") {
      console.log("Права администратора");
    } else if (this.role === "USER") {
      console.log("Обычный пользователь");
    } else {
      console.log("Гость");
    }

    // Попытка разобрать address (но формат может отличаться, возможны ошибки)
    const parts = this.address.split(",");
    if (parts.length >= 3) {
      console.log(`Город: ${parts[0].trim()}, Улица: ${parts[1].trim()}, Дом: ${parts[2].trim()}`);
    } else {
      console.log(`Адрес (сырая строка): ${this.address}`);
    }
  }
}

const user = new User("Иван", "ADMIN", "Москва, Пушкина, 10");
user.printUserInfo();
```

* Здесь «роль» может быть любой строкой (опечатка в `"ADMNI"` не отлавливается компилятором).
* «Адрес» приходится «добывать» из одной строки при каждом использовании.

***

#### Как сделать лучше

**1. Использовать enum для ролей**

Создадим типы и перечисления, чтобы исключить «магические строки»:

```typescript
enum UserRole {
  Admin = "ADMIN",
  User = "USER",
  Guest = "GUEST",
}
```

**2. Выделить класс/Value Object для адреса**

Вместо одной строки с неочевидным форматом, создадим класс (или интерфейс) `Address` с понятными полями:

```typescript
class Address {
  constructor(
    public city: string,
    public street: string,
    public house: string
  ) {}
  
  // При желании, можно добавить методы, например форматированный вывод
  public toString(): string {
    return `${this.city}, ${this.street}, ${this.house}`;
  }
}
```

**3. Переписать `User`, используя более «сильные» типы**

```typescript
class User {
  public name: string;
  public role: UserRole;
  public address: Address;

  constructor(name: string, role: UserRole, address: Address) {
    this.name = name;
    this.role = role;
    this.address = address;
  }

  public printUserInfo(): void {
    console.log(`Имя: ${this.name}`);

    // Теперь не боимся опечаток — если в Role нет значения, TypeScript не позволит скомпилировать код
    switch (this.role) {
      case UserRole.Admin:
        console.log("Права администратора");
        break;
      case UserRole.User:
        console.log("Обычный пользователь");
        break;
      case UserRole.Guest:
        console.log("Гость");
        break;
    }

    // Адрес уже структурирован, не нужно вручную что-то «парсить»
    console.log(`Адрес: ${this.address.toString()}`);
  }
}

// Пример использования
const address = new Address("Москва", "Пушкина", "10");
const user2 = new User("Иван", UserRole.Admin, address);
user2.printUserInfo();
```

Теперь:

* Роль пользователя задаётся `UserRole.Admin` (или другим элементом enum). Ошибки типа `"ADMON"` выявятся на этапе компиляции.
* Адрес описан классом `Address`, и у нас есть чёткие поля `city`, `street`, `house`. Можно добавлять методы валидации, форматирования и т. д.

**4. Преимущества такого подхода**

* Код становится понятнее: «роль» и «адрес» — полноценные сущности со своей логикой, а не «чёрный ящик».
* Снижается вероятность ошибок — никаких «магических» строк для роли, никаких расхождений в структуре адреса.
* При расширении (новые поля, методы и т.п.) всё логично добавляется в соответствующие классы.

</details>

<details>

<summary>Long Parameter List (Длинный список параметров)</summary>

**Long Parameter List** — это антипаттерн, возникающий, когда метод или функция принимает чрезмерно много аргументов (5, 6, 10 и т.д.). Такой подход усложняет понимание кода, увеличивает вероятность путаницы при вызове и делает метод трудным для сопровождения.

#### Пример «как не надо»

Ниже приведён выдуманный пример с методом, который принимает длинный список параметров для «обработки заказа» (не забываем, что реальный код может быть ещё более громоздким).

```typescript
class OrderService {
  public processOrder(
    orderId: number,
    userId: number,
    discountCode: string | null,
    addressCity: string,
    addressStreet: string,
    addressHouse: string,
    paymentType: string,
    deliveryDate: Date,
    isGift: boolean
  ): void {
    console.log("Обработка заказа...");
    
    // Внутри метода много логики, завязанной на все эти параметры
    if (discountCode) {
      console.log(`Применяем скидку по коду: ${discountCode}`);
    }
    console.log(`Отправка на адрес: ${addressCity}, ${addressStreet}, ${addressHouse}`);
    
    if (isGift) {
      console.log("Заказ помечен как подарок");
    }

    console.log(`Тип оплаты: ${paymentType}`);
    console.log(`Дата доставки: ${deliveryDate.toDateString()}`);
    
    // Много другой логики...
  }
}

// Использование:
const service = new OrderService();
service.processOrder(
  123,
  456,
  "WELCOME2025",
  "Москва",
  "Ленина",
  "10",
  "CREDIT_CARD",
  new Date("2025-03-20"),
  true
);
```

* Параметров слишком много: трудно запомнить их порядок и типы.
* Любая передача «не на то место» может привести к ошибкам, которые сложно сразу отследить.
* Добавление нового условия или опции вынудит расширять и без того громоздкий список.

***

#### Как сделать лучше

**1. Использовать объект для параметров**

Вместо передачи множества отдельных параметров можно передавать объект, инкапсулирующий все необходимые данные. Это делает код более читаемым и масштабируемым.

```typescript
interface OrderParams {
  orderId: number;
  userId: number;
  discountCode?: string;  // ? означает, что поле необязательное
  address: {
    city: string;
    street: string;
    house: string;
  };
  paymentType: string;
  deliveryDate: Date;
  isGift?: boolean;       // Тоже необязательное поле
}

class OrderServiceRefactored {
  public processOrder(params: OrderParams): void {
    console.log("Обработка заказа...");

    // Можем деструктурировать для удобства
    const { orderId, userId, discountCode, address, paymentType, deliveryDate, isGift } = params;

    if (discountCode) {
      console.log(`Применяем скидку по коду: ${discountCode}`);
    }
    console.log(`Отправка на адрес: ${address.city}, ${address.street}, ${address.house}`);

    if (isGift) {
      console.log("Заказ помечен как подарок");
    }

    console.log(`Тип оплаты: ${paymentType}`);
    console.log(`Дата доставки: ${deliveryDate.toDateString()}`);
    
    // ... остальная логика
  }
}

// Использование:
const service2 = new OrderServiceRefactored();
service2.processOrder({
  orderId: 123,
  userId: 456,
  discountCode: "WELCOME2025",
  address: {
    city: "Москва",
    street: "Ленина",
    house: "10",
  },
  paymentType: "CREDIT_CARD",
  deliveryDate: new Date("2025-03-20"),
  isGift: true,
});
```

* Теперь мы передаём один объект `OrderParams`.
* Если порядок свойств изменится, ошибки не возникнет — TypeScript сверяет структуру по ключам.
* Код становится читабельнее: сразу видно, что мы устанавливаем `orderId` и т.д.
* Добавлять новые поля проще: достаточно дополнить интерфейс `OrderParams`, не меняя сигнатуру метода.

**2. Разделять параметры по объектам/классам**

Если метод по-прежнему делает слишком много, можно выделить несколько сущностей:

* Отдельный объект `DeliveryInfo` (или класс) для хранения информации о доставке (адрес, дата).
* Отдельный объект `PaymentInfo` для типа оплаты, реквизитов и т.д.
* А уже на их основе составлять общий `OrderParams`.

**3. Принцип единственной ответственности (SRP)**

Если метод слишком «много знает», стоит подумать, не лучше ли часть логики вынести в отдельные методы или классы.\
Например:

* `PaymentService` отвечает за процесс оплаты.
* `DeliveryService` занимается доставкой.
* `OrderService` координирует вызовы между ними.

***

Таким образом, антипаттерн **Long Parameter List** делает метод неудобным для использования и сопровождения. Решается он, в первую очередь, путём агрегирования связанных параметров в объекты или структуры, а при необходимости — разделением логики между несколькими методами/классами.

</details>

<details>

<summary>Data Clumps (Группы данных)</summary>

«Data Clumps» встречается, когда одна и та же группа связанных параметров или полей (две, три и более переменных) постоянно используются совместно во многих местах кода, но при этом не объединены в отдельную структуру (класс, объект). Вместо этого эти переменные передаются «пакетом» из метода в метод или размножаются в нескольких классах. Это усложняет код, усложняет модификации и повышает риск несогласованного изменения данных.

***

#### Пример «как не надо»

Предположим, что у нас есть класс, занимающийся обработкой информации о сотруднике, и в нескольких местах кода мы постоянно передаём одни и те же данные: `firstName`, `lastName`, `department`, `position`, причём они нередко появляются вместе:

```typescript
class EmployeeService {
  public hireEmployee(
    firstName: string,
    lastName: string,
    department: string,
    position: string
  ): void {
    // ... логика найма (добавление в БД, уведомление и т.д.)
    console.log(
      `Новый сотрудник: ${firstName} ${lastName}, отдел: ${department}, должность: ${position}`
    );
  }

  public transferEmployee(
    firstName: string,
    lastName: string,
    currentDepartment: string,
    newDepartment: string
  ): void {
    // ... логика перевода сотрудника из одного отдела в другой
    console.log(
      `Сотрудник: ${firstName} ${lastName} переведён из отдела ${currentDepartment} в отдел ${newDepartment}`
    );
  }

  public promoteEmployee(
    firstName: string,
    lastName: string,
    position: string
  ): void {
    // ... логика повышения сотрудника
    console.log(
      `Сотрудник: ${firstName} ${lastName} повышен до должности: ${position}`
    );
  }
}
```

* Мы видим «кучу» данных: `firstName`, `lastName`, `department`, `position`.
* Вызовы методов могут выглядеть так:

```typescript
const employeeService = new EmployeeService();

// Нанимаем нового сотрудника
employeeService.hireEmployee("Иван", "Иванов", "IT", "Разработчик");

// Переводим сотрудника в другой отдел
employeeService.transferEmployee("Иван", "Иванов", "IT", "QA");

// Повышаем сотрудника
employeeService.promoteEmployee("Иван", "Иванов", "Team Lead");
```

* Видно, что `firstName`, `lastName` фигурируют почти везде. Отдел и должность тоже часто встречаются. В случае расширения логики для сотрудников придётся дублировать ещё больше параметров.
* Если мы решим переименовать `position` в `jobTitle` или добавить новые поля, нужно менять сигнатуры многих методов и места их вызова.

***

#### Как сделать лучше

**1. Объединить связанные данные в класс или интерфейс**

Создадим класс (или интерфейс) `EmployeeInfo`, который будет представлять сотрудника. Если у нас есть разные сценарии (полная информация, сокращённая, и т. д.), можно сделать несколько типов:

```typescript
interface EmployeeInfo {
  firstName: string;
  lastName: string;
  department: string;
  position: string;
}
```

Теперь методы могут выглядеть так:

```typescript
class EmployeeServiceRefactored {
  public hireEmployee(employee: EmployeeInfo): void {
    console.log(
      `Новый сотрудник: ${employee.firstName} ${employee.lastName}, отдел: ${employee.department}, должность: ${employee.position}`
    );
    // ... логика найма
  }

  public transferEmployee(
    employee: EmployeeInfo,
    newDepartment: string
  ): void {
    console.log(
      `Сотрудник: ${employee.firstName} ${employee.lastName} переведён из отдела ${employee.department} в отдел ${newDepartment}`
    );
    // ... логика перевода
    // Допустим, мы можем обновить employee.department, если нужно
    // employee.department = newDepartment;
  }

  public promoteEmployee(employee: EmployeeInfo, newPosition: string): void {
    console.log(
      `Сотрудник: ${employee.firstName} ${employee.lastName} повышен до должности: ${newPosition}`
    );
    // ... логика повышения
  }
}
```

* Теперь у нас нет дублирования «кучи» параметров. Мы работаем с `EmployeeInfo` как с единой сущностью.
* При добавлении новых полей (например, `middleName`, `birthday`, `salary`) мы меняем только `EmployeeInfo`, а не каждую сигнатуру методов.

**2. При необходимости использовать классы**

Если данные требуют логики (например, форматирование полного имени, валидация и т. п.), есть смысл создать класс:

```typescript
class Employee {
  constructor(
    public firstName: string,
    public lastName: string,
    public department: string,
    public position: string
  ) {}

  public getFullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

Это позволяет инкапсулировать методы, связанные с сотрудником, внутри самого класса.\
Методы `EmployeeServiceRefactored` принимают и возвращают уже готовые объекты `Employee`, и если в будущем мы захотим добавить ещё поля или методы, то сделаем это в одном месте.

**3. Разделение ответственности**

Если в коде появляется так много полей, что даже в класс `Employee` сложно уместить их без «распухания», возможно, стоит разделять их по другим объектам:

* `PersonalInfo` (ФИО, день рождения)
* `JobInfo` (должность, дата приёма, отдел, зарплата)
* `ContactInfo` (телефон, email и т. д.) И уже `Employee` будет агрегировать эти структуры.

***

Таким образом, антипаттерн **Data Clumps** проявляется в виде повторяющегося набора связанных параметров/полей, которые логически должны быть «спаянными» и представлены отдельной сущностью или объектом. Объединение данных в классы или структуры решает проблему — упрощает код, делает его более читаемым и облегчает внесение изменений.

</details>

### Нарушители объектно-ориентированного дизайна

Все эти запахи являют собой неполное или неправильное использование возможностей объектно-ориентированного программирования.

<details>

<summary>Switch Statements (Операторы switch)</summary>

Антипаттерн «Switch Statements» возникает, когда в коде встречаются большие операторы `switch-case` (или длинные цепочки `if-else`), которые разрастаются по мере добавления новых условий или типов данных. Чаще всего такие конструкции пытаются определить поведение в зависимости от «типа» или «состояния» объекта — хотя гораздо эффективнее вместо этого использовать полиморфизм, паттерн Strategy или State.

***

#### Пример «как не надо»

Вместо того чтобы распределять логику по соответствующим классам или стратегиям, мы делаем один метод с большим `switch`. Допустим, у нас есть система оповещений, которая по типу уведомления (email, sms, push и т. д.) выбирает, как его отправить:

```typescript
enum NotificationType {
  Email = "EMAIL",
  SMS = "SMS",
  Push = "PUSH",
}

class NotificationService {
  public sendNotification(type: NotificationType, message: string): void {
    switch (type) {
      case NotificationType.Email:
        // Логика отправки email
        console.log(`Отправляем Email: ${message}`);
        break;
      case NotificationType.SMS:
        // Логика отправки SMS
        console.log(`Отправляем SMS: ${message}`);
        break;
      case NotificationType.Push:
        // Логика отправки Push
        console.log(`Отправляем Push: ${message}`);
        break;
      default:
        console.log("Неизвестный тип уведомления");
    }
  }
}

// Использование
const service = new NotificationService();
service.sendNotification(NotificationType.Email, "Тестовое сообщение");
service.sendNotification(NotificationType.SMS, "Тестовое сообщение");
```

* По мере расширения списка типов (`Telegram`, `WhatsApp`, `Slack` и т.д.) код в `switch` будет становиться всё больше.
* Если в разных местах кода нужна разная логика в зависимости от `type`, в проекте может появиться несколько подобных `switch` — это затрудняет поддержку и тестирование.

***

#### Как сделать лучше

**1. Использовать полиморфизм (Strategy)**

Каждый тип уведомления можно оформить в отдельном классе, который реализует общий интерфейс. Тогда мы избавляемся от громоздкого `switch`.

```typescript
// Шаг 1. Создаём общий интерфейс для «отправщиков» уведомлений
interface Notifier {
  send(message: string): void;
}

// Шаг 2. Реализуем конкретные «стратегии» для Email, SMS, Push
class EmailNotifier implements Notifier {
  send(message: string): void {
    console.log(`Отправляем Email: ${message}`);
  }
}

class SMSNotifier implements Notifier {
  send(message: string): void {
    console.log(`Отправляем SMS: ${message}`);
  }
}

class PushNotifier implements Notifier {
  send(message: string): void {
    console.log(`Отправляем Push: ${message}`);
  }
}

// Шаг 3. Контекст (NotificationService) принимает любую реализацию Notifier
class NotificationServiceRefactored {
  private notifier: Notifier;

  constructor(notifier: Notifier) {
    this.notifier = notifier;
  }

  public setNotifier(notifier: Notifier): void {
    this.notifier = notifier;
  }

  public sendNotification(message: string): void {
    this.notifier.send(message);
  }
}

// Использование
function main(): void {
  // Начинаем с Email
  const service = new NotificationServiceRefactored(new EmailNotifier());
  service.sendNotification("Сообщение для Email");

  // При необходимости меняем стратегию на SMS
  service.setNotifier(new SMSNotifier());
  service.sendNotification("Сообщение для SMS");

  // И так далее...
  service.setNotifier(new PushNotifier());
  service.sendNotification("Сообщение для Push");
}

main();
```

* Теперь мы можем просто добавить новый класс-уведомитель (`SlackNotifier`, `TelegramNotifier`) без изменения кода сервиса.
* Код становится гибче: мы используем полиморфизм вместо больших ветвлений.

**2. Паттерн State или Factory (в зависимости от задачи)**

Если логика на самом деле связана с «состоянием» объекта, стоит подумать о паттерне **State**. Если же мы хотим выдавать разные типы уведомлений по запросу, можно применить **Factory**, возвращающую нужный тип «отправщика».

```typescript
// Пример фабрики:
class NotifierFactory {
  public static createNotifier(type: NotificationType): Notifier {
    switch (type) {
      case NotificationType.Email:
        return new EmailNotifier();
      case NotificationType.SMS:
        return new SMSNotifier();
      case NotificationType.Push:
        return new PushNotifier();
      default:
        throw new Error("Неизвестный тип уведомления");
    }
  }
}
```

* Сама фабрика локализует `switch`, но это единственное место, где мы проверяем `type`.
* Дальше, по коду, передаём объект `Notifier` и работаем с ним полиморфно.

**3. Распределить логику в соответствующие классы/модули**

Если имеем дело не только с «отправкой уведомлений», но и с обработкой разных типов сущностей (например, разные форматы), обычно стоит переносить код, специфичный для каждого формата, в собственные классы или модули.

* Пусть «JPEG-обработчик» умеет читать, конвертировать, показывать превью и т. д.
* «PNG-обработчик» реализует то же самое по-своему.
* Код, использующий их, не делает `switch` по формату, а вызывает абстрактный интерфейс.

***

Таким образом, избавляясь от громоздких `switch-case`, мы повышаем гибкость, удобство расширения и читаемость кода. Вместо «больших условных блоков» разумнее использовать полиморфизм, паттерны **Strategy**, **State** или **Factory** — в зависимости от конкретных целей.

</details>

<details>

<summary>Temporary Field (Временное поле)</summary>

**Temporary Field** — антипаттерн, когда в классе существуют поля (свойства), которые иногда не используются или нужны лишь при определённых условиях (при этом большую часть времени пусты/`null`/неактуальны). Это усложняет дизайн: код, работающий с таким полем, часто вынужден проверять его на `null` или наличие/отсутствие значений. Часто бывает, что эти поля нужны всего для пары методов или для определённого участка работы, тогда как в остальных местах класса они бесполезны.

***

#### Пример «как не надо»

Предположим, у нас есть класс, занимающийся обработкой заказов (OrderProcessor). В нём завели поля для различных этапов обработки, но некоторые поля нужны только на этапе валидации, а некоторые — только на этапе доставки. В итоге часть полей может быть не заполнена при большинстве вызовов методов этого класса.

```typescript
class OrderProcessor {
  // Поля, которые не всегда нужны
  private validationError: string | null = null; // Будет заполнен только при ошибке
  private deliveryAddress: string | null = null; // Заполняется, если заказ дошёл до этапа доставки
  private discountCode: string | null = null;    // Может быть пустым

  // Допустим, этот метод пытается валидировать заказ и записывает ошибку, если что-то не так
  public validateOrder(orderId: number): boolean {
    // Логика валидации...
    if (orderId <= 0) {
      this.validationError = "Некорректный номер заказа";
      return false;
    }
    // Если всё ок — обнуляем ошибку
    this.validationError = null;
    return true;
  }

  // Метод, который устанавливает данные для доставки
  public prepareDelivery(address: string): void {
    this.deliveryAddress = address;
    console.log(`Адрес доставки: ${this.deliveryAddress}`);
  }

  // Метод, который применяет скидку
  public applyDiscount(code: string): void {
    this.discountCode = code;
    console.log(`Применён скидочный код: ${this.discountCode}`);
  }

  // Метод, который завершает заказ
  public completeOrder(): void {
    if (this.validationError) {
      console.log(`Невозможно завершить заказ: ${this.validationError}`);
      return;
    }
    if (!this.deliveryAddress) {
      console.log("Адрес доставки не указан!");
      return;
    }
    // Прочая логика завершения...
    console.log(`Заказ завершён. Скидка: ${this.discountCode || "нет"}`);
  }
}

// Использование
const processor = new OrderProcessor();
if (processor.validateOrder(10)) {
  processor.prepareDelivery("Москва, Тверская, 1");
  processor.applyDiscount("NEWYEAR");
  processor.completeOrder();
}
```

* В данном классе:
  * `validationError`, `deliveryAddress`, `discountCode` чаще всего инициализируются `null`.
  * Используются только в определённых методах и могут вводить путаницу — например, что делать, если метод `completeOrder` вызывается раньше `prepareDelivery`?
  * Поля «живут» во всём объекте, хотя их смысл важен только во время отдельных операций.

***

#### Как сделать лучше

**1. Разделить ответственность по меньшим объектам/методам**

Один из способов избавиться от временных полей — разделить класс на несколько, более узконаправленных:

* **Validator** (класс/объект, занимающийся исключительно валидацией заказа и хранящий при необходимости информацию об ошибках).
* **DeliveryService** (отдельная сущность, которая принимает информацию о доставке, формирует доставку и т. п.).
* **DiscountApplier** или логика скидок в отдельном месте, если нужно особое хранение/обработка.

Вместо одного большого `OrderProcessor` с кучей полей, которые «иногда нужны», у нас будет набор сервисов/объектов, и каждый хранит только то, что ему действительно важно.

Пример (очень упрощённый) рефакторинга:

```typescript
class OrderValidator {
  public validate(orderId: number): { isValid: boolean; error?: string } {
    if (orderId <= 0) {
      return { isValid: false, error: "Некорректный номер заказа" };
    }
    return { isValid: true };
  }
}

class DeliveryService {
  public prepareDelivery(orderId: number, address: string): void {
    console.log(`Готовим доставку для заказа ${orderId} на адрес: ${address}`);
  }
}

class DiscountService {
  public applyDiscount(orderId: number, discountCode: string): void {
    console.log(`Применяем скидку к заказу ${orderId}: ${discountCode}`);
  }
}

class OrderProcessorRefactored {
  private validator: OrderValidator;
  private deliveryService: DeliveryService;
  private discountService: DiscountService;

  constructor() {
    this.validator = new OrderValidator();
    this.deliveryService = new DeliveryService();
    this.discountService = new DiscountService();
  }

  public processOrder(orderId: number, address: string, discountCode?: string): void {
    // Валидация
    const validationResult = this.validator.validate(orderId);
    if (!validationResult.isValid) {
      console.log(`Ошибка валидации: ${validationResult.error}`);
      return;
    }

    // Подготовка доставки
    this.deliveryService.prepareDelivery(orderId, address);

    // Применение скидки, если указана
    if (discountCode) {
      this.discountService.applyDiscount(orderId, discountCode);
    }

    // Завершаем заказ
    console.log(`Заказ ${orderId} завершён.`);
  }
}
```

* Теперь нет длинного списка временных полей в одном классе.
* `OrderProcessorRefactored` аккумулирует логику — вызывает нужные сервисы последовательно.
* Информация о том, есть ошибка валидации, возвращается в виде результата работы `validator`, а не хранится в поле «где-то в классе».

**2. Использовать локальные переменные вместо полей**

Если поле нужно лишь внутри одного метода (или небольшой цепочки методов), достаточно ограничиться локальными переменными, не «захламляя» класс:

```typescript
class OrderProcessorLocalVars {
  public processOrder(orderId: number, address: string, discountCode?: string): void {
    // Валидация
    let error: string | null = null;
    if (orderId <= 0) {
      error = "Некорректный номер заказа";
    }

    if (error) {
      console.log(`Ошибка: ${error}`);
      return;
    }

    // Настройка доставки
    console.log(`Адрес доставки: ${address}`);

    // Применение скидки
    if (discountCode) {
      console.log(`Применяем скидку: ${discountCode}`);
    }

    console.log("Заказ завершён!");
  }
}
```

* «Временные» данные `error`, `discountCode` и т. д. не висят полями в объекте, а живут только внутри `processOrder`.

**3. Уточнить жизненный цикл данных**

Если поле необходимо только на время определённой операции или состояния, подумайте о паттерне **State**. Например, на «шаге 1» класс переходит в состояние `Validating`, где хранит некоторые поля, а на «шаге 2» переключается в состояние `Delivering`, с другими полями. Это предотвращает появление полей, которые «живут» в объекте постоянно, а используются только на одном этапе.

***

Таким образом, антипаттерн **Temporary Field** часто сигнализирует о том, что класс несёт слишком много разрозненных обязанностей или что поля используются только в части логики. Решение — разделять класс по ответственностям, использовать локальные переменные или применять паттерны (State, Strategy и т. д.) для чёткого размежевания поведения.

</details>

<details>

<summary>Refused Bequest (Отказ от наследства)</summary>

«Refused Bequest» возникает, когда подкласс унаследовал от родителя методы и/или поля, которые ему на деле не нужны или не подходят. Подкласс фактически «отвергает» часть унаследованной функциональности: не использует, переопределяет пустыми методами либо бросает исключения при их вызове. Это говорит о проблемах в иерархии наследования и часто противоречит принципу подстановки Лисков (LSP), ведь наследник не может выступать полноправной заменой родителя.

***

#### Пример «как не надо»

Рассмотрим, что у нас есть базовый класс `Animal` с методами `eat()`, `move()`, `makeSound()`, а затем мы создаём подкласс `Fish`, который наследует их. Но, например, `move()` в базовом классе подразумевает передвижение по суше (ходьба), а `Fish` передвигается совсем по-другому — и часть логики «ходьбы» ей не нужна.

```typescript
class Animal {
  public eat(): void {
    console.log("Поедаем пищу...");
  }

  public move(): void {
    console.log("Идём по земле...");
  }

  public makeSound(): void {
    console.log("Издаём звук...");
  }
}

class Fish extends Animal {
  // Fish "отвергает" часть методов от Animal

  public move(): void {
    // Игнорируем логику "Идём по земле"
    // Или переопределяем так, что исходное поведение не имеет смысла
    console.log("Я рыба, я плаваю!");
  }

  // Допустим, makeSound нам не очень нужен, рыбы не издают слышимых звуков
  // Но метод приходится иметь, так как он унаследован
  public makeSound(): void {
    // Либо ничего не делаем, либо бросаем исключение,
    // что "Рыбы не говорят"
    console.log("Рыбы обычно не издают звуки.");
  }
}

// Использование
const nemo = new Fish();
nemo.eat();        // "Поедаем пищу..."
nemo.move();       // "Я рыба, я плаваю!"
nemo.makeSound();  // "Рыбы обычно не издают звуки."
```

* `Fish` унаследовал методы, которые частично ему не подходят (логика «ходьбы» и «голоса»).
* Приходится переопределять или оставлять «заглушки», что указывает на неправильное моделирование.

***

#### Как сделать лучше

**1. Уточнить иерархию (использовать композицию вместо наследования)**

Вместо классического «Animal → Fish» можно выделить более корректные абстракции или применить композицию. Например, в отдельных классах описать различное поведение передвижения (Strategy):

```typescript
interface MoveBehavior {
  move(): void;
}

class WalkBehavior implements MoveBehavior {
  public move(): void {
    console.log("Иду по земле...");
  }
}

class SwimBehavior implements MoveBehavior {
  public move(): void {
    console.log("Плаваю в воде...");
  }
}

class Animal2 {
  private moveBehavior: MoveBehavior;

  constructor(moveBehavior: MoveBehavior) {
    this.moveBehavior = moveBehavior;
  }

  public performMove(): void {
    this.moveBehavior.move();
  }

  public eat(): void {
    console.log("Поедаю пищу...");
  }
}

class Fish2 extends Animal2 {
  constructor() {
    super(new SwimBehavior());
  }
}

class Dog extends Animal2 {
  constructor() {
    super(new WalkBehavior());
  }
}

// Использование
const fish = new Fish2();
fish.eat();         // "Поедаю пищу..."
fish.performMove(); // "Плаваю в воде..."

const dog = new Dog();
dog.eat();          // "Поедаю пищу..."
dog.performMove();  // "Иду по земле..."
```

* Теперь нет ненужных методов, которые класс должен «отвергать».
* Если нужно добавить новую логику передвижения, мы создаём новый `MoveBehavior`, не ломая иерархию.

**2. Создать более подходящий родительский класс или интерфейс**

Если часть подклассов имеет кардинально разные характеристики, возможно, им не нужно наследоваться от общего класса.

* Можно использовать более узкую абстракцию (например, `Swimmable` для рыб, `Walkable` для собак).
* Для общих черт (например, «живое существо») иметь базовые поля и методы, которые действительно нужны всем наследникам.

```typescript
abstract class Creature {
  public eat(): void {
    console.log("Поедаю пищу...");
  }
  // makeSound() и move() не делаем абстрактными, если не у всех это есть/нужно
}

interface Swimmable {
  swim(): void;
}

class Fish3 extends Creature implements Swimmable {
  public swim(): void {
    console.log("Плаваю в воде!");
  }
}
```

* Теперь рыбе не приходится «делать вид», что она может ходить или издавать звук.
* Абстракции становятся точнее, «Fish» не «отвергает» методы из родителя.

**3. Перейти к композиции или паттернам**

Вместо наследования, когда функциональность сильно разнится, композиция (складывание поведения из нескольких компонентов) обычно гибче. Паттерны **Strategy**, **State** помогают изолировать или заменять разные части логики, не перенося лишний код в подклассы.

***

Таким образом, антипаттерн **Refused Bequest** говорит о том, что иерархия наследования выбрана неверно: подкласс унаследовал что-то, что ему не нужно. Решение — использовать более корректную абстракцию, проектировать иерархию с учётом реальных общих черт либо заменить наследование композицией и поведением (паттерны Strategy, State и пр.).

</details>

<details>

<summary>Alternative Classes with Different Interfaces (Альтернативные классы с разными интерфейсами)</summary>

Данный антипаттерн возникает, когда в проекте присутствуют два или больше классов, выполняющих схожие задачи, но имеющих при этом различные и несовместимые методы (интерфейсы). Часто эти классы «дублируют» функциональность друг друга, просто по-разному называя методы, принимая разные наборы параметров и т. п. Это вносит путаницу: неясно, какой класс использовать, и зачем их два (или больше), если логика по сути одинаковая.

***

#### Пример «как не надо»

Допустим, у нас есть два класса, занимающихся сохранением и загрузкой пользователей. Оба «работают» с данными пользователя, но их методы и интерфейсы никак не согласованы.

```typescript
class UserStorage {
  public storeUser(name: string, age: number): void {
    console.log(`Сохраняем пользователя в хранилище: имя=${name}, возраст=${age}`);
    // ... логика сохранения
  }

  public fetchUser(): { name: string; age: number } | null {
    console.log("Загружаем пользователя из хранилища...");
    // ... логика загрузки
    return { name: "Иван", age: 30 }; 
  }
}

class UserRepository {
  public save(data: { username: string; years: number }): void {
    console.log(`Сохраняем пользователя в репозитории: ${data.username}, возраст=${data.years}`);
    // ... логика сохранения
  }

  public getUser(): { username: string; years: number } | undefined {
    console.log("Загружаем пользователя из репозитория...");
    // ... логика загрузки
    return { username: "Пётр", years: 25 };
  }
}

// Использование
const storage = new UserStorage();
storage.storeUser("Иван", 30);
const userFromStorage = storage.fetchUser();

const repository = new UserRepository();
repository.save({ username: "Пётр", years: 25 });
const userFromRepo = repository.getUser();
```

В данном примере:

1. **UserStorage** оперирует методами `storeUser` / `fetchUser`, которые принимают и возвращают `(name, age)`.
2. **UserRepository** использует `save` / `getUser`, где данные представлены объектом `{ username, years }`.

Оба класса делают одно и то же — **сохраняют и извлекают** сущность «пользователь». Однако методологии и интерфейсы различны, что порождает путаницу и дублирование.

***

#### Как сделать лучше

**1. Объединить классы или согласовать интерфейс**

Если логика действительно одна и та же, проще объединить всё в одном классе. Или же, если есть веские причины иметь два разных класса (например, разная инфраструктура хранения), желательно **согласовать интерфейсы**, чтобы они были идентичны или соответствовали единому контракту (например, `IUserRepository`).

```typescript
interface IUserRepository {
  saveUser(user: { name: string; age: number }): void;
  loadUser(): { name: string; age: number } | null;
}

class UserStorageRefactored implements IUserRepository {
  public saveUser(user: { name: string; age: number }): void {
    console.log(`Сохраняем в UserStorage: имя=${user.name}, возраст=${user.age}`);
    // ... логика сохранения
  }

  public loadUser(): { name: string; age: number } | null {
    console.log("Загружаем из UserStorage...");
    return { name: "Иван", age: 30 };
  }
}

class UserRepositoryRefactored implements IUserRepository {
  public saveUser(user: { name: string; age: number }): void {
    console.log(`Сохраняем в UserRepository: имя=${user.name}, возраст=${user.age}`);
    // ... логика сохранения
  }

  public loadUser(): { name: string; age: number } | null {
    console.log("Загружаем из UserRepository...");
    return { name: "Пётр", age: 25 };
  }
}

// Использование:
function main(): void {
  // Допустим, мы можем легко подменять одну реализацию другой
  let repo: IUserRepository = new UserStorageRefactored();
  repo.saveUser({ name: "Иван", age: 30 });
  const user1 = repo.loadUser();

  repo = new UserRepositoryRefactored();
  repo.saveUser({ name: "Пётр", age: 25 });
  const user2 = repo.loadUser();
}

main();
```

* Теперь оба класса реализуют единый интерфейс **`IUserRepository`** с одинаковыми названиями и сигнатурами методов.
* Клиентскому коду (использующему `IUserRepository`) не важно, с кем он работает — с `UserStorageRefactored` или `UserRepositoryRefactored`.
* Если логика хранения действительно разная (например, в одном случае — база данных, в другом — локальные файлы), она «спрятана» внутри, а внешний интерфейс одинаков.

**2. Выделить общий базовый класс (если уместно)**

Если есть смысл хранить часть общей логики в базовом классе, а различия в наследниках — тоже вариант. Но зачастую достаточно общего интерфейса (или абстрактного класса).

**3. Соблюдать принцип единства интерфейсов**

При проектировании желательно заранее определить единый «контракт» (интерфейс) для похожих компонентов. Тогда вместо «двух альтернативных классов с разными методами» у нас будут две реализации одного и того же интерфейса — это гораздо понятнее и гибче.

***

Таким образом, антипаттерн **Alternative Classes with Different Interfaces** указывает на дублирующийся функционал, но несовместимые интерфейсы у классов. Решение — **унифицировать** методы в единый «контракт» (интерфейс или базовый класс) или объединить логику, если разделение не оправдано. Это делает код проще для понимания и расширения.

</details>

### Утяжелители изменений

Эти запахи приводят к тому, что при необходимости что-то поменять в одном месте программы, вам приходится вносить множество изменений в других местах. Это серьезно осложняет и удорожает развитие программы.

<details>

<summary>Divergent Change (Расходящиеся модификации)</summary>

**Divergent Change** — это ситуация, когда в одном и том же классе или модуле приходится вносить изменения по совершенно разным поводам. Другими словами, класс или модуль страдает от слишком большого количества потенциальных причин для изменения: то есть одно изменение касается отображения, другое — валидации, третье — бизнес-логики и т. д., и всё это сосредоточено в одном месте. Такой код нарушает **принцип единственной ответственности** (Single Responsibility Principle, SRP) и со временем становится трудно поддерживаемым.

***

#### Пример «как не надо»

Допустим, у нас есть класс `UserController`, в котором сосредоточена логика и по работе с базой данных (CRUD-операции), и по форматированию вывода для пользовательского интерфейса, и по валидации данных пользователя, и даже по логированию. Каждый раз, когда меняется формат вывода, правила валидации или способ сохранения данных, нам приходится модифицировать `UserController`.

```typescript
class UserController {
  // "Бизнес-логика"
  public createUser(data: any): void {
    // 1. Валидируем поля
    if (!data.name) {
      throw new Error("Имя не указано");
    }
    if (data.age < 0) {
      throw new Error("Некорректный возраст");
    }

    // 2. Сохраняем в БД (предположим, без репозитория, напрямую)
    console.log("Подключаемся к БД...");
    console.log("Вставляем запись в таблицу users...");
    // ... логика SQL-запроса / или другой способ

    // 3. Логируем событие
    console.log(`[LOG] Создан пользователь: ${data.name}`);

    // 4. Возвращаем форматированный результат для UI
    console.log(`Пользователь ${data.name} успешно создан!`);
  }

  public getUser(id: number): void {
    // 1. Читаем из БД
    console.log("Подключаемся к БД...");
    // ... SQL-запрос SELECT ...

    // 2. Логируем
    console.log(`[LOG] Получен пользователь с id=${id}`);

    // 3. Форматируем вывод для фронтенда
    console.log(`Пользователь #${id}, Имя: (данные из БД)`);
  }

  // И так далее... Методы updateUser, deleteUser,
  // в каждом могут быть свои "разношерстные" куски логики
}
```

Здесь на лицо **«расхождение изменений»**:

* Меняются правила валидации пользователей — надо лезть в методы.
* Меняется способ хранения в БД — снова правим те же методы.
* Меняется формат вывода (надо, например, возвращать JSON) — опять открываем все методы.

Таким образом, у нас «один класс, сто причин для изменения».

***

#### Как сделать лучше

**1. Разделить ответственность (в соответствии с SRP)**

Вместо того чтобы один класс делал и сохранение в БД, и логику, и форматирование ответа, мы можем выделить несколько классов/уровней:

* **Слой валидации** (например, `UserValidator` или middleware).
* **Слой доступа к данным** (например, `UserRepository` или `UserService` для CRUD), который будет отвечать только за взаимодействие с хранилищем данных.
* **Контроллер** (`UserController`), в задачу которого входит только связать запрос (HTTP) и логику приложения, делегируя остальные детали соответствующим сервисам.

Упрощённо это может выглядеть так:

```typescript
class UserValidator {
  public validate(data: any): void {
    if (!data.name) {
      throw new Error("Имя не указано");
    }
    if (data.age < 0) {
      throw new Error("Некорректный возраст");
    }
  }
}

class UserRepository {
  // Отвечает за доступ к БД (CRUD-операции)
  public create(data: { name: string; age: number }): void {
    console.log("Подключаемся к БД...");
    console.log("INSERT INTO users ...");
    // Реальная логика сохранения
  }

  public findById(id: number): { id: number; name: string; age: number } {
    console.log("SELECT * FROM users WHERE id = " + id);
    return { id, name: "Иван", age: 30 }; // для примера
  }
}

class UserService {
  private validator: UserValidator;
  private repository: UserRepository;

  constructor(validator: UserValidator, repository: UserRepository) {
    this.validator = validator;
    this.repository = repository;
  }

  public createUser(data: any): void {
    this.validator.validate(data);
    this.repository.create({ name: data.name, age: data.age });
    console.log(`[LOG] Создан пользователь: ${data.name}`);
  }

  public getUser(id: number): { id: number; name: string; age: number } {
    const user = this.repository.findById(id);
    console.log(`[LOG] Получен пользователь с id=${id}`);
    return user;
  }
}

class UserControllerRefactored {
  private userService: UserService;

  constructor(userService: UserService) {
    this.userService = userService;
  }

  public createUserHandler(reqBody: any): void {
    // Контроллер только вызывает нужный метод сервиса
    this.userService.createUser(reqBody);
    // Контроллер решает, как отвечать пользователю
    console.log(`Пользователь ${reqBody.name} успешно создан!`);
  }

  public getUserHandler(id: number): void {
    const user = this.userService.getUser(id);
    // Форматируем вывод (JSON, текст и т.д.)
    console.log(`Пользователь: ${JSON.stringify(user)}`);
  }
}
```

Тут каждый класс/слой отвечает за свою задачу:

* **UserValidator** — только валидация.
* **UserRepository** — только взаимодействие с БД (CRUD).
* **UserService** — бизнес-логика, объединяющая валидацию, репозиторий, логи.
* **UserControllerRefactored** — «слой» для работы с HTTP-запросами/ответами (или CLI).

Если меняется **правило валидации**, трогаем только `UserValidator`.\
Если меняется **способ хранения**, меняем `UserRepository`.\
Если меняется **формат ответа**, корректируем логику в `UserControllerRefactored`.

**2. Разделять по функциональному признаку**

Если у вас есть класс, который «обслуживает» сразу несколько подсистем (например, валидация пользователя, логирование транзакций, отчёты о продажах), лучше создать разные модули/классы для каждого направления.\
**Divergent Change** → при любом изменении в одной из подсистем приходится править класс.

**3. Следовать принципам SOLID**

Особенно важно придерживаться **S** (SRP) — принцип единственной ответственности. Также помогает **I** (ISP) — принцип разделения интерфейса, не «перегружая» один интерфейс методами, которые нужны разным слоям/блокам.

***

Таким образом, антипаттерн **Divergent Change** проявляется, когда в одном классе или модуле сосредоточены различные виды логики, требующие изменений по разным причинам. Решение — **разделить** этот модуль по сферам ответственности, чтобы каждый класс/модуль менялся по одной-двум конкретным причинам, а не по множеству одновременно.

</details>

<details>

<summary>Shotgun Surgery (Стрельба дробью)</summary>

**Shotgun Surgery** — это «запах кода», при котором одно-единственное изменение в требованиях вынуждает вносить правки сразу в разные модули или классы. Получается, что мы «выстреливаем дробью» по всему проекту: чуть-чуть меняем в одном месте, чуть-чуть в другом, и так далее. Если пропустить хотя бы одно место, логика ломается. Это говорит о том, что похожая функциональность или одни и те же правила «размазаны» по коду, вместо того чтобы находиться в одном месте.

***

#### Пример «как не надо»

Допустим, у нас несколько модулей в интернет-магазине, и в каждом есть своя логика расчёта стоимости доставки (shipping cost). При этом правила расчёта везде одинаковые (например, если сумма заказа больше 3000, доставка бесплатна; иначе 200₽), но каждый модуль «самостоятельно» прописывает эти правила.

```typescript
// Модуль корзины
class Cart {
  public calculateCartTotal(items: Array<{ price: number }>): number {
    let total = items.reduce((acc, item) => acc + item.price, 0);

    // Логика доставки — если total < 3000, добавляем 200₽
    if (total < 3000) {
      total += 200;
    }
    return total;
  }
}

// Модуль оформления заказа
class Checkout {
  public finalizeOrder(items: Array<{ price: number }>): void {
    let total = items.reduce((acc, item) => acc + item.price, 0);

    // Опять прописано то же условие
    const deliveryCost = total < 3000 ? 200 : 0;
    total += deliveryCost;

    console.log(`Итоговая сумма: ${total}, доставка: ${deliveryCost}`);
    // ... Дальнейшая логика оформления
  }
}

// Модуль акций/скидок
class Promo {
  public applyPromoAndDelivery(orderAmount: number, hasPromo: boolean): number {
    // Ещё раз дублируем логику доставки
    const deliveryCost = orderAmount < 3000 ? 200 : 0;
    let total = orderAmount + deliveryCost;

    // Допустим, при наличии промокода снижаем итог на 10%
    if (hasPromo) {
      total *= 0.9;
    }

    return total;
  }
}
```

* Здесь, если вдруг правила доставки изменятся (скажем, порог 5000, а сама доставка 250₽), придётся править код в **трёх** местах (Cart, Checkout, Promo).
* Если пропустить одно из мест, логика будет неконсистентной.
* Так и проявляется «Shotgun Surgery»: одно правило (ставка доставки) «рассыпано» по проекту.

***

#### Как сделать лучше

**1. Централизовать логику в одном месте**

Вместо того чтобы повторять правило «если меньше 3000, то доставка 200₽» в каждом модуле, можно создать отдельный сервис или функцию, занимающуюся исключительно подсчётом доставки:

```typescript
class DeliveryService {
  private freeDeliveryThreshold = 3000;
  private deliveryPrice = 200;

  public calculateDelivery(orderAmount: number): number {
    return orderAmount < this.freeDeliveryThreshold ? this.deliveryPrice : 0;
  }
}

// Пример использования в других модулях
class CartRefactored {
  constructor(private deliveryService: DeliveryService) {}

  public calculateCartTotal(items: Array<{ price: number }>): number {
    let total = items.reduce((acc, item) => acc + item.price, 0);
    total += this.deliveryService.calculateDelivery(total);
    return total;
  }
}

class CheckoutRefactored {
  constructor(private deliveryService: DeliveryService) {}

  public finalizeOrder(items: Array<{ price: number }>): void {
    let total = items.reduce((acc, item) => acc + item.price, 0);
    const deliveryCost = this.deliveryService.calculateDelivery(total);
    total += deliveryCost;

    console.log(`Итоговая сумма: ${total}, доставка: ${deliveryCost}`);
  }
}

class PromoRefactored {
  constructor(private deliveryService: DeliveryService) {}

  public applyPromoAndDelivery(orderAmount: number, hasPromo: boolean): number {
    const deliveryCost = this.deliveryService.calculateDelivery(orderAmount);
    let total = orderAmount + deliveryCost;

    if (hasPromo) {
      total *= 0.9;
    }
    return total;
  }
}

// Пример демонстрации
function main() {
  const deliveryService = new DeliveryService();
  
  const cart = new CartRefactored(deliveryService);
  console.log("Cart total:", cart.calculateCartTotal([{ price: 1500 }, { price: 1000 }]));

  const checkout = new CheckoutRefactored(deliveryService);
  checkout.finalizeOrder([{ price: 1500 }, { price: 1000 }]);

  const promo = new PromoRefactored(deliveryService);
  console.log("Promo total:", promo.applyPromoAndDelivery(2500, true));
}

main();
```

* Теперь, если нужно изменить логику доставки, мы меняем её в **одном месте** — в `DeliveryService`.
* Нет риска, что забудем внести изменения где-то ещё: все модули «спрашивают» сервис о доставке.

**2. Применять принцип DRY**

Чтобы избежать «рассеянных» фрагментов одинаковой логики, помните о принципе **DRY (Don't Repeat Yourself)**. Если какая-то формула, условие или правило повторяется в нескольких местах — вынесите его в сервис/утилиту/абстракцию.

**3. При необходимости — паттерны Strategy, Factory и т. д.**

Если правила доставки ещё и зависят от контекста (разные регионы, виды доставки), уместно выделить несколько стратегий (`PickupStrategy`, `CourierStrategy` и т. п.). Главное — все эти стратегии управляются из одного места (например, DeliveryService), а не разбросаны по коду.

***

Таким образом, при **Shotgun Surgery** одно небольшое изменение приводит к модификациям во многих местах системы. Лучшая защита от этого — **централизовать** повторяющиеся правила или алгоритмы в одном месте (сервис/модуль), устранять дублирование и следовать более «чистой» архитектуре, где каждая задача сконцентрирована внутри собственного «узла», а не размазана по всему коду.

</details>

<details>

<summary>Parallel Inheritance Hierarchies (Параллельные иерархии наследования)</summary>

«Parallel Inheritance Hierarchies» возникает, когда изменения в одной иерархии классов вынуждают создавать соответствующие классы в другой иерархии. То есть, у нас есть два (или более) набора классов, которые развиваются «параллельно» — при добавлении нового подкласса в одной иерархии автоматически нужно добавить «зеркальный» подкласс в другой. Это усложняет проект и говорит о том, что логика слишком жёстко связана между разными классами.

***

#### Пример «как не надо»

Рассмотрим ситуацию, когда у нас есть иерархия форм оплаты (`PaymentMethod` → `CreditCardPayment`, `PayPalPayment`, `CryptoPayment`, …) и, параллельно, у нас есть иерархия «обработчиков логов» для этих форм оплаты (`PaymentLogger` → `CreditCardLogger`, `PayPalLogger`, `CryptoLogger`, …). Получается, при появлении нового способа оплаты (например, `BankTransferPayment`), придётся **добавлять** не только его класс, но и «зеркальный» класс-логгер (`BankTransferLogger`), который явно соответствует тому же способу оплаты.

```typescript
// Иерархия 1: Способы оплаты
abstract class PaymentMethod {
  public abstract pay(amount: number): void;
}

class CreditCardPayment extends PaymentMethod {
  public pay(amount: number): void {
    console.log(`Оплата ${amount}₽ через кредитную карту`);
  }
}

class PayPalPayment extends PaymentMethod {
  public pay(amount: number): void {
    console.log(`Оплата ${amount}₽ через PayPal`);
  }
}

// Допустим, появляется новый способ оплаты
class CryptoPayment extends PaymentMethod {
  public pay(amount: number): void {
    console.log(`Оплата ${amount}₽ через криптовалюту`);
  }
}

// Иерархия 2: Логгеры для оплаты
abstract class PaymentLogger {
  public abstract logPayment(): void;
}

class CreditCardLogger extends PaymentLogger {
  public logPayment(): void {
    console.log("Логируем транзакцию CreditCardPayment");
  }
}

class PayPalLogger extends PaymentLogger {
  public logPayment(): void {
    console.log("Логируем транзакцию PayPalPayment");
  }
}

// Появился CryptoPayment — придётся создать CryptoLogger!
class CryptoLogger extends PaymentLogger {
  public logPayment(): void {
    console.log("Логируем транзакцию CryptoPayment");
  }
}

// Пример использования
function main(): void {
  const payment1 = new CreditCardPayment();
  payment1.pay(1000);
  const logger1 = new CreditCardLogger();
  logger1.logPayment();

  const payment2 = new CryptoPayment();
  payment2.pay(2000);
  const logger2 = new CryptoLogger();
  logger2.logPayment();
}

main();
```

Здесь видно, что **для каждой** реализации оплаты есть **соответствующая** реализация логгера. Стоит нам добавить новый класс оплаты, **тут же** придётся заводить «зеркальный» класс логгера.

***

#### Как сделать лучше

**1. Не дублировать иерархии, а использовать полиморфизм внутри одного набора классов**

Если логирование привязано к способу оплаты, можно «зашить» логику логирования прямо в класс PaymentMethod (или в отдельный метод/интерфейс), а не делать отдельную параллельную иерархию. Например:

```typescript
abstract class PaymentMethodRefactored {
  public abstract pay(amount: number): void;
  
  // Добавим метод логирования: по умолчанию — базовый, или переопределять в подклассах
  public logPayment(): void {
    console.log(`Лог транзакции (общий), способ: ${this.constructor.name}`);
  }
}

class CreditCardPaymentRefactored extends PaymentMethodRefactored {
  public pay(amount: number): void {
    console.log(`Оплата ${amount}₽ через кредитную карту`);
  }

  public logPayment(): void {
    console.log("Лог CreditCardPayment");
  }
}

class PayPalPaymentRefactored extends PaymentMethodRefactored {
  public pay(amount: number): void {
    console.log(`Оплата ${amount}₽ через PayPal`);
  }
}

// Можно создать CryptoPaymentRefactored, 
// тоже переопределить pay() и при необходимости logPayment()

// Использование
function mainRefactored(): void {
  const payment: PaymentMethodRefactored = new CreditCardPaymentRefactored();
  payment.pay(1000);
  payment.logPayment();
}

mainRefactored();
```

* Теперь, если появляется новый способ оплаты (например, `CryptoPaymentRefactored`), достаточно **одного** класса, а логирование (или иная логика) там уже можно переопределить.
* Таким образом, иерархии не «размножаются параллельно».

**2. Использовать композицию вместо второй иерархии**

Если по каким-то причинам логирование должно быть **внешним** (не хочется «захламлять» классы оплаты логикой логгера), можно применить паттерн **Strategy** или **Decorator**. Например, у нас есть `PaymentLogger`, который «оборачивает» `PaymentMethod`, сохраняя ссылку на базовый объект, и, при вызове `pay()`, выполняет логику логирования. Но при этом нам **не нужно** создавать один класс «логгер» на каждый способ оплаты — мы можем обойтись универсальным «логгером-обёрткой»:

```typescript
class LoggingPayment implements PaymentMethod {
  constructor(private wrappedPayment: PaymentMethod) {}

  public pay(amount: number): void {
    console.log(`Логируем перед оплатой: ${this.wrappedPayment.constructor.name}`);
    this.wrappedPayment.pay(amount);
    console.log(`Логируем после оплаты: ${this.wrappedPayment.constructor.name}`);
  }
}
```

* Теперь, если есть разные реализации `PaymentMethod`, мы можем «оборачивать» их в `LoggingPayment` по мере необходимости, **не** плодя отдельные классы `CreditCardLogger`, `PayPalLogger` и т. д.

**3. Обобщить логику, чтобы не было жёсткой зависимости 1 к 1**

Иногда в «параллельных» иерархиях логика совпадает 1 к 1. Это знак, что, вероятно, нужно использовать **обобщённые** механизмы (интерфейсы, generics, DI) вместо ручного написания классов-близнецов.

***

Таким образом, **Parallel Inheritance Hierarchies** указывают на то, что два (или более) набора классов эволюционируют параллельно и жёстко завязаны друг на друга. Это вызывает избыточность и дублирование. Решения:

1. **Инкапсулировать** дополнительное поведение (например, логирование) в базовом классе или через интерфейс.
2. Применять **композицию** или **декораторы** вместо создания «зеркальных» подклассов.
3. Упрощать иерархии, чтобы новое поведение не требовало множества дублирующих классов.

#### Не стоит трогать, если...

Иногда наличие паралельной иерархии — это необходимое зло, без которого устройство программы было бы еще хуже. Если вы обнаружите, что ваши попытки устранить дублирование приводят к еще большему ухудшению организации кода, то... остановите рефакторинг, откатите все внесенные изменения, выпейте чаю и начните привыкать к этому коду.\


</details>

### Замусориватели

Замусориватели являют собой что-то бесполезное и лишнее, от чего можно было бы избавиться, сделав код чище, эффективней и проще для понимания.

<details>

<summary>Comments (Комментарии)</summary>

В некоторых случаях комментарии действительно необходимы (например, для документирования публичного API, сложных алгоритмов или нестандартных бизнес-требований). Однако, когда мы говорим о «запахе кода» под названием **Comments**, имеется в виду ситуация, когда **избыточные** или **оправдывающие** комментарии используются вместо того, чтобы рефакторить код и сделать его самодокументируемым. То есть комментарии «подменяют» собой несоответствие принципам чистого кода: длинный метод, сложная логика, неочевидные переменные и т. д.

**Суть запаха**:

* Если вам приходится писать много комментариев, чтобы объяснить, _что_ делает код или _почему_ это так сложно, обычно правильнее **упростить** сам код (разделить методы, переименовать переменные, выделить классы), а не оставлять его громоздким и компенсировать пояснениями «на словах».

***

#### Пример «как не надо»

Предположим, что мы имеем метод, который делает сразу несколько задач, и автор пытается компенсировать это подробными комментариями:

```typescript
class OrderService {
  /**
   * Метод processOrder обрабатывает заказ:
   * 1) Валидирует параметры
   * 2) Применяет скидки, если есть
   * 3) Сохраняет заказ в БД
   * 4) Отправляет уведомление пользователю
   * 
   * Очень большой и сложный метод, будь осторожен, меняя что-либо здесь!
   */
  public processOrder(order: any): void {
    // Проверяем, что у заказа есть id
    if (!order.id) {
      // Если нет - выбрасываем ошибку
      throw new Error("Order must have an id");
    }

    // Если есть поле discountCode, то уменьшаем цену
    if (order.discountCode) {
      // Здесь логика расчёта скидки
      order.total = this.calculateDiscount(order.total, order.discountCode);
    }

    // Сохраняем заказ в БД (далее идёт длинный кусок кода)
    //...
    console.log("Сохраняем заказ...");

    // Отправляем уведомление на почту пользователя
    //...
    console.log("Отправляем уведомление...");

    // Обработка завершена
  }

  private calculateDiscount(amount: number, code: string): number {
    // Тут тоже какая-то сложная логика, завязанная на тип кода
    return amount * 0.9; // к примеру, простая 10%-скидка
  }
}
```

* Большое количество комментариев: «что и почему происходит», «какой шаг за чем следует».
* Много логики внутри одного метода, хотя следовало бы разбить: валидация, скидки, сохранение, уведомление.
* Автор явно предупреждает «будь осторожен!», что намекает на сложность и хрупкость.

***

#### Как сделать лучше

**1. Упростить код, сделав его «самодокументируемым»**

Вместо длинных комментариев, разбейте метод на более мелкие и понятные:

```typescript
class OrderServiceRefactored {
  public processOrder(order: any): void {
    this.validate(order);
    this.applyDiscountIfNeeded(order);
    this.saveOrder(order);
    this.notifyUser(order);
  }

  private validate(order: any): void {
    if (!order.id) {
      throw new Error("Order must have an id");
    }
  }

  private applyDiscountIfNeeded(order: any): void {
    if (order.discountCode) {
      order.total = this.calculateDiscount(order.total, order.discountCode);
    }
  }

  private calculateDiscount(amount: number, code: string): number {
    // Простая 10%-скидка для примера
    return amount * 0.9;
  }

  private saveOrder(order: any): void {
    // Сохраняем заказ в БД
    console.log("Сохраняем заказ...", order);
  }

  private notifyUser(order: any): void {
    // Отправляем уведомление пользователю
    console.log("Отправляем уведомление о заказе", order.id);
  }
}
```

* Короткие, говорящие названия методов (`validate`, `applyDiscountIfNeeded`, `saveOrder`, `notifyUser`) сами по себе объясняют, что делает код.
* Теперь нам **не** нужно писать комментарий: «1) Валидируем параметры, 2) Применяем скидки...» — читается непосредственно из последовательности вызовов.

**2. Избегать комментариев «потому что код слишком сложный»**

Если участок кода настолько запутан, что без комментария не обойтись, возможно, надо переписать логику или выделить её в отдельный модуль/класс.\
**Исключение**: алгоритмы, действительно сложные сами по себе (например, специфические формулы или протоколы) — здесь комментарии оправданы, чтобы описать идею алгоритма или источник математической формулы.

**3. Использовать говорящие имена**

* Методы: `calculateDiscountForLoyalCustomer`, `sendEmailNotification`, `handlePaymentFailure`.
* Переменные: `discountedPrice`, `maxDeliveryDays`, `customerEmail` и т. п.\
  Так вы сокращаете необходимость пояснять что-то в комментариях.

**4. Не путать документацию и «комментарии-заплатки»**

Хорошая документация:

* **JSDoc** (или аналог) на публичные методы, описывающая параметры, возвращаемые значения, исключения.
* **Описание архитектуры** или нестандартных решений в Markdown или wiki.

«Плохие» комментарии:

* Поясняют очевидные вещи («`i++` увеличивает i на 1»).
* Предупреждают: «не лезь сюда!» вместо рефакторинга, который бы упростил участок кода.

***

Таким образом, **Comments** как антипаттерн (или «запах кода») не означает, что комментарии — всегда плохо. Проблема возникает, когда комментарии используются, чтобы **маскировать** сложность и плохую структуру кода, тогда как правильным решением было бы **рефакторить** и делать код понятным без обилия вспомогательных надписей.

</details>

<details>

<summary>Duplicate Code (Дублирование кода)</summary>

**Duplicate Code** — это «запах кода», при котором одна и та же логика (или очень похожая) повторяется в нескольких местах программы. Такая ситуация усложняет поддержку: если потребуется изменить логику, придётся искать и править все копии. Пропуск хотя бы одного места приведёт к рассинхронизации поведения, багам и путанице.

***

#### Пример «как не надо»

Предположим, у нас есть код, который работает с заказами и считает итоговую сумму с учётом скидки. Одинаковые вычисления дублируются в двух классах (Cart и Checkout), каждый раз прописываясь заново:

```typescript
class Cart {
  public calculateCartTotal(items: Array<{ price: number }>, discount?: number): number {
    let total = 0;
    for (const item of items) {
      total += item.price;
    }
    
    // Применяем скидку, если она указана
    if (discount && discount > 0) {
      total = total - total * (discount / 100);
    }
    
    return total;
  }
}

class Checkout {
  public finalizePurchase(items: Array<{ price: number }>, discount?: number): number {
    let total = 0;
    for (const item of items) {
      total += item.price;
    }

    // Та же логика расчёта скидки
    if (discount && discount > 0) {
      total = total - total * (discount / 100);
    }

    // ... Дальше идёт логика оформления покупки
    return total;
  }
}

// Использование:
const cart = new Cart();
const cartTotal = cart.calculateCartTotal([{ price: 100 }, { price: 200 }], 10);

const checkout = new Checkout();
const purchaseTotal = checkout.finalizePurchase([{ price: 100 }, { price: 200 }], 10);
```

* В классах **Cart** и **Checkout** видим полное дублирование логики подсчёта суммы и скидки.
* Если поменяется формула скидки (например, нужно минимум 2 товара или исключения для некоторых категорий), придётся менять и в Cart, и в Checkout.

***

#### Как сделать лучше

**1. Выделить общую логику в единый метод (DRY)**

Создадим «утилитный» метод или сервис для расчёта итоговой суммы. Тогда классы Cart и Checkout будут просто вызывать этот метод:

```typescript
class PriceCalculator {
  public static calculateTotal(items: Array<{ price: number }>, discount?: number): number {
    let total = items.reduce((acc, item) => acc + item.price, 0);

    if (discount && discount > 0) {
      total = total - total * (discount / 100);
    }
    return total;
  }
}

class CartRefactored {
  public calculateCartTotal(items: Array<{ price: number }>, discount?: number): number {
    // Теперь дублированного кода нет, вся логика в PriceCalculator
    return PriceCalculator.calculateTotal(items, discount);
  }
}

class CheckoutRefactored {
  public finalizePurchase(items: Array<{ price: number }>, discount?: number): number {
    // Тоже используем общий метод
    const total = PriceCalculator.calculateTotal(items, discount);
    // ... остальная логика оформления
    return total;
  }
}

// Использование:
const cart2 = new CartRefactored();
const cartTotal2 = cart2.calculateCartTotal([{ price: 100 }, { price: 200 }], 10);

const checkout2 = new CheckoutRefactored();
const purchaseTotal2 = checkout2.finalizePurchase([{ price: 100 }, { price: 200 }], 10);
```

* Теперь при изменении правил скидки или структуры товаров нужно править **только** `PriceCalculator`.
* Никакой опасности пропустить одно из мест, где логика дублировалась.

**2. Проверить, нужна ли нам вообще пара классов**

Иногда дублирование возникает, потому что мы раздробили функциональность на два почти одинаковых класса. Возможно, правильнее объединить их или определить точные различия (например, `Cart` отвечает за текущее содержимое корзины, `Checkout` — за финализацию заказа). Но сам механизм вычисления цены/скидки можно вынести отдельно.

**3. Не копировать код между проектами/файлами**

Если вы заметили, что, например, куски валидации или формулы «гуляют» из одного места в другое, используйте методы/модули, которые можно реэкспортировать. Либо оформляйте в библиотеки (npm-пакеты, shared-модули), если это общий код для нескольких проектов.

**4. Принцип DRY (Don’t Repeat Yourself)**

* Как только видите копирование строк кода больше, чем один раз, подумайте о вынесении в отдельную функцию, метод или класс.
* Это не означает слепое удаление любых «повторений» — иногда короткий фрагмент бывает быстрее понять «на месте». Но если логика «повторяется» и может меняться, лучше выделять её в одно место.

***

Таким образом, **Duplicate Code** — сигнал, что код «размножился» по нескольким точкам. Следует выделять общий функционал в единое место (метод, класс, модуль), избегая ситуации, когда одно изменение нужно «ручками» вносить везде, где логика повторена.

</details>

<details>

<summary>Lazy Class (Ленивый класс)</summary>

**Lazy Class** (ленивый класс) — это ситуация, когда класс выполняет настолько мало работы (или содержит настолько мало логики), что не оправдывает своё существование. Часто это признак того, что класс был создан «на будущее» или в рамках первоначального разделения кода, но в итоге так и не получил достаточно функциональности.

В результате мы имеем «пустышку», которая лишь усложняет архитектуру, добавляя дополнительный уровень абстракции без реальной пользы.

***

#### Пример «как не надо»

Допустим, мы решили вынести в отдельный класс «проверку статуса заказа». Однако в итоге такой класс содержит лишь один метод и практически ничего не делает — логику пришлось оставить в другом месте, или её вообще нет.

```typescript
class OrderStatusChecker {
  // Пустой или почти пустой класс
  public isOrderCompleted(order: any): boolean {
    // Проверяем флажок, ничего более
    return order.isCompleted;
  }
}

// Где-то в OrderService или другом месте:
class OrderService {
  private orderStatusChecker: OrderStatusChecker;

  constructor() {
    this.orderStatusChecker = new OrderStatusChecker();
  }

  public processOrder(order: any): void {
    if (this.orderStatusChecker.isOrderCompleted(order)) {
      console.log("Заказ уже завершён, ничего не делаем");
    } else {
      // Логика обработки заказа
      console.log("Обрабатываем заказ...");
    }
  }
}
```

* `OrderStatusChecker` не содержит дополнительных операций, не инкапсулирует никакой сложной логики — фактически это просто «один метод с одним if».
* Класс мог иметь смысл, если бы там была сложная проверка статуса (несколько полей, разные этапы и т. д.), но если этого не произошло и в планах не предвидится, лучше от него избавиться.

***

#### Как сделать лучше

**1. Удалить класс и перенести логику в другой существующий класс/метод**

Если класс делает очень мало, то обычно выгоднее **убрать** его, переместив функциональность в более подходящее место. В примере выше можно прямо в `OrderService` спросить:

```typescript
class OrderServiceRefactored {
  public processOrder(order: any): void {
    if (order.isCompleted) {
      console.log("Заказ уже завершён, ничего не делаем");
    } else {
      console.log("Обрабатываем заказ...");
    }
  }
}
```

* Нет нужды создавать `OrderStatusChecker` — мы сократили абстракцию и избавились от лишнего объекта.

**2. Расширить класс, если он «всё-таки» нужен**

Если предполагается, что класс со временем будет содержать больше логики (например, разные состояния заказа, дополнительные проверки), можно **раскрыть** его функциональность уже сейчас:

```typescript
class OrderStatusCheckerEnhanced {
  public isOrderCompleted(order: any): boolean {
    // Допустим, логика стала более сложной:
    if (!order.startDate) return false;
    if (order.cancelled) return false;
    if (order.isCompleted && order.finishedAt) return true;
    return false;
  }

  public isOrderDelayed(order: any): boolean {
    // Допустим, отдельная проверка «заказ просрочен»
    const dueDate = order.dueDate;
    if (!dueDate) return false;
    return (Date.now() > dueDate.getTime() && !order.isCompleted);
  }
}
```

* Теперь класс действительно **инкапсулирует** различные проверки статуса заказа, а не ограничивается тривиальным `return order.isCompleted`.

**3. Объединять классы, избегающие «ленивости»**

Если у вас много таких «микро-классов», с по паре строчек кода, разумно подумать, не лучше ли объединить их в более крупные блоки (модули, сервисы), чтобы код оставался структурированным, но без излишней фрагментации.

***

Таким образом, **Lazy Class** говорит о том, что класс выполняет ничтожно мало действий и скорее мешает, чем помогает. Если класс создавался «на будущее», но не обрёл нужную функциональность — лучше либо удалить его (переместив логику в более уместное место), либо развить, сделав действительно полноценным участником архитектуры.

</details>

<details>

<summary>Data Class (Класс данных)</summary>

**Data Class** — это класс, который главным образом содержит только поля (данные) и не имеет значимых методов поведения, либо его методы сводятся к геттерам/сеттерам, `toString()`, `equals()` и т. п. Такой класс не инкапсулирует логику и зачастую выступает просто «структурой» для переноса данных из одного места в другое.

Хотя наличие простых DTO (Data Transfer Object) иногда оправдано (например, при передаче данных между слоями или через сеть), проблема в том, что **Data Class** часто становится «центром вымывания логики» — связанный с ним функционал оказывается размазанным по другим местам, а сам класс почти не участвует в логике, кроме хранения данных.

***

#### Пример «как не надо»

Предположим, мы имеем класс `User`, который содержит лишь поля и геттеры/сеттеры, без какой-либо бизнес-логики:

```typescript
class User {
  private _firstName: string;
  private _lastName: string;
  private _email: string;

  constructor(firstName: string, lastName: string, email: string) {
    this._firstName = firstName;
    this._lastName = lastName;
    this._email = email;
  }

  public get firstName(): string {
    return this._firstName;
  }
  public set firstName(value: string) {
    this._firstName = value;
  }

  public get lastName(): string {
    return this._lastName;
  }
  public set lastName(value: string) {
    this._lastName = value;
  }

  public get email(): string {
    return this._email;
  }
  public set email(value: string) {
    this._email = value;
  }

  // Здесь нет никакой логики, класс просто хранит данные
}
```

* Вся логика (например, проверка формата email, генерация полного имени, и т. д.) может оказаться где-то снаружи.
* Если у нас много таких «голых» классов данных, логика валидации/обработки размазана по сервисам, контроллерам и т. д.

***

#### Как сделать лучше

**1. Поместить поведение туда, где хранятся данные**

Если класс представляет реальную бизнес-сущность (например, `User`), можно **инкапсулировать** часть логики, непосредственно связанную с этой сущностью, внутрь самого класса:

```typescript
class UserRefactored {
  private firstName: string;
  private lastName: string;
  private email: string;

  constructor(firstName: string, lastName: string, email: string) {
    // Сразу проводим валидацию email (логика внутри класса)
    if (!this.isValidEmail(email)) {
      throw new Error("Некорректный email: " + email);
    }
    this.firstName = firstName;
    this.lastName = lastName;
    this.email = email;
  }

  // Пример метода, генерирующего полное имя
  public getFullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }

  // Пример метода изменения email с проверкой
  public updateEmail(newEmail: string): void {
    if (!this.isValidEmail(newEmail)) {
      throw new Error("Некорректный новый email: " + newEmail);
    }
    this.email = newEmail;
  }

  private isValidEmail(value: string): boolean {
    // Упрощённая проверка
    return value.includes("@");
  }
}
```

* Теперь класс не просто «мешок» с полями, а действительно **инкапсулирует** логику, связанную с пользователем (например, валидацию).
* Если в будущем появятся требования (форматирование имени, флаги активности, методы смены пароля и т. д.), класс может содержать их, а не «выносить» в разные сервисы.

**2. Вынести лишь необходимые DTO при передаче данных**

Если нам всё же нужны объекты-транспортировщики (DTO) для передачи данных (например, при сериализации JSON), это нормально, но они обычно располагаются **рядом** или внутри слоя передачи данных и не выдаются за полноценные бизнес-сущности.

* В таких случаях мы **осознанно** имеем классы без поведения, но они чётко маркируются как DTO/TransferObject.
* Бизнес-логику держим внутри основной модели (например, `UserEntity` или `UserDomain`).

**3. Следить за распределением логики**

Если данные класса используются повсюду и многие операции над ними «пристраиваются» где-то «снаружи», это может указывать на то, что **Data Class** должен стать полноценным классом с методами. Либо же эту сущность стоит «заинкапсулировать» в сервис или Value Object, который будет напрямую управлять данными.

***

Таким образом, **Data Class** нежелательна, когда мы говорим о полноценной бизнес-сущности, ведь она превращается в «пустой мешок данных», а логику приходится размазывать по другим местам. Решение — **внедрять** в класс поведение и правила, относящиеся к данным (инкапсуляция), либо, если класс действительно нужен лишь для транспортировки, чётко ограничивать его как DTO и не смешивать с доменной логикой.

</details>

<details>

<summary>Dead Code (Мертвый код)</summary>

**Dead Code** — это любой код (классы, функции, переменные), который больше не используется в проекте и не влияет на его работу. Он может оставаться после рефакторинга, изменений требований или переездов на новые архитектуры. Такой код засоряет проект, усложняет поиск актуальной логики и может ввести разработчиков в заблуждение.

***

#### Пример «как не надо»

Ниже простой пример, когда мы оставили старую функцию, которая больше никогда не вызывается:

```typescript
class UserUtils {
  // Использовалась когда-то для синхронизации пользователей, 
  // но теперь это происходит в другом сервисе.
  // Однако функция осталась "на всякий случай".
  public static syncUsersOldWay(): void {
    console.log("Синхронизируем пользователей старым способом...");
    // ... Код, который давно не вызывается
  }

  public static getUserFullName(user: { firstName: string; lastName: string }): string {
    return `${user.firstName} ${user.lastName}`;
  }
}

// Где-то в коде:
function main() {
  const user = { firstName: "Иван", lastName: "Иванов" };
  console.log("Полное имя:", UserUtils.getUserFullName(user));

  // syncUsersOldWay() никогда не вызывается
}
main();
```

* Метод `syncUsersOldWay()` больше не используется, но остаётся в коде и сбивает с толку.
* Разработчикам непонятно, стоит ли трогать этот код, или он «всё-таки нужен».

***

#### Как сделать лучше

**1. Удалять неиспользуемые методы, классы и т. д.**

Самое простое и правильное решение: **выпилить** весь код, который не используется и не планируется к повторному применению.

* Удаляем метод `syncUsersOldWay()`.
* Если вдруг он понадобится в будущем, есть система контроля версий (Git) — всегда можно вернуть его из истории.

```typescript
class UserUtilsRefactored {
  public static getUserFullName(user: { firstName: string; lastName: string }): string {
    return `${user.firstName} ${user.lastName}`;
  }
}
```

**2. Если код теоретически может пригодиться, вынести его в ветку/архив**

В некоторых случаях, когда логика может вернуться или ценна как пример, можно:

* Перенести в отдельную «ветку» (branch) в Git, задокументировав причину.
* Или оформить как архивный модуль/класс (с пометкой `deprecated`), если нужно временно сохранить совместимость.

Но это скорее исключение, чем правило. Оставлять мёртвый код в основном репозитории, надеясь, что «когда-нибудь пригодится», не рекомендуется.

**3. Настраивать инструменты статического анализа**

Многие инструменты (TypeScript compiler с флагом `--noUnusedLocals`, линтеры) могут подсказать, что есть неиспользуемый код:

* Неиспользуемые переменные и импорт.
* Не вызываемые методы.
* Мёртвые ветви в `if-else`.

Регулярный анализ и автоматические проверки при сборке позволят быстро выявлять и устранять «Dead Code».

***

Таким образом, **Dead Code** — лишний «мусор» в проекте, который не несёт пользы и мешает понимать, что действительно актуально. Лучше всего **удалять** мёртвый код, чтобы проект оставался чистым и более поддерживаемым. Если понадобятся старые версии — всегда можно вернуть их из системы контроля версий.

</details>

<details>

<summary>Speculative Generality (Теоретическая общность)</summary>

**Speculative Generality** (иногда называют «заранее подготовленная универсальность») — это ситуация, когда код пытаются сделать более абстрактным и универсальным «на всякий случай», в расчёте на возможные будущие требования, которые на самом деле могут и не появиться. В результате появляются лишние классы, интерфейсы, параметры, настройки, усложняющие проект, но не приносящие реальной пользы.

**Ключевая идея**: «**YAGNI** (You Ain’t Gonna Need It)» — не реализовывай то, что не требуется сейчас.

***

#### Пример «как не надо»

Допустим, мы пишем сервис для загрузки файлов. Нам известно, что пока нужно только скачивать по HTTP. Но разработчик «на перспективу» предполагает, что однажды могут попросить FTP, SFTP и ещё что-то. В результате он делает сложную систему с интерфейсами, дополнительными классами и конфигами, хотя реально используется лишь один вариант.

```typescript
// Интерфейс на будущее (когда-то вдруг понадобятся FTP/SFTP)
interface FileDownloader {
  download(url: string, protocolConfig?: any): void;
}

// Реализация для HTTP
class HttpFileDownloader implements FileDownloader {
  public download(url: string, protocolConfig?: any): void {
    // Псевдокод реальной HTTP-загрузки
    console.log(`HTTP download: ${url}, config: ${JSON.stringify(protocolConfig)}`);
  }
}

// Абстракции «на будущее» для других протоколов
class FtpFileDownloader implements FileDownloader {
  public download(url: string, protocolConfig?: any): void {
    console.log(`FTP download: ${url}, config: ${JSON.stringify(protocolConfig)}`);
  }
}

class SftpFileDownloader implements FileDownloader {
  public download(url: string, protocolConfig?: any): void {
    console.log(`SFTP download: ${url}, config: ${JSON.stringify(protocolConfig)}`);
  }
}

// Много лишних настроек, хотя реальный код пока использует только HTTP
class FileDownloadService {
  private downloader: FileDownloader;

  constructor(protocolType: string) {
    switch (protocolType) {
      case "HTTP":
        this.downloader = new HttpFileDownloader();
        break;
      case "FTP":
        this.downloader = new FtpFileDownloader();
        break;
      case "SFTP":
        this.downloader = new SftpFileDownloader();
        break;
      default:
        this.downloader = new HttpFileDownloader(); // По умолчанию HTTP
    }
  }

  public downloadFile(url: string): void {
    // Допустим, какой-то «protocolConfig», который сейчас не нужен
    const protocolConfig = { retries: 3, secureMode: false };
    this.downloader.download(url, protocolConfig);
  }
}

// Использование
const service = new FileDownloadService("HTTP");
service.downloadFile("http://example.com/file.zip");
```

* Хотя требуется только HTTP, у нас уже есть классы для FTP и SFTP, а также логика выборки «protocolType», свитч, неиспользуемый `protocolConfig`.
* Всё это усложняет код и вводит лишние концепции, которые сейчас вообще не нужны.

***

#### Как сделать лучше

**1. Реализовать только то, что нужно сейчас**

Если по требованиям есть только HTTP-загрузка, не делайте целую «систему плагинов». Ограничьтесь простым классом, отвечающим за скачивание по HTTP:

```typescript
class SimpleFileDownloader {
  public downloadFile(url: string): void {
    console.log(`HTTP download: ${url}`);
    // ... Реальная логика HTTP-загрузки
  }
}

// Использование:
const downloader = new SimpleFileDownloader();
downloader.downloadFile("http://example.com/file.zip");
```

* Если в будущем **действительно** понадобится FTP — добавим новый класс или модифицируем логику, но пока не усложняем.

**2. Использовать паттерны, когда появляется реальная потребность**

Возможно, со временем и вправду потребуется поддержка нескольких протоколов. Тогда можно внедрить паттерн **Strategy** (или **Factory**) **после** появления задачи, а не заблаговременно.

* Принцип: «Не пытайся предугадывать все будущие сценарии, основывайся на реальных требованиях.»

**3. «YAGNI» и «KISS»**

* **YAGNI (You Ain’t Gonna Need It)**: не добавляй код, который может понадобиться «когда-нибудь», если нет чётких требований.
* **KISS (Keep It Simple, Stupid)**: стремись к простым решениям, пока нет необходимости в усложнении.

**4. При появлении новых требований — рефакторить**

Когда в проекте действительно возникает необходимость (например, появляются чёткие требования на FTP-загрузку), тогда:

1. Рефакторим существующий код под нужную архитектуру (Strategy, Factory и т. п.).
2. Добавляем класс `FtpFileDownloader`.
3. Оптимизируем логику выбора протокола.\
   Это будет **обоснованным** усложнением, а не спекулятивным.

***

Таким образом, **Speculative Generality** — это «запах кода», когда в систему добавляются абстракции и структуры «на всякий случай». Правильный подход: **придерживаться реальных бизнес-требований**, не создавать избыточных интерфейсов и классов без практической необходимости. Лучше добавлять новые уровни абстракции по мере необходимости, чем загромождать код «спекулятивными» заделами.

</details>

### Опутыватели связями

Все запахи из этой группы приводят к избыточной связанности между классами, либо показывают, что бывает, если тесная связанность заменяется постоянным делегированием.

<details>

<summary>Feature Envy (Завистливые функции)</summary>

**Feature Envy** — это ситуация, когда метод в одном классе слишком много обращается к данным (полям и методам) другого класса, вместо того чтобы «заботиться» о собственных данных. Другими словами, метод завидует (envy) «фичам» другого объекта, часто вызывая у него геттеры/сеттеры или методы, вместо того чтобы инкапсулировать часть логики в том же месте, где расположены данные.

В результате, большая часть кода метода посвящена работе с чужими полями, что нарушает **принцип инкапсуляции** и может затруднять сопровождение.

***

#### Пример «как не надо»

Предположим, у нас есть класс `Customer`, который содержит информацию о клиенте (имя, адрес, статус). Другой класс — `BillingService`, где метод `calculateDiscount` постоянно лезет в поля `Customer` и выполняет логику, которая «по смыслу» могла бы находиться внутри самого `Customer`.

```typescript
class Customer {
  public name: string;
  public address: string;
  public isLoyal: boolean; // Например, лояльный клиент или нет

  constructor(name: string, address: string, isLoyal: boolean) {
    this.name = name;
    this.address = address;
    this.isLoyal = isLoyal;
  }
}

class BillingService {
  /**
   * Метод calculateDiscount "завидует" данным Customer,
   * т.к. постоянно смотрит на поля isLoyal, address и пр.
   */
  public calculateDiscount(customer: Customer, orderAmount: number): number {
    let discount = 0;

    // Если клиент лояльный, даём 10% скидку
    if (customer.isLoyal) {
      discount = orderAmount * 0.1;
    }

    // Если клиент из города "Москва", допустим, даём ещё 5%
    if (customer.address.includes("Москва")) {
      discount += orderAmount * 0.05;
    }

    // Ещё какая-то логика, завязанная на customer...
    console.log(`Клиент: ${customer.name}, Скидка: ${discount}`);

    return discount;
  }
}

// Пример использования
function main() {
  const customer = new Customer("Иван", "Москва, Тверская 1", true);
  const billing = new BillingService();

  const discount = billing.calculateDiscount(customer, 2000);
  console.log("Итоговая скидка:", discount);
}

main();
```

* Метод `calculateDiscount` «знает» о деталях: `isLoyal`, `address`, ещё и `name`.
* Каждое изменение (добавить проверку на поле, изменить логику лояльности) придётся вносить в `BillingService`, а ведь это по смыслу относится к бизнес-логике самого клиента (или связанной с его параметрами).

***

#### Как сделать лучше

**1. Переместить (или «перетянуть») логику поближе к данным**

Вместо того чтобы класс `BillingService` лазил по полям `Customer`, можно дать `Customer` методы, которые позволяют узнать размер скидки на основе его собственных характеристик. Тогда «зависть» (envy) исчезнет — мы будем спрашивать «Какая твоя скидка?» у самого клиента, не трогая его внутренние поля напрямую.

```typescript
class CustomerRefactored {
  public name: string;
  public address: string;
  public isLoyal: boolean;

  constructor(name: string, address: string, isLoyal: boolean) {
    this.name = name;
    this.address = address;
    this.isLoyal = isLoyal;
  }

  /**
   * Теперь логика определения скидки — внутри Customer.
   * Он сам решает, сколько процентов даёт статус лояльности, местоположение и т.д.
   */
  public calculatePersonalDiscount(orderAmount: number): number {
    let discount = 0;

    if (this.isLoyal) {
      discount += orderAmount * 0.1;
    }

    if (this.address.includes("Москва")) {
      discount += orderAmount * 0.05;
    }

    return discount;
  }
}

class BillingServiceRefactored {
  /**
   * Метод "не завидует" Customer, а просто запрашивает у него скидку.
   */
  public calculateDiscount(customer: CustomerRefactored, orderAmount: number): number {
    const discount = customer.calculatePersonalDiscount(orderAmount);
    console.log(`Клиент: ${customer.name}, Скидка: ${discount}`);
    return discount;
  }
}

// Пример использования
function mainRefactored() {
  const customer = new CustomerRefactored("Иван", "Москва, Тверская 1", true);
  const billing = new BillingServiceRefactored();

  const discount = billing.calculateDiscount(customer, 2000);
  console.log("Итоговая скидка (рефакторинг):", discount);
}

mainRefactored();
```

* Теперь, если хотим расширить логику скидок (учитывать дату регистрации, категорию клиента и т. д.), мы меняем `calculatePersonalDiscount` в `CustomerRefactored`.
* `BillingServiceRefactored` не тревожит поля клиента, не «завидует» им.

**2. Использовать паттерны при необходимости**

Иногда Feature Envy возникает, когда логика на самом деле должна находиться в отдельном сервисе или паттерне **Strategy**. Главное, чтобы этот сервис/стратегия не лазил непосредственно в поля объекта, а работал через осмысленные методы.

* Например, `customer.isLoyalCustomer()` или `customer.getLocation()`, а не `if (customer.isLoyal === true && customer.address === ...)`.

**3. Соблюдать принцип инкапсуляции**

Когда класс A слишком много знает о данных класса B, это намёк на то, что часть данных/логики должна переместиться в класс B или что нужно создать новый объект, объединяющий эти данные с логикой, чтобы не было бесконечных «гостей» с кучей геттеров.

***

Таким образом, **Feature Envy** показывает, что метод не там, где ему следует быть, поскольку он «влюблён» (envy) в данные другого класса. Решение — **переместить** поведение в класс, которому принадлежат эти данные, либо создать дополнительный объект, более логично совмещающий данные и методы.

</details>

<details>

<summary>Inappropriate Intimacy (Неуместная близость)</summary>

**Inappropriate Intimacy** — это ситуация, когда два класса (или модуля) слишком сильно зависят друг от друга и обращаются к приватным или внутренним деталям друг друга, вместо того чтобы взаимодействовать через чёткие, публичные интерфейсы. Они буквально «лезут в душу» друг к другу, нарушая принципы инкапсуляции и повышая связность кода. Подобная «интимная» близость усложняет сопровождение, ведь изменение в одном классе может привести к поломкам в другом.

***

#### Пример «как не надо»

Допустим, у нас есть два класса: `User` и `UserProfileService`. Первый хранит информацию о пользователе, второй должен заниматься логикой профиля (запросы и изменение данных). Вместо того чтобы взаимодействовать через методы `User`, сервис напрямую лазит в поля `User` (даже если они обозначены как приватные или предполагаются к инкапсуляции), используя, например, геттеры/сеттеры пакетной видимости, или (в случае языков, позволяющих это) обходные манёвры.

```typescript
class User {
  // Допустим, по ошибке (или нарочно) поля публичные,
  // либо есть "пакетная" видимость и сервис их напрямую изменяет.
  public firstName: string;
  public lastName: string;
  public isActive: boolean;

  constructor(firstName: string, lastName: string, isActive: boolean) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.isActive = isActive;
  }
}

class UserProfileService {
  /**
   * "Неподобающая близость" — сервис напрямую шарится по внутренним полям User.
   * Он не вызывает методы, а просто меняет поля, как сочтёт нужным.
   */
  public deactivateUser(user: User): void {
    // Входит в поля и меняет их, возможно, пропуская какую-то важную логику
    user.isActive = false;
    console.log(`Пользователь ${user.firstName} ${user.lastName} деактивирован`);
  }

  public updateName(user: User, newFirstName: string, newLastName: string): void {
    // Опять лезем к полям напрямую
    user.firstName = newFirstName;
    user.lastName = newLastName;
    console.log(`Имя пользователя изменено на: ${user.firstName} ${user.lastName}`);
  }
}

// Использование
function main(): void {
  const user = new User("Иван", "Иванов", true);
  const service = new UserProfileService();

  service.deactivateUser(user);
  service.updateName(user, "Пётр", "Петров");
}

main();
```

* `UserProfileService` напрямую меняет внутреннее состояние `User`, не вызывая никаких «правильных» методов, не проверяя логику, которая могла бы быть необходима при изменении имени или статуса.
* Если в будущем в `User` появятся дополнительные проверки (например, «Нельзя выключать пользователя без уведомления»), мы всё равно сможем обойти их, напрямую присвоив `isActive = false`.

***

#### Как сделать лучше

**1. Инкапсулировать внутренние данные и давать методы**

Сделаем поля `User` приватными (или хотя бы защищёнными), а для изменения — публичные методы. Тогда внешний код не сможет просто так влезть и поменять поля, а должен вызывать методы, соблюдающие нужные правила:

```typescript
class UserRefactored {
  private firstName: string;
  private lastName: string;
  private active: boolean;

  constructor(firstName: string, lastName: string, isActive: boolean) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.active = isActive;
  }

  public deactivate(): void {
    // Логика деактивации: например, отмечаем время, шлём уведомление
    this.active = false;
    console.log(`Пользователь деактивирован: ${this.firstName} ${this.lastName}`);
  }

  public updateName(newFirstName: string, newLastName: string): void {
    // При переименовании, возможно, сохраняем историю изменений или валидируем
    this.firstName = newFirstName;
    this.lastName = newLastName;
    console.log(`Имя пользователя обновлено до: ${this.firstName} ${this.lastName}`);
  }

  public isActive(): boolean {
    return this.active;
  }

  public getFullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }
}

class UserProfileServiceRefactored {
  /**
   * Теперь сервис вызывает публичные методы UserRefactored, 
   * не влезая напрямую в его поля.
   */
  public deactivateUser(user: UserRefactored): void {
    user.deactivate();
  }

  public changeUserName(user: UserRefactored, firstName: string, lastName: string): void {
    user.updateName(firstName, lastName);
  }
}

// Использование
function mainRefactored(): void {
  const user = new UserRefactored("Иван", "Иванов", true);
  const service = new UserProfileServiceRefactored();

  service.deactivateUser(user);
  service.changeUserName(user, "Пётр", "Петров");
}

mainRefactored();
```

</details>

<details>

<summary>Message Chains (Цепочка вызовов)</summary>

**Message Chains** — это ситуация, когда в коде встречаются длинные цепочки вызовов методов или свойств: `obj.getA().getB().getC().doSomething()`. При этом каждый «звено» в цепочке возвращает объект, на котором снова вызывается метод, и так далее. Такой код нарушает **Принцип наименьшего знания** (Law of Demeter), потому что объект «лезет» глубоко в структуру других объектов, напрямую работая с их внутренностями или делегатами.

**Что плохо в длинных цепочках сообщений?**

* Они делают код более хрупким: если структура одного из звеньев поменяется (перестанет возвращать объект нужного типа, переедет в другой класс и т.д.), придётся менять код везде, где используется эта цепочка.
* Они усложняют чтение и понимание, особенно когда цепочка проходит через несколько «переходных» объектов (нужно держать в уме всю иерархию).

***

#### Пример «как не надо»

Допустим, у нас есть объект `Order`, который содержит ссылку на `Customer`, а `Customer`, в свою очередь, хранит объект `Address`. Если в коде в нескольких местах приходится делать:

```typescript
class Address {
  public city: string;
  public street: string;
  public house: string;

  constructor(city: string, street: string, house: string) {
    this.city = city;
    this.street = street;
    this.house = house;
  }
}

class Customer {
  public name: string;
  public address: Address;

  constructor(name: string, address: Address) {
    this.name = name;
    this.address = address;
  }
}

class Order {
  public customer: Customer;
  public total: number;

  constructor(customer: Customer, total: number) {
    this.customer = customer;
    this.total = total;
  }
}

function main() {
  const address = new Address("Москва", "Тверская", "10");
  const customer = new Customer("Иван", address);
  const order = new Order(customer, 3000);

  // Длинная цепочка: order -> customer -> address -> city
  console.log("Город:", order.customer.address.city);

  // Если понадобится улица, снова:
  console.log("Улица:", order.customer.address.street);

  // Если нам надо сравнить что-то с городом, получим ещё:
  if (order.customer.address.city === "Москва") {
    console.log("Доставка в пределах столицы!");
  }
}

main();
```

* Мы видим «цепочку» `order.customer.address.city`, причем во многих местах.
* Если вдруг изменится структура (скажем, `Customer` больше не хранит `Address`, а использует другую систему), придётся везде искать эти «цепочки» и рефакторить.

***

#### Как сделать лучше

**1. Сократить цепочки с помощью «Методов-делегатов» (Hide Delegate)**

Пусть `Order` сам предоставляет метод `getCity()`, инкапсулируя доступ к клиенту и его адресу:

```typescript
class AddressRefactored {
  public city: string;
  public street: string;
  public house: string;
  
  constructor(city: string, street: string, house: string) {
    this.city = city;
    this.street = street;
    this.house = house;
  }
}

class CustomerRefactored {
  public name: string;
  private address: AddressRefactored;

  constructor(name: string, address: AddressRefactored) {
    this.name = name;
    this.address = address;
  }

  // Инкапсулируем адрес, предоставляя метод, если нужно
  public getAddress(): AddressRefactored {
    return this.address;
  }
}

class OrderRefactored {
  private customer: CustomerRefactored;
  private total: number;

  constructor(customer: CustomerRefactored, total: number) {
    this.customer = customer;
    this.total = total;
  }

  // Вместо "order.customer.address.city" даём метод:
  public getCity(): string {
    return this.customer.getAddress().city;
  }

  public getTotal(): number {
    return this.total;
  }
  
  // Если часто нужен не только город, но и другие части адреса,
  // можно сделать методы getStreet(), getFullAddress() и т.п.
}

function mainRefactored() {
  const address = new AddressRefactored("Москва", "Тверская", "10");
  const customer = new CustomerRefactored("Иван", address);
  const order = new OrderRefactored(customer, 3000);

  // Теперь обращаемся к городу непосредственно через Order,
  // не зная при этом, что у него внутри есть Customer -> Address.
  console.log("Город:", order.getCity());

  if (order.getCity() === "Москва") {
    console.log("Доставка в пределах столицы!");
  }
}

mainRefactored();
```

* Теперь в клиентском коде не видно «цепочки». Если структура поменяется (допустим, `Customer` заменит `Address` на другое хранилище), меняем только класс `OrderRefactored` (или `CustomerRefactored`), а не все места, где раньше была «цепочка».

**2. Локализовать поведение там, где оно логически должно быть**

Если мы часто делаем проверки вроде `if (order.customer.address.city === "Москва")`, значит, возможно, стоит в `Order` или в `Customer` сделать метод `isCity(cityName: string): boolean`, который сам знает, как проверить. Тогда клиентский код вызывает `order.isCity("Москва")` или `customer.livesIn("Москва")`.

**3. Следовать принципу «Law of Demeter»**

* Объект должен обращаться **только** к методам:
  1. самого себя,
  2. своих полей,
  3. объектов, которые он создаёт,
  4. объектов, переданных ему в метод.
* Если ваш код пишет что-то вроде `obj.getX().getY().getZ()...`, это признак, что вы заходите слишком далеко по цепочке объектов.

***

Таким образом, **Message Chains** приводит к жёсткой связанности и повышенной хрупкости кода. Рекомендуется **сокращать** цепочки за счёт методов-«посредников» (Hide Delegate), а также корректно распределять ответственность (пусть объекты сами предоставляют более удобные, высокоуровневые методы вместо того, чтобы раскрывать всю свою внутреннюю структуру).

</details>

<details>

<summary>Middle Man (Посредник)</summary>

**Middle Man** — это ситуация, когда класс почти полностью делегирует свою работу другим объектам, практически не добавляя никакого собственного поведения. То есть он становится «прослойкой», не содержащей собственной логики, а лишь передаёт вызовы дальше. В таком случае класс (или метод) не приносит особой пользы, а лишь усложняет цепочку вызовов.

***

#### Пример «как не надо»

Допустим, у нас есть класс `UserService`, который, по идее, должен заниматься логикой пользователя. Но в реальности он просто хранит ссылку на `UserRepository` и при любом вызове сам не делает ничего, кроме как делегирует запросы репозиторию.

```typescript
class UserRepository {
  public findUserById(id: number): string {
    // Псевдореализация запроса в БД
    console.log(`Поиск пользователя в БД по id=${id}`);
    return `User${id}`;
  }

  public createUser(name: string): void {
    // Сохранение в БД
    console.log(`Создание пользователя в БД: ${name}`);
  }
}

class UserService {
  private userRepository: UserRepository;

  constructor(userRepository: UserRepository) {
    this.userRepository = userRepository;
  }

  // Все методы просто "прокидывают" вызовы к UserRepository

  public getUser(id: number): string {
    return this.userRepository.findUserById(id);
  }

  public addUser(name: string): void {
    this.userRepository.createUser(name);
  }
}

// Использование
function main() {
  const repo = new UserRepository();
  const service = new UserService(repo);

  const user = service.getUser(100);
  console.log("Найден пользователь:", user);

  service.addUser("Новый пользователь");
}

main();
```

* [ ] `UserService` не добавляет никакой бизнес-логики. Все его методы — это тривиальные делегирования к `UserRepository`.
* [ ] Такой класс — «пустышка» (Middle Man). Если в будущем бизнес-логика так и не появится, он лишь затрудняет понимание архитектуры.

***

#### Как сделать лучше

1. **Убрать «промежуточный» класс**, если логика не планируется
   * Если ваша архитектура требует уровня «сервисов» для бизнес-логики, но в данном случае он не нужен, можно вызвать `UserRepository` напрямую.
   * Лишняя прослойка только добавляет дополнительные точки входа и вызовов, не давая преимуществ.
2. **Добавить (или перенести) реальную логику в «средний» класс**, если она нужна
   * Если у вас есть планы или реальная необходимость хранить правила, валидацию, транзакции и т. п. на уровне сервисов, то пусть `UserService` действительно **что-то делает**, а не просто «прокидывает» вызовы.
   * Например, проверяет права доступа, выполняет валидацию данных, оборачивает операции в транзакции, обрабатывает ошибки и т. д.
3. **Избегать избыточных делегатов**
   * Если какой-то метод на 99% повторяет метод другого объекта, стоит спросить: «Зачем мы это делаем?»
   * Может быть, метод-делегат действительно удобен (скрывает детали реализации), но тогда он должен быть «осмысленным», а не просто копировать сигнатуру.
4. **Следовать принципу «минимальной необходимой абстракции»**
   * Архитектура (например, в стиле «Controller → Service → Repository») полезна, когда каждый слой отвечает за своё дело.
   * Но если слой пуст, его стоит либо заполнить содержательным кодом, либо убрать.

***

Таким образом, **Middle Man** — «запах кода», когда класс не вносит никакой собственной функциональности, а является простой прослойкой между вызовами. Решение — либо **устранить** этот класс, сделав вызовы непосредственно к реальному исполнителю, либо **обогатить** класс бизнес-логикой, если она там&#x20;

</details>

