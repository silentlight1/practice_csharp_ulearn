# Практика: TaxiOrder

## 1. Описание предметной области и сущностей
ВВ рамках задания выполнена переработка анемичной модели заказа такси в богатую (Rich) доменную модель, основанную на принципах DDD.

Базовые инфраструктурные классы:

Entity<TId> - базовый абстрактный класс для всех DDD-сущностей. Обеспечивает уникальную идентификацию через свойство Id и сравнение сущностей по идентификатору.

ValueType<T> - базовый абстрактный класс для всех объектов-значений. Обеспечивает сравнение объектов по содержимому (Equals, GetHashCode) и автоматическое формирование строкового представления на основе публичных свойств. 

Объекты-значения:

PersonName - представляет имя человека. Содержит свойства FirstName и LastName. Неизменяем.

Address - представляет адрес. Содержит свойства Street и Building. Неизменяем.

Car - описывает автомобиль. Содержит свойства Model, Color, PlateNumber. Неизменяем.

Сущности:

Driver - сущность, представляющая водителя. Имеет идентификатор (Id), имя (PersonName) и автомобиль (Car).

TaxiOrder - агрегатная сущность, управляющая жизненным циклом заказа такси. Хранит информацию о клиенте (ClientName), начальном адресе (Start), пункте назначения (Destination), назначенном водителе (Driver), текущем статусе (Status) и временных метках ключевых событий (CreationTime, DriverAssignmentTime, StartRideTime, FinishRideTime, CancelTime).
Предоставляет методы для проверки допустимости переходов (IsDriverAssignable(), IsCancelable(), IsRideStartable(), IsRideFinishable(), CanTransitionTo(TaxiOrderStatus)) и команды для изменения состояния (Cancel, StartRide, FinishRide, AssignDriver, UnassignDriver, UpdateDestination).
Добавлено вспомогательное свойство IsActive, сигнализирующее о том, что заказ не завершён и не отменён.
Внутренняя логика статусов и временных меток вынесена в отдельный класс OrderLifecycle для удобства.

OrderLifecycle - вспомогательный класс, инкапсулирующий управление статусами и временными метками заказа. Хранит словарь _milestones с временем перехода в каждый статус и обеспечивает проверку допустимости переходов через метод IsTransitionAllowed. Используется внутри TaxiOrder для централизованного контроля жизненного цикла.

Репозиторий:

DriversRepository - репозиторий, отвечающий за получение сущности Driver по идентификатору. Не зависит от заказов, работает только с данными о водителях. Для повышения производительности использует внутренний кэш (_cache).

API:

ITaxiApi<TOrder> - интерфейс, определяющий контракт для управления заказами такси. Содержит методы создания заказа, обновления пункта назначения, назначения и отмены водителя, отмены заказа, начала и завершения поездки, а также получения краткой информации и полной информации о водителе.
Порядок методов в интерфейсе изменён для улучшения читаемости и семантической группировки.

TaxiApi - реализация интерфейса ITaxiApi<TaxiOrder>. Является тонким слоем, который преобразует входные данные в объекты-значения (PersonName, Address) и делегирует все бизнес-операции методам TaxiOrder. Сохраняет обратную совместимость с существующими тестами. Использует DriversRepository и внешний источник времени (Func<DateTime>).

Enum(Перечисление):

TaxiOrderStatus - определяет возможные состояния заказа:
WaitingForDriver - ожидание водителя,
WaitingCarArrival - водитель назначен, ожидание прибытия,
InProgress - поездка начата,
Finished - поездка завершена,
Canceled - заказ отменён.


