# BIOS/UEFI — отчёт по Lenovo BIOS Simulator Center

**Симулятор:** https://download.lenovo.com/bsco/index.html  
**Устройство:** ThinkPad T14s Gen 5 (21LS, 21LT)  
**Режим:** Graphics Interface Mode  
**Клавиша сохранения:** F10 — Save and Exit. Подтверждение: «Save configuration changes and exit now?» → Yes.  
**Сброс:** F9 — Setup Defaults, либо Restart → Load Factory Defaults.

---

## Задача 1. Навигация

Разделы меню Setup: **Main**, **Config**, **Date/Time**, **Security**, **Startup**, **Restart**.

### Что показывает Main

| Параметр | Значение |
|---|---|
| UEFI BIOS Version | N46ET09W |
| UEFI BIOS Date | 2024-01-22 |
| Embedded Controller Version | N46HT10W |
| ME Firmware Version | XX.X.XX.XXXX |
| Machine Type Module | 21LS |
| System-unit serial number | PF12345 |
| System board serial number | L112345678 |
| Asset Tag | No Asset Information |
| CPU Type | Intel(R) Core(TM) CPU |
| CPU Speed | 1.X00GHz |
| Installed memory | 8192MB |
| UUID | 79498acc-3133-11b2-a85c-123412341234 |
| MAC Address (Internal LAN) | 54 XX 75 XX 31 XX |
| Preinstalled OS License | XX |
| UEFI Secure Boot | On |
| OA3 ID | XX |
| BIOS Event log | (Simulator not support) |

Где лежат настройки из задач 2–5:

- порядок загрузки — **Startup → Boot**
- Secure Boot — **Security → Secure Boot**
- пароль супервизора — **Security → Password → Supervisor Password**
- заводской сброс — **Restart → Load Factory Defaults** (или F9)

---

## Задача 2. Порядок загрузки

**Путь:** `Startup → Boot → Boot Priority Order`

Устройства перетаскиваются мышью. Исходный порядок:

1. Windows Boot Manager
2. USB CD
3. USB FDD
4. NVMe0 SKHynix_HFS256GD9TNG-L5B0B
5. USB HDD
6. PXE BOOT
7. LENOVO CLOUD
8. ON-PREMISE

Ниже черты **Excluded from Boot Priority Order**: Other CD, Other HDD.

Сделанный порядок: **USB HDD — первый, NVMe0 — второй**, дальше Windows Boot Manager. После повторного открытия `Startup → Boot` порядок на месте. Сохраняется по **F10**.

---

## Задача 3. Secure Boot

**Путь:** `Security → Secure Boot → Secure Boot`  
Статус по умолчанию: **On**. Рядом: Secure Boot Mode = User Mode, Secure Boot Key State = Standard Mode.

Переключение: On → Off → снова On. Отдельного окна «нужна перезагрузка» нет, статус меняется сразу. На реальном ноутбуке изменение начинает действовать после **F10 (Save and Exit)**, то есть после перезагрузки.

**Зачем нужен Secure Boot:** при On прошивка запускает только загрузчики с доверенной подписью и не даёт стартовать неподписанной ОС. Off разрешает загрузку любой ОС (нужно, например, для части дистрибутивов Linux без ключа Microsoft).

---

## Задача 4. Пароль супервизора

**Путь:** `Security → Password → Supervisor Password`

Было: **Disabled**. В окне два поля: Enter New Password и Confirm New Password, раскладка English (United States), кнопки Cancel и Save. Установлен пароль `Test1234`. Статус стал **Enabled**.

Что защищает пароль (текст подсказки): списки приоритета загрузки, сетевые пункты, дату и время. Пункт `Security → Password → Lock UEFI BIOS Settings` (по умолчанию Off) при включении запрещает любые правки Setup без пароля супервизора.

В этом симуляторе смена даты и переключателя Password at Restart **не вызвала** запрос пароля — окно запроса здесь не реализовано. На реальном ThinkPad защищённые пункты без пароля не меняются.

Снять пароль пустыми полями нельзя: сообщение **«Password must has at least 1 character»**, статус остаётся Enabled. На реальном устройстве пароль снимают так: открыть тот же пункт, ввести текущий пароль и оставить новый пустым, затем Save.

---

## Задача 5. Заводские настройки

