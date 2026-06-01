# Добавление действия с помощью модального окна

## Сценарий

В NocoBase во многих местах интерфейса есть кнопка **настройки действий** для добавления кнопок действий в интерфейс.

![img_v3_02b4_51f4918f-d344-43b2-b19e-48dca709467g](https://static-docs.nocobase.com/img_v3_02b4_51f4918f-d344-43b2-b19e-48dca709467g.jpg)

Если существующие кнопки действий не соответствуют нашим требованиям, нужно добавить подпункты в существующее меню **настройки действий**, чтобы появились новые кнопки действий.

## Пример

В этом примере создаётся кнопка, по нажатию открывающая модальное окно. Содержимым окна будет встроенный фрейм с документацией соответствующего блока. Эту кнопку добавляют в меню **настройки действий** в блоках **таблицы**, **деталей** и **формы**.

Полный пример кода к этой статье находится в [репозитории примеров плагинов на GitHub](https://github.com/nocobase/plugin-samples/tree/main/packages/plugins/%40nocobase-sample/plugin-initializer-action-modal).

<video controls width='100%' src="https://static-docs.nocobase.com/20240526172851_rec_.mp4"></video>

## Инициализация плагина

Следуйте документации «[Написать первый плагин](/plugin-development/write-your-first-plugin)»: если проекта ещё нет, сначала создайте его; если проект уже есть или вы клонировали исходный код, этот шаг можно пропустить.

```bash
yarn create nocobase-app my-nocobase-app -d postgres
cd my-nocobase-app
yarn install
yarn nocobase install
```

Затем инициализируйте плагин и подключите его к системе:

```bash
yarn pm create @nocobase-sample/plugin-initializer-action-modal
yarn pm enable @nocobase-sample/plugin-initializer-action-modal
```

Затем запустите проект:

```bash
yarn dev
```

После входа в систему откройте [http://localhost:13000/admin/pm/list/local/](http://localhost:13000/admin/pm/list/local/) и убедитесь, что плагин установлен и включён.

## Реализация

Прежде чем повторять этот пример, необходимо ознакомиться с базовыми материалами:

- [Компонент действия](https://client.docs.nocobase.com/components/action)
- [Учебник по инициализатору схемы](/development/client/ui-schema/initializer): как добавлять в интерфейс блоки, поля, действия и другое
- [API инициализатора схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer): как добавлять в интерфейс блоки, поля, действия и другое
- [Пользовательская схема интерфейса](/development/client/ui-schema/what-is-ui-schema): описание структуры и оформления интерфейса
- [Визуальный редактор Designable](/development/client/ui-schema/designable): изменение схемы в конструкторе

```bash
.
├── client # клиентская часть плагина
│   ├── initializer # инициализатор
│   ├── index.tsx # точка входа клиентской части
│   ├── locale.ts # вспомогательные функции локализации
│   ├── constants.ts # константы
│   ├── schema # схема
│   └── settings # настройки схемы
├── locale # файлы локализации
│   ├── en-US.json # английский
│   └── zh-CN.json # китайский
├── index.ts # точка входа серверной части
└── server # серверная часть плагина
```

### 1. Определите имя

Сначала задаётся имя действия — оно будет использоваться в нескольких местах.

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-action-modal/src/client/constants.ts`:

```ts
export const ActionName = 'Open Document';
export const ActionNameLowercase = 'open-document';
```

### 2. Определить схему

#### 2.1 Определить схему

Все динамические страницы NocoBase строятся по схеме, поэтому нужно описать схему, которую затем можно вставить в интерфейс. Перед реализацией этого раздела необходимо прочитать:

- [Компонент действия](https://client.docs.nocobase.com/components/action)
- [Выдвижная панель действия](https://client.docs.nocobase.com/components/action#actiondrawer)
- [Протокол пользовательской схемы интерфейса](/development/client/ui-schema/what-is-ui-schema): структура схемы и назначение полей

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-action-modal/src/client/schema/index.ts` со следующим содержимым:

```ts
import { ISchema } from "@nocobase/client"
import { tStr } from "../locale";
import { ActionName } from "../constants";

export const createDocumentActionModalSchema = (blockComponent: string): ISchema => {
  return {
    type: 'void',
    'x-component': 'Action',
    title: tStr(ActionName),
    'x-component-props': {
      type: 'primary'
    },
    properties: {
      drawer: {
        type: 'void',
        'x-component': 'Action.Drawer',
        'x-component-props': {
          size: 'large'
        },
        properties: {
          iframe: {
            type: 'void',
            'x-component': 'iframe',
            'x-component-props': {
              src: `https://client.docs.nocobase.com/components/${blockComponent}`,
              style: {
                border: 'none',
                width: '100%',
                height: '100%'
              }
            },
          }
        }
      },
    },
  }
}
```

Функция `createDocumentActionModalSchema` принимает параметр `blockComponent` и возвращает схему. По этой схеме в интерфейс добавляется кнопка: при нажатии открывается модальное окно, а внутри него — встроенный фрейм со страницей документации выбранного блока.

`createDocumentActionModalSchema`:

- `type`: тип узла; здесь `void` — узел без собственных данных, только оболочка для компонента интерфейса
- `x-component: 'Action'`: [компонент действия](https://client.docs.nocobase.com/components/action) для отображения кнопки
- `title`: заголовок кнопки (в примере через локализацию из константы `ActionName`)
- `properties`: дочерние узлы
  - вложенный узел с компонентом `Action.Drawer` ([выдвижная панель действия](https://client.docs.nocobase.com/components/action#actiondrawer)) в составе схемы

Подробнее о схеме см. [документацию по пользовательской схеме интерфейса](/development/client/ui-schema/what-is-ui-schema).

#### 2.2 Проверка схемы

Проверить схему можно двумя способами:

- **Временная страница:** создать страницу, отрисовать на ней схему и убедиться, что поведение соответствует задаче.
- **Примеры в документации:** запустить документацию командой `yarn doc plugins/@nocobase-sample/plugin-initializer-action-modal` и проверить схему через написанные примеры (раздел в планах).

Ниже для примера используется **временная страница**: добавляется маршрут и одна или несколько тестовых схем с разными значениями `blockComponent`.

```tsx | pure
import React from 'react';
import { Plugin, SchemaComponent } from '@nocobase/client';
import { createDocumentActionModalSchema } from './schema';

