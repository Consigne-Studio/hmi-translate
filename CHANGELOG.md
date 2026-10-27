# Changelog

## 1.0.0 — 2026-10-24

First release.

* Import and export: TIA Portal / WinCC project texts and HMI alarms (.xlsx), FactoryTalk View ME/SE language files (Unicode .txt), TwinCAT HMI .localization, TwinCAT 3 PLC / CODESYS localization (.pot/.po), Ignition Translation Manager (.xml/.properties), .xlsx/.csv tables with one column per language.
* Protection of alarm variables, field references, embedded tags, format items, markup and escaped line breaks.
* Engines: Ollama (local), OpenAI-compatible APIs, Anthropic Claude API, DeepL API, translation memory only. Automatic second pass for missing variables and texts too long.
* Checks: variables, field length (Max length column or rule), line count, tags, numbers, glossary, untranslated texts, final colon, safety texts, consistency.
* Built-in glossary of 300 industrial terms in 8 languages; company glossary (CSV) with priority.
* Translation memory with human/machine origin, TMX export.
* Interface in English and French; translation report (HTML) and list of issues (CSV).
* Command line for scripts and build pipelines, with `--fail-on` exit codes.
* Free trial edition: 500 distinct texts per project.
