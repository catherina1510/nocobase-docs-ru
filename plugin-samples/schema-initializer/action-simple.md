# Добавление простого действия

## Сценарий

В NocoBase во многих местах интерфейса есть кнопка **настройки действий** для добавления кнопок действий в интерфейс.

![img_v3_02b4_51f4918f-d344-43b2-b19e-48dca709467g](https://static-docs.nocobase.com/img_v3_02b4_51f4918f-d344-43b2-b19e-48dca709467g.jpg)

Если существующие кнопки действий не соответствуют нашим требованиям, нужно добавить подпункты в существующее меню **настройки действий**, чтобы появились новые кнопки действий.

Под «простым действием» здесь имеются в виду действия **без модального окна**. См. также статью «[Действие с модальным окном](/plugin-samples/schema-initializer/action-modal)».

## Пример

В этом примере создаётся кнопка, по нажатию открывающая документацию соответствующего блока. Эту кнопку добавляют в меню **настройки действий** в блоках **таблицы**, **деталей** и **формы**.

Полный пример кода к этой статье находится в [репозитории примеров плагинов](https://github.com/nocobase/plugin-samples/tree/main/packages/plugins/%40nocobase-sample/plugin-initializer-action-simple).

<video width="100%" controls="">
  <source src="https://static-docs.nocobase.com/20240522-185359.mp4" type="video/mp4" />
</video>

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
yarn pm create @nocobase-sample/plugin-initializer-action-simple
yarn pm enable @nocobase-sample/plugin-initializer-action-simple
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

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-action-simple/src/client/constants.ts`:

```ts
export const ActionName = 'Document';
export const ActionNameLowercase = ActionName.toLowerCase();
```

### 2. Определить схему

#### 2.1 Определить схему

Все динамические страницы NocoBase строятся по схеме, поэтому нужно описать схему, которую затем можно вставить в интерфейс. Перед реализацией этого раздела необходимо прочитать:

- [Компонент действия](https://client.docs.nocobase.com/components/action)
- [Протокол пользовательской схемы интерфейса](/development/client/ui-schema/what-is-ui-schema): структура схемы и назначение полей

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-action-simple/src/client/schema/index.ts` со следующим содержимым:

```ts
import { useFieldSchema } from '@formily/react';
import { ISchema } from "@nocobase/client"
import { useT } from '../locale';
import { ActionName } from '../constants';

export function useDocumentActionProps() {
  const fieldSchema = useFieldSchema();
  const t = useT();
  return {
    title: t(ActionName),
    type: 'primary',
    onClick() {
      window.open(fieldSchema['x-doc-url'])
    }
  }
}

export const createDocumentActionSchema = (blockComponent: string): ISchema & { 'x-doc-url': string } => {
  return {
    type: 'void',
    'x-component': 'Action',
    'x-doc-url': `https://client.docs.nocobase.com/components/${blockComponent}`,
    'x-use-component-props': 'useDocumentActionProps',
  }
}
```

Функция `createDocumentActionSchema` принимает параметр `blockComponent` и возвращает схему. По этой схеме в интерфейс добавляется кнопка: при нажатии открывается документация соответствующего блока.

`createDocumentActionSchema`:

- `type`: тип узла; здесь `void` — узел без собственных данных, только оболочка для компонента интерфейса
- `x-component: 'Action'`: [компонент действия](https://client.docs.nocobase.com/components/action) для отображения кнопки
- заголовок кнопки задаётся в `useDocumentActionProps` через `t(ActionName)` (в исходной константе — `Document`)
- `x-doc-url`: пользовательское поле схемы с адресом страницы документации
- `x-use-component-props: 'useDocumentActionProps'`: динамические свойства компонента; подробнее см. [документацию](/development/client/ui-schema/what-is-ui-schema#x-component-props-和-x-use-component-props)

`useDocumentActionProps()`:

- [useFieldSchema()](https://client.docs.nocobase.com/core/ui-schema/designable#usefieldschema): получить схему текущего узла
- `type: 'primary'`: тип кнопки — **основная** (в глоссарии интерфейса NocoBase этому соответствует стиль основной кнопки)
- `onClick`: обработчик нажатия — открыть документацию соответствующего блока

Подробнее о схеме см. [документацию по пользовательской схеме интерфейса](/development/client/ui-schema/what-is-ui-schema).

#### 2.2 Регистрация области видимости

Нужно зарегистрировать `useDocumentActionProps` в приложении через `addScopes`, чтобы по полю `x-use-component-props` находился соответствующий хук.

```ts
import { useDocumentActionProps } from './schema';
import { Plugin } from '@nocobase/client';