export class PluginInitializerActionModalClient extends Plugin {
  async load() {
    this.app.router.add('admin.open-document-schema', {
      path: '/admin/open-document-schema',
      Component: () => {
        return <>
          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <SchemaComponent schema={{ properties: { test1: createDocumentActionModalSchema('table-v2') } }} />
          </div>
          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <SchemaComponent schema={{ properties: { test2: createDocumentActionModalSchema('details') } }} />
          </div>
        </>
      }
    })
  }
}

export default PluginInitializerActionModalClient;
```

Затем откройте [http://localhost:13000//admin/open-document-schema](http://localhost:13000/admin/open-document-schema) и проверьте временную страницу.

Подробнее о компоненте отрисовки схемы см. [соответствующую документацию](https://client.docs.nocobase.com/core/ui-schema/schema-component#schemacomponent-1).

<video controls width='100%' src="https://static-docs.nocobase.com/20240526171945_rec_.mp4"></video>

После проверки тестовую страницу и код её регистрации нужно удалить.

### 3. Определить элемент инициализатора схемы

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-action-modal/src/client/initializer/index.ts`:

```ts
import { SchemaInitializerItemType, useSchemaInitializer } from "@nocobase/client"

import { useT } from "../locale";
import { createDocumentActionModalSchema } from '../schema';
import { ActionName, ActionNameLowercase } from "../constants";

export const createDocumentActionModalInitializerItem = (blockComponent: string): SchemaInitializerItemType => ({
  type: 'item',
  title: ActionName,
  name: ActionNameLowercase,
  useComponentProps() {
    const { insert } = useSchemaInitializer();
    const t = useT();
    return {
      title: t(ActionName),
      onClick: () => {
        insert(createDocumentActionModalSchema(blockComponent));
      },
    };
  },
})
```

Так как для разных `blockComponent` нужны разные варианты модального действия с документацией, вынесена функция `createDocumentActionModalInitializerItem`, которая создаёт соответствующий элемент инициализатора схемы.

