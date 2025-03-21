# Закон дементры

**Law of Demeter (LoD)** гласит, что объект должен взаимодействовать только с «близкими друзьями» и не «разгуливать» по внутренностям других объектов.\
Формально: «Метод объекта может вызывать методы только следующих “друзей”»:

1. **Самого себя** (его собственные методы).
2. **Своих полей** (объекты, хранящиеся в полях этого класса).
3. **Объектов, созданных внутри метода** (локальные переменные).
4. **Параметров, переданных в метод**.

Если приходится обращаться к чему-то за два–три объекта «вбок» (например, `objA.objB.objC.doSomething()`), мы рискуем нарушать инкапсуляцию, т. е. «лезем» в детали, которые не должны быть доступны.

#### Зачем это нужно

* **Сокращение связности**: объекты знают меньше о внутренней структуре других. Если в этой структуре что-то меняется, не придётся менять код повсюду.
* **Упрощение кода**: мы не пишем длинные цепочки вызовов, нет необходимости удерживать в уме сложные иерархии.
* **Лучше читаемость**: вместо `order.customer.address.city`, мы можем, например, спросить у `order` напрямую «какой город?», если действительно нужно знать.

***

### Пример «как не надо» (нарушение LoD)

Допустим, у нас есть несколько связанных классов:

```typescript
class Address {
  public city: string;
  public street: string;

  constructor(city: string, street: string) {
    this.city = city;
    this.street = street;
  }
}

class Customer {
  public address: Address;

  constructor(address: Address) {
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
```

Где-нибудь в коде (например, в `ShippingService`) мы хотим узнать, в каком городе будет доставка:

```typescript
class ShippingService {
  public calculateShipping(order: Order): number {
    // Нарушение: залезаем через order -> customer -> address -> city
    if (order.customer.address.city === "Москва") {
      return 0; // Бесплатная доставка
    } else {
      return 200; // Фиксированная стоимость
    }
  }
}

// Использование
function main() {
  const address = new Address("Москва", "Тверская");
  const customer = new Customer(address);
  const order = new Order(customer, 3000);

  const shippingService = new ShippingService();
  const shippingCost = shippingService.calculateShipping(order);
  console.log("Стоимость доставки:", shippingCost);
}

main();
```

* Здесь `ShippingService` явно ходит по цепочке `order.customer.address.city`.
* При этом, если структура `Order` или `Customer` изменится, нужно будет чинить код в `ShippingService`.

***

### Как сделать лучше (соблюдение LoD)

1. **Сократить «цепочку» с помощью методов делегирования**\
   Пусть класс `Order` сам предоставляет нужную информацию. Например, метод `getCity()`. Тогда клиент (здесь `ShippingService`) не будет знать, что где-то внутри есть `customer` или `address`.

```typescript
class OrderRefactored {
  private customer: Customer;
  private total: number;

  constructor(customer: Customer, total: number) {
    this.customer = customer;
    this.total = total;
  }

  // Делаем метод, скрывающий детали:
  public getCity(): string {
    return this.customer.address.city;
  }

  public getTotal(): number {
    return this.total;
  }
}

class ShippingServiceRefactored {
  public calculateShipping(order: OrderRefactored): number {
    // Теперь вызываем order.getCity() — нет цепочки obj.obj.obj
    if (order.getCity() === "Москва") {
      return 0;
    }
    return 200;
  }
}

// Использование
function mainRefactored() {
  const address = new Address("Москва", "Тверская");
  const customer = new Customer(address);
  const order = new OrderRefactored(customer, 3000);

  const shippingService = new ShippingServiceRefactored();
  const shippingCost = shippingService.calculateShipping(order);
  console.log("Стоимость доставки (LoD):", shippingCost);
}

mainRefactored();
```

* `ShippingServiceRefactored` теперь не знает, как именно `OrderRefactored` устроен внутри. Ему достаточно вызвать «говорящий» метод `getCity()`.

2. **Перенести поведение, если нужно**\
   Иногда логичнее спросить: а не должен ли `Order` сам решать, сколько будет доставка (особенно если логика завязана на состоянии заказа)? Или мы можем добавить метод `order.calculateShipping()`, и клиентскому коду не придётся знать ни о «городах», ни о «customer».
3. **Разделять ответственность**\
   Если вы видите несколько подобных цепочек, подумайте, не лучше ли дать объекту методы для тех операций, которые часто нужны «извне», избегая вызовов внутренних объектов напрямую.

***

#### Итог

**Law of Demeter** (Принцип Деметры) — рекомендация, которая снижает связанность кода и делает систему гибче. Благодаря этому принципу, объекты общаются через понятные интерфейсы, не раскрывая свою «внутреннюю кухню» (цепочки вложенных объектов). Сокращается риск, что при малейшем изменении структуры цепочка «обвалится» и придётся менять код в многочисленных местах.



### Ссылки

Стандартное определение [https://habr.com/ru/articles/319652/](https://habr.com/ru/articles/319652/)

Альтернативное мнение [https://www.yegor256.com/2016/07/18/law-of-demeter.html](https://www.yegor256.com/2016/07/18/law-of-demeter.html)
