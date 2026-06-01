# Сборка

## Пользовательская конфигурация сборки

Если вы хотите настроить конфигурацию сборки, создайте файл `build.config.ts` в корневой директории плагина со следующим содержимым:

```js
import { defineConfig } from '@nocobase/build';

export default defineConfig({
  modifyViteConfig: (config) => {
    // vite используется для сборки кода `src/client`

    // Изменение конфигурации Vite: https://vitejs.dev/guide/
    return config
  },
  modifyTsupConfig: (config) => {
    // tsup используется для сборки кода `src/server`

    // Изменение конфигурации tsup: https://tsup.egoist.dev/#using-custom-configuration
    return config
  },
  beforeBuild: (log) => {
    // Функция обратного вызова, выполняющаяся перед началом сборки; можно запускать предварительные операции.
  },
  afterBuild: (log: PkgLog) => {
    // Функция обратного вызова, выполняющаяся после завершения сборки; можно запускать пост-обработку.
  };
});
```

