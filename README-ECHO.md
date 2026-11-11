# Azeroth: Эхо Пустоты — ядро

Форк **AzerothCore 3.3.5a** (build 12340) с модулями кастомного реалма, скомпилированными в `worldserver`.

---

## 🔗 Связанные репозитории

| Репозиторий | Что внутри |
|---|---|
| [**azeroth-echo**](https://github.com/Project-Aurora-WoW/azeroth-echo) | Контент: SQL-миграции, Lua-скрипты, модули (исходники), аддон, патчи клиента, инструменты |
| **azeroth-echo-core** ← вы здесь | Ядро: код AzerothCore, вкомпилированные модули, сборка |
| [**Релизы контента**](https://github.com/Project-Aurora-WoW/azeroth-echo/releases) | Готовые клиентские патчи |

**[Полная документация в Wiki](https://github.com/Project-Aurora-WoW/azeroth-echo/wiki)** · [Связь репозиториев](https://github.com/Project-Aurora-WoW/azeroth-echo/wiki/Связь-репозиториев)

---

## Модули в составе

| Модуль | Назначение |
|---|---|
| `mod-campaign` | движок кампаний 1–4 |
| `mod-archivist-chronicle-of-azeroth` | Хронист Ордена, лор-архив |
| `mod-echoes-architecture` | каркас систем реалма (PlayerScript, SpellScript) |
| `mod-stranniki-iz-niotkuda` | странствующие существа |
| `echo-challenge-modes` | режимы испытаний, Алтарь Испытаний (GO 705605) |

Исходники модулей — в контентном репозитории, здесь рабочая копия, скомпилированная в ядро.

---

## Сборка

### Требования

| Компонент | Версия |
|---|---|
| Visual Studio Build Tools | 18 (2026), MSVC 19.24+ |
| CMake | 4.5+ |
| Boost | 1.78 |
| OpenSSL | 4.0 |
| MySQL client libs | 8.0 |

### Конфигурация и сборка

```powershell
cmake -S . -B E:/acbuild -G "Visual Studio 18 2026" -A x64 `
      -DCMAKE_INSTALL_PREFIX=E:/acbuild/bin -DSCRIPTS=static

cmake --build E:/acbuild --config RelWithDebInfo --target worldserver authserver --parallel
```

Результат:

```
E:\acbuild\bin\RelWithDebInfo\worldserver.exe
E:\acbuild\bin\RelWithDebInfo\authserver.exe
```

При успешной конфигурации CMake выводит список найденных модулей:

```
* Modules configuration (static):
  |   +- worldserver
  |   +- echo-challenge-modes
  |   +- mod-archivist-chronicle-of-azeroth
  |   +- mod-campaign
  |   +- mod-echoes-architecture
  |   +- mod-stranniki-iz-niotkuda
```

Если модуля нет в списке — CMake его не нашёл: проверить структуру `modules/<имя>/src/*.cpp`.

### Что нужно после сборки

В `bin/RelWithDebInfo/` должны лежать:

```
configs/                  worldserver.conf, authserver.conf
data/                     dbc, maps, vmaps, Cameras
libmysql.dll              из MySQL
libcrypto-4-x64.dll       из OpenSSL
libssl-4-x64.dll          из OpenSSL
legacy.dll                из предыдущей сборки
```

Каталог `data/sql` в репозиторий не входит — его берут из апстрима:

```powershell
git clone --depth 1 https://github.com/azerothcore/azerothcore-wotlk.git C:\temp\ac
Copy-Item -Recurse C:\temp\ac\data\sql .\data\
```

Без этого `worldserver` упадёт с `Failed to initialize network`.

---

## Запуск

```powershell
cd E:\acbuild\bin\RelWithDebInfo
.\authserver.exe -c configs/authserver.conf

# в другом окне
.\worldserver.exe -c configs/worldserver.conf
```

Флаг `-c` обязателен: по умолчанию конфиг ищется в текущем каталоге, а лежит в `configs\`.

Признак успеха в логе:

```
Opening DatabasePool 'echo_world' ... opened successfully
Updating World database... >> World database is up-to-date!
WORLD: World Initialized In 0 Minutes 10 Seconds
worldserver-daemon ready...
```

---

## Что изменено относительно апстрима

| Изменение | Зачем |
|---|---|
| Добавлены 5 модулей проекта | контент реалма |
| Удалены 4 модуля (`ai-bots`, `cursed-islands`, `night-of-fallen-heroes`, `stalker`) | не относятся к проекту, ломали автообновление БД |
| `.gitignore` дополнен `data/sql/`, `bin/`, `Crashes/`, `logs/`, `configs/` | 828 МБ SQL-дампов не место в git |

---

## Известные проблемы сборки

| Симптом | Причина | Решение |
|---|---|---|
| `LNK2019: Add<...>Scripts` | имя функции модуля не совпало | `Add` + имя папки **без изменений** + `Scripts` |
| `C3668: override не переопределяет` | сигнатура хука не совпала с ядром | сверить с `ScriptDefines/PlayerScript.h` |
| `ACCESS_VIOLATION` в `_checkMemory` | скрипты созданы глобально | создавать внутри `Add<...>Scripts()` |
| `Could not find a proper repository signature` | нет тегов в `.git` | не критично, версия будет `unknown` |

Подробно — в [Wiki: Сборка ядра](https://github.com/Project-Aurora-WoW/azeroth-echo/wiki/Сборка-ядра) и [Устранение неполадок](https://github.com/Project-Aurora-WoW/azeroth-echo/wiki/Устранение-неполадок).

---

## Лицензия

AzerothCore распространяется под **GPL-2.0** (см. [`LICENSE`](LICENSE)). Модули сторонних авторов сохраняют свои лицензии — см. `modules/*/README.md`.
