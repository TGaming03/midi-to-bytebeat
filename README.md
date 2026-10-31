# MIDI → Bytebeat Studio

**Браузерный конвертер MIDI в Bytebeat и Floatbeat.** Загрузи MIDI-файл или поддерживаемый JSON, настрой дорожки и тембры, прослушай результат и получи одно JavaScript-выражение для Bytebeat/Floatbeat-плеера.

[Открыть онлайн-версию](https://tgaming03.github.io/midi-to-bytebeat/) · [Исходный код](https://github.com/TGaming03/midi-to-bytebeat)

---

## Содержание

- [Русский](#русский)
  - [Возможности](#возможности)
  - [Поддерживаемые форматы](#поддерживаемые-форматы)
  - [Настройки конвертации](#настройки-конвертации)
  - [Дорожки и тембры](#дорожки-и-тембры)
  - [Как работает аудиодвижок](#как-работает-аудиодвижок)
  - [Прослушивание и экспорт](#прослушивание-и-экспорт)
  - [Как пользоваться](#как-пользоваться)
  - [Ограничения и примечания](#ограничения-и-примечания)
  - [Конфиденциальность и технологии](#конфиденциальность-и-технологии)
  - [Авторство](#авторство)
  - [Лицензия](#лицензия)
- [English](#english)
  - [Features](#features)
  - [Supported input formats](#supported-input-formats)
  - [Conversion settings](#conversion-settings)
  - [Tracks and timbres](#tracks-and-timbres)
  - [Audio engine](#audio-engine)
  - [Preview and export](#preview-and-export)
  - [How to use](#how-to-use)
  - [Limitations and notes](#limitations-and-notes)
  - [Privacy and technologies](#privacy-and-technologies)
  - [Authorship](#authorship)
  - [License](#license)

# Русский

## Возможности

- Конвертация MIDI в **Bytebeat** (значения `0–255`) или **Floatbeat** (значения `−1…1`).
- Поддержка файлов `.mid`, `.midi` и JSON двух распространённых структур: `@tonejs/midi` и VisiPiano.
- Обработка нескольких дорожек и нот с сохранением их временного положения и длительности.
- Чтение карты темпа и событий смены темпа в MIDI.
- Чтение MIDI Program Change, громкости каналов CC7 и CC11, а также Pitch Bend с диапазоном, заданным через RPN.
- Отдельная настройка каждой дорожки: включение/отключение, выбор одного из 16 синтезированных тембров и громкость.
- Автоматический подбор тембра на основе номера GM-программы или названия инструмента, когда оно доступно.
- Отдельная обработка ударных на MIDI-канале 10 с использованием GM-таблицы ударных.
- Настройка частоты дискретизации, общей громкости, скорости, транспонирования и максимальной продолжительности.
- Предварительное прослушивание прямо в браузере.
- Копирование сгенерированного выражения в буфер обмена и скачивание его как `bytebeat.js`.
- Перетаскивание файла на страницу (drag and drop).
- Интерфейс на русском и английском языках.
- Светлая и тёмная темы с запоминанием выбранных языка и темы в настройках браузера.
- Адаптивная вёрстка для узких и широких экранов.

## Поддерживаемые форматы

### MIDI: `.mid` и `.midi`

Это рекомендуемый формат: в нём доступно больше MIDI-событий, чем в упрощённых JSON-представлениях.

Парсер читает:

- MIDI-заголовок и дорожки (`MThd` и `MTrk`);
- временную сетку PPQ (ticks per quarter note);
- события изменения темпа;
- названия дорожек;
- Note On и Note Off (в том числе Note On с нулевой velocity как завершение ноты);
- Program Change — смену GM-программы инструмента;
- CC7 (Channel Volume) и CC11 (Expression);
- Pitch Bend;
- RPN 0,0 и Data Entry MSB (CC6) для определения диапазона Pitch Bend.

Если диапазон Pitch Bend не задан через RPN, используется стандартное значение в 2 полутона. MIDI-канал 10 (внутренний номер канала 9) распознаётся как канал ударных.

Дорожки в интерфейсе формируются по сочетанию исходной MIDI-дорожки, канала и программы инструмента. Поэтому одна исходная MIDI-дорожка с несколькими Program Change может отображаться несколькими отдельными группами.

### JSON

Поддерживаются JSON-структуры, совместимые с:

- **`@tonejs/midi`** — используются данные заголовка, PPQ, темпа, дорожек, нот, инструментов, каналов и доступных событий Pitch Bend;
- **VisiPiano** — распознаётся массив `tempo` с полями `absoluteTime` и `bpm`; при наличии подходящих данных восстанавливается временная карта темпа.

Для JSON парсер учитывает следующие варианты полей:

- PPQ: `header.ppq` или `header.PPQ` (если отсутствует, используется 480);
- карта темпа: `header.tempos` либо VisiPiano `tempo` с `absoluteTime`; иначе используется BPM из заголовка/первого tempo-события или 120 BPM;
- дорожки: `tracks`;
- инструмент: `instrument` (строка или объект с `name`/`number`), `instrumentName` или `instrumentNumber`;
- MIDI-канал: `channel` или `channelNumber`;
- ноты: `midi`, `velocity`, `ticks`, `time`, `durationTicks` и `duration`;
- Pitch Bend: `pitchBends`; для диапазона изгиба может использоваться `controlChanges['6']`.

JSON-парсер не восстанавливает последовательности Program Change и автоматизацию громкости CC7/CC11 как полноценную временную историю. Для наиболее полного результата используй исходный MIDI-файл. Наличие Pitch Bend в JSON зависит от того, содержит ли исходный JSON соответствующие события; обычный экспорт VisiPiano может их не включать.

## Настройки конвертации

| Настройка | Значения / поведение |
|---|---|
| **Sample rate / Частота дискретизации** | 8000, 11025, 22050, 32000 или 44100 Гц. По умолчанию — 22050 Гц. |
| **Output format / Формат** | Bytebeat (`0–255`) или Floatbeat (`−1…1`). |
| **Max seconds / Макс. секунд** | `0` означает всю песню; положительное значение ограничивает длительность конвертации. По умолчанию — `0`. |
| **Master gain / Общая громкость** | Общий коэффициент громкости; начальное значение — `0.4`. |
| **Speed / Скорость** | Изменение скорости воспроизведения; начальное значение — `1×`, предусмотренный диапазон интерфейса — `0.25×–4×`. |
| **Transpose / Транспонирование** | Перемещение мелодических нот на −48…+48 полутонов; по умолчанию — `0`. На ударные не влияет. |
| **120 BPM before first tempo change** | Включено по умолчанию. Если первый tempo event начинается позже нулевого тика, перед ним применяется 120 BPM. При отключении первый найденный темп применяется с начала. |

Частота дискретизации, выбранная при конвертации, должна совпадать с частотой дискретизации в Bytebeat/Floatbeat-плеере. Для Bytebeat/Floatbeat также выбери в плеере соответствующий режим.

## Дорожки и тембры

После загрузки файла появляется список найденных нотных групп. Для каждой группы доступны:

- **Флажок** — включить или исключить группу из результата;
- **Список тембров** — выбрать другой тембр независимо от исходной GM-программы;
- **Громкость дорожки** — от 0 до 150%.

В информации о группе показываются её порядковый номер, название (если оно есть в MIDI/JSON), MIDI-канал, номер программы и количество нот.

Доступны 16 синтезированных тембров:

| № | Тембр | Характер |
|---:|---|---|
| 1 | Piano | Фортепиано |
| 2 | Electric piano (FM) | Электропиано с FM-синтезом |
| 3 | Organ / accordion | Орган / аккордеон |
| 4 | Plucked / guitar | Щипковый звук / гитара |
| 5 | Overdrive guitar | Перегруженная гитара |
| 6 | Bass | Бас |
| 7 | Strings | Струнные |
| 8 | Choir / voice | Хор / голос |
| 9 | Brass | Медные духовые |
| 10 | Reeds | Язычковые духовые |
| 11 | Mallets (xylophone…) | Ксилофон и другие ударные мелодические инструменты |
| 12 | Synth lead | Синтезаторный лид |
| 13 | Flute / winds | Флейта / деревянные духовые |
| 14 | Pad | Пэд |
| 15 | Bell percussion | Звонкая перкуссия |
| 16 | Noise sweep | Шумовой свип |

Это собственные синтезированные тембры движка, а не воспроизведение оригинальных звуков из GM SoundFont. Номер GM-программы и название инструмента используются для выбора наиболее подходящей категории тембра. Тембр любой группы можно переопределить вручную.

### Автоматическое сопоставление инструментов

Если у инструмента есть номер GM Program, он сопоставляется с одной из 16 категорий тембров по диапазону номера программы. Если номер программы отсутствует, парсер пытается определить тембр по названию инструмента (например, `bass`, `guitar`, `violin`, `choir`, `trumpet`, `flute`, `organ`, `pad` или `synth`). Если определить категорию не удаётся, используется Piano.

### Ударные

Канал 10 MIDI обрабатывается отдельно от мелодических дорожек. Для MIDI-нот ударных используется встроенная таблица параметров GM percussion для нот 35–87 (включая поддерживаемые значения этой таблицы). Разные типы ударов используют разные модели синтеза: тональные и шумовые составляющие, короткие затухающие импульсы и другие формы звука.

Ударные не транспонируются настройкой Transpose. Их тембр определяется номером MIDI-ноты ударного, а не выбранной мелодической категорией.

## Как работает аудиодвижок

Генератор создаёт автономное JavaScript-выражение. Внутри выражения находится сам аудиодвижок, параметры 16 тембров, таблица ударных и закодированные события нот. Для генерации результата не требуется загружать отдельную библиотеку или звуковой банк.

В реализации используются:

- огибающие атаки, затухания и релиза для формирования нот;
- гармонический синтез и индивидуальные наборы гармоник;
- FM-синтез для отдельных тембров;
- расстройка осцилляторов и дополнительные гармонические компоненты;
- модуляция высоты тона для отдельных тембров;
- шумовые компоненты и короткие переходные импульсы;
- частотное ограничение и ослабление высоких гармоник с учётом частоты дискретизации;
- отдельные модели синтеза для разных категорий ударных;
- ограничение чрезмерной громкости и нормализация с учётом числа одновременно звучащих голосов.

В движке предусмотрены номинальные пределы одновременных голосов: 14 мелодических и 6 ударных. Голоса сверх этих значений плавно ослабляются и затем исключаются, чтобы уменьшать перегрузку при плотной полифонии.

Это синтезаторный движок: он не пытается извлечь или встроить исходные сэмплы инструментов из MIDI-файла. Поэтому результат может отличаться от звучания MIDI в SoundFont-плеере или аппаратном синтезаторе.

## Прослушивание и экспорт

После успешной конвертации доступны следующие действия:

- **Preview / Прослушать** — браузерный рендер аудио через Web Audio API и воспроизведение результата;
- **Stop / Стоп** — остановка воспроизведения;
- **Copy / Копировать** — копирование JS-выражения;
- **Download .js / Скачать .js** — сохранение выражения в файл `bytebeat.js`.

Во время рендера отображается прогресс. Предварительное прослушивание сначала рассчитывает PCM-буфер частями, а затем воспроизводит его через `AudioContext`.

### Использование с внешним плеером

Скопируй выражение или открой скачанный `bytebeat.js` и вставь выражение в совместимый Bytebeat/Floatbeat-плеер. Плеер должен:

1. предоставлять переменную `t` — индекс текущего аудиосэмпла;
2. работать в браузерном JavaScript-контексте с объектом `window`;
3. использовать ту же частоту дискретизации, которая выбрана при конвертации;
4. использовать режим Bytebeat для диапазона `0–255` или Floatbeat для диапазона `−1…1`.

Экспортируемый код возвращает значение для одного сэмпла при каждом вызове. Это не отдельный аудиофайл и не самостоятельный проигрыватель: полученное выражение предназначено для исполнения внутри совместимого плеера.

## Как пользоваться

1. Открой страницу MIDI → Bytebeat Studio.
2. Выбери `.mid`, `.midi` или поддерживаемый `.json` через поле файла либо перетащи его на страницу.
3. Проверь найденные дорожки и отключи ненужные.
4. При необходимости измени тембр и громкость каждой дорожки.
5. Выбери частоту дискретизации и Bytebeat/Floatbeat.
6. Настрой общую громкость, скорость, транспонирование и максимальную длительность.
7. Нажми **Convert / Конвертировать**.
8. Прослушай результат или скопируй/скачай сгенерированное выражение.
9. Внешний плеер настрой на ту же частоту дискретизации и тот же режим.

## Ограничения и примечания

- MIDI-файлы с SMPTE time division не поддерживаются. Используй MIDI с PPQ timing.
- Для наиболее точной обработки рекомендуется исходный `.mid`/`.midi`, а не промежуточный JSON.
- В JSON могут отсутствовать Program Change, CC7/CC11, Pitch Bend или другие исходные MIDI-события. Конвертер не может восстановить информацию, которой нет в файле.
- JSON-парсер не обрабатывает Program Change и временную автоматизацию CC7/CC11 как MIDI-парсер.
- Автоматически выбранный тембр — приблизительное соответствие категории инструмента; оригинальные SoundFont-сэмплы не используются.
- Очень длинные композиции и большое количество нот могут создавать большой JS-код и требовать заметного времени/памяти для рендера. Отключение ненужных дорожек или ограничение длительности уменьшает объём данных. Понижение sample rate уменьшает количество рассчитываемых сэмплов в секунду, но само по себе не сокращает список нот в экспортируемом коде.
- Для прослушивания браузер должен поддерживать Web Audio API (`AudioContext`). Некоторые браузеры запускают аудио только после действия пользователя.
- Bytebeat/Floatbeat-плеер может иметь собственные ограничения на размер выражения, длительность или вычислительную сложность.

## Конфиденциальность и технологии

- Вся обработка MIDI/JSON выполняется локально в браузере.
- Файлы не отправляются на сервер; HTML не содержит внешних CDN-библиотек или сетевых запросов для конвертации.
- Для интерфейса используются HTML, CSS и JavaScript без обязательных сторонних зависимостей.
- Предварительное прослушивание использует Web Audio API.
- Выбранные язык и тема могут сохраняться в `localStorage` браузера. Если хранилище недоступно, приложение продолжает работать без сохранения этих предпочтений.

Для локального запуска достаточно скачать `index.html` и открыть его в современном браузере с поддержкой JavaScript и Web Audio API. Сборка и установка пакетов не требуются.

## Авторство

**Весь проект реализован Claude (Anthropic).** Claude создал HTML-разметку, CSS и адаптивный интерфейс, JavaScript-логику, MIDI- и JSON-парсеры, карту темпа, обработку MIDI-контроллеров и Pitch Bend, набор из 16 синтезированных тембров, таблицу и модели ударных, аудиодвижок, генератор Bytebeat/Floatbeat-выражений, браузерное прослушивание, импорт файлов, переключение языков и тем, а также остальные функции приложения.

README подготовлен с помощью Claude на основе исходного `index.html`.

## Лицензия

Проект распространяется по лицензии **MIT**. Подробности см. в файле [`LICENSE`](LICENSE).

---

# English

## Features

- Converts MIDI into **Bytebeat** (`0–255`) or **Floatbeat** (`−1…1`).
- Accepts `.mid`, `.midi`, and JSON files using two common structures: `@tonejs/midi` and VisiPiano.
- Handles multiple note tracks and preserves note timing and duration.
- Reads MIDI tempo maps and tempo-change events.
- Reads MIDI Program Change, channel volume CC7, Expression CC11, and Pitch Bend with an RPN-defined bend range.
- Per-track controls: enable/disable, choose one of 16 synthesized timbres, and adjust track volume.
- Automatically selects a timbre from the GM program number or instrument name when available.
- Handles percussion separately on MIDI channel 10 using a built-in GM percussion table.
- Adjustable sample rate, master gain, playback speed, transpose, and maximum duration.
- In-browser audio preview.
- Copy the generated expression or download it as `bytebeat.js`.
- Drag-and-drop file loading.
- Russian and English interface.
- Light and dark themes; selected language and theme are saved in browser preferences.
- Responsive layout for narrow and wide screens.

## Supported input formats

### MIDI: `.mid` and `.midi`

This is the recommended input format because it can preserve more MIDI events than simplified JSON exports.

The MIDI parser reads:

- MIDI header and track chunks (`MThd` and `MTrk`);
- PPQ (ticks per quarter note) timing;
- tempo meta-events;
- track names;
- Note On and Note Off events (including Note On with zero velocity as a note-off);
- Program Change events;
- CC7 (Channel Volume) and CC11 (Expression);
- Pitch Bend;
- RPN 0,0 and Data Entry MSB (CC6) to determine the Pitch Bend range.

If no Pitch Bend range is specified through RPN, the parser uses a default range of 2 semitones. MIDI channel 10 (zero-based channel index 9) is treated as the percussion channel.

The interface groups events by source MIDI track, channel, and instrument program. A single source track containing multiple Program Change values can therefore appear as multiple groups.

### JSON

The parser supports JSON structures compatible with:

- **`@tonejs/midi`** — header data, PPQ, tempo, tracks, notes, instrument metadata, channels, and available Pitch Bend events;
- **VisiPiano** — a `tempo` array with `absoluteTime` and `bpm` fields is recognised; a tempo map can be reconstructed when the required data is present.

The parser recognises these field variants:

- PPQ: `header.ppq` or `header.PPQ` (fallback: 480);
- tempo map: `header.tempos` or VisiPiano `tempo` entries with `absoluteTime`; otherwise it uses BPM from the header/first tempo entry or 120 BPM;
- tracks: `tracks`;
- instrument: `instrument` (a string or an object containing `name`/`number`), `instrumentName`, or `instrumentNumber`;
- MIDI channel: `channel` or `channelNumber`;
- notes: `midi`, `velocity`, `ticks`, `time`, `durationTicks`, and `duration`;
- Pitch Bend: `pitchBends`; `controlChanges['6']` may be used to obtain the bend range.

The JSON parser does not reconstruct Program Change sequences or time-varying CC7/CC11 volume automation as a full MIDI event history. Use the original MIDI file for the most complete conversion. Pitch Bend can only be imported when the JSON contains the relevant events; a typical VisiPiano export may not include them.

## Conversion settings

| Setting | Values / behaviour |
|---|---|
| **Sample rate** | 8000, 11025, 22050, 32000, or 44100 Hz. Default: 22050 Hz. |
| **Output format** | Bytebeat (`0–255`) or Floatbeat (`−1…1`). |
| **Max seconds** | `0` means the whole song; a positive value limits the converted duration. Default: `0`. |
| **Master gain** | Overall gain coefficient; default: `0.4`. |
| **Speed** | Playback-speed multiplier; default: `1×`; the interface is intended for `0.25×–4×`. |
| **Transpose** | Shifts melodic notes by −48 to +48 semitones; default: `0`. Does not affect percussion. |
| **120 BPM before first tempo change** | Enabled by default. If the first tempo event occurs after tick zero, 120 BPM is used before it. When disabled, the first available tempo is applied from the beginning. |

The sample rate selected during conversion must match the sample rate configured in the Bytebeat/Floatbeat player. Select the matching output mode in the player as well.

## Tracks and timbres

After a file is loaded, the app displays the detected note groups. Each group has:

- **Checkbox** — include or exclude the group;
- **Timbre selector** — replace its automatically selected sound with another timbre;
- **Track volume** — adjustable from 0% to 150%.

Group information includes its index, source name (when available), MIDI channel, program number, and note count.

There are 16 synthesized timbres:

| No. | Timbre | Character |
|---:|---|---|
| 1 | Piano | Acoustic-piano-style synthesis |
| 2 | Electric piano (FM) | FM electric piano |
| 3 | Organ / accordion | Organ / accordion |
| 4 | Plucked / guitar | Plucked sound / guitar |
| 5 | Overdrive guitar | Distorted guitar |
| 6 | Bass | Bass |
| 7 | Strings | Strings |
| 8 | Choir / voice | Choir / voice |
| 9 | Brass | Brass instruments |
| 10 | Reeds | Reed instruments |
| 11 | Mallets (xylophone…) | Xylophone and related mallet sounds |
| 12 | Synth lead | Synth lead |
| 13 | Flute / winds | Flute / woodwinds |
| 14 | Pad | Pad |
| 15 | Bell percussion | Bell-like percussion |
| 16 | Noise sweep | Noise sweep |

These are the audio engine's own synthesized timbres, not original instrument samples from a GM SoundFont. GM program numbers and instrument names are used to choose an approximate timbre category. Any track's timbre can be overridden manually.

### Automatic instrument mapping

When a GM Program number is available, it is mapped to one of the 16 timbre categories according to its program-number range. When the program number is missing, the parser tries to infer a category from the instrument name (for example, `bass`, `guitar`, `violin`, `choir`, `trumpet`, `flute`, `organ`, `pad`, or `synth`). If no category can be identified, Piano is used.

### Percussion

MIDI channel 10 is handled separately from melodic tracks. Percussion notes use a built-in GM percussion parameter table for MIDI notes 35–87 (including the entries defined in that table). Different percussion types use different synthesis models, combining tonal and noisy components, short decaying impulses, and other sound shapes.

The Transpose setting does not transpose percussion. A drum sound is selected by its MIDI percussion note number, not by the melodic timbre categories.

## Audio engine

The converter generates a self-contained JavaScript expression. The expression embeds the audio engine, all 16 timbre definitions, the percussion table, and encoded note events. No separate sound library or sample bank is needed at playback time.

The implementation uses:

- attack, decay, and release envelopes;
- harmonic synthesis with individual harmonic profiles;
- FM synthesis for selected timbres;
- oscillator detuning and additional harmonic components;
- pitch modulation for selected timbres;
- noise components and short transient impulses;
- frequency limiting and high-harmonic attenuation based on the selected sample rate;
- separate synthesis models for percussion categories;
- loudness limiting and normalization based on the number of simultaneously active voices.

The engine has nominal simultaneous-voice limits of 14 melodic voices and 6 percussion voices. Voices beyond those limits are smoothly attenuated and then dropped to help control CPU load and excessive level during dense polyphonic passages.

This is a synthesizer, not a sample-extraction tool. It does not embed or reproduce the original instrument samples from the MIDI file. Its sound can therefore differ from playback through a SoundFont player or a hardware synthesizer.

## Preview and export

After conversion, the following actions are available:

- **Preview** — renders audio in the browser through the Web Audio API and plays the result;
- **Stop** — stops playback;
- **Copy** — copies the generated JS expression to the clipboard;
- **Download .js** — saves the expression as `bytebeat.js`.

A progress indicator is shown while rendering. The preview first calculates the PCM buffer in chunks and then plays it through `AudioContext`.

### Using an external player

Copy the expression or open the downloaded `bytebeat.js`, then paste the expression into a compatible Bytebeat/Floatbeat player. The player must:

1. provide `t` as the current audio-sample index;
2. run the expression in a browser-style JavaScript context with a `window` object;
3. use the same sample rate selected during conversion;
4. use Bytebeat mode for values `0–255` or Floatbeat mode for values `−1…1`.

The exported code returns one sample value per invocation. It is not an audio file or a standalone player; it is an expression intended to run inside a compatible player.

## How to use

1. Open MIDI → Bytebeat Studio.
2. Select a `.mid`, `.midi`, or supported `.json` file, or drag it onto the page.
3. Review the detected tracks and disable any groups you do not need.
4. Change the timbre and volume of individual tracks if needed.
5. Select a sample rate and Bytebeat/Floatbeat output mode.
6. Set master gain, speed, transpose, and maximum duration.
7. Click **Convert**.
8. Preview the result or copy/download the generated expression.
9. Configure the external player with the same sample rate and output mode.

## Limitations and notes

- MIDI files using SMPTE time division are not supported. Use PPQ-timed MIDI files.
- The original `.mid`/`.midi` file is recommended for the most accurate event handling.
- JSON exports may omit Program Change, CC7/CC11, Pitch Bend, or other source MIDI events. Missing information cannot be reconstructed by the converter.
- The JSON parser does not process Program Change and time-varying CC7/CC11 automation as the MIDI parser does.
- Automatic timbre selection is an approximate instrument-category mapping; original SoundFont samples are not used.
- Very long songs and large note counts can produce large JS expressions and require substantial rendering time or memory. Disabling unused tracks or limiting the duration reduces the amount of encoded note data. Lowering the sample rate reduces the number of samples calculated per second, but does not by itself remove notes from the exported code.
- Preview requires a browser with Web Audio API (`AudioContext`) support. Some browsers only allow audio playback after a user interaction.
- A third-party Bytebeat/Floatbeat player may impose its own expression-size, duration, or computation limits.

## Privacy and technologies

- MIDI/JSON processing takes place locally in the browser.
- Files are not uploaded to a server; the HTML does not rely on external CDN libraries or network requests for conversion.
- The interface and converter use HTML, CSS, and JavaScript without required third-party dependencies.
- Audio preview uses the Web Audio API.
- The selected language and theme may be stored in browser `localStorage`. The app still works if browser storage is unavailable, but preferences may not persist.

To run locally, download `index.html` and open it in a modern browser with JavaScript and Web Audio API support. No build step or package installation is required.

## Authorship

**The entire project was implemented by Claude (Anthropic).** Claude created the HTML markup, CSS and responsive interface, JavaScript logic, MIDI and JSON parsers, tempo map, MIDI controller and Pitch Bend handling, all 16 synthesized timbres, the percussion table and synthesis models, the audio engine, the Bytebeat/Floatbeat expression generator, browser preview, file import, language and theme switching, and the rest of the application features.

This README was also prepared with Claude based on the source `index.html`.

## License

The project is distributed under the **MIT License**. See [`LICENSE`](LICENSE) for details.
