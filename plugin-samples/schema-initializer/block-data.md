# Добавление блока данных

## Сценарий

В NocoBase во многих местах есть меню **«Добавить блок»**. Часть пунктов связана с **коллекциями** (таблицами данных) — это **блоки данных**; остальные, без привязки к коллекции, — **простые блоки**.

![img_v3_02b4_170eddb5-d3b4-461e-b74b-f83250941e5g](https://static-docs.nocobase.com/img_v3_02b4_170eddb5-d3b4-461e-b74b-f83250941e5g.jpg)

Встроенных типов блоков может не хватать — тогда добавляют собственные блоки под задачу. Здесь речь именно о **блоках данных** (с загрузкой записей из коллекции).

## Пример

В этом примере создаётся блок **Info** и подключается к меню **«Добавить блок»** на **странице**, в **блоке таблицы** (в т.ч. в модальном окне) и на **мобильной странице**.

Пример показывает работу **инициализатора схемы**. Про расширение блоков см. «[Расширение блока](/plugin-samples/block)».

Полный исходный код — в [репозитории примеров плагинов](https://github.com/nocobase/plugin-samples/tree/main/packages/plugins/%40nocobase-sample/plugin-initializer-block-data).

<video width="100%" controls="">
  <source src="https://static-docs.nocobase.com/20240522-182547.mp4" type="video/mp4" />
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
yarn pm create @nocobase-sample/plugin-initializer-block-data
yarn pm enable @nocobase-sample/plugin-initializer-block-data
```

Затем запустите проект:

```bash
yarn dev
```

После входа в систему откройте [http://localhost:13000/admin/pm/list/local/](http://localhost:13000/admin/pm/list/local/) и убедитесь, что плагин установлен и включён.

## Реализация

Прежде чем повторять пример, полезно прочитать:

- [учебник по инициализатору схемы](/development/client/ui-schema/initializer): как добавлять в интерфейс блоки, поля, действия и другое;
- [API инициализатора схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer): то же на уровне API;
- [пользовательская схема интерфейса](/development/client/ui-schema/what-is-ui-schema): структура и оформление интерфейса;
- [визуальный редактор Designable](/development/client/ui-schema/designable): изменение схемы в конструкторе.

```bash
.
├── client # клиентская часть плагина
│   ├── initializer # инициализатор
│   ├── component # компонент блока
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

Сначала задаётся имя блока — оно используется в нескольких местах.

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data/src/client/constants.ts`:

```ts
export const BlockName = 'Info';
export const BlockNameLowercase = BlockName.toLowerCase();
```

### 2. Реализация компонента блока

#### 2.1 Определить компонент блока

Нужен компонент блока **Info** со следующим поведением:

- показывать имя **коллекции** текущего блока;
- показывать список загруженных записей.

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data/src/client/component/Info.tsx` со следующим содержимым:

```tsx | pure
import React, { FC } from 'react';
import { withDynamicSchemaProps } from '@nocobase/client'
import { BlockName } from '../constants';

export interface InfoProps {
  collectionName: string;
  data?: any[];
  loading?: boolean;
}

export const Info: FC<InfoProps> = withDynamicSchemaProps(({ collectionName, data }) => {
  return <div>
    <div>collection: {collectionName}</div>
    <div>data list: <pre>{JSON.stringify(data, null, 2)}</pre></div>
  </div>
}, { displayName: BlockName })
```

Компонент `Info` — функциональный компонент, обёрнутый в `withDynamicSchemaProps`. [withDynamicSchemaProps](/development/client/ui-schema/what-is-ui-schema#x-component-props-和-x-use-component-props) — компонент высшего порядка для динамических свойств в схеме.

Без учёта `withDynamicSchemaProps` это обычный функциональный компонент.

Экспортируйте его в `packages/plugins/@nocobase-sample/plugin-initializer-block-data/src/client/component/index.ts`:

```tsx | pure
export * from './Info';
```

#### 2.2 Регистрация компонента блока

Зарегистрируйте `Info` в приложении из плагина.

```tsx | pure
import { Plugin } from '@nocobase/client';
import { Info } from './component';

export class PluginInitializerBlockDataClient extends Plugin {
  async load() {
    this.app.addComponents({ Info })
  }
}

export default PluginInitializerBlockDataClient;
```

#### 2.3 Проверка компонента блока

Проверить компонент можно двумя способами:

- **Временная страница:** маршрут и отрисовка `Info` с тестовыми данными.
- **Примеры в документации:** команда `yarn doc plugins/@nocobase-sample/plugin-initializer-block-data` и проверка через примеры в документации (раздел в планах).

Ниже — вариант с **временной страницей**.

```tsx | pure
import React from 'react';
import { Plugin } from '@nocobase/client';
import { Info } from './component';

export class PluginInitializerBlockDataClient extends Plugin {
  async load() {
    this.app.addComponents({ Info })

    this.app.router.add('admin.info-component', {
      path: '/admin/info-component',
      Component: () => {
        return <>
          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <Info collectionName='test' data={[{ id: 1 }, { id: 2 }]} />
          </div>
        </>
      }
    })
  }
}

export default PluginInitializerBlockDataClient;
```

Откройте [http://localhost:13000/admin/info-component](http://localhost:13000/admin/info-component) и проверьте страницу.

![20240526165834](https://static-docs.nocobase.com/20240526165834.png)

После проверки удалите тестовый маршрут и связанный код.

### 3. Определить схему блока

#### 3.1 Определение схемы блока

Интерфейс строится по схеме; нужно описать фрагмент, который инициализатор вставит на страницу как блок **Info**. Полезно прочитать:

- [протокол пользовательской схемы интерфейса](/development/client/ui-schema/what-is-ui-schema): структура схемы и поля;
- [DataBlockProvider](https://client.docs.nocobase.com/core/data-block/data-block-provider): провайдер **блока данных**.

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data/src/client/schema/index.ts`:

```ts
import { useCollection, useDataBlockRequest } from "@nocobase/client";

import { InfoProps } from "../component";
import { BlockName, BlockNameLowercase } from "../constants";

export function useInfoProps(): InfoProps {
  const collection = useCollection();
  const { data, loading } = useDataBlockRequest<any[]>();

  return {
    collectionName: collection.name,
    data: data?.data,
    loading: loading
  }
}

export function getInfoSchema({ dataSource = 'main', collection }) {
  return {
    type: 'void',
    'x-decorator': 'DataBlockProvider',
    'x-decorator-props': {
      dataSource,
      collection,
      action: 'list',
    },
    'x-component': 'CardItem',
    "x-toolbar": "BlockSchemaToolbar",
    properties: {
      [BlockNameLowercase]: {
        type: 'void',
        'x-component': BlockName,
        'x-use-component-props': 'useInfoProps',
      }
    }
  }
}
```

Здесь нужно объяснить 2 момента:

- `getInfoSchema()` вынесен в функцию, потому что **источник данных** и **коллекция** задаются при выборе таблицы в инициализаторе.
- `useInfoProps()` задаёт динамические параметры для `Info`; в схеме в базе хранится **строка** с именем хука (`x-use-component-props`).

Разбор `getInfoSchema()`:

  - `type: 'void'`: узел без собственного значения в данных формы;
  - `x-decorator: 'DataBlockProvider'`: провайдер блока данных; подробнее — [DataBlockProvider](https://client.docs.nocobase.com/core/data-block/data-block-provider);
  - `x-decorator-props`: параметры провайдера;
    - `dataSource`: источник данных;
    - `collection`: коллекция;
    - `action: 'list'`: операция получения списка записей;
  - `x-component: 'CardItem'`: [компонент карточки блока](https://client.docs.nocobase.com/components/card-item); блок оборачивается в карточку для оформления, макета и перетаскивания;
  - `properties`: дочерние узлы схемы;
    - узел с ключом `info` (из `BlockNameLowercase`): сам блок **Info**.

`useInfoProps()` — динамические параметры компонента **Info**:

  - [useCollection](https://client.docs.nocobase.com/core/data-source/collection-provider#usecollection): текущая коллекция из контекста [DataBlockProvider](https://client.docs.nocobase.com/core/data-block/data-block-provider);
  - [useDataBlockRequest](https://client.docs.nocobase.com/core/data-block/data-block-request-provider#usedatablockrequest): запрос данных блока, настроенный [DataBlockProvider](https://client.docs.nocobase.com/core/data-block/data-block-provider).

По смыслу схема близка к такому дереву:

```tsx | pure
<DataBlockProvider collection={collection} dataSource={dataSource} action='list'>
  <CardItem>
    <Info {...useInfoProps()} />
  </CardItem>
</DataBlockProvider>
```

#### 3.2 Регистрация области видимости

Зарегистрируйте `useInfoProps` через `addScopes`, чтобы по полю [x-use-component-props](/development/client/ui-schema/what-is-ui-schema#x-component-props-和-x-use-component-props) находился соответствующий хук.

```tsx | pure
import { Plugin } from '@nocobase/client';
import { Info } from './component';
import { useInfoProps } from './schema';

export class PluginInitializerBlockDataClient extends Plugin {
  async load() {
    this.app.addComponents({ Info })
    this.app.addScopes({ useInfoProps });
  }
}

export default PluginInitializerBlockDataClient;
```

Подробнее см. «[Глобальная регистрация компонента и области видимости](/plugin-samples/component-and-scope/global)».

#### 3.3 Проверка схемы блока

Схему можно проверить на временной странице или через примеры в документации. Ниже — временная страница:

```tsx | pure
import React from 'react';
import { Plugin, SchemaComponent, SchemaComponentOptions } from '@nocobase/client';
import { Info } from './component';
import { getInfoSchema, useInfoProps } from './schema';

export class PluginInitializerBlockDataClient extends Plugin {
  async load() {
    // ...
    this.app.router.add('admin.info-schema', {
      path: '/admin/info-schema',
      Component: () => {
        return <>
          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <SchemaComponent schema={{ properties: { test1: getInfoSchema({ collection: 'users' }) } }} />
          </div>

          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <SchemaComponent schema={{ properties: { test2: getInfoSchema({ collection: 'roles' }) } }} />
          </div>
        </>
      }
    })
  }
}

export default PluginInitializerBlockDataClient;
```

- [SchemaComponentOptions](https://client.docs.nocobase.com/core/ui-schema/schema-component#schemacomponentoptions): передача `components` и `scope` в схему; см. «[Локальная регистрация компонента и области видимости](/plugin-samples/component-and-scope/local)»;
- [SchemaComponent](https://client.docs.nocobase.com/core/ui-schema/schema-component#schemacomponent-1): отрисовка схемы.

Откройте [http://localhost:13000/admin/info-schema](http://localhost:13000/admin/info-schema) и проверьте страницу.

![20240526170053](https://static-docs.nocobase.com/20240526170053.png)

После проверки удалите тестовый маршрут и связанный код.

### 4. Определить элемент инициализатора схемы

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data/src/client/initializer/index.tsx`:

```tsx | pure
import React from 'react';
import { SchemaInitializerItemType, useSchemaInitializer } from '@nocobase/client'
import { CodeOutlined } from '@ant-design/icons';

import { getInfoSchema } from '../schema'
import { useT } from '../locale';
import { BlockName, BlockNameLowercase } from '../constants';

export const infoInitializerItem: SchemaInitializerItemType = {
  name: BlockNameLowercase,
  Component: 'DataBlockInitializer',
  useComponentProps() {
    const { insert } = useSchemaInitializer();
    const t = useT();
    return {
      title: t(BlockName),
      icon: <CodeOutlined />,
      componentType: BlockName,
      useTranslationHooks: useT,
      onCreateBlockSchema({ item }) {
        insert(getInfoSchema({ dataSource: item.dataSource, collection: item.name }))
      },
    };
  },
}
```

Ключевой компонент сценария — `DataBlockInitializer` (см. [документацию по инициализатору схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer)).

`infoInitializerItem`:

  - `Component`: в отличие от примера «[Добавление нового простого блока](/plugin-samples/schema-initializer/block-simple)», где задаётся поле `type`, здесь указан встроенный компонент строкой `'DataBlockInitializer'`; допустимы [оба способа](https://client.docs.nocobase.com/core/ui-schema/schema-initializer#two-ways-to-define-component-and-type);
  - `useComponentProps`: параметры для `DataBlockInitializer`;
    - `title`: заголовок пункта;
    - `icon`: значок; другие значки — в [наборе иконок Ant Design](https://ant.design/components/icon/);
    - `componentType`: тип создаваемого блока — `Info`;
    - `onCreateBlockSchema`: вызывается после выбора коллекции;
      - `item`: сведения о выбранной коллекции;
        - `item.name`: имя коллекции;
        - `item.dataSource`: источник данных коллекции;
    - [useSchemaInitializer](https://client.docs.nocobase.com/core/ui-schema/schema-initializer#useschemainitializer): методы вставки схемы (в т.ч. `insert`);
  - в схеме блока задаётся `"x-toolbar": "BlockSchemaToolbar"`: панель `BlockSchemaToolbar` показывает текущую коллекцию (часто в левом верхнем углу карточки блока) и обычно используется вместе с `DataBlockProvider`.

Подробнее об элементах инициализатора — в [документации по инициализатору схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer).

### 5. Реализация настроек схемы

#### 5.1 Определение настроек схемы

У готового блока обычно есть **настройки схемы**; в этом примере они не разбираются подробно — добавлена только операция **удаления** (`remove`).

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data/src/client/settings/index.ts`:

```ts
import { SchemaSettings } from "@nocobase/client";
import { BlockNameLowercase } from "../constants";

export const infoSettings = new SchemaSettings({
  name: `blockSettings:${BlockNameLowercase}`,
  items: [
    {
      type: 'remove',
      name: 'remove',
      componentProps: {
        removeParentsIfNoChildren: true,
        breakRemoveOn: {
          'x-component': 'Grid',
        },
      }
    }
  ]
})
```

#### 5.2 Регистрация настроек схемы

```ts
import { Plugin } from '@nocobase/client';
import { infoSettings } from './settings';

export class PluginInitializerBlockDataClient extends Plugin {
  async load() {
    // ...
    this.app.schemaSettingsManager.add(infoSettings)
  }
}

export default PluginInitializerBlockDataClient;
```

#### 5.3 Использование настроек схемы

В файле `packages/plugins/@nocobase-sample/plugin-initializer-block-data/src/client/schema/index.ts` добавьте в результат `getInfoSchema` поле `x-settings` со значением `infoSettings.name`.

```diff
+ import { infoSettings } from "../settings";

export function getInfoSchema({ dataSource = 'main', collection }) {
  return {
    type: 'void',
    'x-decorator': 'DataBlockProvider',
+   'x-settings': infoSettings.name,
    // ...
  }
}
```

### 6. Подключение к меню «Добавить блок»

В системе много точек с меню **«Добавить блок»**, но **идентификаторы инициализаторов** у них разные.

![img_v3_02b4_049b0a62-8e3b-420f-adaf-a6350d84840g](https://static-docs.nocobase.com/img_v3_02b4_049b0a62-8e3b-420f-adaf-a6350d84840g.jpg)

#### 6.1 Уровень страницы

Чтобы добавить пункт на уровне страницы, нужно знать имя инициализатора и путь вложенности. Имена ищут в исходниках ядра и плагинов, через инструменты разработчика или отладочные сборки; в исходной англоязычной документации этот шаг был помечен как незаполненный.

На скриншоте меню **«Добавить блок»** на странице соответствует инициализатору `page:addBlock`, подменю **«Блоки данных»** — ветке `dataBlocks`.

Измените файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data/src/client/index.tsx`:

```tsx | pure
import { Plugin } from '@nocobase/client';
import { Info } from './component';
import { useInfoProps } from './schema';
import { infoSettings } from './settings';
import { infoInitializerItem } from './initializer';

export class PluginDataBlockInitializerClient extends Plugin {
  async load() {
    this.app.addComponents({ Info });
    this.app.addScopes({ useInfoProps });

    this.app.schemaSettingsManager.add(infoSettings);

    this.app.schemaInitializerManager.addItem('page:addBlock', `dataBlocks.${infoInitializerItem.name}`, infoInitializerItem)
  }
}

export default PluginDataBlockInitializerClient;
```

<video controls width='100%' src="https://static-docs.nocobase.com/20240526170424_rec_.mp4"></video>

#### 6.2 Модальное окно «Добавить» в блоке таблицы

Тот же пункт добавляют в меню **«Добавить блок»** внутри модального окна **«Добавить»** у **блока таблицы**.

![img_v3_02b4_fc47fe3a-35a1-4186-999c-0b48e6e001dg](https://static-docs.nocobase.com/img_v3_02b4_fc47fe3a-35a1-4186-999c-0b48e6e001dg.jpg)

Для этого сценария инициализатор меню — `popup:addNew:addBlock`, подменю **«Блоки данных»** — снова `dataBlocks`.

Измените файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data/src/client/index.tsx`:

```diff
import { Plugin } from '@nocobase/client';
import { Plugin } from '@nocobase/client';
import { Info } from './component';
import { useInfoProps } from './schema';
import { infoSettings } from './settings';
import { infoInitializerItem } from './initializer';

export class PluginDataBlockInitializerClient extends Plugin {
  async load() {
    this.app.addComponents({ Info });
    this.app.addScopes({ useInfoProps });

    this.app.schemaSettingsManager.add(infoSettings);

    this.app.schemaInitializerManager.addItem('page:addBlock', `dataBlocks.${infoInitializerItem.name}`, infoInitializerItem)
+   this.app.schemaInitializerManager.addItem('popup:addNew:addBlock', `dataBlocks.${infoInitializerItem.name}`, infoInitializerItem)
  }
}

export default PluginDataBlockInitializerClient;

```

![img_v3_02b4_7062bfab-5a7b-439c-b385-92c5704b6b3g](https://static-docs.nocobase.com/img_v3_02b4_7062bfab-5a7b-439c-b385-92c5704b6b3g.jpg)

#### 6.3 Мобильная страница

> Сначала включите плагин мобильного клиента; см. «[Установка и активация плагинов](/get-started/installation/plugins)».

Тот же пункт можно добавить в меню **«Добавить блок»** на мобильной странице; способ узнать имя инициализатора тот же, что в п. 6.1.

Измените файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data/src/client/index.tsx`:

```diff
import { Plugin } from '@nocobase/client';
import { Info } from './component';
import { useInfoProps } from './schema';
import { infoSettings } from './settings';
import { infoInitializerItem } from './initializer';

export class PluginDataBlockInitializerClient extends Plugin {
  async load() {
    this.app.addComponents({ Info });
    this.app.addScopes({ useInfoProps });

    this.app.schemaSettingsManager.add(infoSettings);

    this.app.schemaInitializerManager.addItem('page:addBlock', `dataBlocks.${infoInitializerItem.name}`, infoInitializerItem)
    this.app.schemaInitializerManager.addItem('popup:addNew:addBlock', `dataBlocks.${infoInitializerItem.name}`, infoInitializerItem)
+   this.app.schemaInitializerManager.addItem('mobilePage:addBlock', `dataBlocks.${infoInitializerItem.name}`, infoInitializerItem)
  }
}

export default PluginDataBlockInitializerClient;
```

При необходимости добавьте вызовы `addItem` для других точек меню **«Добавить блок»**, заранее узнав их идентификаторы.

## Упаковка и загрузка в производственную среду

Согласно разделу «[Сборка и упаковка плагина](/plugin-development/write-your-first-plugin#build-and-package-plugin)» в руководстве по разработке плагинов, можно собрать архив и установить его в производственной среде.

Если вы клонировали исходный код, вам необходимо сначала выполнить полную сборку, чтобы также построить зависимости плагина.

```bash
yarn build
```

Если вы использовали `create-nocobase-app` чтобы создать проект, вы можете напрямую выполнить:

```bash
yarn build @nocobase-sample/plugin-initializer-block-data --tar
```

В каталоге `storage/tar/@nocobase-sample/plugin-initializer-block-data.tar.gz` появится архив; установить плагин можно по инструкции [установки плагинов из файла](/get-started/installation/plugins).
