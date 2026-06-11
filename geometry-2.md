# Практика: Геометрия-2

## 1. Описание предметной области и сущностей
В данной работе был использован паттерн Visitor
- IVisitor<TypeResult> - интерфейс посетителя, реализация паттерна Visitor. Имеет метод Visit для каждого типа фигуры.
- Body - абстрактный класс базовая фигура. Содержит позицию (Position) и метод Accept, который принимает посетителя.
- Ball - класс шар с радиусом Radius.
- RectangularCuboid - класс прямоугольный параллелепипед с трёхмерными размерами SizeX, SizeY, SizeZ.
- Cylinder - класс цилиндр с высотой SizeZ и радиусом Radius.
- CompoundBody - класс "составное тело", содержит список частей (Parts).
- BoxingVisitorBase<TypeResult> - абстрактный базовый класс для посетителей, который превращают фигуры в параллелепипеды. Содержит общие методы GetBoundingBox для шара, цилиндра и параллелепипеда. Создан для избегания дублирования кода.
- BoundingBoxVisitor - класс, который вычисляет минимальный ограничивающий параллелепипед для составного тела.
- BoxifyVisitor - класс, который заменяет каждую фигуру в составном теле на её ограничивающий параллелепипед, сохраняя структуру.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    %% Диаграмма слева-направо для лучшей видимости
    direction LR

    class IVisitor~TypeResult~ {
        <<interface>>
        +Visit(Ball ball) TypeResult
        +Visit(RectangularCuboid rectangularCuboid) TypeResult
        +Visit(Cylinder cylinder) TypeResult
        +Visit(CompoundBody compoundBody) TypeResult
    }

    %% Базовый класс для отсутствия дублирования в коде
    class BoxingVisitorBase~TypeResult~ {
        <<abstract>>
        #GetBoundingBox(Ball ball) RectangularCuboid
        #GetBoundingBox(RectangularCuboid box) RectangularCuboid
        #GetBoundingBox(Cylinder cylinder) RectangularCuboid
        +Visit(Ball ball) TypeResult*
        +Visit(RectangularCuboid rectangularCuboid) TypeResult*
        +Visit(Cylinder cylinder) TypeResult*
        +Visit(CompoundBody compoundBody) TypeResult*
    }

    class BoundingBoxVisitor {
        +Visit(Ball ball) RectangularCuboid
        +Visit(RectangularCuboid recCub) RectangularCuboid
        +Visit(Cylinder cylinder) RectangularCuboid
        +Visit(CompoundBody compBody) RectangularCuboid
    }

    class BoxifyVisitor {
        +Visit(Ball ball) Body
        +Visit(RectangularCuboid recCub) Body
        +Visit(Cylinder cylinder) Body
        +Visit(CompoundBody compBody) Body
    }

    class Body {
        <<abstract>>
        +Vector3 Position
        +Accept(IVisitor~TypeResult~ visitor) TypeResult*
    }

    class Ball {
        +double Radius
        +Accept(IVisitor~TypeResult~ visitor) TypeResult
    }

    class RectangularCuboid {
        +double SizeX
        +double SizeY
        +double SizeZ
        +Accept(IVisitor~TypeResult~ visitor) TypeResult
    }

    class Cylinder {
        +double SizeZ
        +double Radius
        +Accept(IVisitor~TypeResult~ visitor) TypeResult
    }

    class CompoundBody {
        +IReadOnlyList~Body~ Parts
        +Accept(IVisitor~TypeResult~ visitor) TypeResult
    }

    IVisitor~TypeResult~ <|.. BoxingVisitorBase~TypeResult~ : реализация
    BoxingVisitorBase~TypeResult~ <|-- BoundingBoxVisitor : наследование
    BoxingVisitorBase~TypeResult~ <|-- BoxifyVisitor : наследование

    Body <|-- Ball : наследование
    Body <|-- RectangularCuboid : наследование
    Body <|-- Cylinder : наследование
    Body <|-- CompoundBody : наследование

    BoxingVisitorBase~TypeResult~ ..> RectangularCuboid : зависимость
    BoundingBoxVisitor ..> RectangularCuboid : зависимость
    BoxifyVisitor ..> Body : зависимость
    Body ..> IVisitor~TypeResult~ : зависимость
    CompoundBody *-- Body : композиция
```