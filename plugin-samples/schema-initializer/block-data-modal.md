# Добавление блока данных с помощью модального окна

## Сценарий

Во многих случаях перед тем, как создать блок, нужно сначала задать параметры конфигурации. Например:
- блок **канбана** — после нажатия нужно выбрать **поле группировки** и **поле сортировки**;
- блок **календаря** — сначала выбрать **поле заголовка**, **поле даты начала** и **поле даты окончания**;
- блок **диаграммы** — сначала задать параметры, связанные с графиком.

<br />

<video width="100%" controls="">
  <source src="https://static-docs.nocobase.com/20240529223753_rec_.mp4" type="video/mp4" />
</video>

## Пример

В этом примере создаётся блок **временной шкалы** (`Timeline`) на основе [компонента Timeline](https://ant.design/components/timeline) библиотеки Ant Design; перед созданием блока пользователь выбирает **поле времени** и **поле заголовка**.

Пример в первую очередь показывает работу **инициализатора схемы**. Про расширение блоков см. статью «[Расширение блока](/plugin-samples/block)».

Полный исходный код приведён в [репозитории примеров плагинов](https://github.com/nocobase/plugin-samples/tree/main/packages/plugins/%40nocobase-sample/plugin-initializer-block-data-modal).

<video width="100%" controls="">
  <source src="https://static-docs.nocobase.com/20240529223457_rec_.mp4" type="video/mp4" />
</video>

## Инициализация плагина

Следуйте документации «[Написать первый плагин](/plugin-development/write-your-first-plugin)»: если проекта ещё нет, сначала создайте его; если проект уже есть или вы клонировали исходный код, этот шаг можно пропустить.

```bash
yarn create nocobase-app my-nocobase-app -d postgres
cd my-nocobase-app
yarn install
yarn nocobase install
```

Затем инициализируйте плагин и добавьте его в систему:

```bash
yarn pm create @nocobase-sample/plugin-initializer-block-data-modal
yarn pm enable @nocobase-sample/plugin-initializer-block-data-modal
```

Затем запустите проект:

```bash
yarn dev
```

После входа в систему откройте [http://localhost:13000/admin/pm/list/local/](http://localhost:13000/admin/pm/list/local/) и убедитесь, что плагин установлен и включён.

## Реализация

Прежде чем повторять этот пример, необходимо ознакомиться с базовыми материалами:

- [компонент Timeline](https://ant.design/components/timeline) библиотеки Ant Design;
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

Сначала задаётся имя блока — оно будет использоваться в нескольких местах.

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/client/constants.ts`:

```ts
export const BlockName = 'Timeline';
export const BlockNameLowercase = BlockName.toLowerCase();
```

### 2. Реализация блочного компонента

#### 2.1 Определить компонент блока

Речь идёт о компоненте блока **временной шкалы** (`Timeline`) со следующими требованиями.

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/client/component/Timeline.tsx` со следующим содержимым:

```tsx | pure
import React, { FC } from 'react';
import { Timeline as AntdTimeline, TimelineProps as AntdTimelineProps, Spin } from 'antd';
import { withDynamicSchemaProps } from "@nocobase/client";
import { BlockName } from '../constants';

export interface TimelineProps {
  data?: AntdTimelineProps['items'];
  loading?: boolean;
}

export const Timeline: FC<TimelineProps> = withDynamicSchemaProps((props) => {
  const { data, loading } = props;
  if (loading) return <div style={{ height: 100, textAlign: 'center' }}><Spin /></div>
  return <AntdTimeline mode='left' items={data}></AntdTimeline>
}, { displayName: BlockName });
```

Компонент `Timeline` — это обёртка над `withDynamicSchemaProps`; среди входных параметров по сути два поля:

- `loading`: признак загрузки данных;
- `data`: массив элементов для свойства `items` компонента временной шкалы.

[withDynamicSchemaProps](/development/client/ui-schema/what-is-ui-schema#x-component-props-和-x-use-component-props) — компонент высшего порядка для динамических свойств в схеме.

#### 2.2 Регистрация компонента блока

Компонент `Timeline` нужно зарегистрировать в приложении из плагина.

```tsx | pure
import { Plugin } from '@nocobase/client';
import { Timeline } from './component';

export class PluginInitializerBlockDataModalClient extends Plugin {
  async load() {
    this.app.addComponents({ Timeline })
  }
}

export default PluginInitializerBlockDataModalClient;
```

#### 2.3 Проверка компонента блока

Проверить компонент можно двумя способами:

- **Временная страница:** создать маршрут, отрисовать на нём `Timeline` и убедиться, что поведение соответствует задаче.
- **Примеры в документации:** запустить `yarn doc plugins/@nocobase-sample/plugin-initializer-block-data-modal` и проверить схему через примеры в документации (раздел в планах).

Ниже для примера используется **временная страница**: добавляется маршрут и один или несколько экземпляров `Timeline` с разными параметрами свойств.

```tsx | pure
import { Plugin } from '@nocobase/client';
import { Timeline } from './component';
import React from 'react';

export class PluginInitializerBlockDataModalClient extends Plugin {
  async load() {
    this.app.addComponents({ Timeline })

    this.app.router.add('admin.timeline-block-component', {
      path: '/admin/timeline-block-component',
      Component: () => {
        return <>
          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <Timeline
              data={[
                {
                  label: '2015-09-01',
                  children: 'user1',
                },
                {
                  label: '2015-09-02',
                  children: 'user2',
                },
                {
                  label: '2015-09-03',
                  children: 'user3',
                },
              ]} />
          </div>
          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <Timeline loading={true} />
          </div>
        </>
      }
    })
  }
}

export default PluginInitializerBlockDataModalClient;
```

Затем откройте [http://localhost:13000/admin/timeline-block-component](http://localhost:13000/admin/timeline-block-component) и проверьте содержимое тестовой страницы.

![20240529210122](https://static-docs.nocobase.com/20240529210122.png)

После проверки тестовую страницу необходимо удалить.

### 3. Определить форму конфигурации

По заданию после выбора **коллекции** (таблицы данных) нужно настроить **поле времени** и **поле заголовка**; для этого определяется форма `TimelineInitializerConfigForm`.

#### 3.1 Определение компонента формы конфигурации

Сначала нужно прочитать:

- [компонент действия](https://client.docs.nocobase.com/components/action);
- [Action.Modal](https://client.docs.nocobase.com/components/action#actionmodal) — модальное окно;
- [ActionContextProvider](https://client.docs.nocobase.com/components/action#actioncontext) — контекст действия;
- [SchemaComponent](https://client.docs.nocobase.com/core/ui-schema/schema-component#schemacomponent-1) — отрисовка схемы.

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/client/initializer/ConfigForm.tsx` со следующим содержимым:

```tsx | pure
import React, { FC, useMemo } from "react";
import { ISchema } from '@formily/react';
import { ActionContextProvider, SchemaComponent, useApp, CollectionFieldOptions } from '@nocobase/client';
import { useT } from "../locale";

const createSchema = (fields: CollectionFieldOptions, t: ReturnType<typeof useT>): ISchema => {
  // полная реализация — в разделе 3.2 ниже
}

interface TimelineConfigFormValues {
  timeField: string;
  titleField: string;
}

export interface TimelineConfigFormProps {
  collection: string;
  dataSource?: string;
  onSubmit: (values: TimelineConfigFormValues) => void;
  visible: boolean;
  setVisible: (visible: boolean) => void;
}

export const TimelineInitializerConfigForm: FC<TimelineConfigFormProps> = ({ visible, setVisible, collection, dataSource, onSubmit }) => {
  const app = useApp();
  const fields = useMemo(() => app.getCollectionManager(dataSource).getCollection(collection).getFields(), [collection, dataSource])
  const t = useT();
  const schema = useMemo(() => createSchema(fields, t), [fields]);

  return <ActionContextProvider value={{ visible, setVisible }}>
    <SchemaComponent schema={schema}  />
  </ActionContextProvider>
}
```

Компонент `TimelineInitializerConfigForm` принимает 4 параметра:

- `visible`: видимость модального окна;
- `setVisible`: смена видимости;
- `collection`: имя коллекции (таблицы данных);
- `dataSource`: имя **источника данных** (глоссарий: «Источник данных»);
- `onSubmit`: обработчик отправки формы.

Значения `collection` и `dataSource` появляются после выбора таблицы в инициализаторе, поэтому на форме они задаются динамически.

- [app](https://client.docs.nocobase.com/core/application/application): экземпляр приложения через [useApp()](https://client.docs.nocobase.com/core/application/application#useapp);
- [app.getCollectionManager](https://client.docs.nocobase.com/core/application/application##appgetcollectionmanager): получить [CollectionManager](https://client.docs.nocobase.com/core/data-source/collection-manager);
- [getCollection](https://client.docs.nocobase.com/core/data-source/collection-manager#getcollectionpath): получить коллекцию;
- [getFields](https://client.docs.nocobase.com/core/data-source/collection#collectiongetfieldspredicate): получить поля коллекции.

[ActionContextProvider](https://client.docs.nocobase.com/components/action#actioncontext) передаёт `visible` и `setVisible` вниз по дереву; [SchemaComponent](https://client.docs.nocobase.com/core/ui-schema/schema-component#schemacomponent-1) отрисовывает схему.

#### 3.2 Реализация схемы формы конфигурации

Сначала необходимо ознакомиться:

- [Форма V2](https://client.docs.nocobase.com/components/form-v2) — компонент формы;
- [Выбор](https://client.docs.nocobase.com/components/action#select) — поле выбора.

```tsx | pure
const useCloseActionProps = () => {
  const { setVisible } = useActionContext();
  return {
    type: 'default',
    onClick() {
      setVisible(false);
    },
  };
};

const useSubmitActionProps = (onSubmit: (values: TimelineConfigFormValues) => void) => {
  const { setVisible } = useActionContext();
  const form = useForm<TimelineConfigFormValues>();

  return {
    type: 'primary',
    async onClick() {
      await form.submit();
      const values = form.values;
      onSubmit(values);
      setVisible(false);
    },
  };
};

const createSchema = (fields: CollectionFieldOptions[]): ISchema => {
  return {
    type: 'void',
    name: uid(),
    'x-component': 'Action.Modal',
    'x-component-props': {
      width: 600,
    },
    'x-decorator': 'FormV2',
    properties: {
      titleField: {
        type: 'string',
        title: 'Title Field',
        required: true,
        enum: fields.map(item => ({ label: item.uiSchema?.title || item.name, value: item.name })),
        'x-decorator': 'FormItem',
        'x-component': 'Select',
      },
      timeField: {
        type: 'string',
        title: 'Time Field',
        required: true,
        enum: fields.filter(item => item.type === 'date').map(item => ({ label: item.uiSchema?.title || item.name, value: item.name })),
        'x-decorator': 'FormItem',
        'x-component': 'Select',
      },
      footer: {
        type: 'void',
        'x-component': 'Action.Modal.Footer',
        properties: {
          close: {
            title: 'Close',
            'x-component': 'Action',
            'x-component-props': {
              type: 'default',
            },
            'x-use-component-props': 'useCloseActionProps',
          },
          submit: {
            title: 'Submit',
            'x-component': 'Action',
            'x-use-component-props': 'useSubmitActionProps',
          },
        },
      },
    }
  };
}
```

Определена функция `createSchema`: по списку полей коллекции `fields` строится схема модальной формы.

В модальном окне — форма с двумя полями выбора (**поле заголовка** и **поле времени**) и кнопками **«Закрыть»** и **«Отправить»** (в коде — строки-ключи локализации `Close` / `Submit`).

- для кнопок используются хуки, поэтому задействовано поле [x-use-component-props](/development/client/ui-schema/what-is-ui-schema#x-component-props-和-x-use-component-props);
- **поле заголовка**: в списке доступны все поля коллекции;
- **поле времени**: только поля с типом `date`.

Далее в `TimelineInitializerConfigForm` нужно передать хуки `useSubmitActionProps` и `useCloseActionProps` в **область видимости** [`SchemaComponent`](https://client.docs.nocobase.com/core/ui-schema/schema-component#schemacomponent-1), см. статью «[Локальная регистрация компонента и области видимости](/plugin-samples/component-and-scope/local)».

```diff
-   <SchemaComponent schema={schema}/>
+   <SchemaComponent schema={schema} scope={{ useSubmitActionProps: useSubmitActionProps.bind(null, onSubmit), useCloseActionProps }} />
```

#### 3.3 Проверка формы конфигурации

```tsx | pure
import { Plugin } from '@nocobase/client';
import { Timeline } from './component';
import React, { useState } from 'react';
import { TimelineInitializerConfigForm } from './initializer/ConfigForm';

export class PluginInitializerBlockDataModalClient extends Plugin {
  async load() {
    this.app.addComponents({ Timeline })

    this.app.router.add('admin.timeline-config-form', {
      path: '/admin/timeline-config-form',
      Component: () => {
        const [visible, setVisible] = useState(true);
        function onSubmit(values) {
          console.log(values);
        }
        return <>
          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <TimelineInitializerConfigForm visible={visible} onSubmit={onSubmit} setVisible={setVisible} collection='users' />
          </div>
        </>
      }
    })
  }
}

export default PluginInitializerBlockDataModalClient;
```

Затем откройте [http://localhost:13000/admin/timeline-config-form](http://localhost:13000/admin/timeline-config-form) и проверьте тестовую страницу.

<video width="100%" controls="">
  <source src="https://static-docs.nocobase.com/20240529215127_rec_.mp4" type="video/mp4" />
</video>

После проверки тестовую страницу необходимо удалить.

### 4. Определить схему блока

#### 4.1 Определение схемы блока

Интерфейс NocoBase строится по схеме; нужно описать фрагмент схемы, который затем вставляет инициализатор и отображает блок **временной шкалы** (`Timeline`). Перед реализацией раздела полезно прочитать:

- [протокол пользовательской схемы интерфейса](/development/client/ui-schema/what-is-ui-schema): структура схемы и поля;
- [DataBlockProvider](https://client.docs.nocobase.com/core/data-block/data-block-provider): провайдер **блока данных**.

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/client/schema/index.tsx`:

```ts
import { useDataBlockProps, useDataBlockRequest } from "@nocobase/client";
import { TimelineProps } from '../component';
import { BlockName, BlockNameLowercase } from "../constants";

interface GetTimelineSchemaOptions {
  dataSource?: string;
  collection: string;
  titleField: string;
  timeField: string;
}

export function getTimelineSchema(options: GetTimelineSchemaOptions) {
  return {
    type: 'void',
    "x-toolbar": "BlockSchemaToolbar",
    'x-decorator': 'DataBlockProvider',
    'x-decorator-props': {
      dataSource,
      collection,
      action: 'list',
      params: {
        sort: `-${timeField}`
      },
      [BlockNameLowercase]: {
        titleField,
        timeField,
      }
    },
    'x-component': 'CardItem',
    properties: {
      [BlockNameLowercase]: {
        type: 'void',
        'x-component': BlockName,
        'x-use-component-props': 'useTimelineProps',
      }
    }
  }
}

export function useTimelineProps(): TimelineProps {
  const dataProps = useDataBlockProps();
  const props = dataProps[BlockNameLowercase];
  const { loading, data } = useDataBlockRequest<any[]>();
  return {
    loading,
    data: data?.data?.map((item) => ({
      label: item[props.timeField],
      children: item[props.titleField],
    }))
  }
}
```

Здесь нужно объяснить 2 момента:

`getTimelineSchema()` принимает `dataSource`, `collection`, `titleField`, `timeField` и возвращает схему для отображения блока `Timeline`:

  - `type: 'void'`: узел без собственного значения в данных формы;
  - `x-decorator: 'DataBlockProvider'`: провайдер блока данных; подробнее — [DataBlockProvider](https://client.docs.nocobase.com/core/data-block/data-block-provider);
  - `x-decorator-props`: параметры провайдера;
  - `dataSource`: источник данных;
  - `collection`: коллекция;
  - `action: 'list'`: операция чтения списка записей;
  - `params: { sort }`: параметры запроса; здесь сортировка по `timeField` по убыванию; см. также [useRequest](https://client.docs.nocobase.com/core/request#userequest);
  - `x-component: 'CardItem'`: [компонент карточки блока](https://client.docs.nocobase.com/components/card-item); блоки оборачиваются в карточку для оформления, макета и перетаскивания;
  - вложенный узел с `'x-component': 'Timeline'`: наш компонент блока;
  - `'x-use-component-props': 'useTimelineProps'`: динамические параметры для `Timeline`; в базе хранится **строка** с именем хука.

`useTimelineProps()` — хук, возвращающий динамические параметры для временной шкалы:

  - [useDataBlockProps](https://client.docs.nocobase.com/core/data-block/data-block-provider#usedatablockprops): параметры [DataBlockProvider](https://client.docs.nocobase.com/core/data-block/data-block-provider), то есть значение `x-decorator-props`;
  - [useDataBlockRequest](https://client.docs.nocobase.com/core/data-block/data-block-request-provider#usedatablockrequest): запрос данных блока, который настраивает [DataBlockProvider](https://client.docs.nocobase.com/core/data-block/data-block-provider).

Схема по смыслу соответствует такому дереву React:

```tsx | pure
<DataBlockProvider collection={collection} dataSource={dataSource} action='list' params={{ sort: `-${timeField}` }} timeline={{ titleField, timeField }}>
  <CardItem>
    <Timeline {...useTimelineProps()} />
  </CardItem>
</DataBlockProvider>
```

#### 4.2 Регистрация области видимости

В файле `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/client/index.tsx` зарегистрируйте `useTimelineProps` через `addScopes`, чтобы по полю `x-use-component-props` находился соответствующий хук.

```tsx | pure
import { Plugin } from '@nocobase/client';
import { Timeline } from './component';
import { useTimelineProps } from './schema';

export class PluginInitializerBlockDataModalClient extends Plugin {
  async load() {
    this.app.addComponents({ Timeline })
    this.app.addScopes({ useTimelineProps });
  }
}

export default PluginInitializerBlockDataModalClient;
```

Подробнее об областях видимости см. «[Глобальная регистрация компонента и области видимости](/plugin-samples/component-and-scope/global)».

#### 4.3 Проверка схемы блока

Как и для компонента, схему можно проверить на **временной странице** или через примеры в документации. Ниже — вариант с временной страницей:

```tsx | pure
import { Plugin, SchemaComponent } from '@nocobase/client';
import { Timeline, getTimelineSchema, useTimelineProps } from './component';
import React from 'react';

export class PluginInitializerBlockDataModalClient extends Plugin {
  async load() {
    // ...

    this.app.router.add('admin.timeline-schema', {
      path: '/admin/timeline-schema',
      Component: () => {
        return <>
          <div style={{ marginTop: 20, marginBottom: 20 }}>
            <SchemaComponent schema={{ properties: { test1: getTimelineSchema({ collection: 'users' })({ timeField: 'createdAt', titleField: 'nickname' }) } }} />
          </div>
        </>
      }
    })
  }
}

export default PluginInitializerBlockDataModalClient;

```

- [SchemaComponentOptions](https://client.docs.nocobase.com/core/ui-schema/schema-component#schemacomponentoptions): передача `components` и `scope` в схему; см. «[Локальная регистрация компонента и области видимости](/plugin-samples/component-and-scope/local)»;
- [SchemaComponent](https://client.docs.nocobase.com/core/ui-schema/schema-component#schemacomponent-1): отрисовка схемы.

Откройте [http://localhost:13000/admin/timeline-schema](http://localhost:13000/admin/timeline-schema) и проверьте страницу.

<video width="100%" controls="">
  <source src="https://static-docs.nocobase.com/20240529220626_rec_.mp4" type="video/mp4" />
</video>

После проверки тестовую страницу необходимо удалить.

### 5. Определить элемент инициализатора схемы

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/client/initializer/index.tsx` с описанием элемента инициализатора:

```tsx | pure
import React, { useCallback, useState } from 'react';
import { FieldTimeOutlined } from '@ant-design/icons';
import { DataBlockInitializer, SchemaInitializerItemType, useSchemaInitializer } from "@nocobase/client";

import { getTimelineSchema } from '../schema';
import { useT } from '../locale';
import { TimelineConfigFormProps, TimelineInitializerConfigForm } from './ConfigForm';
import { BlockName, BlockNameLowercase } from '../constants';

export const TimelineInitializerComponent = () => {
  const { insert } = useSchemaInitializer();
  const [collection, setCollection] = useState<string>();
  const [dataSource, setDataSource] = useState<string>();
  const [showConfigForm, setShowConfigForm] = useState(false);
  const t = useT()

  const onSubmit: TimelineConfigFormProps['onSubmit'] = useCallback((values) => {
    const schema = getTimelineSchema({ collection, dataSource, timeField: values.timeField, titleField: values.titleField });
    insert(schema);
  }, [collection, dataSource])

  return <>
    {showConfigForm && <TimelineInitializerConfigForm
      visible={showConfigForm}
      setVisible={setShowConfigForm}
      onSubmit={onSubmit}
      collection={collection}
      dataSource={dataSource}
    />}
    <DataBlockInitializer
      name={BlockNameLowercase}
      title={t(BlockName)}
      icon={<FieldTimeOutlined />}
      componentType={BlockName}
      onCreateBlockSchema={({ item }) => {
        const { name: collection, dataSource } = item;
        setCollection(collection);
        setDataSource(dataSource);
        setShowConfigForm(true);
      }}>

    </DataBlockInitializer>
  </>
}

export const timelineInitializerItem: SchemaInitializerItemType = {
  name: 'Timeline',
  Component: TimelineInitializerComponent,
}
```

Сначала пользователь выбирает коллекцию в `DataBlockInitializer` — сохраняются `collection` и `dataSource`, затем в форме `TimelineInitializerConfigForm` задаются `timeField` и `titleField`; после отправки формы по этим данным строится схема и вставляется на страницу.

Ключевой узел сценария — компонент `DataBlockInitializer` (см. [документацию по инициализатору схемы и блокам данных](https://client.docs.nocobase.com/core/ui-schema/schema-initializer)).

`timelineInitializerItem`:

  - `name`: уникальный идентификатор элемента (операции создания, чтения, обновления, удаления в меню);
  - `Component`: в отличие от примера «[Добавление нового простого блока](/plugin-samples/schema-initializer/block-simple)», где задаётся поле `type`, здесь используется собственный `Component`; допустимы [оба способа](https://client.docs.nocobase.com/core/ui-schema/schema-initializer#two-ways-to-define-component-and-type).

`TimelineInitializerComponent`:

  - `DataBlockInitializer`
    - `title`: заголовок пункта;
    - `icon`: значок; другие значки — в [наборе иконок Ant Design](https://ant.design/components/icon/);
    - `componentType`: тип блока, в примере — `Timeline`;
    - `onCreateBlockSchema`: вызывается после выбора коллекции;
      - `item`: сведения о выбранной коллекции;
        - `item.name`: имя коллекции;
        - `item.dataSource`: источник данных коллекции;
    - [useSchemaInitializer](https://client.docs.nocobase.com/core/ui-schema/schema-initializer#useschemainitializer): методы вставки схемы (в частности `insert`).

Определения элементов инициализатора см. в [документации по инициализатору схемы](https://client.docs.nocobase.com/core/ui-schema/schema-initializer).

### 6. Реализация настроек схемы

#### 6.1 Определение настроек схемы

У готового блока обычно есть **настройки схемы** для параметров и действий; в этом примере они не разбираются подробно — добавлена только операция **удаления** (`remove`).

Создайте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/client/settings/index.ts` со следующим содержимым:

```ts
import { SchemaSettings } from "@nocobase/client";

export const timelineSettings = new SchemaSettings({
  name: 'blockSettings:info',
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

- `componentProps` для пункта `remove`:
  - `removeParentsIfNoChildren`: удалять ли родителя, если не осталось дочерних узлов;
  - `breakRemoveOn`: условие остановки при удалении вверх по дереву. Меню **«Добавить блок»** оборачивает вложенность в узел с компонентом `Grid`, поэтому задано `breakRemoveOn: { 'x-component': 'Grid' }`: при удалении не подниматься выше узла сетки.

#### 6.2 Регистрация настроек схемы

```ts
import { Plugin } from '@nocobase/client';
import { timelineSettings } from './settings';

export class PluginInitializerBlockDataModalClient extends Plugin {
  async load() {
    // ...
    this.app.schemaSettingsManager.add(timelineSettings)
  }
}

export default PluginInitializerBlockDataModalClient;
```

#### 6.3 Использование настроек схемы

Нужно добавить в функцию `getTimelineSchema()` поле `x-settings` с именем набора настроек в файле `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/client/schema/index.tsx`:

```diff
+ import { timelineSettings } from '../settings';

export function getTimelineSchema(options: GetTimelineSchemaOptions) {
  const { dataSource, collection, titleField, timeField } = options;
  return {
    type: 'void',
    'x-decorator': 'DataBlockProvider',
+   'x-settings': timelineSettings.name,
    // ...
  }
}
```

### 7. Подключение к меню «Добавить блок»

В системе много точек с меню **«Добавить блок»**, но **идентификаторы инициализаторов** у них разные.

![img_v3_02b4_049b0a62-8e3b-420f-adaf-a6350d84840g](https://static-docs.nocobase.com/img_v3_02b4_049b0a62-8e3b-420f-adaf-a6350d84840g.jpg)

#### 7.1 Уровень страницы

Чтобы добавить пункт на уровне страницы, нужно знать имя инициализатора (`name`) и путь вложенности. Имена можно найти в исходном коде ядра и плагинов, через инструменты разработчика в браузере или в отладочных сборках; в исходной англоязычной документации этот шаг был помечен как незаполненный.

На скриншоте выше меню **«Добавить блок»** на странице соответствует инициализатору `page:addBlock`, а подменю **«Блоки данных»** — ветке `dataBlocks`.

Измените файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/client/index.tsx`:

```tsx | pure
import { Plugin } from '@nocobase/client';
import { Timeline } from './component';
import { useTimelineProps } from './schema';
import { timelineSettings } from './settings';
import { timelineInitializerItem } from './timelineInitializerItem';

export class PluginInitializerBlockDataModalClient extends Plugin {
  async load() {
    this.app.addComponents({ Timeline })
    this.app.addScopes({ useTimelineProps });
    this.app.schemaSettingsManager.add(timelineSettings)

    this.app.schemaInitializerManager.addItem('page:addBlock', `dataBlocks.${timelineInitializerItem.name}`, timelineInitializerItem)
  }
}

export default PluginInitializerBlockDataModalClient;
```

<video controls width='100%' src="https://static-docs.nocobase.com/20240529222118_rec_.mp4"></video>

#### 7.2 Модальное окно «Добавить» в блоке таблицы

Тот же элемент нужно добавить не только в меню страницы **«Добавить блок»**, но и в **«Добавить блок»** внутри модального окна **«Добавить»** у **блока таблицы**.

![img_v3_02b4_fc47fe3a-35a1-4186-999c-0b48e6e001dg](https://static-docs.nocobase.com/img_v3_02b4_fc47fe3a-35a1-4186-999c-0b48e6e001dg.jpg)

По аналогии с п. 7.1 для этого сценария инициализатор меню **«Добавить блок»** в модальном окне таблицы имеет имя `popup:addNew:addBlock`, подменю **«Блоки данных»** по-прежнему — `dataBlocks`.

Измените файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/client/index.tsx`:

```diff
import { Plugin } from '@nocobase/client';
import { Timeline } from './component';
import { useTimelineProps } from './schema';
import { timelineSettings } from './settings';
import { timelineInitializerItem } from './timelineInitializerItem';

export class PluginInitializerBlockDataModalClient extends Plugin {
  async load() {
    this.app.addComponents({ Timeline })
    this.app.addScopes({ useTimelineProps });
    this.app.schemaSettingsManager.add(timelineSettings)
    this.app.schemaInitializerManager.addItem('page:addBlock', `dataBlocks.${timelineInitializerItem.name}`, timelineInitializerItem)
+   this.app.schemaInitializerManager.addItem('popup:addNew:addBlock', `dataBlocks.${timelineInitializerItem.name}`, timelineInitializerItem);
  }
}

export default PluginInitializerBlockDataModalClient;
```

![20240529223046](https://static-docs.nocobase.com/20240529223046.png)

#### 7.3 Мобильная страница

> Сначала включите плагин мобильного клиента; см. «[Установка и активация плагинов](/get-started/installation/plugins)».

Тот же элемент можно добавить в меню **«Добавить блок»** на мобильной странице; способ узнать имя инициализатора тот же, что выше, поэтому не повторяется.

Измените файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/client/index.tsx`:

```ts
// ...

export class PluginInitializerBlockDataModalClient extends Plugin {
  async load() {
    // ...
    this.app.schemaInitializerManager.addItem('mobilePage:addBlock', `dataBlocks.${timelineInitializerItem.name}`, timelineInitializerItem);
  }
}

export default PluginInitializerBlockDataModalClient;
```

![20240529223307](https://static-docs.nocobase.com/20240529223307.png)

При необходимости добавьте вызовы `addItem` для других точек меню **«Добавить блок»**, заранее узнав их идентификаторы.

### 8. Многоязычность

#### 8.1 Английский

Отредактируйте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/locale/en-US.json`:

```diff
{
  "Timeline": "Timeline",
  "Title Field": "Title Field",
  "Time Field": "Time Field"
}
```

#### 8.2 Китайский

Отредактируйте файл `packages/plugins/@nocobase-sample/plugin-initializer-block-data-modal/src/locale/zh-CN.json`:

```diff
{
  "Timeline": "时间线",
  "Title Field": "标题字段",
  "Time Field": "时间字段"
}
```

Дополнительные языки задаются по тому же принципу. Список языков интерфейса настраивается в [настройках системы](http://localhost:13000/admin/settings/system-settings); переключатель обычно в правом верхнем углу.

![20240611113758](https://static-docs.nocobase.com/20240611113758.png)

## Упаковка и загрузка в производственную среду

Согласно [Build and Package Plugin](/plugin-development/write-your-first-plugin#build-and-package-plugin) документации, мы можем упаковать плагин и загрузить его в производственную среду.

Если вы клонировали исходный код, вам необходимо сначала выполнить полную сборку, чтобы также построить зависимости плагина.

```bash
yarn build
```

Если вы использовали `create-nocobase-app` чтобы создать проект, вы можете напрямую выполнить:

```bash
yarn build @nocobase-sample/plugin-initializer-block-data-modal --tar
```

В каталоге `storage/tar/@nocobase-sample/plugin-initializer-block-data-modal.tar.gz` появится архив; установить плагин можно по инструкции [установки плагинов из файла](/get-started/installation/plugins).