- `type`: тип элемента; здесь `item` — пункт с подписью и обработчиком клика, по клику в схему вставляется новый фрагмент
- `name`: уникальный идентификатор для различения элементов и операций создания/чтения/обновления/удаления
- `useComponentProps`: возвращает объект со свойствами `title` и `onClick`: `title` — текст на экране, `onClick` — обработчик после нажатия
- [Документация по хуку инициализатора схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer#useschemainitializer): доступ к контексту `SchemaInitializerContext` с методами работы со схемой

Описание элементов инициализатора см. в [документации по элементам инициализатора схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer#built-in-components-and-types).

### 4. Реализация настроек схемы

#### 4.1 Определение настроек схемы

Сейчас после вставки через `createDocumentActionModalInitializerItem()` узел нельзя удалить из интерфейса. Это исправляется подключением [настроек схемы](https://client.docs.nocobase.com/core/ui-schema/schema-settings).

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-action-modal/src/client/settings/index.ts`:

```ts
import { SchemaSettings } from "@nocobase/client";
import { ActionNameLowercase } from "../constants";

export const documentActionModalSettings = new SchemaSettings({
  name: `actionSettings:${ActionNameLowercase}`,
  items: [
    {
      name: 'remove',
      type: 'remove',
    }
  ]
});
```

#### 4.2 Регистрация настроек схемы

```ts
import { Plugin } from '@nocobase/client';
import { documentActionModalSettings } from './settings';

export class PluginInitializerActionModalClient extends Plugin {
  async load() {
    this.app.schemaSettingsManager.add(documentActionModalSettings);
  }
}

export default PluginInitializerActionModalClient;
```

#### 4.3 Использование настроек схемы

Измените функцию `createDocumentActionModalSchema` в файле `packages/plugins/@nocobase-sample/plugin-initializer-action-modal/src/client/schema/index.ts`, чтобы в схеме появилось поле `x-settings` с именем `documentActionModalSettings`.

```diff
export const createDocumentActionModalSchema = (blockComponent: string): ISchema => {
  return {
    type: 'void',
    'x-component': 'Action',
+   'x-settings': documentActionModalSettings.name,
    // ..
  }
}
```

### 5. Добавление на страницу: настройка действий

В системе много точек меню **настройки действий**, но **идентификаторы инициализаторов у них разные**. Нужно добавить элемент в те меню, которые соответствуют блокам **таблицы**, **деталей** и **формы** в вашем сценарии.

Сначала подберите подходящие имена инициализаторов (они зависят от версии и шаблонов страниц). После сопоставления с вашим интерфейсом отредактируйте вызовы `addItem` в коде ниже.

Затем измените файл `packages/plugins/@nocobase-sample/plugin-initializer-action-modal/src/client/index.tsx`:

```diff
import { Plugin } from '@nocobase/client';
import { documentActionModalSettings } from './documentActionModalSettings';
import { createDocumentActionModalInitializerItem } from './documentActionModalInitializerItem';

export class PluginInitializerActionModalClient extends Plugin {
  async load() {
    this.app.schemaSettingsManager.add(documentActionModalSettings);
+   this.app.schemaInitializerManager.addItem('table:configureActions', 'open-document', createDocumentActionModalInitializerItem('table-v2'));
+   this.app.schemaInitializerManager.addItem('details:configureActions', 'open-document', createDocumentActionModalInitializerItem('details'));
+   this.app.schemaInitializerManager.addItem('createForm:configureActions', 'open-document', createDocumentActionModalInitializerItem('form-v2'));
  }
}

export default PluginInitializerActionModalClient;
```

<video controls width='100%' src="https://static-docs.nocobase.com/20240526172851_rec_.mp4"></video>

### 6. Многоязычность

:::предупреждение
После изменения многоязычных файлов необходимо перезапустить службу, чтобы изменения вступили в силу.
:::

##### 6.1 Английский

Отредактируйте `packages/plugins/@nocobase-sample/plugin-initializer-action-modal/locale/en-US.json` следующим образом:

```json
{
  "Document": "Document"
}
```

##### 6.2 Китайский

Отредактируйте `packages/plugins/@nocobase-sample/plugin-initializer-action-modal/locale/zh-CN.json` следующим образом:

```json
{
  "Document": "文档"
}
```

При необходимости добавьте другие языки по тому же принципу.

Языки интерфейса задаются в [настройках системы](http://localhost:13000/admin/settings/system-settings); переключатель обычно находится в правом верхнем углу.

![20240611113758](https://static-docs.nocobase.com/20240611113758.png)

## Упаковка и выкладка в продуктивную среду

Согласно [Build and Package Plugin](/plugin-development/write-your-first-plugin#build-and-package-plugin) документации, мы можем упаковать плагин и загрузить его в производственную среду.

Если вы клонировали исходный код, вам необходимо сначала выполнить полную сборку, чтобы также построить зависимости плагина.

```bash
yarn build
```

Если вы использовали `create-nocobase-app` чтобы создать проект, вы можете напрямую выполнить:

```bash
yarn build @nocobase-sample/plugin-initializer-action-modal --tar
```

В каталоге `storage/tar/@nocobase-sample/plugin-initializer-action-modal.tar.gz` появится архив; установить плагин можно по инструкции [установки плагинов из файла](/get-started/installation/plugins).
