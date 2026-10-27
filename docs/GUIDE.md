# hmi-translate — user guide

*Version française : [GUIDE.fr.md](GUIDE.fr.md).*

1. [The round trip, tool by tool](#1-the-round-trip-tool-by-tool)
2. [Translation engines](#2-translation-engines)
3. [Field lengths](#3-field-lengths)
4. [Glossary](#4-glossary)
5. [Translation memory](#5-translation-memory)
6. [Review: what each check means](#6-review-what-each-check-means)
7. [Command line](#7-command-line)
8. [Troubleshooting](#8-troubleshooting)
9. [FAQ](#9-faq)

The interface has five steps: **Files**, **Languages and engine**, **Translate**, **Review**, **Export**. You can go back to any step at any time; the project is saved automatically and reopened at the next start.

## 1. The round trip, tool by tool

General rule: **export from your tool, translate here, import the result into a copy of your project first**, then open a few screens and alarms in each language. Menu names can differ slightly between versions of each tool.

| Tool | What to export | What hmi-translate produces | What to import |
|---|---|---|---|
| TIA Portal (STEP 7, WinCC Comfort/Advanced/Unified) — project texts | Tools › Export project texts (also in Languages & resources › Project texts), with the source language and the target languages ticked, as .xlsx | `<name>_translated.xlsx`: the same workbook, with the target columns filled (new columns are added in the same style if they were missing) | Tools › Import project texts, with that file |
| TIA Portal — HMI alarms | HMI alarms editor › Export, as .xlsx | `<name>_translated.xlsx` | HMI alarms editor › Import |
| FactoryTalk View ME / SE | Tools › Languages › Export, "Unicode text file", one file per language | `<application>_fr_FR.txt` (one file per target language, same encoding and layout) | Tools › Languages: add the language to the application, then Import with that file |
| TwinCAT HMI | The files of the `Localization` folder of the HMI project (`en.localization`, …) | `fr.localization`, `de.localization`… (same key order and indentation) | Copy the files into the same `Localization` folder and add them to the project |
| TwinCAT 3 PLC / CODESYS | Project › Localization › Create localization template (.pot) | `<name>.fr_FR.po`, … | Project › Localization › Manage localization: add the .po files |
| Ignition (Translation Manager) | Designer › Tools › Translation Manager › Export, format XML (UTF-8) or PROPERTIES | `<name>_fr.xml` (or `.properties`) per target language | Translation Manager › Import |
| Any other tool (AVEVA, B&R, WinCC Unified text lists, in-house tools…) | A .xlsx or .csv table with one column per language, headed by codes such as `en-US`, `de-DE`, `fr-FR` | `<name>_translated.xlsx` / `.csv` | Your tool's import |

Notes:

* **Several files at once**: import all the exports of a machine together (for example project texts + HMI alarms + the PLC .pot). A text that appears in several files is translated once and identically everywhere.
* **Existing translations** in the files are kept and checked; tick "Replace translations already present in the files" in step 2 to translate them again.
* **Source language** is detected from the files (column headers, file names, locale fields); change it in step 2 if needed.
* **Context**: the alarm class, the trigger tag, the key or the comment of each text are sent with it to the engine, and shown in the review.
* **Language codes**: hmi-translate writes each language in the style the file already uses (`fr-FR`, `fr_FR` or `fr`).

## 2. Translation engines

Choose the engine in step 2, then click **Test the connection**: it makes a real call to the engine (the list of models for Ollama and OpenAI-compatible servers, the model for Anthropic, the usage of your key for DeepL), so a wrong address, key or model name shows up before the translation.

The texts are sent in batches. If a batch fails (no answer, an answer the engine refuses to give, a server error), its texts are marked **No answer from the engine** with the reason, and the translation goes on with the next batch; translate again later to fill them. A wrong key, address or model, an engine unreachable from the start, or three failed batches in a row stop the translation with a clear message.

### Ollama (local)

Nothing leaves your computer. Install [Ollama](https://ollama.com), then download a model once:

| Model | Download | Comfortable on | Result |
|---|---|---|---|
| `qwen2.5:14b` (default) | about 9 GB | GPU with 12 GB or more | good for European languages and Chinese |
| `qwen2.5:7b` | about 5 GB | GPU with 8 GB, or CPU | usable, more texts to review |
| `qwen2.5:32b` | about 20 GB | GPU with 24 GB | best of the three, slower |

```
ollama pull qwen2.5:14b
```

Any other chat model installed in Ollama can be typed in the **Model** field. If Ollama runs on another computer of your network, put its address in **Address** (for example `http://192.168.1.20:11434`) and start Ollama there with `OLLAMA_HOST=0.0.0.0`.

Speed depends mostly on the graphics card: a GPU that holds the whole model is many times faster than the CPU. Try the sample project or a small export first to see the speed on your computer. The translation runs in the background, shows its progress and can be stopped; what is done is kept.

### OpenAI-compatible APIs

OpenAI, Azure OpenAI, Mistral, LM Studio, vLLM and any server that offers `/chat/completions`. Set **Address** (for example `https://api.mistral.ai/v1` or `http://localhost:1234/v1`), **Model** (a model your account offers) and the **API key**.

### Anthropic Claude API, DeepL API

Set the API key. For DeepL, free keys (ending in `:fx`) use the free endpoint automatically.

DeepL translates the texts with their variables protected, but it does not take instructions: the glossary and the field lengths are not sent to it. The checks still run after the translation, so terms and lengths to fix are flagged in the review as with the other engines.

### Keys

A key can be read from the environment variables `OPENAI_API_KEY`, `ANTHROPIC_API_KEY` or `DEEPL_API_KEY` (the interface shows "Key found in the environment variable"), or typed in the interface. A typed key is kept in memory only, unless you tick "Remember the key on this computer": it is then stored in `settings.json` of your workspace folder (readable by your user only on Linux/macOS). Keys are never written in projects, reports or logs.

With a cloud engine, the texts and their context are sent to the provider, under your own contract with it. Use Ollama for confidential projects.

### Memory only

Uses only the translations of your translation memory: useful for a new version of a machine already translated, or to check what the memory covers.

## 3. Field lengths

For each text, the limit comes from, in this order:

1. a **Max length** column in the export (TIA Portal HMI alarms and many tables have one: `Max length`, `Maximum length`, `MaxLength`, `Longueur max`, `Max. Länge`…);
2. otherwise the rule chosen in step 2: **source length × factor** (default 1.3), **not longer than the source**, **fixed number of characters**, or **no limit**.

Lengths are counted the way an operator sees them: a variable counts as 4 characters, a Chinese, Japanese or Korean character as 2, and for a text on several lines the longest line counts. When a translation is too long, the engine is asked once more for a shorter version (standard HMI abbreviations allowed); if it is still too long, the text is flagged in the review.

## 4. Glossary

The built-in glossary has 300 terms (alarms, states, operating modes, process equipment, safety, units of work) in English, French, German, Spanish, Italian, Portuguese, Polish and Chinese. Download it from step 2 (**Download the built-in glossary**) to see it.

Your company glossary is a CSV file (comma or semicolon, UTF-8) with one column per language, headed by the language codes. Optional columns: `id`, `domain`, `note`.

```
en,fr,de,note
Reject conveyor,Convoyeur de rejet,Ausschleusband,line FL-200
Fault reset,Acquit défaut,Störungsquittierung,
```

Import it in step 2. It takes priority over the built-in glossary for the same term. Terms of several words and every term of your glossary are mandatory: a translation that does not use them is flagged. Single words of the built-in glossary are suggestions given to the engine.

## 5. Translation memory

Every translation you export goes into the memory, with its origin: **human** (edited or approved by you) or **machine**. A human translation is never replaced by a machine one. The memory is used first at the next translation: the engine is called only for the texts it does not know.

Export it as TMX (step 5) to share it with a translation agency or another CAT tool. The memory file is `memory.json` in the workspace folder: copy it to give your memory to a colleague.

## 6. Review: what each check means

| Check | Level | Meaning |
|---|---|---|
| Variable missing / Unexpected variable | error | A variable, tag reference or markup of the source is missing in the translation, or one was added. The import would break the alarm. |
| Too long for the field | error (hard limit from the export) or warning (your rule) | The longest line is longer than the limit. |
| Line count differs | warning | The source has line breaks the translation does not have, or the reverse. |
| Tag or identifier missing | error | An equipment tag such as `P-101` or `LT101` is missing. |
| Number missing | error | A number of the source (setpoint, limit, address) is not in the translation; only the decimal separator may change. |
| Glossary term not used | warning | A mandatory glossary term is not translated as the glossary says. |
| Same as source | warning | A text of several words came back untranslated. |
| Final colon differs | warning | The source ends with a colon and the translation does not, or the reverse. |
| Safety text: human review required | review | Emergency stop, danger, warning, protective device, hot surface, crushing… Must be reviewed by a person fluent in the language. The words are recognised in the source text in English, French, German, Spanish, Italian, Portuguese, Dutch, Swedish, Polish, Czech, Turkish, Russian, Chinese, Japanese and Korean. |
| Translated differently elsewhere | warning | The same source text has different translations in the project. |

The filters show **all** texts, those **to check**, **errors** only, **safety** texts, texts **too long**, texts **not approved**. Edit a translation in place: the checks run again when you leave the field. **Approve** marks a translation as reviewed by a person; **Approve the visible texts (except errors and safety texts)** approves the rest of the page shown (up to 100 rows of the current filter) in one click; safety texts are approved one by one, after reading.

Export is possible with remaining issues: step 5 says how many texts still have an error, and the report lists them, so that nothing is hidden.

## 7. Command line

```
hmi-translate [serve] [--port 4870] [--no-open]
hmi-translate translate <files…> --to fr-FR,de-DE [--from en-US] [--engine ollama|openai|anthropic|deepl|memory]
                        [--model …] [--base-url …] [--length ratio:1.3|source|fixed:40|none]
                        [--overwrite] [--out translated] [--fail-on error|review|warning]
hmi-translate check <files…> --to fr-FR [--from en-US] [--out dir] [--fail-on …]
hmi-translate glossary --export <file.csv> | --import <file.csv> | --clear
hmi-translate memory --export <file.tmx>
```

`translate` writes the translated files, `translation-report.html` and `translation-issues.csv` in the output folder. `check` checks the translations already present in the files (useful in a build pipeline, after a translation agency). Exit codes: 0 done, 1 error (file, engine), 2 issues of the `--fail-on` level remain.

## 8. Troubleshooting

| Symptom | Fix |
|---|---|
| "Ollama is not reachable" | Start Ollama (icon in the notification area on Windows, or `ollama serve`). Check the address in step 2. |
| "Model … is not installed in Ollama" | Run the `ollama pull` command shown in the message, or choose one of the installed models it lists. |
| 401 / 403 with a cloud engine | The API key is wrong or has no access to that model. |
| 404 with a cloud engine | The address or the model name is wrong: use a model your account offers. |
| 429 | The provider limits your rate: hmi-translate makes up to 3 attempts per batch, waiting longer each time; the texts still without an answer are marked, translate again later for them (what is done is kept). |
| "No answer from the engine" on a few texts | The reason is shown with each text (server error, refusal). Click **Translate** again: only the missing texts are sent. |
| "Format not recognised" | Check the extension and that the file comes from the export function of your tool (not a copy pasted into another workbook). |
| .xls file refused | Save it as .xlsx in Excel, or export again choosing .xlsx. |
| Accents wrong after import in FactoryTalk View | Use the "Unicode text file" export; hmi-translate keeps the encoding of the source file. |
| The browser does not open | Open `http://127.0.0.1:4870` by hand. If the port is busy, hmi-translate takes the next free one and prints it in the window. |
| The texts of the trial stop at 500 | The trial translates 500 distinct texts per project; the others are flagged "Beyond the 500 texts of the trial". |

## 9. FAQ

**Is my project sent anywhere?** Not with Ollama or memory only: everything stays on your computer. The interface is served on `127.0.0.1` only and refuses requests from other sites. With a cloud engine, the texts are sent to that provider only.

**Does it change anything else in my files?** No. In .xlsx files, only the translated cells are rewritten; formatting, other sheets and columns stay as they were. Text formats keep their encoding, line endings, order and comments.

**Which languages?** Any language the engine knows. The interface proposes 37 common ones and accepts any code (`ro-RO`, `sv-SE`, `ar-SA`…). The built-in glossary covers 8 of them.

**Can a translation agency review the result?** Yes: give them `translation-issues.csv` or the translated .xlsx, then run `hmi-translate check` on what they send back. The TMX export carries your memory to their tools.

**Several engineers?** Each one installs hmi-translate on their computer. Share the company glossary CSV and the memory file, or point `HMI_TRANSLATE_HOME` to a shared folder used by one person at a time.

**Is it certified for safety texts?** No. It flags them for review, and they can only be approved one by one, by a person.

**How is the licence checked?** It is not: there is no activation and nothing is sent to us. The full edition is downloaded after purchase and is used under the terms of [LICENSE.md](../LICENSE.md) (one named person, or up to 5 for the Machine builder licence, for 12 months).
