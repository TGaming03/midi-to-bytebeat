# MIDI → Bytebeat Studio

**Браузерный конвертер MIDI в Bytebeat и Floatbeat.** Загрузи MIDI-файл или поддерживаемый JSON, настрой дорожки и тембры, прослушай результат и получи одно JavaScript-выражение для Bytebeat/Floatbeat-плеера.

[Открыть онлайн-версию](https://tgaming03.github.io/midi-to-bytebeat/) · [Исходный код](https://github.com/TGaming03/midi-to-bytebeat)

---

## Содержание

- [Русский](#русский)
- [English](#english)

# Русский

## Возможности

- Конвертация MIDI в **Bytebeat** (значения `0–255`) или **Floatbeat** (значения `−1…1`).
- Файлы `.mid`, `.midi` и JSON двух структур: `@tonejs/midi` и VisiPiano.
- **Все 128 инструментов General MIDI: у каждой программы свой синтезированный тембр**, а также полный набор GM-ударных (ноты 35–87).
- Несколько дорожек с сохранением времени и длительности нот.
- Карта темпа и события смены темпа (ускорения и замедления).
- Program Change (смена инструментов в любой момент), громкость CC7 и CC11, Pitch Bend с диапазоном из RPN.
- Настройка каждой дорожки: включение, выбор любого из 128 тембров или ударных, громкость 0–150%.
- Частота дискретизации, общая громкость, скорость, транспонирование, максимальная длительность.
- Прослушивание в браузере с графиком волны и перемоткой по клику, копирование выражения и скачивание как `bytebeat.js`; статистика результата (ноты, дорожки, длительность, размер).
- Перетаскивание файла на страницу, интерфейс на русском и английском, светлая и тёмная темы.

## Поддерживаемые форматы

### MIDI: `.mid` и `.midi` (рекомендуется)

Читаются: заголовок и дорожки (`MThd`, `MTrk`), PPQ, смены темпа, названия дорожек, Note On/Off (Note On с velocity 0 считается окончанием ноты), Program Change, CC7, CC11, Pitch Bend, RPN 0,0 с Data Entry MSB (CC6) для диапазона изгиба. Без RPN диапазон равен 2 полутонам. Канал 10 (индекс 9) считается ударным.

Дорожки в интерфейсе строятся по сочетанию исходной дорожки, канала и программы. Если на одной дорожке инструмент меняется несколько раз, она показывается несколькими группами.

### JSON

- **`@tonejs/midi`**: заголовок, PPQ, темп, дорожки, ноты, инструменты, каналы, `pitchBends`.
- **VisiPiano**: массив `tempo` с `absoluteTime` и `bpm`. Время нот в этом формате посчитано при постоянном BPM, поэтому парсер восстанавливает тики и применяет реальную карту темпа.

Распознаваемые поля: `header.ppq` / `header.PPQ` (иначе 480), `header.tempos` или `tempo`, `tracks`, `instrument` (строка или объект с `name`/`number`), `instrumentName`, `instrumentNumber`, `channel` / `channelNumber`, `midi`, `velocity`, `ticks`, `time`, `durationTicks`, `duration`, `pitchBends`, `controlChanges['6']`.

Ограничения JSON: смена программ внутри дорожки и автоматизация CC7/CC11 в нём теряются, а в экспорте VisiPiano нет Pitch Bend. Если инструменты слиты в одну дорожку, восстановить их уже нельзя. Для полного результата используй исходный MIDI.

## Настройки конвертации

| Настройка | Значения / поведение |
|---|---|
| **Частота дискретизации** | 8000, 11025, 22050, 32000, 44100 Гц. По умолчанию 22050. |
| **Формат вывода** | Bytebeat (`0–255`) или Floatbeat (`−1…1`). |
| **Макс. секунд** | `0` — вся песня; положительное значение ограничивает длительность. |
| **Громкость** | Общий коэффициент, по умолчанию `0.4`. |
| **Скорость** | `0.25×–4×`, по умолчанию `1×`. |
| **Транспонирование** | −48…+48 полутонов; ударных не касается. |
| **120 BPM до первой смены темпа** | Включено по умолчанию (стандарт MIDI). Если выключить, первый найденный темп применяется с начала. |

Частота и режим в плеере должны совпадать с выбранными при конвертации.

## Дорожки и тембры

Для каждой группы доступны флажок, выбор тембра и громкость 0–150%. В описании группы указаны номер, название, канал, программа и число нот.

### 128 тембров General MIDI

Тембр выбирается по номеру GM-программы **напрямую**: у каждой из 128 программ свои параметры синтеза (набор гармоник, огибающая, FM, расстройка, вибрато, форманты, шум и т. д.). Например, скрипка, альт, виолончель и контрабас, или труба, тромбон, туба и валторна звучат по-разному. Любой тембр можно заменить вручную. В списке тембров также есть пункт «Ударные (GM)». Названия инструментов и семейств переведены: в русском интерфейсе они русские, в английском — стандартные GM-названия. Тембры в списке сгруппированы по 16 семействам.

| Программы | Семейство | Что входит |
|---|---|---|
| 1–8 | Piano | рояли, хонки-тонк, электропиано, клавесин, клавинет |
| 9–16 | Chromatic Percussion | челеста, глокеншпиль, музыкальная шкатулка, вибрафон, маримба, ксилофон, колокола, цимбалы |
| 17–24 | Organ | органы, аккордеоны, гармоника |
| 25–32 | Guitar | нейлон, сталь, джаз, клин, приглушённая, овердрайв, дисторшн, флажолеты |
| 33–40 | Bass | акустический, пальцы, медиатор, фретлесс, слэп, синтбасы |
| 41–48 | Strings | скрипка, альт, виолончель, контрабас, тремоло, пиццикато, арфа, литавры |
| 49–56 | Ensemble | струнные ансамбли, синтструнные, хоры, оркестровый удар |
| 57–64 | Brass | труба, тромбон, туба, валторна, секция, синтмедь |
| 65–72 | Reed | саксофоны, гобой, английский рожок, фагот, кларнет |
| 73–80 | Pipe | пикколо, флейта, рекордер, пан-флейта, бутылка, сякухати, свисток, окарина |
| 81–88 | Synth Lead | квадрат, пила, каллиопа, charang, solo vox и др. |
| 89–96 | Synth Pad | Fantasia, Warm, Polysynth, Space Voice, Bowed Glass, Metal, Halo, Sweep |
| 97–104 | Synth Effects | Ice Rain, Soundtrack, Crystal, Atmosphere, Brightness, Goblin, Echo Drops, Star Theme |
| 105–112 | Ethnic | ситар, банджо, сямисэн, кото, калимба, волынка, фидл, шанаи |
| 113–120 | Percussive | колокольчик, агого, стальные барабаны, вудблок, тайко, мелодический том, синтбарабан, реверс-тарелка |
| 121–128 | Sound Effects | шум струн, дыхание, прибой, птица, телефон, вертолёт, аплодисменты, выстрел |

Это собственные синтезированные тембры движка, а не сэмплы из SoundFont. Они передают характер инструмента, но звучат иначе, чем в сэмплерном плеере или на аппаратном синтезаторе.

### Автоматическое сопоставление

Если в файле есть номер GM-программы, используется именно он (0–127). Если номера нет, парсер пытается определить инструмент по названию (`bass`, `guitar`, `violin`, `choir`, `trumpet`, `flute`, `organ`, `pad`, `synth` и т. п.) и берёт подходящую программу. Если определить не удалось, используется фортепиано.

### Ударные

Канал 10 обрабатывается отдельно. Для нот 35–87 используется таблица параметров GM-ударных; у бочки, малого, томов, хэтов, тарелок, клэпа, ковбелла, конгов, бонго и остальных свои модели синтеза: тональные и шумовые составляющие, свип высоты, серия шумовых всплесков, металлические обертоны. Ударные не транспонируются, их звук определяется номером ноты.

## Как работает аудиодвижок

Генератор создаёт автономное JavaScript-выражение. В него входят сам движок, параметры **только использованных** тембров, таблица ударных и закодированные события нот. Внешние библиотеки и звуковые банки не нужны.

В движке используются: огибающие атаки, затухания и релиза; гармонический и FM-синтез; расстройка, вибрато и форманты; шумовые и переходные компоненты; негармоничные наборы обертонов для колоколов и маллетов; плавное ослабление высоких гармоник у границы Найквиста; отдельные модели для категорий ударных; нормализация по числу активных голосов и мягкий лимитер.

Одновременно звучат не более 14 мелодических и 6 ударных голосов; лишние плавно ослабляются и исключаются. Pitch Bend применяется внутри ноты как плавное изменение высоты.

## Прослушивание и экспорт

После конвертации доступны: **Прослушать** (рендер PCM-буфера частями через Web Audio API, с прогрессом), **Стоп**, **Копировать** и **Скачать .js** (`bytebeat.js`).

### Внешний плеер

Плеер должен: предоставлять переменную `t` (индекс сэмпла), выполнять код в браузерном JavaScript с объектом `window`, использовать ту же частоту дискретизации и режим Bytebeat или Floatbeat. Выражение возвращает значение одного сэмпла за вызов. Это не аудиофайл и не отдельный проигрыватель.

## Как пользоваться

1. Открой страницу и выбери `.mid`, `.midi` или `.json` (или перетащи файл).
2. Проверь дорожки, отключи лишние, при желании измени тембр и громкость.
3. Выбери частоту, формат вывода и остальные настройки.
4. Нажми **Конвертировать**.
5. Прослушай результат или скопируй/скачай выражение.
6. В плеере выставь ту же частоту и режим.

## Ограничения и примечания

- **Bank Select не читается.** Варианты банков (например, Ukulele, Church Organ 2, Wet Vibraphone) звучат как базовая программа своего номера.
- SMPTE time division не поддерживается, нужен MIDI с PPQ.
- Для максимально точного результата используй `.mid`, а не JSON.
- В JSON может не быть Program Change, CC7/CC11 и Pitch Bend; конвертер не восстанавливает отсутствующие данные.
- Автоматически выбранный тембр — приблизительное соответствие; настоящие сэмплы не используются.
- Длинные песни и большое число нот дают большой JS-код и долгий рендер. Помогают отключение дорожек и ограничение длительности. Понижение частоты уменьшает число считаемых сэмплов, но не сокращает список нот.
- Для прослушивания нужен Web Audio API; некоторые браузеры запускают звук только после действия пользователя.
- У сторонних плееров могут быть свои лимиты на размер и сложность выражения.

## Конфиденциальность и технологии

- Вся обработка выполняется локально в браузере, файлы никуда не отправляются.
- HTML не использует внешние CDN и сетевые запросы для конвертации; только HTML, CSS и JavaScript без зависимостей.
- Язык и тема могут сохраняться в `localStorage`; без хранилища приложение работает, но настройки не запоминаются.

Для локального запуска скачай `index.html` и открой его в современном браузере.

## Авторство

**Весь проект реализован Claude (Anthropic):** интерфейс и вёрстка, MIDI- и JSON-парсеры, карта темпа, CC и Pitch Bend, 128 синтезированных тембров, GM-ударные, аудиодвижок, генератор выражений, прослушивание, импорт файлов, переключение языка и темы. README подготовлен с помощью Claude на основе исходного `index.html`.

## Лицензия

MIT. См. [`LICENSE`](LICENSE).

---

# English

## Features

- Converts MIDI into **Bytebeat** (`0–255`) or **Floatbeat** (`−1…1`).
- Accepts `.mid`, `.midi`, and JSON in two structures: `@tonejs/midi` and VisiPiano.
- **All 128 General MIDI instruments, each with its own synthesized timbre**, plus a full GM drum kit (notes 35–87).
- Multiple tracks with preserved note timing and duration.
- Tempo maps and tempo changes (accelerando and ritardando).
- Program Change (instrument changes at any moment), CC7 and CC11 volume, Pitch Bend with an RPN-defined range.
- Per-track controls: enable, choose any of 128 timbres or drums, volume 0–150%.
- Sample rate, master gain, speed, transpose, and maximum duration.
- In-browser preview with a waveform and click-to-seek, copy the expression or download it as `bytebeat.js`; result statistics (notes, tracks, duration, size).
- Drag and drop, Russian and English UI, light and dark themes.

## Supported input formats

### MIDI: `.mid` and `.midi` (recommended)

The parser reads the header and tracks (`MThd`, `MTrk`), PPQ, tempo changes, track names, Note On/Off (Note On with velocity 0 ends a note), Program Change, CC7, CC11, Pitch Bend, and RPN 0,0 with Data Entry MSB (CC6) for the bend range. Without RPN the range is 2 semitones. Channel 10 (index 9) is treated as percussion.

The interface groups events by source track, channel, and program. A track with several instrument changes appears as several groups.

### JSON

- **`@tonejs/midi`**: header, PPQ, tempo, tracks, notes, instruments, channels, `pitchBends`.
- **VisiPiano**: a `tempo` array with `absoluteTime` and `bpm`. This format computes note times at a constant BPM, so the parser reconstructs ticks and applies the real tempo map.

Recognised fields: `header.ppq` / `header.PPQ` (fallback 480), `header.tempos` or `tempo`, `tracks`, `instrument` (string or object with `name`/`number`), `instrumentName`, `instrumentNumber`, `channel` / `channelNumber`, `midi`, `velocity`, `ticks`, `time`, `durationTicks`, `duration`, `pitchBends`, `controlChanges['6']`.

JSON limitations: program changes inside a track and CC7/CC11 automation are lost, and VisiPiano exports contain no Pitch Bend. If instruments are merged into one track, they cannot be recovered. Use the original MIDI for the most complete result.

## Conversion settings

| Setting | Values / behaviour |
|---|---|
| **Sample rate** | 8000, 11025, 22050, 32000, 44100 Hz. Default 22050. |
| **Output format** | Bytebeat (`0–255`) or Floatbeat (`−1…1`). |
| **Max seconds** | `0` means the whole song; a positive value limits the duration. |
| **Master gain** | Overall coefficient, default `0.4`. |
| **Speed** | `0.25×–4×`, default `1×`. |
| **Transpose** | −48 to +48 semitones; does not affect percussion. |
| **120 BPM before first tempo change** | On by default (MIDI standard). When off, the first tempo found applies from the start. |

The sample rate and mode in the player must match those chosen during conversion.

## Tracks and timbres

Each group has a checkbox, a timbre selector, and a volume slider (0–150%). Group information shows its index, name, channel, program, and note count.

### 128 General MIDI timbres

The timbre is chosen from the GM program number **directly**: every one of the 128 programs has its own synthesis parameters (harmonic set, envelope, FM, detune, vibrato, formants, noise, and so on). For example, violin, viola, cello, and contrabass, or trumpet, trombone, tuba, and French horn, sound different from each other. Any timbre can be overridden manually. The timbre list also contains a "Drums (GM)" entry. Instrument and family names are localized: Russian in the Russian UI, standard GM names in English. Timbres are grouped into 16 families in the list.

| Programs | Family | Includes |
|---|---|---|
| 1–8 | Piano | grand pianos, honky-tonk, electric pianos, harpsichord, clavinet |
| 9–16 | Chromatic Percussion | celesta, glockenspiel, music box, vibraphone, marimba, xylophone, bells, dulcimer |
| 17–24 | Organ | organs, accordions, harmonica |
| 25–32 | Guitar | nylon, steel, jazz, clean, muted, overdrive, distortion, harmonics |
| 33–40 | Bass | acoustic, finger, pick, fretless, slap, synth basses |
| 41–48 | Strings | violin, viola, cello, contrabass, tremolo, pizzicato, harp, timpani |
| 49–56 | Ensemble | string ensembles, synth strings, choirs, orchestra hit |
| 57–64 | Brass | trumpet, trombone, tuba, horn, section, synth brass |
| 65–72 | Reed | saxophones, oboe, English horn, bassoon, clarinet |
| 73–80 | Pipe | piccolo, flute, recorder, pan flute, bottle, shakuhachi, whistle, ocarina |
| 81–88 | Synth Lead | square, saw, calliope, charang, solo vox, and others |
| 89–96 | Synth Pad | Fantasia, Warm, Polysynth, Space Voice, Bowed Glass, Metal, Halo, Sweep |
| 97–104 | Synth Effects | Ice Rain, Soundtrack, Crystal, Atmosphere, Brightness, Goblin, Echo Drops, Star Theme |
| 105–112 | Ethnic | sitar, banjo, shamisen, koto, kalimba, bagpipe, fiddle, shanai |
| 113–120 | Percussive | tinkle bell, agogo, steel drums, woodblock, taiko, melodic tom, synth drum, reverse cymbal |
| 121–128 | Sound Effects | fret noise, breath, seashore, bird, telephone, helicopter, applause, gunshot |

These are the engine's own synthesized timbres, not SoundFont samples. They capture each instrument's character but sound different from a sampler or a hardware synthesizer.

### Automatic mapping

If the file contains a GM program number, that exact number (0–127) is used. If there is none, the parser tries to infer the instrument from its name (`bass`, `guitar`, `violin`, `choir`, `trumpet`, `flute`, `organ`, `pad`, `synth`, and similar) and picks a matching program. If nothing matches, piano is used.

### Percussion

Channel 10 is handled separately. Notes 35–87 use a GM percussion parameter table; kick, snare, toms, hi-hats, cymbals, clap, cowbell, congas, bongos, and the rest each have their own synthesis model: tonal and noise components, pitch sweeps, bursts of noise, and metallic partials. Percussion is not transposed; its sound is determined by the note number.

## Audio engine

The converter generates a self-contained JavaScript expression containing the engine, the parameters of **only the timbres actually used**, the percussion table, and the encoded note events. No external library or sound bank is needed.

The engine uses attack, decay, and release envelopes; harmonic and FM synthesis; detune, vibrato, and formants; noise and transient components; inharmonic partial sets for bells and mallets; smooth attenuation of high harmonics near the Nyquist limit; separate models for percussion categories; normalization by the number of active voices; and a soft limiter.

At most 14 melodic and 6 percussion voices sound at once; extra voices are smoothly attenuated and dropped. Pitch Bend is applied inside the note as a smooth pitch change.

## Preview and export

After conversion you can **Preview** (the PCM buffer is rendered in chunks through the Web Audio API, with progress), **Stop**, **Copy**, or **Download .js** (`bytebeat.js`).

### External player

The player must provide `t` (the sample index), run the code in a browser-style JavaScript context with a `window` object, and use the same sample rate and Bytebeat or Floatbeat mode. The expression returns one sample per call. It is not an audio file or a standalone player.

## How to use

1. Open the page and select a `.mid`, `.midi`, or `.json` file (or drag it in).
2. Review the tracks, disable unneeded ones, and change timbre and volume if you want.
3. Choose the sample rate, output format, and other settings.
4. Click **Convert**.
5. Preview the result or copy/download the expression.
6. Set the same sample rate and mode in the player.

## Limitations and notes

- **Bank Select is not read.** Bank variants (for example Ukulele, Church Organ 2, Wet Vibraphone) sound like the base program of their number.
- SMPTE time division is not supported; use PPQ-timed MIDI.
- Use `.mid` rather than JSON for the most accurate result.
- JSON may lack Program Change, CC7/CC11, and Pitch Bend; the converter cannot recover missing data.
- Automatic timbre selection is approximate; real samples are not used.
- Long songs and large note counts produce large JS code and long rendering. Disabling tracks and limiting the duration helps. A lower sample rate reduces the samples calculated per second but does not shorten the note list.
- Preview needs the Web Audio API; some browsers start audio only after a user interaction.
- Third-party players may have their own limits on expression size and complexity.

## Privacy and technologies

- All processing happens locally in the browser; files are never uploaded.
- The HTML uses no external CDN or network requests for conversion; only HTML, CSS, and JavaScript with no dependencies.
- Language and theme may be stored in `localStorage`; without storage the app still works, but preferences are not remembered.

To run locally, download `index.html` and open it in a modern browser.

## Authorship

**The entire project was implemented by Claude (Anthropic):** the interface and layout, MIDI and JSON parsers, tempo map, CC and Pitch Bend handling, 128 synthesized timbres, GM percussion, the audio engine, the expression generator, preview, file import, and language and theme switching. This README was prepared with Claude based on the source `index.html`.

## License

MIT. See [`LICENSE`](LICENSE).
