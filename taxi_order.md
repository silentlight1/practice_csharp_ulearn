# Практика: TaxiOrder

## 1. Описание предметной области и сущностей
В рамках задания выполнена переработка анемичной модели заказа такси в богатую(Rich) доменную модель, основанную на принципах DDD.

Сущности:
- TaxiOrder - сущность, управляющая циклом заказа. Хранит информацию о клиенте, маршруте (начальный адрес и пункт назначения), назначенном водителе, текущем статусе и временных метках ключевых событий. Предоставляет методы для безопасного изменения состояния с проверкой допустимости операций в каждом статусе (UpdateDestination, AssignDriver, UnassignDriver, Cancel, StartRide, FinishRide).
- Driver - сущность, представляющая водителя. Содержит идентификатор, имя (PersonName) и автомобиль (Car).
- PersonName, Address, Car - объекты-значения, неизменяемые и сравниваемые по всем свойствам.
- DriversRepository - репозиторий, отвечающий за получение сущности Driver по идентификатору.  
- TaxiApi - тонкий слой, реализующий интерфейс ITaxiApi<TaxiOrder>. Делегирует все бизнес операции методам самого заказа, сохраняя обратную совместимость с существующими тестами.

Взаимодействие строится через TaxiApi. клиент создаёт заказ, обновляет пункт назначения, назначает или отменяет водителя, запускает и завершает поездку. Все проверки на допустимость действий инкапсулированы внутри TaxiOrder. Это делает модель самодостаточной и устойчивой к некорректным вызовам.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction TD

    class Entity~TId~ {
        <<abstract>>
        +TId Id
        +Equals(object) bool
        +GetHashCode() int
        +ToString() string
    }

    class ValueType~T~ {
        <<abstract>>
        +Equals(object) bool
        +GetHashCode() int
        +ToString() string
    }

    class PersonName {
        +string FirstName
        +string LastName
        +PersonName(firstName, lastName)
    }

    class Address {
        +string Street
        +string Building
        +Address(street, building)
    }

    class Car {
        +string Model
        +string Color
        +string PlateNumber
        +Car(model, color, plateNumber)
    }

    class Driver {
        +PersonName Name
        +Car Car
        +Driver(id, name, car)
    }

    class TaxiOrder {
        +PersonName ClientName
        +Address Start
        +Address Destination
        +Driver Driver
        +TaxiOrderStatus Status
        +DateTime CreationTime
        +DateTime DriverAssignmentTime
        +DateTime CancelTime
        +DateTime StartRideTime
        +DateTime FinishRideTime
        +TaxiOrder(id, clientName, start, creationTime)
        +UpdateDestination(destination) void
        +AssignDriver(driver, assignmentTime) void
        +UnassignDriver() void
        +Cancel(cancelTime) void
        +StartRide(startTime) void
        +FinishRide(finishTime) void
        +GetDriverFullInfo() string
        +GetShortOrderInfo() string
    }

    class TaxiOrderStatus {
        <<enumeration>>
        WaitingForDriver
        WaitingCarArrival
        InProgress
        Finished
        Canceled
    }


    class DriversRepository {
        +GetDriver(driverId) Driver
    }

    class ITaxiApi~TOrder~ {
        <<interface>>
        +CreateOrderWithoutDestination(firstName, lastName, street, building) TOrder
        +UpdateDestination(order, street, building) void
        +AssignDriver(order, driverId) void
        +UnassignDriver(order) void
        +Cancel(order) void
        +StartRide(order) void
        +FinishRide(order) void
        +GetDriverFullInfo(order) string
        +GetShortOrderInfo(order) string
    }

    class TaxiApi {
        -DriversRepository _driversRepo
        -Func~DateTime~ _currentTime
        -int _idCounter
        +TaxiApi(driversRepo, currentTime)
        +CreateOrderWithoutDestination(firstName, lastName, street, building) TaxiOrder
        +UpdateDestination(order, street, building) void
        +AssignDriver(order, driverId) void
        +UnassignDriver(order) void
        +Cancel(order) void
        +StartRide(order) void
        +FinishRide(order) void
        +GetDriverFullInfo(order) string
        +GetShortOrderInfo(order) string
    }
    %% 1. Наслед
    ValueType~T~ <|-- PersonName : наследование
    ValueType~T~ <|-- Address : наследование
    ValueType~T~ <|-- Car : наследование
    Entity~TId~ <|-- Driver : наследование
    Entity~TId~ <|-- TaxiOrder : наследование

    %% 2. Реализация интерфейса
    ITaxiApi~TOrder~ <|.. TaxiApi : реализация

    %% 3. Композиция 
    TaxiOrder *-- PersonName : композиция(ClientName)
    TaxiOrder *-- Address : композиция(Start)
    TaxiOrder *-- Address : композиция(Destination)
    Driver *-- PersonName : композиция(Name)
    Driver *-- Car : композиция(Car)

    %% 4. Агрегация
    TaxiOrder o-- Driver : агрегация(Driver)

    %% 5. Ассоциация
    TaxiApi --> DriversRepository : ассоциация(_driversRepo)
    TaxiApi --> Func~DateTime~ : ассоциация(_currentTime)
    TaxiOrder --> TaxiOrderStatus : использует

    %% 6. Зависимость
    TaxiApi ..> TaxiOrder : создаёт, вызывает методы
    TaxiApi ..> PersonName : создаёт
    TaxiApi ..> Address : создаёт
    DriversRepository ..> Driver : создаёт
    DriversRepository ..> Car : создаёт
    DriversRepository ..> PersonName : создаёт

```