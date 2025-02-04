# design-patterns-frontend
Моя интерпретация паттернов проектирования для frontend на typescript
Оригинал: https://github.com/kamranahmedse/design-patterns-for-humans

| [[README#Creational Design Patterns\|Creational Design Patterns]] | [[README#Structural Design Patterns\| Structural Design Patterns]] | [[README#Behavioral Design Patterns \| Behavioral Design Patterns]] |
| :---------------------------------------------------------------- | :----------------------------------------------------------------- | :------------------------------------------------------------------ |
| [[README#🏠 Simple Factory \| Simple Factory]]                    | [[README#🔌 Adapter \| Adapter]]                                   | [[README#🔗 Chain of Responsibility \| Chain of Responsibility]]    |
| [[README#🏭 Factory Method\|Factory Method]]                      | [[README#🚡 Bridge \| Bridge]]                                     | [[README#👮 Command \| Command]]                                    |
| [[README#🔨 Abstract Factory\|Abstract Factory]]                  | [[README#🌿 Composite \| Composite]]                               | [[README#➿ Iterator \| Iterator]]                                   |
| [[README#👷 Builder\| Builder]]                                   | [[README#☕ Decorator \| Decorator]]                                | [[README#👽 Mediator \| Mediator]]                                  |
| [[README#🐑 Prototype \| Prototype]]                              | [[README#📦 Facade\| Facade]]                                      | [[README#💾 Memento \| Memento]]                                    |
| Singleton (не практичный)                                         | [[README#🍃 Flyweight \| Flyweight]]                               | [[README#😎 Observer \| Observer]]                                  |
|                                                                   | [[README#🎱 Proxy \| Proxy]]                                       | [[README#🏃 Visitor \| Visitor]]                                    |
|                                                                   |                                                                    | [[README#💡 Strategy \| Strategy]]                                  |
|                                                                   |                                                                    | [[README#💢 State \| State]]                                        |
|                                                                   |                                                                    | [[README#📒 Template Method \| Template Method]]                    |

# Creational Design Patterns
---
## 🏠 Simple Factory
> Простая фабрика обеспечивает создание объекта. Можно использовать чтобы предотвратить дублирования логики создания объекта.
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
> Делегирование логики создания дочерним классам

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
>Фабрика фабрик, паттерн при котором инкапсулируются группы фабрик с общей идеей, не завязываясь на конкретных классах.

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

>Этот паттерн помогает победить анти-паттерн `telescoping constructor`, вместо огромного количества входных параметров в конструктор, мы передаём объект, при этом создаётся вспомогательный класс Builder, который упрощает создание экземпляра объекта

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
>Суть паттерна заключается в создании метода внутри класса, который позволяет полностью скопировать другой класс
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
---
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
> Обеспечивает упрощённый интерфейс для взаимодействия с подсистемами

```typescript
class Computer {
    getElectricShock(): void {
        console.log("Ouch!");
    }

    makeSound(): void {
        console.log("Beep beep!");
    }

    showLoadingScreen(): void {
        console.log("Loading..");
    }

    bam(): void {
        console.log("Ready to be used!");
    }

    closeEverything(): void {
        console.log("Bup bup bup buzzzz!");
    }

    sooth(): void {
        console.log("Zzzzz");
    }

    pullCurrent(): void {
        console.log("Haaah!");
    }
}
```

```typescript
class ComputerFacade {
    private computer: Computer;

    constructor(computer: Computer) {
        this.computer = computer;
    }

    turnOn(): void {
        this.computer.getElectricShock();
        this.computer.makeSound();
        this.computer.showLoadingScreen();
        this.computer.bam();
    }

    turnOff(): void {
        this.computer.closeEverything();
        this.computer.pullCurrent();
        this.computer.sooth();
    }
}
```

```typescript
const computer = new ComputerFacade(new Computer());

computer.turnOn(); // Ouch! Beep beep! Loading.. Ready to be used!
computer.turnOff(); // Bup bup buzzz! Haah! Zzzzz
```

## 🍃 Flyweight
> Суть паттерна в переиспользовании ресурсов в целях экономии памяти

```typescript
// Anything that will be cached is flyweight.
// Types of tea here will be flyweights.
class KarakTea {}

// Acts as a factory and saves the tea
class TeaMaker {
    private availableTea: { [key: string]: KarakTea } = {};

    make(preference: string): KarakTea {
        if (!this.availableTea[preference]) {
            this.availableTea[preference] = new KarakTea();
        }

        return this.availableTea[preference];
    }
}
```

```typescript
class TeaShop {
    private orders: { [key: number]: KarakTea } = {};
    private teaMaker: TeaMaker;

    constructor(teaMaker: TeaMaker) {
        this.teaMaker = teaMaker;
    }

    takeOrder(teaType: string, table: number): void {
        this.orders[table] = this.teaMaker.make(teaType);
    }

    serve(): void {
        for (const table in this.orders) {
            console.log(`Serving tea to table# ${table}`);
        }
    }
}
```

```typescript
const teaMaker = new TeaMaker();
const shop = new TeaShop(teaMaker);

shop.takeOrder('less sugar', 1);
shop.takeOrder('more milk', 2);
shop.takeOrder('without sugar', 5);

shop.serve();
// Serving tea to table# 1
// Serving tea to table# 2
// Serving tea to table# 5
```

## 🎱 Proxy
> В этом паттерне класс предоставляет функциональность другого класса

```typescript
interface Door {
    open(): void;
    close(): void;
}

class LabDoor implements Door {
    open(): void {
        console.log("Opening lab door");
    }

    close(): void {
        console.log("Closing the lab door");
    }
}
```

```typescript
class SecuredDoor implements Door {
    private door: Door;

    constructor(door: Door) {
        this.door = door;
    }

    open(password: string): void {
        if (this.authenticate(password)) {
            this.door.open();
        } else {
            console.log("Big no! It ain't possible.");
        }
    }

    private authenticate(password: string): boolean {
        return password === '$ecr@t';
    }

    close(): void {
        this.door.close();
    }
}
```

```typescript
const door = new SecuredDoor(new LabDoor());

door.open('invalid'); // Big no! It ain't possible.
door.open('$ecr@t');   // Opening lab door
door.close();         // Closing lab door
```

Также этот паттерн, к примеру, можно использовать для кеширования данных

# Behavioral Design Patterns
---
Поведенческие паттерны больше ориентированы на распределение ответственности между объектами. Принципиально отличаются они от структурных паттернов тем, что ориентированы больше на передачу сообщений и коммуникацию между сущностями.
## 🔗 Chain of Responsibility
> Суть цепочки заключается в наличие ссылки на себе подобный объект, к которому будет делегирована логики выполнения при определённом условии

```typescript
abstract class Account {
    protected successor: Account | null = null;
    protected balance: number;

    public setNext(account: Account): void {
        this.successor = account;
    }

    public pay(amountToPay: number): void | never {
        if (this.canPay(amountToPay)) {
            console.log(`Paid ${amountToPay} using ${this.constructor.name}`);
        } else if (this.successor) {
            console.log(`Cannot pay using ${this.constructor.name}. Proceeding..`);
            this.successor.pay(amountToPay);
        } else {
            throw new Error('None of the accounts have enough balance');
        }
    }

    public canPay(amount: number): boolean {
        return this.balance >= amount;
    }
}

class Bank extends Account {
    constructor(balance: number) {
        super();
        this.balance = balance;
    }
}

class Paypal extends Account {
    constructor(balance: number) {
        super();
        this.balance = balance;
    }
}

class Bitcoin extends Account {
    constructor(balance: number) {
        super();
        this.balance = balance;
    }
}
```

Выстраиваем банки по приоритетам и запускаем метод платежа
```typescript
// Let's prepare a chain like below
//      bank -> paypal -> bitcoin
//
// First priority bank
//      If bank can't pay then paypal
//      If paypal can't pay then bitcoin

const bank = new Bank(100);    // Bank with balance 100
const paypal = new Paypal(200); // Paypal with balance 200
const bitcoin = new Bitcoin(300); // Bitcoin with balance 300

bank.setNext(paypal);
paypal.setNext(bitcoin);

// Let's try to pay using the first priority i.e. bank
bank.pay(259);

// Output will be
// ==============
// Cannot pay using Bank. Proceeding ..
// Cannot pay using Paypal. Proceeding ..
// Paid 259 using Bitcoin!
```

## 👮 Command
> Суть паттерна заключается инкапсуляции действий в объектах

Прежде всего, класс - раб, который отвечает за весь функционал, который может быть выполнен
```typescript
// Receiver
class Bulb {
    public turnOn(): void {
        console.log("Bulb has been lit");
    }

    public turnOff(): void {
        console.log("Darkness!");
    }
}
```

Есть интерфейс `Command`, который выполняет команду
```typescript
interface Command {
    execute(): void;
    undo(): void;
    redo(): void;
}

// Command
class TurnOn implements Command {
    protected bulb: Bulb;

    constructor(bulb: Bulb) {
        this.bulb = bulb;
    }

    public execute(): void {
        this.bulb.turnOn();
    }

    public undo(): void {
        this.bulb.turnOff();
    }

    public redo(): void {
        this.execute();
    }
}

class TurnOff implements Command {
    protected bulb: Bulb;

    constructor(bulb: Bulb) {
        this.bulb = bulb;
    }

    public execute(): void {
        this.bulb.turnOff();
    }

    public undo(): void {
        this.bulb.turnOn();
    }

    public redo(): void {
        this.execute();
    }
}
```

`Invoker` запускает и взаимодействует с любыми командами
```typescript
// Invoker
class RemoteControl {
    public submit(command: Command): void {
        command.execute();
    }
}
```

Использование
```typescript
cosnt bulb = new Bulb();

const turnOn = new TurnOn(bulb);
const turnOff = new TurnOff(bulb);

const remote = new RemoteControl();
remote.submit(turnOn); // Bulb has been lit!
remote.submit(turnOff); // Darkness!
```

Паттерн хорошо подходит для систем основанных на транзакциях. В случае если какая-то команда не выполнена, мы можем перемещаясь по системе операции вызывать метод undo.

## ➿ Iterator
> Создание итерируемого класса, с целью упрощения взаимодействия с объектом

```typescript
class RadioStation {
    protected frequency: number;

    constructor(frequency: number) {
        this.frequency = frequency;
    }

    getFrequency(): number {
        return this.frequency;
    }
}
```

```typescript
class StationList implements Iterable<RadioStation> {
    private stations: RadioStation[] = [];
    private counter: number = 0;

    public addStation(station: RadioStation): void {
        this.stations.push(station);
    }

    public removeStation(toRemove: RadioStation): void {
        const toRemoveFrequency = toRemove.getFrequency();
        this.stations = this.stations.
	        filter(station => station.getFrequency() !== toRemoveFrequency);
    }

    public count(): number {
        return this.stations.length;
    }

    public current(): RadioStation {
        return this.stations[this.counter];
    }

    public key(): number {
        return this.counter;
    }

    public next(): void {
        this.counter++;
    }

    public rewind(): void {
        this.counter = 0;
    }

    public valid(): boolean {
        return this.counter >= 0 && this.counter < this.stations.length;
    }

    [Symbol.iterator](): Iterator<RadioStation> {
        this.rewind();
        return {
            next: () => {
                if (this.valid()) {
                    return { value: this.current(), done: false };
                } else {
                    return { done: true } as IteratorResult<RadioStation>;
                }
            }
        };
    }
}
```

Использование
```typescript
const stationList = new StationList();

stationList.addStation(new RadioStation(89));
stationList.addStation(new RadioStation(101));
stationList.addStation(new RadioStation(102));
stationList.addStation(new RadioStation(103.2));

for (const station of stationList) {
    console.log(station.getFrequency());
}

stationList.removeStation(new RadioStation(89)); // Will remove station 89
```

Паттерн позволяет итерироваться и взаимодействовать с объектом без понимания его внутренней реализации

## 👽 Mediator
>Паттерн посредника (сущности со стороны), для обеспечения взаимодействия между объектами

```typescript
interface ChatRoomMediator {
    showMessage(user: User, message: string): void;
}

// Mediator
class ChatRoom implements ChatRoomMediator {
    showMessage(user: User, message: string): void {
        const time: string = new Date()
			        .toLocaleString('en-US', { timeZone: 'UTC', hour12: false });
        const sender: string = user.getName();

        console.log(`${time} [${sender}]: ${message}`);
    }
}
```

```typescript
class User {
    protected name: string;
    protected chatMediator: ChatRoomMediator;

    constructor(name: string, chatMediator: ChatRoomMediator) {
        this.name = name;
        this.chatMediator = chatMediator;
    }

    getName(): string {
        return this.name;
    }

    send(message: string): void {
        this.chatMediator.showMessage(this, message);
    }
}
```

```typescript
const mediator = new ChatRoom();

const john = new User('John Doe', mediator);
const jane = new User('Jane Doe', mediator);

john.send('Hi there!');
jane.send('Hey!');
// Output will be
// Feb 14, 10:58 [John]: Hi there!
// Feb 14, 10:58 [Jane]: Hey!
```

> NOTE: честно говоря, не знаю как это ещё использовать кроме как логировать информацию

## 💾 Memento
> Паттерн, который позволяет восстанавливать состояние объекта

```typescript
class EditorMemento {
    protected content: string;

    constructor(content: string) {
        this.content = content;
    }

    getContent(): string {
        return this.content;
    }
}
```

```typescript
class Editor {
    protected content: string = '';

    type(words: string): void {
        this.content = this.content + ' ' + words;
    }

    getContent(): string {
        return this.content;
    }

    save(): EditorMemento {
        return new EditorMemento(this.content);
    }

    restore(memento: EditorMemento): void {
        this.content = memento.getContent();
    }
}
```

Использование
```typescript
const editor = new Editor();

// Type some stuff
editor.type('This is the first sentence.');
editor.type('This is second.');

// Save the state to restore to : This is the first sentence. This is second.
const saved = editor.save();

// Type some more
editor.type('And this is third.');

// Output: Content before Saving
console.log(editor.getContent()); // This is the first sentence. This is second. And this is third.

// Restoring to last saved state
editor.restore(saved);

editor.getContent(); // This is the first sentence. This is second.
```

Таким образом можно создавать новый экземпляр объекта в нужном нам состоянии

## 😎 Observer
> Определение зависимости от некоторого объекта с целью реагирования на изменение его состояния

```typescript
class JobPost {
    protected title: string;

    constructor(title: string) {
        this.title = title;
    }

    getTitle(): string {
        return this.title;
    }
}

class JobSeeker implements Observer {
    protected name: string;

    constructor(name: string) {
        this.name = name;
    }

    onJobPosted(job: JobPost): void {
        console.log(`Hi ${this.name}! New job posted: ${job.getTitle()}`);
    }
}
```

```typescript
class EmploymentAgency implements Observable {
    protected observers: Observer[] = [];

    protected notify(jobPosting: JobPost): void {
        this.observers.forEach(observer => observer.onJobPosted(jobPosting));
    }

    attach(observer: Observer): void {
        this.observers.push(observer);
    }

    addJob(jobPosting: JobPost): void {
        this.notify(jobPosting);
    }
}
```

```typescript
// Create subscribers
const johnDoe = new JobSeeker('John Doe');
const janeDoe = new JobSeeker('Jane Doe');

// Create publisher and attach subscribers
const jobPostings = new EmploymentAgency();
jobPostings.attach(johnDoe);
jobPostings.attach(janeDoe);

// Add a new job and see if subscribers get notified
jobPostings.addJob(new JobPost('Software Engineer'));

// Output
// Hi John Doe! New job posted: Software Engineer
// Hi Jane Doe! New job posted: Software Engineer
```

Наблюдатель `JobSeeker` получает от наблюдаемого объекта `EmploymentAgency`, "уведомление", т.к. наблюдаемый объект вызывает необходимый метод у всех своих наблюдателей

## 🏃 Visitor
> Позволяет добавлять опции в объект, без его изменения

```typescript
// Visitee
interface Animal {
    accept(operation: AnimalOperation): void;
}

// Visitor
interface AnimalOperation {
    visitMonkey(monkey: Monkey): void;
    visitLion(lion: Lion): void;
    visitDolphin(dolphin: Dolphin): void;
}
```

```typescript
interface Animal {
    accept(operation: AnimalOperation): void;
}

class Monkey implements Animal {
    public shout(): void {
        console.log('Ooh oo aa aa!');
    }

    public accept(operation: AnimalOperation): void {
        operation.visitMonkey(this);
    }
}

class Lion implements Animal {
    public roar(): void {
        console.log('Roaaar!');
    }

    public accept(operation: AnimalOperation): void {
        operation.visitLion(this);
    }
}

class Dolphin implements Animal {
    public speak(): void {
        console.log('Tuut tuttu tuutt!');
    }

    public accept(operation: AnimalOperation): void {
        operation.visitDolphin(this);
    }
}
```

```typescript
class Speak implements AnimalOperation {
    public visitMonkey(monkey: Monkey): void {
        monkey.shout();
    }

    public visitLion(lion: Lion): void {
        lion.roar();
    }

    public visitDolphin(dolphin: Dolphin): void {
        dolphin.speak();
    }
}
```

```typescript
const monkey = new Monkey();
const lion = new Lion();
const dolphin = new Dolphin();

const speak = new Speak();

monkey.accept(speak);    // Ooh oo aa aa!    
lion.accept(speak);      // Roaaar!
dolphin.accept(speak);   // Tuut tutt tuutt!
```

Также мы можем добавить новый функционал к классам животных просто добавив новых `visitor`
```typescript
class Jump implements AnimalOperation {
    public visitMonkey(monkey: Monkey): void {
        console.log('Jumped 20 feet high! on to the tree!');
    }

    public visitLion(lion: Lion): void {
        console.log('Jumped 7 feet! Back on the ground!');
    }

    public visitDolphin(dolphin: Dolphin): void {
        console.log('Walked on water a little and disappeared');
    }
}
```

Использование
```typescript
const jump = new Jump();

monkey.accept(speak);        // Ooh oo aa aa!
monkey.accept(jump);         // Jumped 20 feet high! on to the tree!

lion.accept(speak);          // Roaaar!
lion.accept(jump);           // Jumped 7 feet! Back on the ground!

dolphin.accept(speak);       // Tuut tutt tuutt!
dolphin.accept(jump);        // Walked on water a little and disappeared
```

## 💡 Strategy
> Паттерн стратегии позволяет переключать поведение в зависимости от ситуации

```typescript
interface SortStrategy {
    sort(dataset: any[]): any[];
}

class BubbleSortStrategy implements SortStrategy {
    sort(dataset: any[]): any[] {
        console.log("Sorting using bubble sort");

        // Do sorting
        return dataset;
    }
}

class QuickSortStrategy implements SortStrategy {
    sort(dataset: any[]): any[] {
        console.log("Sorting using quick sort");

        // Do sorting
        return dataset;
    }
}
```

```typescript
class Sorter {
    protected sorterSmall: SortStrategy;
    protected sorterBig: SortStrategy;

    constructor(sorterSmall: SortStrategy, sorterBig: SortStrategy) {
        this.sorterSmall = sorterSmall;
        this.sorterBig = sorterBig;
    }

    public sort(dataset: any[]): any[] {
        if (dataset.length > 5) {
            return this.sorterBig.sort(dataset);
        } else {
            return this.sorterSmall.sort(dataset);
        }
    }
}
```

Использование
```typescript
const smallDataset = [1, 3, 4, 2];
const bigDataset = [1, 4, 3, 2, 8, 10, 5, 6, 9, 7];

const sorter: Sorter = new Sorter(new BubbleSortStrategy(), new QuickSortStrategy());

sorter.sort(smallDataset); // Output : Sorting using bubble sort
sorter.sort(bigDataset); // Output : Sorting using quick sort
```

## 💢 State
> Изменение поведения класса при изменении состояния (стейт-машина)

```typescript
// PhoneState interface representing the different states of a phone
interface PhoneState {
    pickUp(): PhoneState;
    hangUp(): PhoneState;
    dial(): PhoneState;
}

// Implementation of the idle state of the phone
class PhoneStateIdle implements PhoneState {
    pickUp(): PhoneState {
        return new PhoneStatePickedUp();
    }
    hangUp(): PhoneState {
        throw new Error("already idle");
    }
    dial(): PhoneState {
        throw new Error("unable to dial in idle state");
    }
}

// Implementation of the picked up state of the phone
class PhoneStatePickedUp implements PhoneState {
    pickUp(): PhoneState {
        throw new Error("already picked up");
    }
    hangUp(): PhoneState {
        return new PhoneStateIdle();
    }
    dial(): PhoneState {
        return new PhoneStateCalling();
    }
}

// Implementation of the calling state of the phone
class PhoneStateCalling implements PhoneState {
    pickUp(): PhoneState {
        throw new Error("already picked up");
    }
    hangUp(): PhoneState {
        return new PhoneStateIdle();
    }
    dial(): PhoneState {
        throw new Error("already dialing");
    }
}
```

Класс телефон, который переключает состояния
``` typescript
class Phone {
    private currentState: PhoneState;

    constructor() {
        this.currentState = new PhoneStateIdle();
    }

    public pickUp(): void {
        this.currentState = this.currentState.pickUp();
    }

    public hangUp(): void {
        this.currentState = this.currentState.hangUp();
    }

    public dial(): void {
        this.currentState = this.currentState.dial();
    }
}
```

```typescript
const phone = new Phone();

phone.pickUp();
phone.dial();
```

## 📒 Template Method
