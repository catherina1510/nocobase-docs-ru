# Встроенный инициализатор блока — настройка действий

## Сценарий

Если вновь созданный блок является **сложным блоком данных**, в нём может быть несколько динамически добавляемых частей. Инициализатор **«Настройка действий»** в основном отвечает за динамическое добавление кнопок для разных операций. Например, в блоке **«Детали»** через **«Настройку действий»** можно добавить кнопки **«Редактировать»**, **«Печать»** и другие.

![img_v3_02b4_9b80a4a0-6d9b-4e53-a544-f92c17d81d2g](https://static-docs.nocobase.com/img_v3_02b4_9b80a4a0-6d9b-4e53-a544-f92c17d81d2g.jpg)

## Пример

Этот пример продолжает материал «[Добавление блока данных](/plugin-samples/schema-initializer/data-block)»: получается поведение, похожее на блок **«Детали»**, а кнопки настраиваются через **«Настройку действий»**.

Полный исходный код — в [репозитории примеров плагинов](https://github.com/nocobase/plugin-samples/tree/main/packages/plugins/%40nocobase-sample/plugin-initializer-configure-actions).

<video width="100%" controls="">
  <source src="https://static-docs.nocobase.com/20240522-191602.mp4" type="video/mp4" />
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
yarn pm create @nocobase-sample/plugin-initializer-configure-actions
yarn pm enable @nocobase-sample/plugin-initializer-configure-actions
```

Затем запустите проект:

```bash
yarn dev
```

После входа в систему откройте [http://localhost:13000/admin/pm/list/local/](http://localhost:13000/admin/pm/list/local/) и убедитесь, что плагин установлен и включён.

## Реализация

Прежде чем повторять этот пример, полезно ознакомиться с базовыми материалами:

- [учебник по инициализатору схемы](/development/client/ui-schema/initializer): как добавлять в интерфейс блоки, поля, операции и другое;
- [API инициализатора схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer): то же на уровне API;
- [пользовательская схема интерфейса](/development/client/ui-schema/what-is-ui-schema): структура и оформление интерфейса;
- [визуальный редактор Designable](/development/client/ui-schema/designable): изменение схемы в конструкторе.

### 1. Создайте блок

Как уже говорилось, пример опирается на «[Добавление блока данных](/plugin-samples/schema-initializer/data-block)»: скопируйте каталог `packages/plugins/@nocobase-sample/plugin-initializer-block-data/src/client` поверх `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client`.

Затем измените `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/index.tsx`:

```diff
import { Plugin } from '@nocobase/client';
- import { Info } from './component';
+ import { InfoV2 } from './component';

- export class PluginInitializerBlockDataClient extends Plugin {
+ export class PluginInitializerConfigureActionsClient extends Plugin {
  async load() {
-   this.app.addComponents({ Info })
+   this.app.addComponents({ InfoV2 })
    // ...
  }
}

- export default PluginInitializerBlockDataClient;
+ export default PluginInitializerConfigureActionsClient;
```

Затем измените `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/constants.ts`:

```ts
export const BlockName = 'InfoV2';
export const BlockNameLowercase = 'info-v2';
```

### 2. Реализация инициализатора

#### 2.1 Определите инициализатор

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/configureActionsInitializer.ts`:

```tsx | pure
import { SchemaInitializer } from "@nocobase/client";
import { BlockNameLowercase } from "../../constants";

export const configureActionsInitializer = new SchemaInitializer({
  name: `${BlockNameLowercase}:configureActions`,
  icon: 'SettingOutlined',
  title: 'Configure actions',
  style: {
    marginLeft: 8,
  },
  items: [

  ]
});
```

Так задаётся новый экземпляр `SchemaInitializer`; подпункты пока пустые.

- [SchemaInitializer](https://client.docs.nocobase.com/core/ui-schema/schema-initializer): создание экземпляра инициализатора схемы;
- `icon`: значок кнопки; набор значков Ant Design — в разделе [Icons](https://ant.design/components/icon/);
- `title`: подпись кнопки;
- [items](https://client.docs.nocobase.com/core/ui-schema/schema-initializer#built-in-components-and-types): подпункты под этой кнопкой.

Экспортируйте инициализатор в `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/index.ts`:

```tsx | pure
export * from './configureActionsInitializer';
```

Измените `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/index.tsx`, чтобы реэкспортировать модуль `configureActions`:

```diff
import React from 'react';
import { SchemaInitializerItemType, useSchemaInitializer } from '@nocobase/client'
import { CodeOutlined } from '@ant-design/icons';

+ export * from './configureActions'
// ...
```

#### 2.2 Зарегистрируйте инициализатор

В `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/index.tsx` импортируйте и зарегистрируйте инициализатор:

```tsx | pure
// ...
import { infoInitializerItem, configureActionsInitializer } from './initializer';

export class PluginInitializerConfigureActionsClient extends Plugin {
  async load() {
    this.app.schemaInitializerManager.add(configureActionsInitializer)

    // ...
  }
}
```

#### 2.3 Подключите инициализатор к схеме

Измените `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/schema/index.ts`, добавив дочерний узел `actions`:

```diff
// ...
+ import { configureActionsInitializer } from "../initializer";

function getInfoBlockSchema({ dataSource, collection }) {
  return {
    // ...
    properties: {
      info: {
        'x-component': BlockName,
        'x-use-component-props': 'useInfoProps',
+       properties: {
+         actions: {
+           type: 'void',
+           'x-component': 'ActionBar',
+           'x-component-props': {
+             layout: 'two-column',
+             style: { marginBottom: 20 }
+           },
+           'x-initializer': configureActionsInitializer.name,
+         }
+       }
      }
    }
  }
}
```

Настройка действий обычно сочетается с компонентом [ActionBar](https://client.docs.nocobase.com/components/action#actionbar) (панель действий).

К узлу **Info** добавлено поле `actions` среди дочерних узлов `properties`:

- `type: 'void'`: тип `void` — контейнер без собственных данных;
- `x-component: 'ActionBar'`: для отображения кнопок используется [ActionBar](https://client.docs.nocobase.com/components/action#actionbar);
- `x-initializer: configureActionsInitializer.name`: подключается созданный выше инициализатор схемы;
- `x-component-props.layout: 'two-column'`: раскладка в две колонки (слева и справа); примеры — в [документации по ActionBar (режим двух колонок)](https://client.docs.nocobase.com/components/action#two-column).

#### 2.4 Блок отображает дочерние узлы

Измените `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/component/Info.tsx`, чтобы компонент **Info** (здесь **InfoV2**) выводил дочерние узлы схемы:

```diff
import React, { FC } from 'react';
import { withDynamicSchemaProps } from '@nocobase/client'

export interface InfoV2Props {
  collectionName: string;
  data?: any[];
  loading?: boolean;
+ children?: React.ReactNode;
}

export const InfoV2: FC<InfoV2Props> = withDynamicSchemaProps(({ children, collectionName, data }) => {
  return <div>
+   {children}
-   <div>collection: {collectionName}</div>
-   <div>data list: <pre>{JSON.stringify(data, null, 2)}</pre></div>
+   <div>data length: {data?.length}</div>
  </div>
}, { displayName: BlockName })
```

- `children`: содержимое из `properties` передаётся в `children` компонента **InfoV2**, поэтому достаточно отрендерить `children`.

![img_v3_02b4_4c6cb675-789e-48d5-99ce-072984dcfc9g](https://static-docs.nocobase.com/img_v3_02b4_4c6cb675-789e-48d5-99ce-072984dcfc9g.jpg)

### 3. Реализация элементов инициализатора

#### 3.1 Повторное использование: действие «Пользовательский запрос»

Дальше правьте `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/configureActionsInitializer.ts`:

```diff
export const configureActions = new SchemaInitializer({
  name: 'info:configureActions',
  title: 'Configure actions',
  icon: 'SettingOutlined',
  items: [
+   {
+     name: 'customRequest',
+     title: '{{t("Custom request")}}',
+     Component: 'CustomRequestInitializer',
+   },
  ]
});
```

Здесь напрямую подключается компонент `CustomRequestInitializer` для пункта **«Пользовательский запрос»** (в продукте — **«Действие: пользовательский запрос»**; ключ локализации `Custom request`). Другие готовые элементы инициализатора см. в документации и примерах ядра (*в разработке*).

![img_v3_02b4_0d439087-cfe1-4681-bfab-4e4bc3e34cbg](https://static-docs.nocobase.com/img_v3_02b4_0d439087-cfe1-4681-bfab-4e4bc3e34cbg.jpg)

#### 3.2 Свой сценарий: действие «Обновить данные»

Помимо повторного использования готовых элементов инициализатора можно описывать собственные **действия**. Пошагово это разобрано в статьях «[Добавление простого действия](/plugin-samples/schema-initializer/action-simple)» и «[Действие с модальным окном](/plugin-samples/schema-initializer/action-modal)».

Ниже реализовано действие **«Обновить данные»** (в коде примера — `Custom Refresh`).

#### 3.2.1 Определите имя

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/items/customRefresh/constants.ts`:

```ts
export const ActionName = 'Custom Request';
export const ActionNameLowercase = 'customRequest';
```

#### 3.2.2 Определите схему

##### 3.2.2.1 Определите схему

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/items/customRefresh/schema.ts`:

```ts
import { ActionProps, useDataBlockRequest, ISchema } from "@nocobase/client";
import { useT } from "../../../../locale";

export const useCustomRefreshActionProps = (): ActionProps => {
  const { runAsync } = useDataBlockRequest();
  const t = useT();
  return {
    type: 'primary',
    title: t('Custom Refresh'),
    async onClick() {
      await runAsync();
    },
  }
}

export const customRefreshActionSchema: ISchema = {
  type: 'void',
  'x-component': 'Action',
  'x-toolbar': 'ActionSchemaToolbar',
  'x-use-component-props': 'useCustomRefreshActionProps'
}
```

Описаны `customRefreshActionSchema` и динамические свойства через `useCustomRefreshActionProps`.

`customRefreshActionSchema`:

- `type: 'void'`: тип `void` — простой интерфейс без полей данных коллекции;
- `x-component: 'Action'`: кнопка на базе компонента [Action](https://client.docs.nocobase.com/components/action);
- `title` в пропсах задаётся в хуке (например `t('Custom Refresh')`) — подпись кнопки;
- `x-use-component-props: 'useCustomRefreshActionProps'`: свойства берутся из хука `useCustomRefreshActionProps`. Схема сохраняется на сервере, поэтому имя хука указывается **строкой**;
- `'x-toolbar': 'ActionSchemaToolbar'`: типичная связка с компонентом **Action**. В отличие от панели по умолчанию, скрывает **инициализатор** в правом верхнем углу действия, остаются только **перетаскивание** и **настройки**.

`useCustomRefreshActionProps` — React-хук, возвращающий пропсы компонента **Action**:

- [useDataBlockRequest()](https://client.docs.nocobase.com/core/data-block/data-block-request-provider): объект запроса **блока данных** внутри `DataBlockProvider`, для автоматической загрузки данных блока;
  - `runAsync`: асинхронный перезапрос данных блока;
- `type: 'primary'`: тип кнопки **основная** (`primary`);
- `onClick`: обработчик нажатия.

Экспорт в `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/items/customRefresh/index.ts`:

```ts
export * from './schema';
```

Измените `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/index.ts`, чтобы реэкспортировать модуль `customRefresh`:

```diff
export * from './configureActionsInitializer';
+ export * from './items/customRefresh';
```

##### 3.2.2.2 Регистрация в контексте

Зарегистрируйте `useCustomRefreshActionProps` в контексте приложения. В `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/index.tsx`:

```diff
// ...
- import { infoInitializerItem } from './initializer';
+ import { infoInitializerItem, useCustomRefreshActionProps } from './initializer';

export class PluginInitializerConfigureActionsClient extends Plugin {
  async load() {
    // ...
-   this.app.addScopes({ useInfoProps });
+   this.app.addScopes({ useInfoProps, useCustomRefreshActionProps });
  }
}
```

Подробнее об использовании `SchemaComponentOptions` см. [документацию SchemaComponentOptions](https://client.docs.nocobase.com/core/ui-schema/schema-component#schemacomponentoptions) и статью «[Глобальная регистрация компонента и области видимости](/plugin-samples/component-and-scope/global)».

#### 3.3.2 Реализация настроек

##### 3.3.2.1 Определите настройки

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/items/customRefresh/settings.ts`:

```tsx | pure
import { SchemaSettings } from "@nocobase/client";
import { ActionNameLowercase } from "./constants";

export const customRefreshActionSettings = new SchemaSettings({
  name: `actionSettings:${ActionNameLowercase}`,
  items: [
    {
      name: 'remove',
      type: 'remove',
    }
  ]
})
```

`customRefreshActionSettings`: в примере добавлена только операция **удаления** (`remove`). Параметры настроек схемы описаны в [документации по настройкам схемы](https://client.docs.nocobase.com/core/ui-schema/schema-settings).

Измените `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/items/customRefresh/index.ts`, чтобы экспортировать настройки:

```tsx | pure
export * from './settings';
```

##### 3.3.2.2 Зарегистрируйте настройки

Зарегистрируйте `customRefreshActionSettings` в системе. В `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/index.tsx`:

```diff
- import { infoInitializerItem, useCustomRefreshActionProps } from './initializer';
+ import { infoInitializerItem, useCustomRefreshActionProps, customRefreshActionSettings } from './initializer';

export class PluginInitializerConfigureActionsClient extends Plugin {
  async load() {
+   this.app.schemaSettingsManager.add(customRefreshActionSettings);
  }
}
```

##### 3.3.2.3 Подключите настройки к схеме

В `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/items/customRefresh/schema.ts` у объекта `customRefreshActionSchema` задайте `x-settings` равным `customRefreshActionSettings.name`:

```diff
+ import { customRefreshActionSettings } from "./settings";

export const customRefreshActionSchema: ISchema = {
  type: 'void',
  'x-component': 'Action',
+ "x-settings": customRefreshActionSettings.name,
  title: 'Custom Refresh',
  'x-use-component-props': 'useCustomRefreshActionProps'
}
```

##### 3.3.3 Элемент инициализатора схемы

###### 3.3.3.1 Определите элемент

Добавьте в `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/items/customRefresh/initializer.ts`:

```tsx | pure
import { SchemaInitializerItemType, useSchemaInitializer } from "@nocobase/client";
import { customRefreshActionSchema } from "./schema";
import { ActionName } from "./constants";
import { useT } from "../../../../locale";

export const customRefreshActionInitializerItem: SchemaInitializerItemType = {
  type: 'item',
  name: ActionName,
  useComponentProps() {
    const { insert } = useSchemaInitializer();
    const t = useT();
    return {
      title: t(ActionName),
      onClick() {
        insert(customRefreshActionSchema)
      },
    };
  },
};
```

- `type: 'item'`: тип `item` — пункт меню, по нажатию вызывается `onClick`;
- `name`: уникальный идентификатор элемента схемы (в т.ч. для операций создания, чтения, обновления и удаления);
- `title`: подпись пункта меню (здесь через `t(ActionName)`).

Подробнее о полях элемента см. [документацию по типам элементов инициализатора схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer#built-in-components-and-types).

Измените `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/index.ts`, чтобы экспортировать новый модуль:

```tsx | pure
export * from './initializer';
```

###### 3.3.3.2 Добавьте элемент в `items`

В `packages/plugins/@nocobase-sample/plugin-initializer-configure-actions/src/client/initializer/configureActions/configureActionsInitializer.ts` добавьте `customRefreshActionInitializerItem` в массив `items`:

```diff
import { SchemaInitializer } from "@nocobase/client";
+ import { customRefreshActionInitializerItem } from "./items/customRefresh";

export const configureActionsInitializer = new SchemaInitializer({
  name: 'info:configureActions',
  title: 'Configure actions',
  icon: 'SettingOutlined',
  style: {
    marginLeft: 8,
  },
  items: [
    {
      name: 'customRequest',
      title: '{{t("Custom request")}}',
      Component: 'CustomRequestInitializer',
      'x-align': 'right',
    },
+   customRefreshActionInitializerItem
  ]
});
```

<video width="100%" controls="">
  <source src="https://static-docs.nocobase.com/20240522-191602.mp4" type="video/mp4" />
</video>

При необходимости добавьте другие **действия** по той же схеме.

## Упаковка и загрузка в производственную среду

Согласно разделу «[Сборка и упаковка плагина](/plugin-development/write-your-first-plugin#build-and-package-plugin)» в руководстве по разработке плагинов, можно собрать архив и установить его в производственной среде.

Если вы клонировали исходный код, сначала выполните полную сборку, чтобы собрать и зависимости плагина.

```bash
yarn build
```

Если вы использовали `create-nocobase-app` чтобы создать проект, вы можете напрямую выполнить:

```bash
yarn build @nocobase-sample/plugin-initializer-configure-actions --tar
```

В каталоге `storage/tar/@nocobase-sample/plugin-initializer-configure-actions.tar.gz` появится архив; установить плагин можно по инструкции [установки плагинов из файла](/get-started/installation/plugins).
