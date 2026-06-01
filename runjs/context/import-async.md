# Контекст: ctx.importAsync()

Динамически загружает **ESM-модули** или **CSS** по URL; применим во всех сценариях RunJS. Для сторонних ESM используйте `ctx.importAsync()`, для UMD/AMD — `ctx.requireAsync()`. URL с `.css` загружает и внедряет стили.

## Сценарии использования

| Сценарий | Описание |
|----------|----------|
| **JS-блок** | Загрузка Vue, ECharts, Tabulator и т. д. для пользовательских графиков, таблиц, досок |
| **Поле JS / Элемент JS / JS-столбец таблицы** | Загрузка небольших ESM-утилит (например, плагинов dayjs) для рендера |
| **Поток событий / события действий** | Подгрузка зависимостей перед выполнением логики |

## Тип

```ts
importAsync<T = any>(url: string): Promise<T>;
```

## Параметры

| Параметр | Тип | Описание |
|----------|-----|----------|
| `url` | `string` | URL ESM или CSS. Поддерживает сокращённую запись `<package>@<version>` или подпуть `<package>@<version>/<path>` (например, `vue@3.4.0`, `dayjs@1/plugin/relativeTime.js`) — разрешается через настроенный префикс CDN; также поддерживается полный URL. Для `.css` загружает и внедряет стили. Для библиотек, зависящих от React, добавляйте `?deps=react@18.2.0,react-dom@18.2.0`, чтобы использовать тот же экземпляр React. |

## Возвращаемое значение

- Объект пространства имён загруженного модуля (значение Promise).

## Формат URL

