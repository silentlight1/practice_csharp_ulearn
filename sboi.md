# Практика: Сбои

## 1. Описание предметной области и сущностей
1. ReportMaker - класс, который фильтрует серьёзные сбои (UnexpectedShutdown, HardwareFailures) до указанной даты, в нём реализованы методы: устаревший(FindDevicesFailedBeforeDateObsolete) и новый (FindDevicesFailedBeforeDate)
2. Device - это устройство (Id, Name)
3. Failure - это сбои (DeviceId, Date, Type)
4. FailureType - перечисленные типы сбоя (UnexpectedShutdown, ShortNonResponding, HardwareFailures, ConnectionProblems)
5. Common - старый класс, который не используется в коде, но он хранит в себе логику для преобразований, поэтому было решено его оставить.
## 2. Диаграмма классов (Mermaid)


```mermaid
classDiagram
    direction TB
    
    class ReportMaker {
        +List~string~ FindDevicesFailedBeforeDateObsolete(int day, int month, int year, int[] failureTypes, int[] deviceId, object[][] times, List~Dictionary~ devices)

        +List~string~ FindDevicesFailedBeforeDate(DateTime betterDate, Failure[] failures, Device[] devices)
    }

    class Device {
        +int Id
        +string Name
        +Device(int id, string name)
    }

     class FailureType {
        <<enum>>
        UnexpectedShutdown
        ShortNonResponding
        HardwareFailures
        ConnectionProblems
    }
    
    class Failure {
        +int DeviceId
        +DateTime Date
        +FailureType TypeFal
        +Failure(int deviceid, DateTime date, FailureType typefal)
        +bool IsSerious()
        +bool IsHappenEarlier(DateTime date)
    }  

    Failure --> FailureType : ассоциация(поле класса)
    
    ReportMaker ..> Failure : зависимость(параметр метода)
    
    ReportMaker ..> Device : зависимость(параметр метода)
    
```