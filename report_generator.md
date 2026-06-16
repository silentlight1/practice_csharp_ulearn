# Практика: Генератор отчётов

## 1. Описание предметной области и сущностей
Система генерирует статистические отчёты по данным погодных измерений, вычисляя различные показатели (среднее, стандартное отклонение, медиана) и выводя их в разных форматах (HTML, Markdown).

Measurement - исходные данные: температура и влажность

MeanAndStd - результат статистики: среднее и стандартное отклонение, форматируется ввиде "среднее +- отклонение"

ReportBuilder - отвечает за визуальное оформление отчёта, использует делегаты для гибкого форматирования, поддерживает HTML, Markdown и другие форматы

StatsProcessor - вычисляет статистические показатели, поддерживает различные алгоритмы (среднее+отклонение, медиана), содержит название статистики для отчёта

ReportGenerator - координирует процесс создания отчёта, комбинирует построитель и обработчик, формирует итоговый отчёт из набора измерений

ReportComponentsFactory - создаёт готовые компоненты для отчётов, централизует создание построителей и обработчиков

ReportMakerHelper - предоставляет простой API для создания отчётов, содержит четыре стандартных метода создания отчётов

Реализация состоит из двух частей: ReportBuilder - визуальная составляющая и StatsProcessor для статистических расчётов. Они объединяются в ReportGenerator.

## 2. Диаграмма классов (Mermaid)

```mermaid
classDiagram
    direction TB
    class ProcessedData {
        -string Label
        -object Value
        +ProcessedData(string, object)
        +Label string
        +Value object
    }

    class ReportBuilder {
        -Func~string, string~ _buildHeader
        -Func~string~ _buildPrefix
        -Func~string, string, string~ _buildEntry
        -Func~string~ _buildSuffix
        +ReportBuilder(Func~string, string~, Func~string~, Func~string, string, string~, Func~string~)
        +BuildHeader(string) string
        +BuildPrefix() string
        +BuildEntry(string, string) string
        +BuildSuffix() string
    }

    class StatsProcessor {
        -string _title
        -Func~IEnumerable~double~, object~ _process
        +StatsProcessor(string, Func~IEnumerable~double~, object~)
        +Title string
        +Process(IEnumerable~double~) object
    }

    class ReportGenerator {
        -ReportBuilder _builder
        -StatsProcessor _processor
        +ReportGenerator(ReportBuilder, StatsProcessor)
        +Generate(IEnumerable~Measurement~) string
    }

    class ReportComponentsFactory {
        <<static>>
        +CreateHtmlBuilder() ReportBuilder
        +CreateMarkdownBuilder() ReportBuilder
        +CreateMeanAndStdProcessor() StatsProcessor
        +CreateMedianProcessor() StatsProcessor
    }

    class ReportMakerHelper {
        <<static>>
        +MeanAndStdHtmlReport(IEnumerable~Measurement~) string
        +MedianMarkdownReport(IEnumerable~Measurement~) string
        +MeanAndStdMarkdownReport(IEnumerable~Measurement~) string
        +MedianHtmlReport(IEnumerable~Measurement~) string
    }

    class Measurement {
        +double Temperature
        +double Humidity
    }

    class MeanAndStd {
        +double Mean
        +double Std
        +ToString() string
    }

    ReportGenerator o-- ReportBuilder : агрегация
    ReportGenerator o-- StatsProcessor : агрегация
    
    StatsProcessor ..> MeanAndStd : экземпляр в Process
    
    ReportGenerator ..> Measurement : параметр в Generate
    
    ReportComponentsFactory ..> ReportBuilder : создаёт экземпляры
    ReportComponentsFactory ..> StatsProcessor : создаёт экземпляры
    
    ReportMakerHelper ..> ReportGenerator : создаёт экземпляр
    ReportMakerHelper ..> ReportComponentsFactory : вызывает методы
    ReportMakerHelper ..> Measurement : принимает как параметр
```