# Практика: TaxiOrder

## 1. Описание предметной области и сущностей
В рамках задания выполнена переработка анемичной модели заказа такси в богатую (Rich) доменную модель, основанную на принципах DDD.

Базовые инфраструктурные классы:

Entity<TId> - базовый абстрактный класс для всех DDD-сущностей. Обеспечивает уникальную идентификацию через свойство Id и сравнение сущностей по идентификатору. Добавлен защищённый метод HasSameIdentity для централизованной проверки идентичности.

ValueType<T> - базовый абстрактный класс для всех объектов-значений. Обеспечивает сравнение объектов по содержимому (переопределение Equals/GetHashCode) и автоматическое формирование строкового представления на основе публичных свойств. Содержит защищённый метод AreValuesEqual для унифицированного сравнения.

Объекты-значения:

PersonName - представляет имя человека. Содержит свойства FirstName и LastName. Неизменяем.

Address - представляет адрес. Содержит свойства Street и Building. Неизменяем.

Car - описывает автомобиль. Содержит свойства Model, Color, PlateNumber. Неизменяем.

Сущности:

Driver - сущность, представляющая водителя. Имеет идентификатор (Id), имя (PersonName) и автомобиль (Car).

TaxiOrder - агрегатная сущность, управляющая жизненным циклом заказа такси. Хранит информацию о клиенте (ClientName), начальном адресе (Start), пункте назначения (Destination), назначенном водителе (Driver), текущем статусе (Status) и временных метках ключевых событий (CreationTime, DriverAssignmentTime, StartRideTime, FinishRideTime, CancelTime).
Предоставляет методы для проверки допустимости переходов (IsDriverAssignable(), IsCancelable(), IsRideStartable(), IsRideFinishable()) и команды для изменения состояния (Cancel, StartRide, FinishRide, AssignDriver, UnassignDriver, UpdateDestination).
Добавлено вспомогательное свойство IsActive, сигнализирующее о том, что заказ не завершён и не отменён.
Внутренняя логика статусов и временных меток вынесена в иерархию состояний (паттерн «Состояние»).

Паттерн «Состояние»:

IOrderState - интерфейс, определяющий контракт для всех состояний заказа. Содержит свойства для проверки допустимости действий и методы для переходов между состояниями.

OrderStateBase - абстрактный базовый класс, предоставляющий стандартную реализацию для всех состояний.

Конкретные состояния: WaitingForDriverState, WaitingCarArrivalState, InProgressState, FinishedState, CanceledState. Каждое состояние определяет, какие действия разрешены, и управляет переходом к следующему состоянию.

Репозиторий:

DriversRepository - репозиторий, отвечающий за получение сущности Driver по идентификатору. Не зависит от заказов, работает только с данными о водителях. Для повышения производительности использует внутренний кэш (_cache).

API слой:

ITaxiApi<TOrder> - интерфейс, определяющий контракт для управления заказами такси. Содержит методы создания заказа, обновления пункта назначения, назначения и отмены водителя, отмены заказа, начала и завершения поездки, а также получения краткой информации и полной информации о водителе.
Порядок методов в интерфейсе изменён для улучшения читаемости и семантической группировки.

TaxiApi - реализация интерфейса ITaxiApi<TaxiOrder>. Является тонким слоем, который преобразует входные данные в объекты-значения (PersonName, Address) и делегирует все бизнес-операции методам TaxiOrder. Сохраняет обратную совместимость с существующими тестами. Использует DriversRepository и внешний источник времени (Func<DateTime>).

Перечисление:

TaxiOrderStatus - определяет возможные состояния заказа:
WaitingForDriver - ожидание водителя,
WaitingCarArrival - водитель назначен, ожидание прибытия,
InProgress - поездка начата,
Finished - поездка завершена,
Canceled - заказ отменён.

