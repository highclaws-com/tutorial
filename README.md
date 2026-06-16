# Tutorial Repository

Welcome to the **Tutorial** repository. This repository is designed to host onboarding tutorials and sandbox verification files for both human users and agentic AIs across multiple locales.

## Repository Structure

The tutorials are organized by language codes, containing localized directories for each category.

* **[translate.json](./translate.json)**: Contains mappings from language codes to their translated directory names (e.g. `"en": "Quick Start"`).
* **[en/](./en)**: English tutorials (e.g. `001-Quick-Start/`).
* **[fr/](./fr)**: French tutorials (e.g. `001-Démarrage-rapide/`).
* **[ja/](./ja)**: Japanese tutorials (e.g. `001-クイックスタート/`).
* **[zh/](./zh)**: Chinese tutorials (e.g. `001-快速上手/`).

---

## How to Add a New Tutorial

To add a new tutorial topic or category, follow these steps:

### 1. Identify or Create the Category Folder
Each category folder is prefixed with a three-digit sequence (e.g., `001-`).
* Check **[translate.json](./translate.json)** to find the translated name of the category for each language.
* If you are creating a **new category**, add the name translations for all supported language codes to **[translate.json](./translate.json)**, and create the directories in each language folder (e.g., `002-Get-to-know-sandbox` for English, `002-了解沙箱环境` for Chinese).

### 2. Add the Source Document
Create the tutorial markdown file under the appropriate language and category directory. Use a three-digit index prefix for the filename:
* Example: `zh/001-快速上手/002-了解沙箱环境.md`

### 3. Translate and Distribute
Translate the newly added document into all other supported locales (`en`, `fr`, `ja`):
* Translate the content while keeping the structure (like headings and formatting) identical.
* Translate the filename to match the target language (e.g., `002-Understand-the-Sandbox-Environment.md` for English).
* Place each translated file in its respective category directory (e.g. `en/001-Quick-Start/002-Understand-the-Sandbox-Environment.md`).
