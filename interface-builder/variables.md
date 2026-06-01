# Переменные

## Введение

Переменные — это набор токенов, используемых для идентификации значения в текущем контексте. Их можно использовать в сценариях вроде настройки области данных блоков, значений полей по умолчанию, правил связывания и рабочего процесса.

![20251030114458](https://static-docs.nocobase.com/20251030114458.png)

## Поддерживаемые переменные

### Текущий пользователь

Представляет данные текущего авторизованного пользователя.

![20240416154950](https://static-docs.nocobase.com/20240416154950.png)

### Текущая роль

Представляет идентификатор роли (имя роли) текущего авторизованного пользователя.

![20240416155100](https://static-docs.nocobase.com/20240416155100.png)

### Текущая форма

Значения текущей формы. Используется только в блоках формы. Сценарии применения:

- правила связывания для текущей формы;
- значения по умолчанию для полей формы (действуют только при добавлении новых данных);
- настройка области данных для полей связи;
- настройка присвоения значения полю для действий отправки.

#### Правила связывания для текущей формы

![20251027114920](https://static-docs.nocobase.com/20251027114920.png)

#### Значения по умолчанию для полей формы (только форма добавления)

![20251027115016](https://static-docs.nocobase.com/20251027115016.png)

<!-- 

![20240416171129_rec_](https://static-docs.nocobase.com/20240416171129_rec_.gif)

 -->

#### Настройка области данных для полей связи

Используется для динамической фильтрации опций нижележащего поля на основе вышележащего поля, обеспечивая точный ввод данных.

**Пример:**

1. Пользователь выбирает значение для поля **Владелец**.
2. Система автоматически фильтрует опции для поля **Аккаунт** на основе **Имени пользователя** выбранного **Владельца**.

![20251030151928](https://static-docs.nocobase.com/20251030151928.png)

<!-- 

![20240416171743_rec_](https://static-docs.nocobase.com/20240416171743_rec_.gif)

 -->

<!-- #### Field value assignment configuration for submit actions


![20240416171215_rec_](https://static-docs.nocobase.com/20240416171215_rec_.gif)

 -->

<!-- ### Current Object

Currently used only for field configuration in sub-forms and sub-tables within a form block, representing the value of each item:

- Default value for sub-fields
- Data scope for sub-association fields

#### Default value for sub-fields


![20240416172933_rec_](https://static-docs.nocobase.com/20240416172933_rec_.gif)


#### Data scope for sub-association fields


![20240416173043_rec_](https://static-docs.nocobase.com/20240416173043_rec_.gif)

 -->

<!-- ### Parent Object

Similar to "Current Object", it represents the parent object of the current object. Supported in NocoBase v1.3.34-beta and above. -->

### Текущая запись

Запись — это строка в коллекции, где каждая строка представляет отдельную запись. Переменная «Текущая запись» доступна в **правилах связывания для действий строки** в блоках отображения.

Пример: отключить кнопку удаления для документов со статусом "Оплачено".

![20251027120217](https://static-docs.nocobase.com/20251027120217.png)

### Текущая запись во всплывающем окне

Действия всплывающих окон играют очень важную роль в настройке интерфейса NocoBase.

- Всплывающее окно для действий строки: каждое всплывающее окно имеет переменную «Текущая запись всплывающего окна», представляющую текущую запись строки.
- Всплывающее окно для полей связи: каждое всплывающее окно имеет переменную «Текущая запись всплывающего окна», представляющую текущую выбранную запись связи.

Блоки внутри всплывающего окна могут использовать переменную «Текущая запись всплывающего окна». Связанные сценарии:

- настройка области данных блока;
- настройка области данных поля связи;
- настройка значений полей по умолчанию (в форме добавления новых данных);
- настройка правил связывания для действий.

<!-- #### Configuring the data scope of a block


![20251027151107](https://static-docs.nocobase.com/20251027151107.png)


#### Configuring the data scope of an association field


![20240416224641_rec_](https://static-docs.nocobase.com/20240416224641_rec_.gif)


#### Configuring default values for fields (in a form for adding new data)


![20240416223846_rec_](https://static-docs.nocobase.com/20240416223846_rec_.gif)


#### Configuring linkage rules for actions


![20240416223101_rec_](https://static-docs.nocobase.com/20240416223101_rec_.gif)


<!--
#### Field value assignment configuration for form submit actions


![20240416224014_rec_](https://static-docs.nocobase.com/20240416224014_rec_.gif)

 -->

<!-- ### Selected Table Records

Currently used only for the default value of form fields in the Add record action of a table block

#### Default value of form fields for the Add record action -->

<!-- ### Parent Record (Deprecated)

Used only in association blocks, representing the source record of the association data.

:::warning
"Parent Record" is deprecated. It is recommended to use the equivalent "Current Popup Record" instead.
::::

<!-- ### Date Variables

Date variables are dynamically parsable date placeholders that can be used in the system to set data scopes for blocks, data scopes for association fields, date conditions in action linkage rules, and default values for date fields. The parsing method of date variables varies depending on the use case: in assignment scenarios (such as setting default values), they are parsed into specific moments in time; in filtering scenarios (such as data scope conditions), they are parsed into time period ranges to support more flexible filtering.

#### Filtering Scenarios

Related use cases include:

- Setting date field conditions for block data scopes
- Setting date field conditions for association field data scopes
- Setting date field conditions for action linkage rules


![20250522211606](https://static-docs.nocobase.com/20250522211606.png)


Related variables include:

- Current time
- Yesterday
- Today
- Tomorrow
- Last week
- This week
- Next week
- Last month
- This month
- Next month
- Last quarter
- This quarter
- Next quarter
- Last year
- This year
- Next year
- Last 7 days
- Next 7 days
- Last 30 days
- Next 30 days
- Last 90 days
- Next 90 days

#### Сценарии присваивания

В сценариях присваивания одна и та же переменная даты автоматически разбирается в разные форматы в зависимости от типа целевого поля. Например, при использовании переменной «Сегодня» для присваивания значения разным типам полей даты:

- Для полей Timestamp и DateTime with timezone переменная разбирается в полную строку времени UTC, например 2024-04-20T16:00:00.000Z. Такой формат включает информацию о часовом поясе и подходит для задач синхронизации между часовыми поясами.

- Для полей DateTime without timezone переменная разбирается в строку локального времени, например 2025-04-21 00:00:00, без информации о часовом поясе, что больше подходит для локальной бизнес-логики.

- Для полей DateOnly переменная разбирается в строку только с датой, например 2025-04-21, содержащую только год, месяц и день без времени.

Система интеллектуально разбирает переменную в зависимости от типа поля, обеспечивая корректный формат при присваивании и предотвращая ошибки данных или исключения из-за несовпадения типов.


![20250522212802](https://static-docs.nocobase.com/20250522212802.png)


Связанные сценарии использования включают:

- Установка значений по умолчанию для полей даты в блоках формы
- Установка атрибута value для полей даты в правилах связывания
- Присваивание значений полям даты в кнопках отправки

Связанные переменные включают:

- Сейчас
- Вчера
- Сегодня
- Завтра -->

### Параметры запроса URL

Эта переменная представляет параметры запроса в URL текущей страницы. Она доступна только если в URL страницы есть строка запроса. Удобнее всего использовать ее вместе с [действием ссылки](/interface-builder/actions/types/link).

![20251027173017](https://static-docs.nocobase.com/20251027173017.png)

![20251027173121](https://static-docs.nocobase.com/20251027173121.png)

### API-токен

Значение этой переменной — строка, являющаяся учетными данными для доступа к NocoBase API. Ее можно использовать для проверки личности пользователя.

### Текущий тип устройства

Пример: не отображать действие «Печать шаблона» на устройствах, отличных от настольных компьютеров.

![20251029215303](https://static-docs.nocobase.com/20251029215303.png)