Взаимодействие:
Взаимодействие строится через TaxiApi. Клиент создаёт заказ, обновляет пункт назначения, назначает или отменяет водителя, запускает и завершает поездку. Все проверки на допустимость действий инкапсулированы внутри TaxiOrder (с использованием паттерна «Состояние»). Это делает модель самодостаточной, устойчивой к некорректным вызовам и легко тестируемой.
## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction TB

    class ValueType~T~ {
        <<abstract>>
        +Equals(object) bool
        +Equals(T) bool
        +GetHashCode() int
        +ToString() string
        #AreValuesEqual(T) bool
    }

    class Entity~TId~ {
        <<abstract>>
        +TId Id
        +Equals(object) bool
        +GetHashCode() int
        +ToString() string
        #HasSameIdentity(Entity~TId~) bool
        #Equals(Entity~TId~) bool
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

    class IOrderState {
        <<interface>>
        +Status TaxiOrderStatus
        +CanAssignDriver bool
        +CanCancel bool
        +CanStartRide bool
        +CanFinishRide bool
        +AssignDriver(driver, time) IOrderState
        +UnassignDriver(time) IOrderState
        +Cancel(time) IOrderState
        +StartRide(time) IOrderState
        +FinishRide(time) IOrderState
        +GetLastProgressTime(order) DateTime
    }

    class OrderStateBase {
        <<abstract>>
        +Status TaxiOrderStatus
        +CanAssignDriver bool
        +CanCancel bool
        +CanStartRide bool
        +CanFinishRide bool
        +AssignDriver(driver, time) IOrderState
        +UnassignDriver(time) IOrderState
        +Cancel(time) IOrderState
        +StartRide(time) IOrderState
        +FinishRide(time) IOrderState
        +GetLastProgressTime(order) DateTime
    }

    class WaitingForDriverState {
        +Status WaitingForDriver
        +CanAssignDriver true
        +CanCancel true
        +AssignDriver(driver, time) IOrderState
        +Cancel(time) IOrderState
    }

    class WaitingCarArrivalState {
        +Driver Driver
        +DateTime AssignedAt
        +Status WaitingCarArrival
        +CanCancel true
        +CanStartRide true
        +UnassignDriver(time) IOrderState
        +Cancel(time) IOrderState
        +StartRide(time) IOrderState
        +GetLastProgressTime(order) DateTime
    }

    class InProgressState {
        +DateTime StartedAt
        +Status InProgress
        +CanFinishRide true
        +FinishRide(time) IOrderState
        +GetLastProgressTime(order) DateTime
    }

    class FinishedState {
        +DateTime FinishedAt
        +Status Finished
        +GetLastProgressTime(order) DateTime
    }

    class CanceledState {
        +DateTime CanceledAt
        +Status Canceled
        +GetLastProgressTime(order) DateTime
    }

    class TaxiOrder {
        -IOrderState _state
        -Driver _assignedDriver
        +PersonName ClientName
        +Address Start
        +Address Destination
        +TaxiOrderStatus Status
        +DateTime CreationTime
        +DateTime DriverAssignmentTime
        +DateTime StartRideTime
        +DateTime FinishRideTime
        +DateTime CancelTime
        +Driver Driver
        +bool IsActive
        +TaxiOrder(id, clientName, start, creationTime)
        +AssignDriver(driverEntity, assignedAt) void
        +UnassignDriver() void
        +Cancel(cancelTime) void
        +StartRide(startTime) void
        +FinishRide(finishTime) void
        +UpdateDestination(destination) void
        +GetShortOrderInfo() string
        +GetDriverFullInfo() string
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

    %% 1. Наследование
    OrderStateBase <|-- WaitingForDriverState
    OrderStateBase <|-- WaitingCarArrivalState
    OrderStateBase <|-- InProgressState
    OrderStateBase <|-- FinishedState
    OrderStateBase <|-- CanceledState
    ValueType~T~ <|-- PersonName
    ValueType~T~ <|-- Address
    ValueType~T~ <|-- Car
    Entity~TId~ <|-- Driver
    Entity~TId~ <|-- TaxiOrder

    %% 2. Реализация интерфейса
    IOrderState <|.. OrderStateBase
    ITaxiApi~TOrder~ <|.. TaxiApi

    %% 3. Композиция
    TaxiOrder *-- IOrderState : состояние
    TaxiOrder *-- PersonName : имя клиента
    TaxiOrder *-- Address : начало пути
    TaxiOrder *-- Address : место назначения
    Driver *-- PersonName : имя
    Driver *-- Car : машина

    %% 4. Агрегация
    TaxiOrder o-- Driver : назначенный водитель

    %% 5. Ассоциация
    TaxiApi --> DriversRepository : ссылка _driverRepo
    TaxiApi --> Func~DateTime~ : ссылка _clock

    %% 6. Зависимость
    TaxiApi ..> TaxiOrder : зависимость
    TaxiApi ..> PersonName : зависимость
    TaxiApi ..> Address : зависимость
    DriversRepository ..> Driver : создаёт
    DriversRepository ..> Car : создаёт
    DriversRepository ..> PersonName : создаёт
    TaxiOrder ..> TaxiOrderStatus : использует
    TaxiOrder ..> IOrderState : делегирует
```