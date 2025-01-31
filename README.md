# design-patterns-frontend
Моя интерпретация паттернов проектирования для frontend на typescript

| [[README#Creational Design Patterns\|Creational Design Patterns]] | [Structural Design Patterns](app://obsidian.md/index.html#structural-design-patterns) | [Behavioral Design Patterns](app://obsidian.md/index.html#behavioral-design-patterns) |
| :---------------------------------------------------------------- | :------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------ |
| [[README#🏠 Simple Factory \| Simple Factory]]                    | [Adapter](app://obsidian.md/index.html#-adapter)                                      | [Chain of Responsibility](app://obsidian.md/index.html#-chain-of-responsibility)      |
| [[README#🏭 Factory Method\|Factory Method]]                      | [Bridge](app://obsidian.md/index.html#-bridge)                                        | [Command](app://obsidian.md/index.html#-command)                                      |
| [[README#🔨 Abstract Factory\|Abstract Factory]]                  | [Composite](app://obsidian.md/index.html#-composite)                                  | [Iterator](app://obsidian.md/index.html#-iterator)                                    |
| [[README#design-patterns-frontend \| Builder]]                    | [Decorator](app://obsidian.md/index.html#-decorator)                                  | [Mediator](app://obsidian.md/index.html#-mediator)                                    |
| [[README#🐑 Prototype \| Prototype]]                              | [Facade](app://obsidian.md/index.html#-facade)                                        | [Memento](app://obsidian.md/index.html#-memento)                                      |
| Singleton (не практичный)                                         | [Flyweight](app://obsidian.md/index.html#-flyweight)                                  | [Observer](app://obsidian.md/index.html#-observer)                                    |
|                                                                   | [Proxy](app://obsidian.md/index.html#-proxy)                                          | [Visitor](app://obsidian.md/index.html#-visitor)                                      |
|                                                                   |                                                                                       | [Strategy](app://obsidian.md/index.html#-strategy)                                    |
|                                                                   |                                                                                       | [State](app://obsidian.md/index.html#-state)                                          |
|                                                                   |                                                                                       | [Template Method](app://obsidian.md/index.html#-template-method)                      |

# Creational Design Patterns
## 🏠 Simple Factory
---
Простая фабрика обеспечивает создание объекта. Можно использовать чтобы предотвратить дублирования логики создания объекта.
```typescript
interface Door {
    getWidth(): number;
    getHeight(): number;
}

class WoodenDoor implements Door {
    protected width: number;
    protected height: number;

    constructor(width: number, height: number) {
        this.width = width;
        this.height = height;
    }

    getWidth(): number {
        return this.width;
    }

    getHeight(): number {
        return this.height;
    }
}
```

```typescript
class DoorFactory {
    public static makeDoor(width: number, height: number): Door {
        return new WoodenDoor(width, height);
    }
}
```
Использование:
```typescript
import { DoorFactory } from './DoorFactory';

// Make me a door of 100x200
const door = DoorFactory.makeDoor(100, 200);

console.log('Width:', door.getWidth());
console.log('Height:', door.getHeight());

// Make me a door of 50x100
const door2 = DoorFactory.makeDoor(50, 100);
```
## 🏭 Factory Method
---
Делегирование логики создания дочерним классам

```typescript
interface Interviewer {
    askQuestions(): void;
}

class Developer implements Interviewer {
    askQuestions(): void {
        console.log('Asking about design patterns!');
    }
}

class CommunityExecutive implements Interviewer {
    askQuestions(): void {
        console.log('Asking about community building');
    }
}
```

Логика взятия интервью едина - поэтому реализуется сразу в абстрактном классе HiringManager.
```typescript
abstract class HiringManager {
    abstract protected makeInterviewer(): Interviewer;

    public takeInterview() {
        const interviewer = this.makeInterviewer();
        interviewer.askQuestions();
    }
}
```

В итоге получаем класс прослойку.
```typescript
class DevelopmentManager extends HiringManager {
    protected makeInterviewer(): Interviewer {
        return new Developer();
    }
}

class MarketingManager extends HiringManager {
    protected makeInterviewer(): Interviewer {
        return new CommunityExecutive();
    }
}
```

```typescript

const devManager = new DevelopmentManager();
devManager.takeInterview();

const marketingManager = new MarketingManager();
marketingManager.takeInterview();
```

## 🔨 Abstract Factory
Фабрика фабрик, паттерн при котором инкапсулируются группы фабрик с общей идеей, не завязываясь на конкретных классах.

Есть интерфейс `Door` и несколько его реализаций
```typescript
interface Door {
    getDescription(): void;
}

class WoodenDoor implements Door {
    getDescription(): void {
        console.log('I am a wooden door');
    }
}

class IronDoor implements Door {
    getDescription(): void {
        console.log('I am an iron door');
    }
}
```

Также есть мастера по установке этих дверей, каждый из который работает со своим типом двери
```typescript
interface DoorFittingExpert {
    getDescription(): void;
}

class Welder implements DoorFittingExpert {
    getDescription(): void {
        console.log('I can only fit iron doors');
    }
}

class Carpenter implements DoorFittingExpert {
    getDescription(): void {
        console.log('I can only fit wooden doors');
    }
}
```

И вот интерфейс абстрактной фабрики
```typescript
interface DoorFactory {
    makeDoor(): Door;
    makeFittingExpert(): DoorFittingExpert;
}

class WoodenDoorFactory implements DoorFactory {
    makeDoor(): Door {
        return new WoodenDoor();
    }

    makeFittingExpert(): DoorFittingExpert {
        return new Carpenter();
    }
}

class IronDoorFactory implements DoorFactory {
    makeDoor(): Door {
        return new IronDoor();
    }

    makeFittingExpert(): DoorFittingExpert {
        return new Welder();
    }
}
```

```typescript
// Usage
const woodenFactory = new WoodenDoorFactory();
const woodenDoor = woodenFactory.makeDoor();
const woodenExpert = woodenFactory.makeFittingExpert();

console.log(woodenDoor.getDescription());  // Output: I am a wooden door
console.log(woodenExpert.getDescription()); // Output: I can only fit wooden doors

const ironFactory = new IronDoorFactory();
const ironDoor = ironFactory.makeDoor();
const ironExpert = ironFactory.makeFittingExpert();

console.log(ironDoor.getDescription());  // Output: I am an iron door
console.log(ironExpert.getDescription()); // Output: I can only fit iron doors
```

Использовать такую фабрику стоит когда есть некоторая связь между сущностями

## 👷 Builder

Этот паттерн помогает победить анти-паттерн `telescoping constructor`, вместо огромного количества входных параметров в конструктор, мы передаём объект, при этом создаётся вспомогательный класс Builder, который упрощает создание экземпляра объекта

Класс объекта
```typescript
class Burger {
    protected size: string;
    protected cheese: boolean;
    protected pepperoni: boolean;
    protected lettuce: boolean;
    protected tomato: boolean;

    constructor(builder: BurgerBuilder) {
        this.size = builder.size;
        this.cheese = builder.cheese;
        this.pepperoni = builder.pepperoni;
        this.lettuce = builder.lettuce;
        this.tomato = builder.tomato;
    }
}
```

Билдер этого объекта
```typescript
class BurgerBuilder {
    size: number;
    cheese: boolean = false;
    pepperoni: boolean = false;
    lettuce: boolean = false;
    tomato: boolean = false;

    constructor(size: number) {
        this.size = size;
    }

    addPepperoni(): this {
        this.pepperoni = true;
        return this;
    }

    addLettuce(): this {
        this.lettuce = true;
        return this;
    }

    addCheese(): this {
        this.cheese = true;
        return this;
    }

    addTomato(): this {
        this.tomato = true;
        return this;
    }

    build(): Burger {
        return new Burger(this);
    }
}
```

Пример создания объекта
```typescript
const burger = new BurgerBuilder(9)
                    .addPepperoni()
                    .addLettuce()
                    .addTomato()
                    .build();
```

## 🐑 Prototype
