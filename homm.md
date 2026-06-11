# Практика: HoMM

## 1. Описание предметной области и сущностей

Player - класс игрок, обладает золотом (Gold), может умереть (Dead), имеет идентификатор (Id) - это свойства. Он может сразиться с армией (CanBeat), собрать сокровища (Consume) и умереть (Die) - это методы класса.

Army - вспомогательный класс армия, есть параметр силы (Power).

Treasure - вспомогательный класс сокровища с количеством золота (Amount).

Dwelling - класс жилище, которое игрок может захватить (становится его владельцем Owner).

Mine - класс шахта, она охраняется армией, после победы игрок забирает сокровища (Treasure) и становится владельцем (Owner).

Creeps - класс нейтральные крипы, они охраняют сокровища, после победы игрок забирает сокровища.

Wolves - класс волки, имеет армию, игрок либо побеждает (ничего не получает, так как у них нет сокровищ), либо умирает.

ResourcePile - класс ресурсы без охраны, игрок просто забирает сокровища.

IOwner - интерфейс для объекта, у которого можно установить владельца (Owner).

IHaveArmy - интерфейс для объекта, имеющего армию для сражения (Army).

IHaveTreasure - интерфейс объекта, имеющего сокровища (Treasure).

Interaction - статический класс - точка входа для взаимодействия игрока с объектом на карте. Содержит единственный метод Make, который проверяет наличие армии и если игрок не может победить, он умирает, также он проверяет возможность присвоения владельца и ещё он проверяет наличие сокровищ, которые игрок забирает.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction TB

    class Player {
        -int Gold
        -bool Dead
        +int Id
        +CanBeat(Army army) bool
        +Consume(Treasure treasure)
        +Die()
    }

    class Army{
        +int Power
    }

    class Treasure {
        +int Amount
    }

    class IOwner {
        <<interface>>
        +int Owner
    }

    class IHaveArmy {
        <<interface>>
        +Army Army
    }

    class IHaveTreasure {
        <<interface>>
        +Treasure Treasure
    }

    class Dwelling {
        +int Owner
    }

    class Mine {
        +int Owner
        +Army Army
        +Treasure Treasure
    }

    class Creeps {
        +Army Army
        +Treasure Treasure
    }

    class Wolves {
        +Army Army
    }

    class ResourcePile {
        +Treasure Treasure
    }


    class Interaction {
        <<static>>
        +Make(Player player, object obj)
    }

    %% Реализации интерфейсов - Интерфейс <|.. Класс
    IOwner <|.. Dwelling
    IOwner <|.. Mine
    IHaveArmy <|.. Mine
    IHaveTreasure <|.. Mine
    IHaveArmy <|.. Creeps
    IHaveTreasure <|.. Creeps
    IHaveArmy <|.. Wolves
    IHaveTreasure <|.. ResourcePile

    %% Ассоциации - КлассА --> КлассБ
    Mine --> Army
    Mine --> Treasure
    Creeps --> Army
    Creeps --> Treasure
    Wolves --> Army
    ResourcePile --> Treasure
    Player --> Army : используется в CanBeat
    Player --> Treasure : используется в Consume

    %% Зависимости - КлассА ..> КлассБ
    Interaction ..> Player : параметр метода
    Interaction ..> Dwelling : проверка
    Interaction ..> Mine : проверка
    Interaction ..> Creeps : проверка
    Interaction ..> Wolves : проверка
    Interaction ..> ResourcePile : проверка
    Interaction ..> IOwner : приведение типа
    Interaction ..> IHaveArmy : приведение типа
    Interaction ..> IHaveTreasure : приведение типа
```