Взаимодействие строится через TaxiApi. Клиент создаёт заказ, обновляет пункт назначения, назначает или отменяет водителя, запускает и завершает поездку. Все проверки на допустимость действий инкапсулированы внутри TaxiOrder (с использованием OrderLifecycle). Это делает модель самодостаточной, устойчивой к некорректным вызовам и легко тестируемой.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction TB

    class ValueType~T~ {
        <<abstract>>
        +Equals(object) bool
        +GetHashCode() int
        +ToString() string
        +Equals(T) bool
    }

    class Entity~TId~ {
        <<abstract>>
        +TId Id
        +Equals(object) bool
        +GetHashCode() int
        +ToString() string
    }

    class ITaxiApi~TOrder~ {
        <<interface>>
        +CreateOrderWithoutDestination(firstName, lastName, street, building) TOrder
        +Cancel(order) void
        +StartRide(order) void
        +FinishRide(order) void
        +AssignDriver(order, driverId) void
        +UnassignDriver(order) void
        +UpdateDestination(order, street, building) void
        +GetShortOrderInfo(order) string
        +GetDriverFullInfo(order) string
    }

    class TaxiApi {
        -DriversRepository _driverRepo
        -Func~DateTime~ _clock
        -int _orderIdCounter
        +TaxiApi(driverRepo, clock)
        +CreateOrderWithoutDestination(...) TaxiOrder
        +Cancel(order) void
        +StartRide(order) void
        +FinishRide(order) void
        +AssignDriver(order, driverId) void
        +UnassignDriver(order) void
        +UpdateDestination(order, street, building) void
        +GetShortOrderInfo(order) string
        +GetDriverFullInfo(order) string
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
        +Car Car
        +PersonName Name
        +Driver(id, name, car)
    }

    class OrderLifecycle {
        -Dictionary~TaxiOrderStatus, DateTime~ _milestones
        -TaxiOrderStatus _currentStatus
        +OrderLifecycle(creationTime)
        +CurrentStatus TaxiOrderStatus
        +GetTime(status) DateTime
        +SetStatus(newStatus, timestamp) void
        +IsTransitionAllowed(newStatus) bool
    }

    class TaxiOrder {
        -OrderLifecycle _lifecycle
        -Driver _assignedDriver
        +PersonName ClientName
        +Address Start
        +Address Destination
        +Driver Driver
        +TaxiOrderStatus Status
        +DateTime CreationTime
        +DateTime CancelTime
        +DateTime DriverAssignmentTime
        +DateTime StartRideTime
        +DateTime FinishRideTime
        +bool IsActive
        +TaxiOrder(id, clientName, start, creationTime)
        +AssignDriver(driverEntity, assignedAt) void
        +UnassignDriver() void
        +UpdateDestination(destination) void
        +Cancel(cancelTime) void
        +StartRide(startTime) void
        +FinishRide(finishTime) void
        +IsDriverAssignable() bool
        +IsCancelable() bool
        +IsRideStartable() bool
        +IsRideFinishable() bool
        +CanTransitionTo(newStatus) bool
        +GetShortOrderInfo() string
        +GetDriverFullInfo() string
        -GetLastProgressTime() DateTime
        -FormatPerson(name) string
        -FormatLocation(address) string
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
        -Dictionary~int, Driver~ _cache
        +GetDriver(driverId) Driver
    }

    %% 1. Наследование
    ValueType~T~ <|-- PersonName : наследование
    ValueType~T~ <|-- Address : наследование
    ValueType~T~ <|-- Car : наследование
    Entity~TId~ <|-- Driver : наследование
    Entity~TId~ <|-- TaxiOrder : наследование

    %% 2. Реализация интерфейса
    ITaxiApi~TOrder~ <|.. TaxiApi : реализация

    %% 3. Композиция
    TaxiOrder *-- OrderLifecycle : композиция (_lifecycle)
    TaxiOrder *-- PersonName : композиция (ClientName)
    TaxiOrder *-- Address : композиция (Start)
    TaxiOrder *-- Address : композиция (Destination)
    Driver *-- PersonName : композиция (Name)
    Driver *-- Car : композиция (Car)

    %% 4. Агрегация
    TaxiOrder o-- Driver : агрегация (_assignedDriver)

    %% 5. Ассоциация
    TaxiApi --> DriversRepository : ассоциация (_driverRepo)
    TaxiApi --> Func~DateTime~ : ассоциация (_clock)
    TaxiOrder --> TaxiOrderStatus : ассоциация (использует)

    %% 6. Зависимость
    TaxiApi ..> TaxiOrder : зависимость (создаёт, вызывает)
    TaxiApi ..> PersonName : зависимость (создаёт)
    TaxiApi ..> Address : зависимость (создаёт)
    DriversRepository ..> Driver : зависимость (создаёт)
    DriversRepository ..> Car : зависимость (создаёт)
    DriversRepository ..> PersonName : зависимость (создаёт)
```