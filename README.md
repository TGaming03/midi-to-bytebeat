# MIDI → Bytebeat Studio

🇬🇧 English · [🇷🇺 Русский](#русский)

Convert a MIDI file into a **single JavaScript expression** for any bytebeat / floatbeat player (e.g. dollchan bytebeat composer). Runs 100% in the browser, one static `index.html`, no dependencies, nothing is uploaded.

**Features**
- Reads **.mid / .midi** directly, plus `@tonejs/midi` JSON and VisiPiano JSON
- Exact **tempo map** (accelerando / ritardando), **instrument changes** at any moment (Program Change), CC7/CC11 volume, **pitch bend** with real RPN range
- 16 synthesized timbres chosen by GM program (piano, FM e-piano, organ, guitars, bass, strings, choir, brass, reeds, mallets, leads, pads…) and a **full GM drum kit** (notes 35–87), each sound different
- Per-track on/off, timbre override and volume; transpose, speed, master gain, max length
- Sample rate 8–44.1 kHz, Bytebeat (0–255) or Floatbeat (−1…1) output
- In-browser preview, copy, download `.js`; drag & drop; RU/EN UI; light/dark theme

**Usage:** open `index.html` → drop a file → *Convert* → paste the code into the player and set the **same sample rate and mode**.

**GitHub Pages:** push the files → *Settings → Pages → Deploy from branch → `main` / root*.

**Notes:** per the MIDI standard, tempo is 120 BPM until the first tempo event (toggle available). Long songs make long code; use *Max seconds*, a lower sample rate, or mute tracks if your player struggles.

## Русский

Конвертер MIDI в **одно JS-выражение** для любого bytebeat/floatbeat-плеера. Работает целиком в браузере: один файл `index.html`, без зависимостей, файлы никуда не отправляются.

**Возможности:** чтение `.mid` напрямую (а также JSON `@tonejs/midi` и VisiPiano); точная карта темпа, смена инструментов в любой момент, громкость CC7/CC11, pitch bend; 16 тембров по GM-программам и полный набор барабанов GM (35–87); настройки по дорожкам, транспонирование, скорость; частота 8–44,1 кГц; Bytebeat или Floatbeat; прослушивание, копирование, скачивание; интерфейс RU/EN, светлая и тёмная темы.

**Как пользоваться:** открой `index.html` → перетащи файл → «Конвертировать» → вставь код в плеер и выставь **ту же частоту и режим**.

**GitHub Pages:** залей файлы → *Settings → Pages → Deploy from branch → `main` / root*.
