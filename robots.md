# Практика: Роботы

## 1. Описание предметной области и сущностей
В системе моделируется робот, который генерирует и выполняет команды перемещения (и при необходимости, стрельбы с укрытием).  
Части робота:
1. AI - генерирует команды (к примеру ShooterAI формирует команды стрельбы, BuilderAI - строительство).
2. Device - исполняет команды (Mover просто перемещает, ShooterMover перемещает с укрытием и т.д.).

Сущности:
- Robot - статический класс, фабрика, создающая экземпляры TemplateRobot<Command>.
- TemplateRobot<Command> - обобщённый класс, собственно робот, связывающий AI и Device.
- IRobot<out Command> - интерфейс, предоставляет метод получения команды. Является ковариантным.
- IDevice<in Command> - интерфейс, предоставляет метод выполнения команды. Является контрвариантным.
- RobotAI<Command> - абстрактный базовый класс для всех AI.
- ShooterAI - конкретный класс, генерирует команды IShooterMoveCommand.
- BuilderAI - конкретный класс, генерирует команды IMoveCommand.
- Device<Command> - абстрактный базовый класс, реализация девайсов с методами форматирования.
- Mover - конкретный класс, выполняет команды IMoveCommand (простое перемещение).
- ShooterMover - конкретный класс, выполняет команды IShooterMoveCommand.
- ShooterCommand - класс, реализация IShooterMoveCommand с фабрикой.
- BuilderCommand - класс, реализация IMoveCommand с фабрикой.
- Point - класс, хранит координаты X и Y.
- IMoveCommand - интерфейс, команда перемещения (содержит точку Point).
- IShooterMoveCommand - интерфейс, который расширяет IMoveCommand, добавляя так называемый ShouldHide.

Обобщённые интерфейсы с out или in позволяют использовать AI, возвращающие более конкретные команды, с Device, принимающими более общие команды и наоборот.

Базовый класс я вывел чтобы избежать дублирования кода, улучшить общую логику создания команд и их форматирования для всех наследников.
## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction TB

    class IRobot~Command~ {
        <<interface>>
        +Command GetCommand()
    }

    class IDevice~Command~ {
        <<interface>>
        +string ExecuteCommand(Command command)
    }

    class IMoveCommand {
        <<interface>>
        +Point Destination
    }

    class IShooterMoveCommand {
        <<interface>>
        +bool ShouldHide
    }

    class ShooterCommand {
        +Point Destination
        +bool Shoot
        +bool ShouldHide
        +ShooterCommand ForCounter(int counter)$
    }

    class BuilderCommand {
        +Point Destination
        +bool Build
        +BuilderCommand ForCounter(int counter)$
    }

    class Point {
        +double X
        +double Y
    }

    class RobotAI~Command~ {
        <<abstract>>
        #int counter
        +Command GetCommand()*
        #Command CreateCommand(Func~int,Command~ factory)
        #void ResetCounter()
    }

    class Device~Command~ {
        <<abstract>>
        +string ExecuteCommand(Command command)*
        #string FormatMoveCommand(IMoveCommand command)
        #string FormatShooterMoveCommand(IShooterMoveCommand command)
    }

    class ShooterAI {
        +IShooterMoveCommand GetCommand()
    }

    class BuilderAI {
        +IMoveCommand GetCommand()
    }

    class Mover {
        +string ExecuteCommand(IMoveCommand command)
    }

    class ShooterMover {
        +string ExecuteCommand(IShooterMoveCommand command)
    }

    class Robot {
        <<static>>
        +TemplateRobot~Command~ Create(IRobot~Command~ ai, IDevice~Command~ device)$
    }

    class TemplateRobot~Command~ {
        -IRobot~Command~ ai
        -IDevice~Command~ device
        +TemplateRobot(IRobot~Command~ ai, IDevice~Command~ device)
        +IEnumerable~string~ Start(int steps)
    }

    RobotAI~IShooterMoveCommand~ <|-- ShooterAI : наследуется
    RobotAI~IMoveCommand~ <|-- BuilderAI : наследуется
    Device~IMoveCommand~ <|-- Mover : наследуется
    Device~IShooterMoveCommand~ <|-- ShooterMover : наследуется
    IMoveCommand <|-- IShooterMoveCommand : наследуется

    IRobot~Command~ <|.. RobotAI~Command~ : реализует
    IDevice~Command~ <|.. Device~Command~ : реализует
    IShooterMoveCommand <|.. ShooterCommand : реализует
    IMoveCommand <|.. BuilderCommand : реализует

    TemplateRobot~Command~ o-- IRobot~Command~ : агрегирует
    TemplateRobot~Command~ o-- IDevice~Command~ : агрегирует

    ShooterCommand --> Point : ассоциация
    BuilderCommand --> Point : ассоциация

    Robot ..> TemplateRobot~Command~ : использует
    ShooterAI ..> ShooterCommand : использует
    BuilderAI ..> BuilderCommand : использует
    TemplateRobot~Command~ ..> IMoveCommand : использует
```