export class PluginInitializerActionSimpleClient extends Plugin {
  async load() {
    this.app.addScopes({ useDocumentActionProps });
  }
}

export default PluginInitializerActionSimpleClient;
```

#### 2.3 Проверка схемы блока

Проверить схему можно двумя способами:

- **Временная страница:** создать страницу, отрисовать на ней схему и убедиться, что поведение соответствует задаче.
- **Примеры в документации:** запустить документацию командой `yarn doc plugins/@nocobase-sample/plugin-initializer-action-simple` и проверить схему через написанные примеры (раздел в планах).

Ниже для примера используется **временная страница**: добавляется маршрут и одна или несколько тестовых схем с разными значениями параметров.

```tsx | pure
import { Plugin, SchemaComponent } from '@nocobase/client';
import { createDocumentActionSchema, useDocumentActionProps } from './schema';
import React from 'react';

export class PluginInitializerActionSimpleClient extends Plugin {
  async load() {
    this.app.addScopes({ useDocumentActionProps });
    this.app.router.add('admin.document-action-schema', {
      path: '/admin/document-action-schema',
      Component: () => {
        return <>
          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <SchemaComponent schema={{ properties: { test1: createDocumentActionSchema('table-v2') } }} />
          </div>
          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <SchemaComponent schema={{ properties: { test2: createDocumentActionSchema('details') } }} />
          </div>
        </>
      }
    })
  }
}

export default PluginInitializerActionSimpleClient;
```

Затем откройте [http://localhost:13000/admin/document-action-schema](http://localhost:13000/admin/document-action-schema) и проверьте временную страницу.

Подробнее о компоненте отрисовки схемы см. [соответствующую документацию](https://client.docs.nocobase.com/core/ui-schema/schema-component#schemacomponent-1).

<video controls width='100%' src="https://static-docs.nocobase.com/20240526171318_rec_.mp4"></video>

После проверки тестовую страницу необходимо удалить.

### 3. Определить элемент инициализатора схемы

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-action-simple/src/client/initializer/index.ts`:

```tsx | pure
import { SchemaInitializerItemType, useSchemaInitializer } from "@nocobase/client"

import { createDocumentActionSchema } from '../schema';
import { ActionNameLowercase, ActionName } from "../constants";
import { useT } from "../locale";

export const createDocumentActionInitializerItem = (blockComponent: string): SchemaInitializerItemType => ({
  type: 'item',
  name: ActionNameLowercase,
  useComponentProps() {
    const { insert } = useSchemaInitializer();
    const t = useT();
    return {
      title: t(ActionName),
      onClick: () => {
        insert(createDocumentActionSchema(blockComponent));
      },
    };
  },
})
```

Так как для разных значений `blockComponent` нужны разные варианты кнопки с документацией, вынесена функция `createDocumentActionInitializerItem`, которая создаёт соответствующий элемент инициализатора схемы.