**Путь:** `Restart → Load Factory Defaults`  
Текст окна: «Load the BIOS factory default configuration now?» → Yes.  
То же действие — клавиша **F9 (Setup Defaults)**.

Предупреждение в подсказке: сброс не отменяет пункты **Security** и **Date & Time**, их нужно сбрасывать отдельно в Setup.

После подтверждения симулятор вернулся на Main, и проверенные пункты снова стали исходными:

- загрузка: Windows Boot Manager первый, NVMe0 четвёртый, USB HDD пятый
- Secure Boot: On
- Supervisor Password: Disabled
- Always On USB: On
- дата: November

---

## Задача 6. Встроенные USB

**Путь:** `Config → USB → Always On USB`  
Отдельного пункта USB BIOS Support на этой модели нет.

Always On USB по умолчанию **On**: порты USB питают внешнее устройство в гибернации и при выключении, когда подключена зарядка. Off отключает питание портов в этих состояниях. Рядом: `Charge in Battery Mode` (по умолчанию Off) — зарядка от батареи в тех же состояниях.

Переключатель выключен и включён обратно. Итог: **On**.

Полное отключение портов USB как устройств — другой путь: `Security → I/O Port Access → USB Port` (On/Off).

---

## Задача 7. SATA / NVMe

Пути `Config → Storage` и режимов **AHCI / RAID / Intel RST** у ThinkPad T14s Gen 5 в симуляторе нет. Диск один, NVMe: **NVMe0 SKHynix_HFS256GD9TNG-L5B0B** (виден в `Startup → Boot`). Пароль диска: `Security → Password → NVMe1 Password`.

На моделях, где режим контроллера есть, смена AHCI ↔ RAID / Intel RST после установки ОС обычно приводит к тому, что система не загружается: другой драйвер и другая разметка метаданных. Возвращать режим нужно до загрузки ОС либо переустанавливать систему. Поэтому режим смотрят, при необходимости переключают и сразу возвращают, читая предупреждение на экране.

---

## Задача 8. Временное меню загрузки (F12)

**Путь:** `Startup → Boot device List F12 Option` — по умолчанию **On**.  
Подсказка: клавиша F12 вызывает всплывающий список устройств. Пункт доступен, когда в Setup входит супервизор.

Это не то же самое, что `Startup → Boot Mode`:

- **Quick** — диагностический экран при загрузке скрыт, пока не нажать Esc
- **Diagnostics** — диагностический экран показывается всегда

Разница:

- `Startup → Boot → Boot Priority Order` — постоянный порядок, он сохраняется в настройках
- меню F12 — выбор устройства **только на эту загрузку**, сохранённый порядок не меняется

---

## Задача 9. Логи

**Путь:** `Main → BIOS Event log`  
Значение: **(Simulator not support)**. Раздела POST/событий и кнопки **Clear Event Log** в этой модели симулятора нет. System Summary и Config → Diagnostics как отдельных экранов тоже нет.

На реальном ThinkPad журнал событий ищут в Main (System Event Log / BIOS Event log), очистка — пункт Clear на том же экране. Отдельная диагностика Lenovo — свой симулятор в списке устройств (UEFI Lenovo Diagnostics Tools).

---

## Задача 10. Подготовка к переустановке ОС

Сделано по шагам после заводского сброса:

1. **Load Factory Defaults** — подтверждено, настройки исходные.
2. **Secure Boot** — On (`Security → Secure Boot → Secure Boot`).
3. **Supervisor Password** — `Test1234`, статус Enabled.
4. Порядок загрузки — USB HDD первый, NVMe0 второй (`Startup → Boot`).
5. **F10 → Yes** («Save configuration changes and exit now?»). Симулятор остаётся в Setup, изменения не пропадают.
6. Повторная проверка: пароль Enabled, Secure Boot On, порядок USB HDD → NVMe0 на месте. Порядок загрузки возвращён к исходному (Windows Boot Manager, USB CD, USB FDD, NVMe0, USB HDD). Пароль в симуляторе пустыми полями не снимается (см. задачу 4); на реальном ноутбуке его снимают, оставляя новый пароль пустым.

Зачем так готовят переустановку: Secure Boot не даёт стартовать неподписанному загрузчику, USB первым позволяет загрузиться с флешки, пароль супервизора не даёт сменить этот порядок без пароля, F10 записывает всё это до выхода.
