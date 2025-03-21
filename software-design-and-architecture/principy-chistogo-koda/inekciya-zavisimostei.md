# Инъекция зависимостей

**Dependency Injection** (DI) — это способ организации кода, при котором объект (или модуль) **не сам** создаёт свои зависимости, а **получает** их извне (через параметры конструктора, сеттеры или специальные механизмы). Такой подход реализует **принцип инверсии управления (IoC)** и принцип «Program Against Abstractions»: мы программируем против интерфейсов, а конкретные реализации внедряются (inject) нам извне.

***

### 1. Зачем нужен Dependency Injection

1. **Слабая связность (Low Coupling)**
   * Когда класс сам создаёт (`new`) другие объекты, он жёстко привязан к конкретным реализациям. При DI класс только «знает», что ему нужна некая абстракция (например, `IRepository`), а конкретный экземпляр «RepositoryImpl» поставляется извне.
2. **Упрощение тестирования**
   * При DI мы можем в тестах подставлять (mock) зависимости, без реальных баз, сетевых вызовов и т. д.
   * Это существенно облегчает написание unit-тестов.
3. **Гибкая архитектура**
   * При добавлении нового варианта зависимости (новая реализация интерфейса) мы не меняем код, который её использует. Достаточно зарегистрировать новую реализацию в IoC-контейнере или при создании объекта.
4. **Следование принципам SOLID (особенно DIP)**
   * DIP (Dependency Inversion Principle) говорит: модули верхнего уровня не должны зависеть от модулей низкого уровня; оба вида должны зависеть от абстракций. DI помогает этого достичь, так как высокоуровневый код зависит только от интерфейсов, а низкоуровневые реализации инжектируются.

***

### 2. Способы внедрения зависимостей (DI)

#### 2.1. Constructor Injection (через конструктор)

Самый распространённый и рекомендуемый способ — передавать зависимости как параметры конструктора.

```ts
interface UserRepository {
  getUser(id: number): string;
}

class UserService {
  private repo: UserRepository;
  constructor(repo: UserRepository) {
    this.repo = repo;
  }

  public findUser(id: number) {
    return this.repo.getUser(id);
  }
}
```

* В коде выше `UserService` не порождает (`new`) репозиторий, а **получает** его извне.

**Использование**:

```ts
class InMemoryUserRepo implements UserRepository {
  private data = { 1: "Alice", 2: "Bob" };
  getUser(id: number) {
    return this.data[id] || "Unknown";
  }
}

// "Собираем" зависимости
const repo = new InMemoryUserRepo();
const service = new UserService(repo);
console.log(service.findUser(1)); // "Alice"
```

#### 2.2. Setter Injection (через сеттеры/свойства)

Иногда DI выполняется через сеттеры (или публичные поля), особенно если зависимость может смениться в рантайме или задним числом:

```ts
class PaymentProcessor {
  private paymentMethod: PaymentMethod;

  public setPaymentMethod(method: PaymentMethod) {
    this.paymentMethod = method;
  }

  public process(amount: number) {
    this.paymentMethod.pay(amount);
  }
}
```

* Однако это чуть менее безопасно, так как объект может существовать без инициализации зависимости. Constructor Injection обычно предпочтительнее.

#### 2.3. Interface Injection (реже встречается)

Когда фреймворк или IoC-контейнер вызывает метод интерфейса, передавая зависимость. Не очень популярно в современных DI-фреймворках.

***

### 3. Инверсия управления (IoC-контейнеры)

**IoC-контейнер** (англ. Inversion of Control Container) — это библиотека/механизм, который автоматически:

1. **Сканирует** или **регистрирует** классы и их зависимости (как key-value: `IUserRepository -> UserRepositoryImpl`).
2. **Создаёт объекты** при запросе, внедряя нужные зависимости (Constructor Injection).
3. **Управляет** их жизненным циклом (singleton, transient и т. д.).

#### 3.1. Примеры IoC-фреймворков

* **Angular** (TypeScript) имеет встроенный DI-контейнер (decorators `@Injectable`, `@Inject`).
* **NestJS** (TypeScript) — также декораторы `@Injectable()`, `@Module()`.
* **Spring** (Java) — `@Autowired`, `@Component`, `@Service` и др.



#### 3.2. Пример: NestJS

```ts
@Injectable()
export class UserRepositoryImpl implements UserRepository {
  getUser(id: number) {
    return `User #${id}`;
  }
}

@Injectable()
export class UserService {
  constructor(private repo: UserRepository) {}
  
  findUser(id: number) {
    return this.repo.getUser(id);
  }
}
```

* Nest сам сканирует классы, видит `UserRepositoryImpl` как реализацию `UserRepository`, и внедряет её в `UserService`.
* Код `UserService` не привязан к конкретному репозиторию.

***

### 4. Простая демонстрация DI без фреймворка (Vanilla)

```ts
// Интерфейс
interface Logger {
  log(msg: string): void;
}

// Реализация
class ConsoleLogger implements Logger {
  log(msg: string) {
    console.log("[ConsoleLogger]", msg);
  }
}

// Класс, зависящий от Logger (получает через конструктор)
class OrderService {
  constructor(private logger: Logger) {}

  createOrder(orderId: number) {
    // Логика...
    this.logger.log(`Order #${orderId} created`);
  }
}

// "Сборка"
function main() {
  const logger = new ConsoleLogger();
  const service = new OrderService(logger);
  service.createOrder(1234);
}

main();
```

* Здесь DI вручную: мы «собираем» объекты в `main()`, а `OrderService` довольствуется зависимостью `Logger`, не зная, что это `ConsoleLogger`.

***

### 5. Когда DI может быть избыточным

1. **Очень маленькие скрипты**
   * Когда проект крошечный, и нет планов на масштаб, DI-контейнер может быть оверхедом.
2. **Статически связанная логика**
   * Если у вас нет никакого варианта реализации, и объект **всегда** использует одну и ту же зависимость, DI может казаться «лишним».
   * Но всё равно, если код растёт, DI может пригодиться для тестов.
3. **Микросервисы** (где сервис маленький)
   * Часто микросервис итак мал, а код почти самодостаточен. Небольшая инъекция конструктора может быть, но сложного IoC-контейнера не нужно.
4. **Злоупотребление**
   * Слишком абстрагироваться, когда, например, 1–2 зависимости, может сделать код избыточно формальным.

***

### Итог

**Dependency Injection** — ключевой механизм для построения гибких, слабо связанных систем. Он:

* Позволяет **программировать против абстракций**.
* Упрощает **тестирование** (замена реальных зависимостей на mock).
* Следует **DIP** и поддерживает **IoC** (инверсию управления).

Хотя в небольших скриптах DI бывает избыточным, при разработке средних и крупных приложений (особенно на Angular, NestJS, Spring, и т.д.) это стандартная практика, которая повышает масштабируемость, удобство сопровождения и переиспользования кода.
