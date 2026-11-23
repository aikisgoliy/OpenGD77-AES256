# OpenGD77-AES256 — Custom Build для TYT MD-UV390 Plus (10W)

**Язык / Language:** [🇷🇺 Русский](#-русский) | [🇬🇧 English](#-english)

---

## 🇷🇺 Русский

Кастомная сборка прошивки [OpenGD77](https://opengd77.com) для радиостанции **TYT MD-UV390 Plus (10W)** (на базе официальных исходников **R20260131**, таргет `MDUV380_10W_PLUS_FW`, профиль сборки **`PROFILE=full`**) с поддержкой **DMRA AES-256 шифрования голоса и SMS**, двухстрочным экраном Dual Screen, ускоренным анализатором спектра с водопадом, индикацией активных ключей AES и эргономичным управлением одной рукой.

---

### ⚠️ Правовое предупреждение и отказ от ответственности (Legal Disclaimer)

* **Запрет шифрования на любительских диапазонах:** использование шифрования голоса и данных (AES-256) **запрещено на любительских радиочастотах** законодательством большинства стран (в РФ — Приказ Минкомсвязи № 184 и ст. 13.4 КоАП РФ). Используйте данный функционал исключительно на выделенных коммерческих, служебных или ведомственных частотах при наличии соответствующего разрешения и лицензии, либо в экранированных лабораторных условиях.
* **Некоммерческий статус Open Source:** исходный код и бинарные сборки распространяются на условиях лицензии OpenGD77 (BSD-3-Clause с запретом коммерческого использования — см. [`license.txt`](license.txt)) **исключительно для некоммерческого, образовательного и исследовательского применения**. Продажа прошивки или радиостанций с предустановленной прошивкой запрещена.
* **Отсутствие встроенных ключей и проприетарного ПО:** в исходном коде и релизных сборках **нет предустановленных ключей шифрования** (все 15 слотов пусты), а также **отсутствует проприетарный вокодер AMBE+2** (для работы DMR пользователь при прошивке самостоятельно объединяет чистый `.bin` со своим файлом-донором TYT MD-9600 V5).
* **Отказ от ответственности (`AS IS`):** программное обеспечение предоставляется «как есть» без каких-либо гарантий. Вся ответственность за соблюдение правил радиосвязи, частотных распределений и местного законодательства лежит исключительно на конечном пользователе.

---

### ✨ Что добавлено в этой сборке (`PROFILE=full`)

#### 1. Базовые улучшения форка [`dondch/OpenGD77-AES256`](https://github.com/dondch/OpenGD77-AES256)
* **Криптографическое ядро DMRA AES-256 (OFB):** полная совместимость со штатной прошивкой TYT «Universal» AES256, Motorola и Hytera (ETSI TS 102 361). Передача 32-битного Message Indicator (MI) в PI-заголовке и внутри голосовых кадров AMBE (Late-Entry MI + Golay(24,12)+CRC4 + EMB Single Block), что обеспечивает мгновенный перезахват MI при быстрых последовательных вызовах (back-to-back calls) и позднем входе в эфир.
* **Хранение и управление 15 ключами во Flash (`0xDA000`):** встроенный наэкранный HEX-редактор на 64 символа (`Menu -> AES Keys`), консольная утилита `aes_key_tool.py` (команда CPS `0x80`), жёсткая блокировка передачи без загруженного ключа (`No AES key`, никогда не откатывается в открытый текст) и автоматическое ограничение размера базы DMR-ID, чтобы она не затирала таблицу ключей в памяти.
* **Зашифрованные DMR SMS (`ENABLE_DMR_DATA`):** обмен текстовыми сообщениями в формате AES-256-ECB (совместимо со стоковыми TYT), папки `Inbox` и `Sent`, индикатор непрочитанных сообщений (конверт в правом верхнем углу экрана) и поддержка шаблонов текста.
* **Удалённая настройка по эфиру (`ENABLE_REMOTE_PROVISION`):** протокол безопасной передачи записей каналов, зон, настроек VFO и ключей AES от мастер-рации к ведомым станциям по DMR-каналу (шифрование AES-256-CTR + подпись 128-битным AES-CMAC, утилита `tools/provision.py`).
* **Ускоренный анализатор спектра и быстрый сканер (`ENABLE_FAST_SCAN` / `ENABLE_SPECTRUM`):** ускорение шага свипирования до ~4 мс на отсчёт, панорамный режим `Wide 20 MHz` (10 подвыборок на столбец с шагом 12.5 кГц без пропуска частот), цветной водопад, авто-масштаб шумовой полки, удержание пиков (`Max Hold`), маркер частоты пика, автопрокрутка диапазона (`Auto scroll` с задержкой `Dwell 4x`) и автопарковка на активном сигнале, а также сверхбыстрое аналоговое сканирование от `6 мс` с ранним отсечением пустых каналов.
* **Аппаратные исправления для MD-UV390 Plus 10W:** корректная инициализация `.ccmram` при старте, поддержка новых ревизий дисплеев и управление усилителем мощности 10W.

#### 2. Кастомные доработки этого форка (`aikisgoliy`)
* **Индикация статуса шифрования и активного ключа в статус-баре (`uiHomeScreen.c`):**
  * `AES<N>` — режим ожидания на цифровом канале (показывает номер ключа канала или глобального TX-ключа `1..15`).
  * `TX A<N>` — передача с активным шифрованием (номер ключа, которым шифруется эфир).
  * `RX A<N>` — приём и расшифровка входящего вызова (номер ключа, которым расшифрован сигнал).
  * `ENC ?` — принимается зашифрованный DMR-вызов, но подходящий ключ отсутствует в памяти рации.
  * `OPEN` — на шифрованном канале принимается открытый (незашифрованный) DMR-вызов.
  * `DMR` / `FM` / `SCAN` — стандартные режимы без шифрования.
* **Отображение ключа в режиме двух строк (`uiDualScreen.c`):**
  * На каждой из двух строк (A и B) цветной бейдж режима автоматически отображает **`DMR AES<N>`**, если для канала/VFO задан ключ шифрования (или во время приёма шифрованного вызова), либо **`DMR`** / **`ANA`** для открытых каналов.
* **Умное переключение Dual Screen ↔ экран вызова (`uiUtilities.c`):**
  * В режиме ожидания отображается двухстрочный экран Dual Screen, а при начале приёма или нажатии PTT автоматически открывается информативный главный экран с S-метром, графиком сигнала, позывным и телеметрией; после окончания вызова экран возвращается к двум строкам.
* **Эргономичная блокировка клавиатуры под правую руку (`applicationMain.c`, `uiChannelMode.c`, `uiVFOMode.c`):**
  * Долгое удержание **`*`** (1 секунда) в режиме каналов или VFO блокирует клавиатуру; повторное удержание **`*`** (1 секунда) снимает блокировку (работает через прямой опрос матрицы клавиатуры `keyboardRead()`).
* **Гибкая настройка блокировки PTT (`Options -> General Options -> Star lock PTT`):**
  * **`Off` (по умолчанию):** долгое удержание `*` блокирует **только клавиатуру**, при этом кнопка **PTT остаётся активной**.
  * **`On`:** долгое удержание `*` блокирует **и клавиатуру, и PTT**.
  * Стандартный пункт главного меню **`Lock -> Keypad & PTT`** по-прежнему принудительно блокирует обе функции независимо от этой настройки.

---

### 🧩 Архитектура и авторы базовых модулей

| Модуль / Фича | Источник / Автор | Вклад в сборку |
| :--- | :--- | :--- |
| **База OpenGD77** | **VK3KYY**, **F1RMB**, **DG4KLU** и др. (`R20260131`) | Ядро прошивки для `TYT MD-UV390 Plus 10W`, DMR/FM стек, хотспот, GPS, спутники. |
| **DMRA AES-256, SMS и Provisioning** | **`dondch/OpenGD77-AES256`** | Шифрование голоса AES-256 OFB (совместимость со стоком TYT), хранение 15 ключей, зашифрованные SMS, удалённая настройка по эфиру, быстрый скан и водопад. |
| **Графический интерфейс и телеметрия** (`ENABLE_TA2TWC_SCREENS`) | **`josephddt/OpenGD77-TA2TWC`** (**TA2TWC**) | Новый главный экран с крупной частотой, S-метром (dBm/SNR), графиком истории сигнала, крупным позывным, шкалой TOT, стрелочным S-метром в RSSI, тёмной темой и 9-строчными меню. |
| **Двухстрочный экран и Row Watch** (`ENABLE_DUAL_SCREEN`) | **`erenbozaci/md-uv390-firmware`** (**Eren TA3BZC**) | Экран в стиле Anytone (строки A и B для VFO или каналов зон), фоновый мониторинг второй строки (`Row watch`) и меню `DW Options`. |
| **Анализатор спектра с водопадом** | **`dondch`** (по мотивам **`mangelajo`** и **`fagci`**) | Цветной водопад, авто-масштаб, удержание пиков (`Max Hold`), маркер частоты пика, автопрокрутка диапазона и парковка на сигнале. |
| **Кастомные доработки этого форка** | **`aikisgoliy/OpenGD77-AES256`** | Индикация статуса и номера ключа (`AES<N>`, `TX A<N>`, `RX A<N>`, `ENC ?`, `OPEN`) в статус-баре и на строках Dual Screen; умное переключение Dual Screen; блокировка/разблокировка удержанием `*` (1 сек) и опция `Star lock PTT`. |

---

### 📖 Справочник по меню радиостанции

Вход в главное меню — кнопка **`Green` (Зелёная)**. Навигация — **`Up` / `Down`**, изменение значений — **`Left` / `Right`**, сохранение/подтверждение — **`Green`**, выход без сохранения/назад — **`Red` (Красная)**.

#### 1. Главное меню (`Main Menu`)

| Пункт меню | Назначение |
| :--- | :--- |
| **Zone** | Выбор рабочей зоны каналов (включая виртуальную зону `All Channels`). |
| **RSSI** | Графический и стрелочный S-метр в реальном времени в dBm и единицах S. |
| **Radio Info** | Напряжение АКБ, температура процессора, версия сборки, статус GPS и свободная память. |
| **Contacts** | Список цифровых контактов (TG / Private Call) и DTMF-контактов. |
| **Last Heard** | Журнал принятых DMR-корреспондентов (позывной/ID, TG, таймслот, время, RSSI). |
| **Firmware Info** | Дата/время компиляции, целевая платформа и параметры сборки. |
| **AES Keys** | Управление 15 слотами ключей AES-256 и глобальным TX-ключом. |
| **Messages** | Зашифрованные DMR SMS (`Inbox` / `Sent` / `New Message`). |
| **Options** | Системные настройки рации (4 подраздела ниже). |
| **Channel Details** | Редактирование параметров текущего канала/VFO (частоты RX/TX, TS, CC, мощность, CTCSS/DCS, слот ключа AES). |
| **Dual Watch** | Настройки двухстрочного экрана (`Dual screen`) и двойного приёма (`DW Options`). |
| **Satellite** | Трекер радиолюбительских спутников с расчётом доплера. |
| **GPS** | Текущие координаты, высота, скорость, курс и статус спутников. |
| **Lock** | Ручной выбор блокировки: `Keypad` (только кнопки) или `Keypad & PTT` (кнопки и передача). |

#### 2. Подменю `AES Keys` и `Messages`

| Раздел | Пункт / Элемент | Управление и описание |
| :--- | :--- | :--- |
| **AES Keys** | **`Key 1` .. `Key 15`** | Слоты ключей (`set` — записан, `---` — пуст). **`Green`** открывает HEX-редактор (64 символа: `0–9` прямой ввод, `Up/Down` перебор `0-F`, `Left/Right` курсор, `Green` сохранить). **`SK2 + Green`** очищает слот. |
| **AES Keys** | **`Global TX key`** | Выбор глобального ключа передачи (`Off` или `1..15`), если в самом канале не задан свой ключ. |
| **Messages** | **`Inbox (N)` / `Sent (N)`** | Папки входящих (`*` — непрочитанные) и отправленных SMS. **`Green`** — чтение, **`SK2 + Green`** — удалить сообщение (или `[Delete all]`). |
| **Messages** | **`New Message`** | Ввод текста с цифровой клавиатуры (Multi-tap), **`Up`** — вставка шаблона, **`Left`** — стереть символ, **`Green`** — выбор ID/TG получателя (`Up/Down` смена `Group`/`Private`, `Green` отправить). |

#### 3. Подменю `Dual Watch` (`DW Options`)

| Пункт меню | Значения | Описание |
| :--- | :--- | :--- |
| **Dual screen** | `On` / `Off` | Включает двухстрочный главный экран (строки A и B). |
| **Row watch** | `On` / `Off` | Фоновое прослушивание второй (аналоговой) строки каждые 1.5 сек (на 80 мс) при тишине на активной строке. |
| **2nd side** | `VFO B` / `Channel` | Тип второй строки по умолчанию для классического Dual Watch. |
| **Home** | `Current` / `VFO A` / `VFO B` / `Channel` | Приоритетная (домашняя) строка при работе классического Dual Watch. |
| **Speed** | `Default` / `90ms` / `200ms` / `400ms` | Интервал переключения сторон в классическом Dual Watch. |
| **On signal** | `Switch` / `Stay` | `Switch` — переключиться на активный сигнал; `Stay` — подать сигнал и остаться на домашней строке. |

#### 4. Подменю `Options` (Системные настройки)

##### `General Options` (Общие настройки)

| Пункт меню | Значения | Описание |
| :--- | :--- | :--- |
| **Key long** | `0.5s` .. `5.0s` | Время удержания кнопок для срабатывания длинного нажатия. |
| **Key rpt** | `0.1s` .. `1.0s` | Скорость автоповтора при удержании кнопок навигации. |
| **Hotspot** | `Off` / `MMDVM` / `BlueDV` | Режим цифрового хотспота при подключении по USB. |
| **Auto lock** | `Off` / `0.5m` .. `15m` | Таймер автоблокировки клавиатуры при бездействии. |
| **APO** | `Off` / `30m` .. `240m` | Таймер автовыключения рации при бездействии. |
| **APO with RF** | `On` / `Off` | Сбрасывать ли таймер автовыключения при приёме сигнала из эфира. |
| **Safe Pwr-On** | `Off` / `On` | Безопасное включение (требует удержания `SK1` при повороте ручки питания). |
| **Eco level** | `0` .. `5` | Уровень энергосбережения приёмника в режиме ожидания. |
| **Star lock PTT** | `Off` / `On` | *(Кастом)* `Off` — долгое `*` блокирует только кнопки (PTT работает); `On` — блокирует кнопки и PTT. |

##### `Radio Options` (Настройки приёмопередатчика)

| Пункт меню | Значения | Описание |
| :--- | :--- | :--- |
| **Band limits** | `Off` / `On` / `CPS` | Ограничение передачи любительскими диапазонами (`Off` — передача разрешена везде). |
| **TX inhibit** | `Off` / `On` | Программный запрет передачи (режим «только приём»). |
| **DMR filter time** | `2s` .. `90s` | Время удержания захваченного TS/CC при отключённых фильтрах. |
| **Scan delay** | `1s` .. `30s` | Задержка возобновления сканирования после пропадания сигнала. |
| **Scan dwell** | `6ms` .. `480ms` | Время задержки на канале при сканировании (поддерживается быстрый скан от `6ms`). |
| **Scan mode** | `Hold` / `Pause` / `Stop` | Поведение сканера при обнаружении сигнала. |
| **Scan on boot** | `Off` / `On` | Автоматический запуск сканирования при включении станции. |
| **Squelch VHF / 220 / UHF** | `0%` .. `100%` | Порог шумоподавителя по умолчанию для каждого диапазона. |
| **PTT toggle** | `Off` / `On` | Режим «защёлки» PTT (нажал — передача идёт, нажал ещё раз — стоп; требует TOT > 0). |
| **Private calls** | `Off` / `On` / `PTT` | Приём и обработка индивидуальных (Private) вызовов. |
| **User power** | `0` .. `4100` | Тонкая калибровка выходной мощности для уровня `User Power` (шаг 100, с `SK2` — шаг 10). |
| **DMR CRC** | `On` / `Off` | Проверка контрольной суммы DMR (`Off` позволяет принимать слабые сигналы с ошибками). |

##### `Display Options` (Настройки экрана)

| Пункт меню | Значения | Описание |
| :--- | :--- | :--- |
| **Text size** | `1` / `2` | Стандартный или увеличенный шрифт на классическом экране. |
| **Brightness / Night** | `1%` .. `100%` | Яркость подсветки дисплея в дневном и ночном режимах. |
| **Min bright.** | `0%` .. `100%` | Минимальная фоновая подсветка в режиме ожидания. |
| **Mode** | `Auto` / `Squelch` / `Manual` / `Buttons` | Режим работы подсветки экрана. |
| **Timeout** | `No` / `5s` .. `30s` | Время до отключения подсветки. |
| **Invert** | `Normal` / `Invert` | Инверсия цветов дисплея. |
| **Auto night** | `Off` / `On` | Автоматическое переключение дневной/ночной темы по восходу и закату. |
| **Order** | `Ct/DB/TA` и др. | Приоритет отображения позывного/имени (Кодплаг / База DMR ID / Talker Alias). |
| **Contact** | `1 line` / `2 lines` / `Auto` | Перенос длинного Talker Alias на вторую строку. |
| **Time / Battery** | `On/Off` и `%/V` | Отображение часов и выбор индикации АКБ (в процентах или вольтах). |
| **Info** | `Off` / `TS` / `Pwr` / `Both` | Выделение жирным шрифтом вручную изменённого таймслота или мощности. |
| **Volume / LEDs** | `On` / `Off` | Всплывающая шкала громкости и работа светодиодов RX/TX (стелс-режим). |
| **Time zone / UTC** | `-12:00` .. `+14:00` | Часовой пояс и формат времени (локальное или UTC). |
| **Show distance** | `Off` / `On` | Показ расстояния до ретранслятора/канала (при наличии координат). |
| **Last talker** | `Off` / `1s` .. `30s` | Периодический показ позывного последнего корреспондента в режиме ожидания. |
| **Main screen** | `New` / `Classic` | `New` — экран с S-метром, графиком сигнала и бейджами `AES<N>`/`TX A<N>`/`RX A<N>`; `Classic` — классический вид. |

##### `Sound Options` (Настройки звука)

| Пункт меню | Значения | Описание |
| :--- | :--- | :--- |
| **Timeout beep** | `N/A` / `5s` .. `20s` | Предупреждающий сигнал перед истечением таймера передачи (TOT). |
| **Beep volume** | `-24dB` .. `+6dB` | Громкость системных сигналов и нажатий клавиш. |
| **DMR beep** | `None` / `Start` / `Stop` / `Both` | Сигнал начала/окончания передачи (Roger Beep) в DMR. |
| **RX beep** | `None` / `Carrier` / `Talker` / `Both` | Звуковой сигнал при приёме несущей или окончании фразы корреспондента. |
| **DMR / FM mic gain** | `-12dB` .. `+21dB` | Усиление микрофона раздельно для DMR и аналогового FM. |
| **VOX threshold / tail** | `2..30` / `0.0s..5.0s` | Чувствительность и задержка отпускания голосовой активации (VOX). |
| **Audio prompt** | `Silent` / `Beep` / `No keys` / `Voice L1..L3` | Режим звуковых сигналов и голосовых подсказок. |
| **DMR Rx AGC** | `Off` / `0dB` .. `21dB` | Автоматическая регулировка усиления приёма в DMR. |

---

### ⌨️ Шпаргалка по горячим клавишам (TYT MD-UV390)

#### 1. На главном экране (Каналы и VFO)

| Кнопка / Комбинация | Режим Каналов (`Channel Mode`) | Режим Частот (`VFO Mode`) |
| :--- | :--- | :--- |
| **Удержание `*` (1 сек)** | **Блокировка / Разблокировка клавиатуры** | **Блокировка / Разблокировка клавиатуры** |
| **Короткое `*`** | В DMR: смена таймслота (`TS1` ↔ `TS2`). В FM: полоса (`25 кГц` ↔ `12.5 кГц`) | В DMR: смена таймслота (`TS1` ↔ `TS2`). В FM: полоса (`25 кГц` ↔ `12.5 кГц`) |
| **`SK2 (синяя)` + `*`** | Переключение режима канала (`DMR` ↔ `FM`) | Переключение режима VFO (`DMR` ↔ `FM`) |
| **Удержание `#`** | **Реверс частот приёма и передачи** (`Reverse Repeater`) | **Запуск анализатора спектра** (`Sweep Scan` с водопадом) |
| **Короткое `#`** | Ввод номера TG / Private ID | Ввод номера TG / Private ID |
| **`SK2` + `#`** | Быстрый список контактов (`Contact Quicklist`) | Быстрый список контактов (`Contact Quicklist`) |
| **Удержание `0`** | Быстрое отключение/включение звука (`Mute`) | Быстрое отключение/включение звука (`Mute`) |
| **Удержание `8`** | Азимут и расстояние до ретранслятора (по GPS) | — |
| **`Up` / `Down`** | В Dual Screen: смена активной строки (`A` ↔ `B`). Обычный режим: смена канала | В Dual Screen: смена активной строки (`A` ↔ `B`). Обычный режим: шаг частоты |
| **Удержание `Up`** | Запуск сканирования каналов зоны | Запуск сканирования диапазона частот VFO |
| **`Red` (Красная)** | В Dual Screen: смена типа строки (`Канал` ↔ `VFO`). Обычный режим: переход в VFO | В Dual Screen: смена типа строки (`VFO` ↔ `Канал`). Обычный режим: переход в Каналы (долгое: `VFO A` ↔ `VFO B`) |
| **`SK2` + `Up` / `Down`** | Смена зоны каналов (`Zone`) | Выбор частоты для ввода (`RX` / `TX`) |
| **`SK2` + `Left` / `Right`** | Уменьшение / увеличение выходной мощности | Уменьшение / увеличение выходной мощности |
| **`SK2` + `Green`** | Быстрый вход в `Channel Details` | Быстрый вход в `Channel Details` |
| **`Orange` (Оранжевая)** | Быстрое меню канала (`Quick Menu`) | Быстрое меню VFO (`Quick Menu`) |
| **Удержание `SK2`** | Режим монитора (`Monitor Mode` — отключает фильтры CC/TS/TG и шумодав) | Режим монитора (`Monitor Mode`) |
| **`SK1` (верхняя) + `Red`** | Переключение дневной и ночной темы экрана | Переключение дневной и ночной темы экрана |
| **`SK2` + цифра `0..9`** | Вызов быстрой клавиши (`QuickKey`). Удержание в меню назначает пункт на цифру | Вызов быстрой клавиши (`QuickKey`) |

#### 2. Управление внутри анализатора спектра (`Sweep Scan` в VFO)

| Кнопка / Комбинация | Действие внутри анализатора спектра |
| :--- | :--- |
| **`Up` / `Down` (или энкодер)** | Сдвиг центральной частоты обзора вверх или вниз. |
| **`SK2` + `Up` / `Down`** | Изменение полосы обзора (`Span`) вплоть до панорамного режима **`Wide 20 MHz`** (10 подвыборок на столбец). |
| **`Left` / `Right`** | Регулировка усиления графика (`Gain` — отключает авто-масштаб). |
| **`SK2` + `Left` / `Right`** | Регулировка уровня шумовой полки (`Noise Floor` — отключает авто-масштаб). |
| **`SK1` + `Up` / `Down`** | Сброс настроек шкалы, возврат в **авто-масштаб** и очистка удержанных пиков (`Max Hold`). |
| **Короткое `*`** | В режиме сканирования: переключение автопрокрутки диапазона (**`Off` → `Auto scroll` (1 проход) → `Dwell 4x` (4 прохода на окно)**).<br>При захвате сигнала: **пропуск текущей частоты** и продолжение сканирования. |
| **`SK2` + `*`** | Переключение вида отображения: **Водопад + столбцы** ↔ **Полноэкранные столбцы**. |
| **Любая другая кнопка / `Red`** | Выход из анализатора спектра в обычный режим VFO. |

---

### 🛠 Сборка и прошивка

Автоматическая сборка всех профилей выполняется через **GitHub Actions** при каждом коммите в ветку `main`.

Для ручной сборки и загрузки в рацию (требуется файл-донор вокодера от **TYT MD-9600 V5**):
```bash
cd MDUV380_firmware
make PROFILE=full -j$(nproc)
python3 tools/opengd77_stm32_firmware_loader.py -s <MD9600-V5-donor.bin>         -f build/openuv380-10w.bin -m MD-UV380
```
> **Внимание:** прошивка предназначена исключительно для 10-ваттной версии **TYT MD-UV390 Plus (10W)**. Не прошивайте сборку для 5W в 10W-рацию (и наоборот).

---
---

## 🇬🇧 English

Custom firmware build of [OpenGD77](https://opengd77.com) for the **TYT MD-UV390 Plus (10W)** transceiver (based on official source release **R20260131**, target `MDUV380_10W_PLUS_FW`, build profile **`PROFILE=full`**) featuring **DMRA AES-256 encrypted voice & SMS**, Anytone-style Dual Screen, fast waterfall spectrum analyzer, on-screen AES key indicators, and ergonomic one-handed keypad lock.

---

### ⚠️ Legal Disclaimer

* **Amateur Radio Restrictions:** AES-256 encrypted voice and data transmissions are **illegal on amateur radio bands** in most jurisdictions. Use encryption strictly on licensed commercial, PMR, or government frequencies where authorized, or into a dummy load for testing.
* **Non-Commercial Open Source License:** This firmware inherits the OpenGD77 license (BSD-3-Clause with a non-commercial clause — see [`license.txt`](license.txt)). **Commercial use or sale of the source code, binaries, or pre-flashed radios is strictly forbidden.**
* **No Pre-loaded Keys or Proprietary Codec:** Neither the source code nor the release binaries contain any encryption keys (all 15 key slots are empty by default) or the proprietary **AMBE+2 vocoder**. Users must supply their own **TYT MD-9600 V5** donor firmware at flash time to merge the DMR codec.
* **Provided "AS IS":** This software is provided "as is" without warranty of any kind. The end user is solely responsible for compliance with local radio regulations and laws.

---

### ✨ What's Included in This Build (`PROFILE=full`)

#### 1. Upstream Improvements from [`dondch/OpenGD77-AES256`](https://github.com/dondch/OpenGD77-AES256)
* **DMRA AES-256 Voice Engine (OFB):** Full interoperability with stock TYT "Universal" AES256, Motorola, and Hytera radios (ETSI TS 102 361). Transmits 32-bit Message Indicator (MI) in PI headers and in-band AMBE Late-Entry MI (Golay(24,12)+CRC4 + EMB Single Block) for instant sync on back-to-back calls and late entry.
* **Persistent 15-Slot Key Store (`0xDA000`):** On-radio 64-character HEX editor (`Menu -> AES Keys`), host CLI loader `aes_key_tool.py` (CPS command `0x80`), strict TX protection when a selected key slot is empty (`No AES key`, never falls back to clear voice), and automatic DMR-ID database capping to prevent key table overwrite.
* **Encrypted DMR SMS (`ENABLE_DMR_DATA`):** AES-256-ECB encrypted text messaging interoperable with stock TYT radios, `Inbox` / `Sent` folders, unread message envelope indicator, and quick-text presets.
* **Secure Remote Provisioning (`ENABLE_REMOTE_PROVISION`):** Over-the-air codeplug & AES key provisioning from master to slave radios over DMR data (AES-256-CTR + 128-bit AES-CMAC, `tools/provision.py`).
* **Fast Waterfall Spectrum Analyzer & Fast Scan (`ENABLE_FAST_SCAN` / `ENABLE_SPECTRUM`):** ~4 ms per sample sweep step, panoramic `Wide 20 MHz` span (10 sub-samples per column at 12.5 kHz spacing with zero frequency gaps), color waterfall, auto noise floor scaling, `Max Hold`, peak frequency marker, `Auto scroll` with `Dwell 4x`, auto-park on signal, and sub-30 ms analog scanning down to `6 ms`.
* **MD-UV390 Plus 10W Hardware Fixes:** Proper `.ccmram` initialization at boot, low-brightness LCD panel support, and 10W PA control.

#### 2. Custom Additions in This Fork (`aikisgoliy`)
* **Dynamic Encryption & Key Badge in Status Bar (`uiHomeScreen.c`):**
  * `AES<N>` — standby on a digital channel (displays active channel or global TX key slot `1..15`).
  * `TX A<N>` — transmitting with encryption enabled (displays active TX key slot).
  * `RX A<N>` — receiving and decrypting an incoming call (displays matched key slot).
  * `ENC ?` — receiving an encrypted DMR call with no matching key in memory.
  * `OPEN` — receiving a clear (unencrypted) DMR call on an encrypted channel.
  * `DMR` / `FM` / `SCAN` — standard unencrypted modes.
* **Per-Row Key Badge in Dual Screen (`uiDualScreen.c`):**
  * Each row (A and B) dynamically renders **`DMR AES<N>`** when an encryption key is assigned or active on RX, or **`DMR`** / **`ANA`** otherwise.
* **Smart Dual Screen ↔ Call Screen Switching (`uiUtilities.c`):**
  * Displays the two-row Dual Screen in standby, automatically switches to the full telemetry/caller home screen during RX or TX, and returns to Dual Screen once the call ends.
* **One-Handed Keypad Lock (`applicationMain.c`, `uiChannelMode.c`, `uiVFOMode.c`):**
  * Long-pressing **`*`** (1 second) in Channel or VFO mode locks the keypad; long-pressing **`*`** (1 second) again unlocks it via direct hardware matrix polling (`keyboardRead()`).
* **Configurable PTT Lock (`Options -> General Options -> Star lock PTT`):**
  * **`Off` (default):** Long-press `*` locks **keypad only**, keeping **PTT active**.
  * **`On`:** Long-press `*` locks **both keypad and PTT**.
  * Standard **`Menu -> Lock -> Keypad & PTT`** continues to lock both regardless of this setting.

---

### 🧩 Credits & Base Modules

| Module / Feature | Source / Author | Contribution |
| :--- | :--- | :--- |
| **OpenGD77 Core** | **VK3KYY**, **F1RMB**, **DG4KLU** et al. (`R20260131`) | Core firmware for `TYT MD-UV390 Plus 10W`, DMR/FM stack, hotspot, GPS, satellite tracker. |
| **DMRA AES-256, SMS & Provisioning** | **`dondch/OpenGD77-AES256`** | AES-256 OFB voice encryption (stock TYT compatible), 15-slot key store, encrypted SMS, remote provisioning, fast scan & waterfall. |
| **Redesigned UI & Telemetry** (`ENABLE_TA2TWC_SCREENS`) | **`josephddt/OpenGD77-TA2TWC`** (**TA2TWC**) | Redesigned home screen with large frequency, S-meter (dBm/SNR), signal history graph, large caller callsign, TX timeout bar, needle RSSI meter, dark theme & compact 9-line menus. |
| **Dual Screen & Row Watch** (`ENABLE_DUAL_SCREEN`) | **`erenbozaci/md-uv390-firmware`** (**Eren TA3BZC**) | Anytone-style two-row home screen (VFO or zone channels), background analog `Row watch`, and `DW Options` menu. |
| **Waterfall Spectrum Scope** | **`dondch`** (inspired by **`mangelajo`** & **`fagci`**) | Color waterfall, auto-scale, `Max Hold`, peak frequency marker, auto-scroll, and signal auto-park. |
| **Custom Fork Additions** | **`aikisgoliy/OpenGD77-AES256`** | Encryption & key ID badges (`AES<N>`, `TX A<N>`, `RX A<N>`, `ENC ?`, `OPEN`) in status bar and Dual Screen rows; smart Dual Screen switching; 1s `*` lock/unlock & `Star lock PTT` option. |

---

### 📖 Menu Reference

Press **`Green`** to open the Main Menu. Use **`Up` / `Down`** to navigate, **`Left` / `Right`** to change values, **`Green`** to confirm/save, and **`Red`** to cancel/go back.

#### 1. Main Menu

| Menu Item | Description |
| :--- | :--- |
| **Zone** | Select active channel zone (including `All Channels`). |
| **RSSI** | Real-time needle & bar S-meter (dBm and S-units). |
| **Radio Info** | Battery voltage, CPU temperature, build version, GPS status, and free memory. |
| **Contacts** | Digital contacts (TG / Private Call) and DTMF contacts. |
| **Last Heard** | Log of recently received DMR stations (callsign/ID, TG, TS, time, RSSI). |
| **Firmware Info** | Build timestamp, target hardware, and compile flags. |
| **AES Keys** | Manage 15 AES-256 key slots and select Global TX key. |
| **Messages** | Encrypted DMR SMS (`Inbox` / `Sent` / `New Message`). |
| **Options** | System settings (`General`, `Radio`, `Display`, `Sound`). |
| **Channel Details** | Edit current channel/VFO parameters (RX/TX freq, TS, CC, power, CTCSS/DCS, AES key slot). |
| **Dual Watch** | Configure Dual Screen and Dual Watch behavior (`DW Options`). |
| **Satellite** | Amateur satellite pass predictor with Doppler tracking. |
| **GPS** | Coordinates, altitude, speed, heading, and satellite fix status. |
| **Lock** | Manual lock mode: `Keypad` (keys only) or `Keypad & PTT` (keys + TX). |

#### 2. `AES Keys` & `Messages` Submenus

| Submenu | Item | Controls & Description |
| :--- | :--- | :--- |
| **AES Keys** | **`Key 1` .. `Key 15`** | Key slots (`set` = stored, `---` = empty). Press **`Green`** to open the 64-nibble HEX editor (`0–9` direct entry, `Up/Down` cycle `0-F`, `Left/Right` cursor, `Green` save). Press **`SK2 + Green`** to clear slot. |
| **AES Keys** | **`Global TX key`** | Select fallback TX key (`Off` or `1..15`) when no per-channel key is set. |
| **Messages** | **`Inbox (N)` / `Sent (N)`** | Received (`*` = unread) and sent encrypted SMS folders. **`Green`** to read, **`SK2 + Green`** to delete message (or `[Delete all]`). |
| **Messages** | **`New Message`** | Multi-tap keypad text entry, **`Up`** cycles quick-text presets, **`Left`** backspace, **`Green`** opens recipient screen (`0–9` ID/TG, `Up/Down` `Group`/`Private`, `Green` send). |

#### 3. `Dual Watch` (`DW Options`)

| Option | Values | Description |
| :--- | :--- | :--- |
| **Dual screen** | `On` / `Off` | Enable Anytone-style two-row home screen (rows A and B). |
| **Row watch** | `On` / `Off` | Background peek at the inactive analog row every 1.5 s (80 ms dwell) when the active row is quiet. |
| **2nd side** | `VFO B` / `Channel` | Default second side for classic Dual Watch scan. |
| **Home** | `Current` / `VFO A` / `VFO B` / `Channel` | Priority home side for classic Dual Watch scan. |
| **Speed** | `Default` / `90ms` / `200ms` / `400ms` | Polling interval for classic Dual Watch scan. |
| **On signal** | `Switch` / `Stay` | `Switch` tunes to active signal; `Stay` beeps and remains on home side. |

#### 4. `Options` Submenus

##### `General Options`

| Option | Values | Description |
| :--- | :--- | :--- |
| **Key long** | `0.5s` .. `5.0s` | Hold duration required to trigger long-press actions. |
| **Key rpt** | `0.1s` .. `1.0s` | Key auto-repeat rate. |
| **Hotspot** | `Off` / `MMDVM` / `BlueDV` | USB digital hotspot mode. |
| **Auto lock** | `Off` / `0.5m` .. `15m` | Automatic keypad lock inactivity timer. |
| **APO** | `Off` / `30m` .. `240m` | Auto Power Off inactivity timer. |
| **APO with RF** | `On` / `Off` | Reset APO timer when receiving RF signals. |
| **Safe Pwr-On** | `Off` / `On` | Require holding `SK1` at power-on. |
| **Eco level** | `0` .. `5` | Receiver power-saving level in standby. |
| **Star lock PTT** | `Off` / `On` | *(Custom)* `Off` = long-press `*` locks keypad only (PTT active); `On` = locks both keypad and PTT. |

##### `Radio Options`

| Option | Values | Description |
| :--- | :--- | :--- |
| **Band limits** | `Off` / `On` / `CPS` | Restrict TX to ham bands (`Off` allows wide-band TX). |
| **TX inhibit** | `Off` / `On` | Disable all transmissions (RX-only mode). |
| **DMR filter time** | `2s` .. `90s` | Hold time for captured TS/CC when filters are off. |
| **Scan delay** | `1s` .. `30s` | Resume delay after signal drops during scan. |
| **Scan dwell** | `6ms` .. `480ms` | Per-channel dwell time during scan (supports fast scan down to `6ms`). |
| **Scan mode** | `Hold` / `Pause` / `Stop` | Scanner behavior upon detecting an active signal. |
| **Scan on boot** | `Off` / `On` | Automatically start scanning on power-up. |
| **Squelch VHF / 220 / UHF** | `0%` .. `100%` | Default squelch threshold per band. |
| **PTT toggle** | `Off` / `On` | Latching PTT mode (press once to TX, again to stop; requires TOT > 0). |
| **Private calls** | `Off` / `On` / `PTT` | Incoming DMR Private Call handling. |
| **User power** | `0` .. `4100` | PA DAC drive value for `User Power` level (step 100, with `SK2` step 10). |
| **DMR CRC** | `On` / `Off` | DMR frame CRC check (`Off` allows decoding weak/corrupted frames). |

##### `Display Options`

| Option | Values | Description |
| :--- | :--- | :--- |
| **Text size** | `1` / `2` | Standard or double-height font on the classic screen. |
| **Brightness / Night** | `1%` .. `100%` | Day and night backlight brightness levels. |
| **Min bright.** | `0%` .. `100%` | Idle backlight brightness level. |
| **Mode** | `Auto` / `Squelch` / `Manual` / `Buttons` | Backlight activation trigger mode. |
| **Timeout** | `No` / `5s` .. `30s` | Backlight off timeout. |
| **Invert** | `Normal` / `Invert` | Invert display colors. |
| **Auto night** | `Off` / `On` | Automatic day/night theme switching based on sunrise/sunset. |
| **Order** | `Ct/DB/TA` etc. | Caller ID display priority (Codeplug / DMR-ID DB / Talker Alias). |
| **Contact** | `1 line` / `2 lines` / `Auto` | Multi-line wrapping for long Talker Alias strings. |
| **Time / Battery** | `On/Off` & `%/V` | Header clock display and battery unit (percentage or voltage). |
| **Info** | `Off` / `TS` / `Pwr` / `Both` | Bold highlight for manually overridden timeslot or power level. |
| **Volume / LEDs** | `On` / `Off` | Volume bar popup and RX/TX LED enable/disable (stealth mode). |
| **Time zone / UTC** | `-12:00` .. `+14:00` | Timezone offset and UTC/local time toggle. |
| **Show distance** | `Off` / `On` | Display distance to repeater/channel when coordinates are set. |
| **Last talker** | `Off` / `1s` .. `30s` | Periodically alternate last heard callsign on standby screen. |
| **Main screen** | `New` / `Classic` | `New` = telemetry screen with S-meter, history graph & `AES<N>`/`TX A<N>`/`RX A<N>` badges; `Classic` = stock layout. |

##### `Sound Options`

| Option | Values | Description |
| :--- | :--- | :--- |
| **Timeout beep** | `N/A` / `5s` .. `20s` | Warning beep before TOT expiration. |
| **Beep volume** | `-24dB` .. `+6dB` | Keypad and alert beep volume. |
| **DMR beep** | `None` / `Start` / `Stop` / `Both` | DMR TX start/end roger beep. |
| **RX beep** | `None` / `Carrier` / `Talker` / `Both` | Audio alert on carrier or talker end. |
| **DMR / FM mic gain** | `-12dB` .. `+21dB` | Separate microphone gain for DMR and FM. |
| **VOX threshold / tail** | `2..30` / `0.0s..5.0s` | VOX sensitivity and hang delay. |
| **Audio prompt** | `Silent` / `Beep` / `No keys` / `Voice L1..L3` | Audio feedback and voice prompt verbosity. |
| **DMR Rx AGC** | `Off` / `0dB` .. `21dB` | Automatic gain control for DMR digital audio. |

---

### ⌨️ Keypad Shortcuts (TYT MD-UV390)

#### 1. Main Screen (Channel Mode & VFO Mode)

| Key / Combo | Channel Mode | VFO Mode |
| :--- | :--- | :--- |
| **Long-press `*` (1 s)** | **Lock / Unlock keypad** | **Lock / Unlock keypad** |
| **Short-press `*`** | DMR: toggle timeslot (`TS1` ↔ `TS2`). FM: toggle BW (`25 kHz` ↔ `12.5 kHz`) | DMR: toggle timeslot (`TS1` ↔ `TS2`). FM: toggle BW (`25 kHz` ↔ `12.5 kHz`) |
| **`SK2 (blue)` + `*`** | Toggle channel mode (`DMR` ↔ `FM`) | Toggle VFO mode (`DMR` ↔ `FM`) |
| **Long-press `#`** | **Reverse RX/TX frequencies** (`Reverse Repeater`) | **Launch Spectrum Analyzer** (`Sweep Scan` with waterfall) |
| **Short-press `#`** | Direct TG / Private ID entry | Direct TG / Private ID entry |
| **`SK2` + `#`** | Contact Quicklist | Contact Quicklist |
| **Long-press `0`** | Toggle audio mute (`Mute`) | Toggle audio mute (`Mute`) |
| **Long-press `8`** | Show bearing & distance to repeater (GPS) | — |
| **`Up` / `Down`** | Dual Screen: switch active row (`A` ↔ `B`). Normal: change channel | Dual Screen: switch active row (`A` ↔ `B`). Normal: step frequency |
| **Long-press `Up`** | Start zone channel scan | Start VFO frequency scan |
| **`Red`** | Dual Screen: toggle active row (`Channel` ↔ `VFO`). Normal: switch to VFO | Dual Screen: toggle active row (`VFO` ↔ `Channel`). Normal: switch to Channels (long: `VFO A` ↔ `VFO B`) |
| **`SK2` + `Up` / `Down`** | Change zone | Toggle frequency input target (`RX` / `TX`) |
| **`SK2` + `Left` / `Right`** | Decrease / increase TX power | Decrease / increase TX power |
| **`SK2` + `Green`** | Open `Channel Details` | Open `Channel Details` |
| **`Orange`** | Open `Quick Menu` | Open `Quick Menu` |
| **Hold `SK2`** | Monitor Mode (bypasses CC/TS/TG filters & squelch) | Monitor Mode |
| **`SK1 (top)` + `Red`** | Toggle Day / Night theme | Toggle Day / Night theme |
| **`SK2` + `0..9`** | Trigger assigned QuickKey (long-press inside menus to assign) | Trigger assigned QuickKey |

#### 2. Inside Spectrum Analyzer (`Sweep Scan` in VFO)

| Key / Combo | Action |
| :--- | :--- |
| **`Up` / `Down` (or Rotary)** | Shift center frequency up / down. |
| **`SK2` + `Up` / `Down`** | Adjust frequency span up to panoramic **`Wide 20 MHz`** (10 sub-samples per column). |
| **`Left` / `Right`** | Adjust display gain (switches to manual scaling). |
| **`SK2` + `Left` / `Right`** | Adjust noise floor offset (switches to manual scaling). |
| **`SK1` + `Up` / `Down`** | Reset scale, re-enable **auto-scaling**, and clear `Max Hold` peaks. |
| **Short-press `*`** | While scanning: cycle auto-scroll (**`Off` → `Auto scroll` (1 pass/window) → `Dwell 4x` (4 passes/window)**).<br>While parked on a signal: **skip current frequency** and resume sweep. |
| **`SK2` + `*`** | Toggle display layout: **Waterfall + Bars** ↔ **Full-Height Bars**. |
| **Any other key / `Red`** | Exit Spectrum Analyzer back to VFO mode. |

---

### 🛠 Build & Flash

Automated builds for all profiles run via **GitHub Actions** on every push to `main`.

To build and flash manually (requires an official **TYT MD-9600 V5** firmware file as the AMBE+2 codec donor):
```bash
cd MDUV380_firmware
make PROFILE=full -j$(nproc)
python3 tools/opengd77_stm32_firmware_loader.py -s <MD9600-V5-donor.bin>         -f build/openuv380-10w.bin -m MD-UV380
```
> **Warning:** This firmware is built exclusively for the **TYT MD-UV390 Plus (10W)** hardware variant. Do **not** flash a 5W image onto a 10W radio (or vice versa).
