# Добавление простого блока

## Сценарий

В NocoBase во многих местах есть меню **«Добавить блок»**. Часть пунктов связана с **таблицами данных** — это **блоки данных**; остальные, без привязки к таблице, — **простые блоки**.

![img_v3_02b4_a4529308-62e3-4fa7-be4d-5dcae332c49g](https://static-docs.nocobase.com/img_v3_02b4_a4529308-62e3-4fa7-be4d-5dcae332c49g.jpg)

Встроенных типов блоков может не хватать — тогда добавляют собственные блоки под задачу. Здесь речь именно о **простых блоках** (без загрузки коллекции).

## Пример

В примере создаётся тип блока с изображением и пункт меню для него добавляется в **«Добавить блок»** на **странице**, в **блоке таблицы** (в т.ч. в модальном окне) и на **мобильной странице**.

Пример показывает работу **инициализатора схемы**. Про расширение блоков см. «[Расширение блока](/plugin-samples/block)».

Полный исходный код — в [репозитории примеров плагинов](https://github.com/nocobase/plugin-samples/tree/main/packages/plugins/%40nocobase-sample/plugin-initializer-block-simple).

<video width="100%" controls="">
  <source src="https://static-docs.nocobase.com/20240522-181816.mp4" type="video/mp4" />
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
yarn pm create @nocobase-sample/plugin-initializer-block-simple
yarn pm enable @nocobase-sample/plugin-initializer-block-simple
```

Затем запустите проект:

```bash
yarn dev
```

После входа в систему откройте [http://localhost:13000/admin/pm/list/local/](http://localhost:13000/admin/pm/list/local/) и убедитесь, что плагин установлен и включён.

## Реализация

Прежде чем повторять пример, полезно прочитать:

- [учебник по инициализатору схемы](/development/client/ui-schema/initializer): как добавлять в интерфейс блоки, поля, операции и другое;
- [API инициализатора схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer): то же на уровне API;
- [пользовательская схема интерфейса](/development/client/ui-schema/what-is-ui-schema): структура схемы и назначение полей;
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

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-simple/src/client/constants.ts`:

```ts
export const BlockName = 'Image';
export const BlockNameLowercase = BlockName.toLowerCase();
```

### 2. Реализация компонента блока

#### 2.1 Определите компонент блока

Нужен компонент блока с картинкой; в примере он назван **Image**.

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-simple/src/client/component/Image.tsx`:

```tsx | pure
import React, { FC } from 'react';
import { withDynamicSchemaProps } from '@nocobase/client';
import { BlockName } from '../constants';

export const Image: FC<{ height?: number }> = withDynamicSchemaProps(({ height = 500 }) => {
  return <div style={{ height }}>
    <img
      style={{ width: '100%', height: '100%', objectFit: 'cover' }}
      src="https://picsum.photos/2000/500"
    />
  </div>
}, { displayName: BlockName })
```

Компонент **Image** — это функциональный компонент, обёрнутый в `withDynamicSchemaProps`. [withDynamicSchemaProps](/development/client/ui-schema/what-is-ui-schema#x-component-props-和-x-use-component-props) — компонент высшего порядка для динамических свойств из схемы.

Без `withDynamicSchemaProps` **Image** остаётся обычным функциональным компонентом.

Экспортируйте его в `packages/plugins/@nocobase-sample/plugin-initializer-block-simple/src/client/component/index.ts`:

```tsx | pure
export * from './Image';
```

#### 2.2 Зарегистрируйте компонент блока

Зарегистрируйте **Image** в приложении из плагина:

```tsx | pure
import { Plugin } from '@nocobase/client';
import { Image } from './component'

export class PluginInitializerBlockSimpleClient extends Plugin {
  async load() {
    this.app.addComponents({ Image })
  }
}

export default PluginInitializerBlockSimpleClient;
```

#### 2.3 Проверка компонента блока

Есть два подхода:

- **Временная тестовая страница** — зарегистрировать маршрут и вывести **Image** с разными параметрами;
- **Примеры в документации** — поднять документацию командой `yarn doc plugins/@nocobase-sample/plugin-initializer-block-simple` и проверить виджеты в статьях (по желанию).

Ниже — вариант с **временной страницей**. Добавьте маршрут и несколько экземпляров **Image** с разной высотой:

```tsx | pure
import React from 'react';
import { Plugin } from '@nocobase/client';
import { Image } from './component'

export class PluginInitializerBlockSimpleClient extends Plugin {
  async load() {
    this.app.addComponents({ Image })

    this.app.router.add('admin.image-component', {
      path: '/admin/image-component',
      Component: () => {
        return <>
          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <Image />
          </div>

          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <Image height={400} />
          </div>
        </>
      }
    })
  }
}

export default PluginInitializerBlockSimpleClient;
```

Откройте `http://localhost:13000/admin/image-component` и проверьте отображение.

![20240526165057](https://static-docs.nocobase.com/20240526165057.png)

После проверки код тестовой страницы удалите.

### 3. Определите схему блока

#### 3.1 Определение схемы блока

Динамические страницы NocoBase строятся по **схеме**, поэтому нужно описать схему, которую потом вставит инициализатор. Полезно ещё раз пройти:

- [пользовательская схема интерфейса](/development/client/ui-schema/what-is-ui-schema): структура схемы и назначение полей.

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-simple/src/client/schema/index.ts`:

```tsx | pure
import { ISchema } from "@nocobase/client";
import { BlockName, BlockNameLowercase } from "../constants";

export const imageSchema: ISchema = {
  type: 'void',
  'x-component': 'CardItem',
  properties: {
    [BlockNameLowercase]: {
      'x-component': BlockName,
    }
  }
};
```

Поле `imageSchema`:

- `type: 'void'`: узел интерфейса без данных коллекции;
- `'x-component': 'CardItem'`: оболочка [CardItem](https://client.docs.nocobase.com/components/card-item) — карточка со стилями, макетом и перетаскиванием;
- внутри `properties` узел с `'x-component': BlockName` — наш компонент **Image**.

Эта схема по смыслу соответствует такому дереву React:

```tsx | pure
<CardItem>
  <Image />
</CardItem>
```

#### 3.2 Проверка схемы блока

Как и для компонента, можно проверить схему на временной странице или в документации. Ниже — временная страница:

```tsx | pure
import React from 'react';
import { Plugin, SchemaComponent } from '@nocobase/client';
import { Image } from './component'
import { imageSchema } from './schema'

export class PluginInitializerBlockSimpleClient extends Plugin {
  async load() {
    this.app.addComponents({ Image })

    this.app.router.add('admin.image-schema', {
      path: '/admin/image-schema',
      Component: () => {
        return <div style={{ marginTop: 20, marginBottom: 20 }}>
          <SchemaComponent schema={{ properties: { test: imageSchema } }} />
        </div>
      }
    })
  }
}

export default PluginInitializerBlockSimpleClient;
```

Подробнее о **SchemaComponent** см. [документацию SchemaComponent](https://client.docs.nocobase.com/core/ui-schema/schema-component#schemacomponent-1).

Откройте [http://localhost:13000/admin/image-schema](http://localhost:13000/admin/image-schema) и проверьте результат.

![20240526165408](https://static-docs.nocobase.com/20240526165408.png)

После проверки код тестового маршрута удалите.

### 4. Элемент инициализатора схемы

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-simple/src/client/initializer/index.ts`:

```ts
import { SchemaInitializerItemType, useSchemaInitializer } from '@nocobase/client';

import { useT } from '../locale';
import { imageSchema } from '../schema';
import { BlockName, BlockNameLowercase } from '../constants';

export const imageInitializerItem: SchemaInitializerItemType = {
  type: 'item',
  name: BlockNameLowercase,
  icon: 'FileImageOutlined',
  useComponentProps() {
    const { insert } = useSchemaInitializer();
    const t = useT()
    return {
      title: t(BlockName),
      onClick: () => {
        insert(imageSchema);
      },
    };
  },
}
```

- `type: 'item'`: пункт меню, по нажатию вызывается обработчик;
- `name`: уникальный идентификатор элемента (в т.ч. для операций со схемой);
- `icon`: значок; набор значков Ant Design — в разделе [Icons](https://ant.design/components/icon);
- `useComponentProps`: возвращает объект с полями `title` (подпись) и `onClick` (обработчик нажатия);
- [useSchemaInitializer()](https://client.docs.nocobase.com/core/ui-schema/schema-initializer#useschemainitializer): доступ к контексту `SchemaInitializerContext`;
  - `insert`: вставка фрагмента схемы.

Дополнительно о полях элемента см. [документацию по типам элементов инициализатора схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer#built-in-components-and-types).

### 5. Настройки схемы

#### 5.1 Определение настроек схемы

У полноценного блока обычно есть **настройки схемы** для свойств и команд. В этом примере достаточно одной операции **удаления** (`remove`).

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-simple/src/client/settings/index.ts`:

```ts | pure
import { SchemaSettings } from "@nocobase/client";
import { BlockNameLowercase } from "../constants";

export const imageSettings = new SchemaSettings({
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
});
```

- **`componentProps`** (для пункта `remove`):
  - `removeParentsIfNoChildren`: удалять ли родителя, если дочерних узлов не осталось;
  - `breakRemoveOn`: на каком узле остановить каскадное удаление вверх. Меню **«Добавить блок»** оборачивает вложенность в **Grid**, поэтому задано `breakRemoveOn: { 'x-component': 'Grid' }` — при удалении до **Grid** выше по дереву удаление не поднимается.

#### 5.2 Регистрация настроек схемы

```ts
import { Plugin } from '@nocobase/client';
import { imageSettings } from './settings';

export class PluginInitializerBlockSimpleClient extends Plugin {
  async load() {
    // ...
    this.app.schemaSettingsManager.add(imageSettings)
  }
}

export default PluginInitializerBlockSimpleClient;
```

#### 5.3 Использование настроек схемы

В файле `packages/plugins/@nocobase-sample/plugin-initializer-block-simple/src/client/schema/index.ts` у `imageSchema` добавьте `x-settings`:

```diff
+ import { imageSettings } from "../settings";

const imageSchema: ISchema = {
  type: 'void',
  'x-decorator': 'CardItem',
+ 'x-settings': imageSettings.name,
  // ...
};
```

### 6. Подключение к меню «Добавить блок»

В системе много точек с меню **«Добавить блок»**, но **идентификаторы инициализаторов** у них разные.

![img_v3_02b4_049b0a62-8e3b-420f-adaf-a6350d84840g](https://static-docs.nocobase.com/img_v3_02b4_049b0a62-8e3b-420f-adaf-a6350d84840g.jpg)

#### 6.1 Уровень страницы

Чтобы добавить пункт на уровне страницы, нужно знать имя инициализатора и путь вложенности. Имена ищут в исходниках ядра и плагинов, через инструменты разработчика или отладочные сборки; в исходной англоязычной документации этот шаг был помечен как незаполненный.

На скриншоте меню **«Добавить блок»** на странице соответствует инициализатору `page:addBlock`, подменю **«Другие блоки»** — ветке `otherBlocks`.

Измените файл `packages/plugins/@nocobase-sample/plugin-initializer-block-simple/src/client/index.tsx`:

```tsx | pure
import { Plugin } from '@nocobase/client';

import { Image } from './component'
import { imageSettings } from './settings';
import { imageInitializerItem } from './initializer';

export class PluginInitializerBlockSimpleClient extends Plugin {
  async load() {
    this.app.addComponents({ Image })
    this.app.schemaSettingsManager.add(imageSettings)
    this.app.schemaInitializerManager.addItem('page:addBlock', `otherBlocks.${imageInitializerItem.name}`, imageInitializerItem)
  }
}

export default PluginInitializerBlockSimpleClient;
```

Сначала в систему регистрируется компонент **Image**, чтобы значение `x-component: 'Image'` в `imageSchema` находило реализацию. Подробнее — «[Глобальная регистрация компонента и области видимости](/plugin-samples/component-and-scope/global)».

Затем настройки регистрируются через [app.schemaSettingsManager.add](https://client.docs.nocobase.com/core/ui-schema/schema-settings-manager#schemasettingsmanageradd).

Пункт меню добавляется через [app.schemaInitializerManager.addItem](https://client.docs.nocobase.com/core/ui-schema/schema-initializer-manager#schemainitializermanageradditem): первый аргумент — инициализатор меню **«Добавить блок»** на странице (`page:addBlock`), путь `otherBlocks.${...}` — ветка **«Другие блоки»**.

Наведите курсор на **«Добавить блок»** — появится новый тип **Image**; нажмите его, чтобы вставить блок.

<video width="100%" controls="">
  <source src="https://static-docs.nocobase.com/20240522-175523.mp4" type="video/mp4" />
</video>

#### 6.2 Модальное окно «Добавить» в блоке таблицы

Тот же пункт добавляют в меню **«Добавить блок»** внутри модального окна **«Добавить»** у **блока таблицы**.

![img_v3_02b4_fc47fe3a-35a1-4186-999c-0b48e6e001dg](https://static-docs.nocobase.com/img_v3_02b4_fc47fe3a-35a1-4186-999c-0b48e6e001dg.jpg)

Для этого сценария инициализатор меню — `popup:addNew:addBlock`, подменю **«Другие блоки»** — снова `otherBlocks`.

Измените файл `packages/plugins/@nocobase-sample/plugin-initializer-block-simple/src/client/index.tsx`:

```diff
export class PluginInitializerBlockSimpleClient extends Plugin {
  async load() {
    this.app.addComponents({ Image })
    this.app.schemaSettingsManager.add(imageSettings)

    this.app.schemaInitializerManager.addItem('page:addBlock', `otherBlocks.${imageInitializerItem.name}`, imageInitializerItem)
+   this.app.schemaInitializerManager.addItem('popup:addNew:addBlock', `otherBlocks.${imageInitializerItem.name}`, imageInitializerItem)
  }
}
```

![img_v3_02b4_7062bfab-5a7b-439c-b385-92c5704b6b3g](https://static-docs.nocobase.com/img_v3_02b4_7062bfab-5a7b-439c-b385-92c5704b6b3g.jpg)

#### 6.3 Мобильная страница

> Сначала включите плагин мобильного клиента; см. «[Установка и активация плагинов](/get-started/installation/plugins)».

Тот же пункт можно добавить в меню **«Добавить блок»** на мобильной странице; способ узнать имя инициализатора тот же, что в п. 6.1.

Измените файл `packages/plugins/@nocobase-sample/plugin-initializer-block-simple/src/client/index.tsx`:

```diff
export class PluginInitializerBlockSimpleClient extends Plugin {
  async load() {
    this.app.addComponents({ Image })
    this.app.schemaSettingsManager.add(imageSettings)

    this.app.schemaInitializerManager.addItem('page:addBlock', `otherBlocks.${imageInitializerItem.name}`, imageInitializerItem)
    this.app.schemaInitializerManager.addItem('popup:addNew:addBlock', `otherBlocks.${imageInitializerItem.name}`, imageInitializerItem)
+   this.app.schemaInitializerManager.addItem('mobilePage:addBlock', `otherBlocks.${imageInitializerItem.name}`, imageInitializerItem)
  }
}
```

![img_v3_02b4_ec873b25-5a09-4f3a-883f-1d722035799g](https://static-docs.nocobase.com/img_v3_02b4_ec873b25-5a09-4f3a-883f-1d722035799g.jpg)

При необходимости добавьте вызовы `addItem` для других точек меню **«Добавить блок»**, заранее узнав их идентификаторы.

## Упаковка и загрузка в производственную среду

Согласно разделу «[Сборка и упаковка плагина](/plugin-development/write-your-first-plugin#build-and-package-plugin)» в руководстве по разработке плагинов, можно собрать архив и установить его в производственной среде.

Если вы клонировали исходный код, вам необходимо сначала выполнить полную сборку, чтобы также построить зависимости плагина.

```bash
yarn build
```

Если вы использовали `create-nocobase-app` чтобы создать проект, вы можете напрямую выполнить:

```bash
yarn build @nocobase-sample/plugin-initializer-block-simple --tar
```

В каталоге `storage/tar/@nocobase-sample/plugin-initializer-block-simple.tar.gz` появится архив; установить плагин можно по инструкции [установки плагинов из файла](/get-started/installation/plugins).
