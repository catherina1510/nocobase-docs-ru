# RelationRepository - Репозиторий отношений

`RelationRepository` — это объект `Repository` для типов ассоциации. `RelationRepository` позволяет работать со связанными данными без загрузки ассоциации. На основе `RelationRepository` каждый тип ассоциации имеет соответствующую производную реализацию:

- `HasOneRepository`
- `HasManyRepository`
- `BelongsToRepository`
- `BelongsToManyRepository`

## Конструктор

**Сигнатура**

- `constructor(sourceCollection: Collection, association: string, sourceKeyValue: string | number)`

**Параметры**

| Имя параметра | Тип | Значение по умолчанию | Описание |
| :--- | :--- | :--- | :--- |
| `sourceCollection` | `Collection` | - | `Collection`, соответствующий ссылочному отношению в ассоциации |
| `association` | `string` | - | Название ассоциации |
| `sourceKeyValue` | `string \| number` | - | Соответствующее ключевое значение в ссылочном отношении |

## Свойства базового класса

### `db: Database`

Объект базы данных

### `sourceCollection`

`Collection`, соответствующий ссылочному отношению в ассоциации.

### `targetCollection`

`Collection`, соответствующий указанному отношению в ассоциации.

### `association`

Объект ассоциации в продолжении, соответствующий текущей ассоциации.

### `associationField`

Поле в коллекции, соответствующее текущей ассоциации.

### `sourceKeyValue`

Соответствующее ключевое значение в ссылочном отношении