- **ESM и CSS**: помимо ESM можно загружать CSS (передайте URL с `.css`; стиль будет внедрён в страницу).
- **Сокращённая форма**: если не настроено иначе, используется префикс CDN **https://esm.sh**. Например, `vue@3.4.0` → `https://esm.sh/vue@3.4.0`.
- **?deps**: для библиотек, зависящих от React (например, `@dnd-kit/core`, `react-big-calendar`), добавляйте `?deps=react@18.2.0,react-dom@18.2.0`, чтобы избежать некорректного вызова хука из-за нескольких экземпляров React.
- **Развёрнутый локально CDN**: для интрасети или пользовательского сервиса задайте переменные:
  - **ESM_CDN_BASE_URL**: базовый URL ESM CDN (по умолчанию `https://esm.sh`)
  - **ESM_CDN_SUFFIX**: необязательный суффикс (например, `/+esm` у jsDelivr)
  - Ссылка: [nocobase/esm-server](https://github.com/nocobase/esm-server)

## Сравнение с ctx.requireAsync()

- **ctx.importAsync()**: загружает **ESM**, возвращает пространство имён модуля; подходит для Vue, dayjs и т. д. (ESM-сборки).
- **ctx.requireAsync()**: загружает **UMD/AMD** или глобальные скрипты; подходит для ECharts, FullCalendar (UMD) и т. д. Если у библиотеки есть ESM, предпочтителен `ctx.importAsync()`.

## Примеры

### Базовое использование

```javascript
const Vue = await ctx.importAsync('vue@3.4.0');

const relativeTime = await ctx.importAsync('dayjs@1/plugin/relativeTime.js');

const pkg = await ctx.importAsync('https://cdn.example.com/my-module.js');

await ctx.importAsync('https://cdn.example.com/theme.css');
```

### ECharts

```ts
const echarts = await ctx.importAsync('echarts@5.4.3');

const chartEl = document.createElement('div');
chartEl.style.width = '100%';
chartEl.style.height = '400px';
ctx.render(chartEl);

const chart = echarts.init(chartEl);

const option = {
  title: { text: 'Sales Overview', left: 'center' },
  tooltip: { trigger: 'axis' },
  legend: { data: ['Sales', 'Profit'], top: '10%' },
  xAxis: { type: 'category', data: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'] },
  yAxis: { type: 'value' },
  series: [
    { name: 'Sales', type: 'bar', data: [120, 200, 150, 80, 70, 110] },
    { name: 'Profit', type: 'line', data: [20, 40, 30, 15, 12, 25] },
  ],
};

chart.setOption(option);

window.addEventListener('resize', () => chart.resize());

chart.on('click', (params) => {
  ctx.message.info(`Clicked ${params.seriesName} on ${params.name}, value: ${params.value}`);
});
```

### Tabulator

```ts
await ctx.importAsync('tabulator-tables@6.2.5/dist/css/tabulator.min.css');

const { TabulatorFull } = await ctx.importAsync('tabulator-tables@6.2.5');

const tableEl = document.createElement('div');
ctx.render(tableEl);

const table = new TabulatorFull(tableEl, {
  data: [
    { id: 1, name: 'Alice', age: 25, city: 'Beijing' },
    { id: 2, name: 'Bob', age: 30, city: 'Shanghai' },
    { id: 3, name: 'Charlie', age: 28, city: 'Guangzhou' },
  ],
  columns: [
    { title: 'ID', field: 'id', width: 80 },
    { title: 'Name', field: 'name', width: 150 },
    { title: 'Age', field: 'age', width: 100 },
    { title: 'City', field: 'city', width: 150 },
  ],
  layout: 'fitColumns',
  pagination: true,
  paginationSize: 10,
});

table.on('rowClick', (e, row) => {
  const rowData = row.getData();
  ctx.message.info(`Row clicked: ${rowData.name}`);
});
```

### FullCalendar (ESM)

```ts
const { Calendar } = await ctx.importAsync('@fullcalendar/core@6.1.20');
const dayGridPlugin = await ctx.importAsync('@fullcalendar/daygrid@6.1.20');

const calendarEl = document.createElement('div');
calendarEl.id = 'calendar';
ctx.render(calendarEl);

const calendar = new Calendar(calendarEl, {
  plugins: [dayGridPlugin.default || dayGridPlugin],
  headerToolbar: {
    left: 'prev,next today',
    center: 'title',
    right: 'dayGridMonth',
  },
});

calendar.render();
```

### dnd-kit (с ?deps)

Для React-библиотек, таких как `@dnd-kit/core`, используйте `?deps=react@18.2.0,react-dom@18.2.0`, чтобы использовать тот же экземпляр React и корректную работу хуков:

```ts
const React = await ctx.importAsync('react@18.2.0');
const { createRoot } = await ctx.importAsync('react-dom@18.2.0/client');
const core = await ctx.importAsync('@dnd-kit/core@6.3.1?deps=react@18.2.0,react-dom@18.2.0');
// Используйте core (DndContext, useDraggable, useDroppable и т. д.) вместе с React
```

### react-big-calendar

```tsx
await ctx.importAsync('react-big-calendar@1.11.4/lib/css/react-big-calendar.css');

const React = await ctx.importAsync('react@18.2.0');
const { Calendar, dateFnsLocalizer } = await ctx.importAsync('react-big-calendar@1.11.4?deps=react@18.2.0,react-dom@18.2.0');
const { format, parse, startOfWeek, getDay } = await ctx.importAsync('date-fns@2.30.0');
const enUS = await ctx.importAsync('date-fns@2.30.0/locale/en-US.js');

const localizer = dateFnsLocalizer({
  format,
  parse,
  startOfWeek,
  getDay,
  locales: { 'en-US': enUS },
});

const events = [
  { title: 'All Day Event', start: new Date(2026, 0, 28), end: new Date(2026, 0, 28), allDay: true },
  { title: 'Meeting', start: new Date(2026, 0, 29, 10, 0), end: new Date(2026, 0, 29, 11, 0) },
];

ctx.render(
  <Calendar
    localizer={localizer}
    events={events}
    startAccessor="start"
    endAccessor="end"
    style={{ height: '80vh' }}
  />
);
```

## Примечания

- Зависит от сети и CDN; для интрасети используйте собственный сервис через **ESM_CDN_BASE_URL**.
- Если библиотека имеет и ESM, и UMD сборки, предпочтителен `ctx.importAsync()` для более корректной модульной семантики.
- Для библиотек, зависящих от React, добавляйте `?deps=react@18.2.0,react-dom@18.2.0` (или версии вашего приложения), чтобы избежать некорректного вызова хука.

## Связанные материалы

- [ctx.requireAsync()](./require-async.md): загрузка UMD/AMD и глобальных скриптов; подходит для ECharts, FullCalendar (UMD)
- [ctx.render()](./render.md): рендер в контейнер
