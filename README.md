
# minifs

Лёгкая библиотека виртуальной файловой системы поверх IndexedDB (браузер).

Версия: beta 1.8

Кратко
- `minifs` предоставляет простой API для создания директорий и файлов, чтения/записи текстовых и бинарных данных, а также операций копирования/перемещения/удаления. Данные сохраняются в `IndexedDB` в объектном хранилище.
- Основная точка входа: функция `createFs(props)` (в файле `minifs.js`). По умолчанию экспортируется инстанс `fs = createFs()`.

Где использовать
- В браузере/приложениях, где доступен `indexedDB`.
- Не предназначено для Node.js без полифилов для IndexedDB.

Инициализация

Можно использовать готовый инстанс:

```js
// Если подключили minifs.js в окружении, он уже создаёт `fs`:
// const fs = createFs(); // при необходимости создать с кастомными опциями
```

Опции `createFs(props)`
- `databaseName` (string) — имя IndexedDB базы (по умолчанию `indexeddb-fs`).
- `databaseVersion` (number) — версия БД (по умолчанию `1`).
- `objectStoreName` (string) — имя objectStore (по умолчанию `files`).
- `rootDirectoryName` (string) — имя корневой директории (по умолчанию `root`).

Основные сущности
- Типы: `directory` и `file`.
- Записи хранятся с ключом `fullPath` и полями `name`, `directory`, `type`, `data` (для файлов).

Как работают пути
- Путь нормализуется: обратные слеши заменяются на `/`, множественные слэши сжимаются.
- Если путь не содержит префикса корня — библиотека автоматически добавляет `root/`.
- Примеры допустимых путей: `docs/readme.txt`, `root/docs/readme.txt`, `.` (корень).

API (основные методы)
- `exists(path)` — возвращает `true/false`, существует ли запись.
- `isFile(path)` — true если путь указывает на файл.
- `isDirectory(path)` — true если путь указывает на директорию.
- `createDirectory(path)` — создает директорию. Бросает ошибку, если родительской директории нет.
- `readDirectory(path)` — возвращает объект { isEmpty, filesCount, directoriesCount, files, directories }.
- `writeFile(path, content)` — записывает текстовый файл (сохраняет текст в зашифрованном виде в базе). Возвращает метаданные (без поля data).
- `writeFileBin(path, base64Data)` — записывает бинарные данные. Ожидает base64-представление (без data: URI).
- `readFile(path)` — читает текстовый файл, возвращает текст.
- `readFileBin(path)` — читает бинарный файл, возвращает data URI вида `data:text/plain;base64,...`.
- `removeFile(path)` — удаляет файл.
- `removeDirectory(path)` — рекурсивно удаляет директорию и содержимое.
- `renameFile(oldPath, newName)` — переименовывает файл (в той же директории).
- `moveFile(srcPath, dstPath)` — перемещает (и переименовывает) файл.
- `copyFile(srcPath, dstPath)` — копирует файл (создает новый с теми же данными).
- `copyDirectory(srcPath, dstPath)` — рекурсивно копирует директорию.
- `details(path)` — возвращает запись (метаданные) по пути.
- `fileDetails(path)` — возвращает метаданные файла или бросает если не файл.
- `directoryDetails(path)` — возвращает метаданные директории или бросает если не директория.
- `remove(path)` — удаляет запись: если директория — рекурсивно, иначе файл.

Примеры использования

```js
// Инициализация
const fs = createFs();

// Создать директорию
await fs.createDirectory('docs');

// Записать текстовый файл
await fs.writeFile('docs/notes.txt', 'Привет, мир!');

// Прочитать файл
const text = await fs.readFile('docs/notes.txt');
console.log(text); // 'Привет, мир!'

// Список директории
const listing = await fs.readDirectory('docs');
console.log(listing.files);

// Бинарная запись: передаём base64 (без data:URI)
const base64 = btoa('binary content');
await fs.writeFileBin('docs/bin.dat', base64);
const dataUri = await fs.readFileBin('docs/bin.dat');
console.log(dataUri); // data:text/plain;base64,....

// Перемещение и удаление
await fs.moveFile('docs/notes.txt', 'docs/notes-renamed.txt');
await fs.removeFile('docs/notes-renamed.txt');

// Удалить директорию рекурсивно
await fs.removeDirectory('docs');
```

Особенности реализации и заметки
- Данные файлов хранятся с префиксом `TSSSFILE>>>` — это внутренний маркер библиотеки (см. `minifs.js`).
- `writeFile` сохраняет текст через `enc.txt(...)`, `readFile` декодирует обратно.
- `writeFileBin` ожидает сырое base64-строчное содержимое; `readFileBin` возвращает data URI.
- Все методы возвращают промисы; используйте `await` или `.then()`.
- Ошибки генерируются как `Error` с текстом: проверяйте сообщения для диагностики (например, "Dir \"...\" missing", "File \"...\" missing").
- Операции, затрагивающие много элементов (копирование/удаление директорий), выполняются рекурсивно и могут занять время.
- По умолчанию создаётся объект `fs` в конце файла: `const fs = createFs();` — вы можете создать свои инстансы с кастомными опциями.

Советы по интеграции
- Если нужно экспортировать данные пользователю, используйте `readFileBin` и создавайте ссылку с `href` = возвращённый data URI.
- Для обмена между вкладками используйте синхронизацию поверх IndexedDB (внешняя логика), т.к. библиотека сама по себе не пушит события.

Проблемы и отладка
- Убедитесь, что в браузере разрешён IndexedDB и нет ограничений приватного режима.
- Для просмотра содержимого IndexedDB используйте DevTools -> Application -> IndexedDB.


