# Регистрация модели потока

## Начните с пользовательской модели потока

```tsx pure
class HelloModel extends FlowModel {
  render() {
    return (
      <div>
        <h1>Hello, NocoBase!</h1>
        <p>This is a simple block rendered by HelloModel.</p>
      </div>
    );
  }
}
```

## Доступные базовые классы модели потока (FlowModel)

| Имя базового класса | Описание |
| ------------------- | -------- |
| `BlockModel` | Базовый класс для всех блоков |
| `CollectionBlockModel` | Блок коллекции, наследуется от `BlockModel` |
| `ActionModel` | Базовый класс для всех действий |

## Зарегистрируйте модель потока

```ts
export class PluginHelloClient extends Plugin {
  async load() {
    this.engine.registerModels({ HelloModel });
  }
}
```