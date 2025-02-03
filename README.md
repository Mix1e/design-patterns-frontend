# design-patterns-frontend
Моя интерпретация паттернов проектирования для frontend на typescript

| [[README#Creational Design Patterns\|Creational Design Patterns]] | [[README#Structural Design Patterns\| Structural Design Patterns]] | [Behavioral Design Patterns](app://obsidian.md/index.html#behavioral-design-patterns) |
| :---------------------------------------------------------------- | :----------------------------------------------------------------- | :------------------------------------------------------------------------------------ |
| [[README#🏠 Simple Factory \| Simple Factory]]                    | [[README#🔌 Adapter \| Adapter]]                                   | [Chain of Responsibility](app://obsidian.md/index.html#-chain-of-responsibility)      |
| [[README#🏭 Factory Method\|Factory Method]]                      | [[README#🚡 Bridge \| Bridge]]                                     | [Command](app://obsidian.md/index.html#-command)                                      |
| [[README#🔨 Abstract Factory\|Abstract Factory]]                  | [Composite](app://obsidian.md/index.html#-composite)               | [Iterator](app://obsidian.md/index.html#-iterator)                                    |
| [[README#👷 Builder\| Builder]]                                   | [Decorator](app://obsidian.md/index.html#-decorator)               | [Mediator](app://obsidian.md/index.html#-mediator)                                    |
| [[README#🐑 Prototype \| Prototype]]                              | [Facade](app://obsidian.md/index.html#-facade)                     | [Memento](app://obsidian.md/index.html#-memento)                                      |
| Singleton (не практичный)                                         | [Flyweight](app://obsidian.md/index.html#-flyweight)               | [Observer](app://obsidian.md/index.html#-observer)                                    |
|                                                                   | [Proxy](app://obsidian.md/index.html#-proxy)                       | [Visitor](app://obsidian.md/index.html#-visitor)                                      |
|                                                                   |                                                                    | [Strategy](app://obsidian.md/index.html#-strategy)                                    |
|                                                                   |                                                                    | [State](app://obsidian.md/index.html#-state)                                          |
|                                                                   |                                                                    | [Template Method](app://obsidian.md/index.html#-template-method)                      |

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
Суть паттерна заключается в создании метода внутри класса, который позволяет полностью скопировать другой класс
```typescript
interface Prototype<T> {
    clone(): T
}

class UserHistory implements Prototype<UserHistory> {
    createdAt: Date;
    constructor(public email: string, public name: string) {
        this.createdAt = new Date();
    }

    clone(): UserHistory {
        let target = new UserHistory(this.email, this.name);
        target.createdAt = this.createdAt;
        return target
    }
}

let user = new UserHistory('vladislav.pestsov@gmail.com', 'Vladislav')
console.log(user);
let user2 = user.clone()
console.log(user2)
```

# Structural Design Patterns

>Паттерны, созданные на создание связи между сущностями

## 🔌 Adapter
> Класс адаптер позволяет обернуть несоответствующий интерфейсу объект, чтобы он соответствовал другому классу

```typescript
interface Lion {
    roar(): void;
}

class AfricanLion implements Lion {
    roar(): void {
        // Implementation for African Lion's roar
    }
}

class AsianLion implements Lion {
    roar(): void {
        // Implementation for Asian Lion's roar
    }
}
```

```typescript
class Hunter {
    hunt(lion: Lion): void {
        lion.roar();
    }
}
```

При этом, мы можем добавить дикую собаку, которая гавкает, но охотник также может на неё охотиться, чтобы добавиться желаемого поведения - пишем класс адаптер
```typescript
// This needs to be added to the game
class WildDog {
    bark(): void {
        // Implementation of bark
    }
}

// Adapter around wild dog to make it compatible with our game
class WildDogAdapter implements Lion {
    protected dog: WildDog;

    constructor(dog: WildDog) {
        this.dog = dog;
    }

    roar(): void {
        this.dog.bark();
    }
}
```

Использование
```typescript
const wildDog = new WildDog();
const wildDogAdapter = new WildDogAdapter(wildDog);

const hunter = new Hunter();
hunter.hunt(wildDogAdapter);
```

## 🚡 Bridge
>Это паттерн суть которого заключается в отделении абстракции от реализации, чтобы они могли изменяться независимо друг от друга
 
![[Pasted image 20250203143740.png]]

Есть иерархия страниц
```typescript
interface WebPage {
    getContent(): string;
}

class About implements WebPage {
    protected theme: Theme;

    constructor(theme: Theme) {
        this.theme = theme;
    }

    getContent(): string {
        return "About page in " + this.theme.getColor();
    }
}

class Careers implements WebPage {
    protected theme: Theme;

    constructor(theme: Theme) {
        this.theme = theme;
    }

    getContent(): string {
        return "Careers page in " + this.theme.getColor();
    }
}
```

Также есть иерархия тем
```typescript
interface Theme {
    getColor(): string;
}

class DarkTheme implements Theme {
    getColor(): string {
        return 'Dark Black';
    }
}

class LightTheme implements Theme {
    getColor(): string {
        return 'Off white';
    }
}

class AquaTheme implements Theme {
    getColor(): string {
        return 'Light blue';
    }
}
```

```typescript
const darkTheme = new DarkTheme();
const about = new About(darkTheme);
const careers = new Careers(darkTheme);

console.log(about.getContent()); // "About page in Dark Black"
console.log(careers.getContent()); // "Careers page in Dark Black"
```

## 🌿 Composite
>Данный паттерн позволяет единообразно взаимодействовать с разными объектами

```typescript
interface Employee {
    getName(): string;
    setSalary(salary: number): void;
    getSalary(): number;
    getRoles(): string[];
}

class Developer implements Employee {
    private salary: number;
    private name: string;
    private roles: string[] = [];

    constructor(name: string, salary: number) {
        this.name = name;
        this.salary = salary;
    }

    getName(): string {
        return this.name;
    }

    setSalary(salary: number): void {
        this.salary = salary;
    }

    getSalary(): number {
        return this.salary;
    }

    getRoles(): string[] {
        return this.roles;
    }
}

class Designer implements Employee {
    private salary: number;
    private name: string;
    private roles: string[] = [];

    constructor(name: string, salary: number) {
        this.name = name;
        this.salary = salary;
    }

    getName(): string {
        return this.name;
    }

    setSalary(salary: number): void {
        this.salary = salary;
    }

    getSalary(): number {
        return this.salary;
    }

    getRoles(): string[] {
        return this.roles;
    }
}
```

При наличии разных типов работников, у нас есть организация, которой не так важны различия между работниками, к примеру, когда вопрос касается денег:
```typescript
class Organization {
    private employees: Employee[] = [];

    addEmployee(employee: Employee): void {
        this.employees.push(employee);
    }

    getNetSalaries(): number {
        return this.employees.reduce((total, employee) => total + employee.getSalary(), 0);
    }
}
```

## ☕ Decorator
> Паттерн позволяет динамически во время выполнения программы изменять поведение объекта подставляя его в другой декоратор (аддон) 

```typescript
interface Coffee {
    getCost(): number;
    getDescription(): string;
}

class SimpleCoffee implements Coffee {
    getCost(): number {
        return 10;
    }

    getDescription(): string {
        return 'Simple coffee';
    }
}
```

```typescript
class MilkCoffee implements Coffee {
    private coffee: Coffee;

    constructor(coffee: Coffee) {
        this.coffee = coffee;
    }

    getCost(): number {
        return this.coffee.getCost() + 2;
    }

    getDescription(): string {
        return this.coffee.getDescription() + ', milk';
    }
}

class WhipCoffee implements Coffee {
    private coffee: Coffee;

    constructor(coffee: Coffee) {
        this.coffee = coffee;
    }

    getCost(): number {
        return this.coffee.getCost() + 5;
    }

    getDescription(): string {
        return this.coffee.getDescription() + ', whip';
    }
}

class VanillaCoffee implements Coffee {
    private coffee: Coffee;

    constructor(coffee: Coffee) {
        this.coffee = coffee;
    }

    getCost(): number {
        return this.coffee.getCost() + 3;
    }

    getDescription(): string {
        return this.coffee.getDescription() + ', vanilla';
    }
}
```

Использование:
```typescript
const someCoffee: Coffee = new SimpleCoffee();
console.log(someCoffee.getCost()); // 10
console.log(someCoffee.getDescription()); // Simple coffee

const milkCoffee: Coffee = new MilkCoffee(someCoffee);
console.log(milkCoffee.getCost()); // 12
console.log(milkCoffee.getDescription()); // Simple coffee, milk

const whipCoffee: Coffee = new WhipCoffee(milkCoffee);
console.log(whipCoffee.getCost()); // 17
console.log(whipCoffee.getDescription()); // Simple coffee, milk, whip

const vanillaCoffee: Coffee = new VanillaCoffee(whipCoffee);
console.log(vanillaCoffee.getCost()); // 20
console.log(vanillaCoffee.getDescription()); // Simple coffee, milk, whip, vanilla
```

Таким образом, изначальный класс обрастаем классами обёртками, который на вход принимают `Coffee like class`
Я бы назвал это ~~Луковый паттерн~~

## 📦 Facade