- `type`: тип элемента; здесь `item` — пункт с подписью и обработчиком клика, по клику в схему вставляется новый фрагмент
- `name`: уникальный идентификатор для различения элементов и операций создания, чтения, обновления и удаления
- `useComponentProps`: возвращает объект со свойствами `title` и `onClick`: `title` — текст на экране, `onClick` — функция обратного вызова после нажатия
- [Документация по хуку инициализатора схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer#useschemainitializer): доступ к контексту `SchemaInitializerContext` с методами работы со схемой

Описание элементов инициализатора см. в [документации по элементам инициализатора схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer#built-in-components-and-types).

### 4. Реализация настроек схемы

#### 4.1 Определение настроек схемы

В настоящее время после добавления через `createDocumentActionInitializerItem()`, его нельзя удалить. Мы можем использовать [Schema Settings](https://client.docs.nocobase.com/core/ui-schema/schema-settings) чтобы установить его.

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-action-simple/src/client/settings/index.ts`:

```ts
import { SchemaSettings } from "@nocobase/client";

import { ActionNameLowercase } from "../constants";

export const documentActionSettings = new SchemaSettings({
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

```diff
import { Plugin } from '@nocobase/client';
import { useDocumentActionProps } from './schema';
+ import { documentActionSettings } from './settings';

export class PluginInitializerActionSimpleClient extends Plugin {
  async load() {
    this.app.addScopes({ useDocumentActionProps });
+   this.app.schemaSettingsManager.add(documentActionSettings);
  }
}

export default PluginInitializerActionSimpleClient;
```

#### 4.3 Использование настроек схемы

Измените функцию `createDocumentActionSchema` в файле `packages/plugins/@nocobase-sample/plugin-initializer-action-simple/src/client/schema/index.ts`, чтобы в схеме появилось поле `x-settings` с именем набора настроек.

```diff
+ import { documentActionSettings } from '../settings';

export const createDocumentActionSchema = (blockComponent: string): ISchema & { 'x-doc-url': string } => {
  return {
    type: 'void',
    'x-component': 'Action',
+   'x-settings': documentActionSettings.name,
    // ...
  }
}
```

### 5. Добавление на страницу: настройка действий

В системе много точек меню **настройки действий**, но **идентификаторы инициализаторов у них разные**. Нужно добавить элемент в те меню, которые соответствуют блокам **таблицы**, **деталей** и **формы** в вашем сценарии.

Сначала подберите подходящие имена инициализаторов (они зависят от версии и шаблонов страниц). Подробный разбор поиска имён вынесен в планы дополнения исходной документации; после сопоставления с вашим интерфейсом отредактируйте вызовы `addItem` в коде ниже.

Затем измените файл `packages/plugins/@nocobase-sample/plugin-initializer-action-simple/src/client/index.tsx`:

```diff
import { Plugin } from '@nocobase/client';
import { useDocumentActionProps } from './schema';
import { documentActionSettings } from './settings';
+ import { createDocumentActionInitializerItem } from './initializer';

export class PluginInitializerActionSimpleClient extends Plugin {
  async load() {
    this.app.addScopes({ useDocumentActionProps });
    this.app.schemaSettingsManager.add(documentActionSettings);
+   this.app.schemaInitializerManager.addItem('table:configureActions', 'document', createDocumentActionInitializerItem('table-v2'));
+   this.app.schemaInitializerManager.addItem('details:configureActions', 'document', createDocumentActionInitializerItem('details'));
+   this.app.schemaInitializerManager.addItem('createForm:configureActions', 'document', createDocumentActionInitializerItem('form-v2'));
  }
}

export default PluginInitializerActionSimpleClient;
```

<video width="100%" controls="">
  <source src="https://static-docs.nocobase.com/20240522-185359.mp4" type="video/mp4" />
</video>

### 6. Многоязычность

:::предупреждение
После изменения файлов локализации необходимо перезапустить службу, чтобы изменения вступили в силу.
:::

##### 6.1 Английский

Отредактируйте `packages/plugins/@nocobase-sample/plugin-initializer-action-simple/src/locale/en-US.json` следующим образом:

```json
{
  "Document": "Document"
}
```

##### 6.2 Китайский

Отредактируйте `packages/plugins/@nocobase-sample/plugin-initializer-action-simple/src/locale/zh-CN.json` следующим образом:

```json
{
  "Document": "文档"
}
```

При необходимости добавьте другие языки по тому же принципу.

Языки интерфейса задаются в [настройках системы](http://localhost:13000/admin/settings/system-settings); переключатель обычно находится в правом верхнем углу.

![20240611113758](https://static-docs.nocobase.com/20240611113758.png)

## Упаковка и загрузка в производственную среду

Согласно разделу «[Сборка и упаковка плагина](/plugin-development/write-your-first-plugin#build-and-package-plugin)» в руководстве по разработке плагинов, можно собрать архив плагина и установить его в продуктивной среде.

Если вы клонировали исходный код, сначала выполните полную сборку, чтобы пересобрать и зависимости плагина.

```bash
yarn build
```

Если вы использовали `create-nocobase-app` чтобы создать проект, вы можете напрямую выполнить:

```bash
yarn build @nocobase-sample/plugin-initializer-action-simple --tar
```

В каталоге `storage/tar/@nocobase-sample/plugin-initializer-action-simple.tar.gz` появится архив; установить плагин можно по инструкции [установки плагинов из файла](/get-started/installation/plugins).
