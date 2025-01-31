# design-patterns-frontend
Моя интерпретация паттернов проектирования для frontend на typescript

| [[README#Creational Design Patterns\|Creational Design Patterns]]  | [Structural Design Patterns](app://obsidian.md/index.html#structural-design-patterns) | [Behavioral Design Patterns](app://obsidian.md/index.html#behavioral-design-patterns) |
| :----------------------------------------------------------------- | :------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------ |
| [[README#🏠 Simple Factory \| Simple Factory]]                     | [Adapter](app://obsidian.md/index.html#-adapter)                                      | [Chain of Responsibility](app://obsidian.md/index.html#-chain-of-responsibility)      |
| [[README#🏭 Factory Method\|Factory Method]]                       | [Bridge](app://obsidian.md/index.html#-bridge)                                        | [Command](app://obsidian.md/index.html#-command)                                      |
| [Abstract Factory](app://obsidian.md/index.html#-abstract-factory) | [Composite](app://obsidian.md/index.html#-composite)                                  | [Iterator](app://obsidian.md/index.html#-iterator)                                    |
| [Builder](app://obsidian.md/index.html#-builder)                   | [Decorator](app://obsidian.md/index.html#-decorator)                                  | [Mediator](app://obsidian.md/index.html#-mediator)                                    |
| [Prototype](app://obsidian.md/index.html#-prototype)               | [Facade](app://obsidian.md/index.html#-facade)                                        | [Memento](app://obsidian.md/index.html#-memento)                                      |
| Singleton (не практичный)                                          | [Flyweight](app://obsidian.md/index.html#-flyweight)                                  | [Observer](app://obsidian.md/index.html#-observer)                                    |
|                                                                    | [Proxy](app://obsidian.md/index.html#-proxy)                                          | [Visitor](app://obsidian.md/index.html#-visitor)                                      |
|                                                                    |                                                                                       | [Strategy](app://obsidian.md/index.html#-strategy)                                    |
|                                                                    |                                                                                       | [State](app://obsidian.md/index.html#-state)                                          |
|                                                                    |                                                                                       | [Template Method](app://obsidian.md/index.html#-template-method)                      |

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
