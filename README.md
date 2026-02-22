# mdTranslategemma

**mdTranslategemma** is a completely vibecoded browser-based utility designed to translate large batches of Markdown (`.md`) files using a local **Ollama** instance running a [translategemma]([https://ollama.com/](https://blog.google/innovation-and-ai/technology/developers-tools/translategemma/) model. While optimized for the `translategemma` model, it should be compatible with various LLMs that follow standard translation prompt patterns. The tool runs entirely in your browser using the **File System Access API**, keeping your data in local environment.

## The Pipeline
### 1. File Scanning & Memory Loading
* The user selects an **Input Folder** and an **Output Folder**.
* The script recursively scans the input directory for all `.md` files.
* To prevent "stale file handles" (a common browser security issue where access is lost during long tasks), the script reads the text content of all files into memory before starting.

### 2. Markdown Chunking
To stay within the specified `Context Length`, the tool splits documents into manageable pieces.
* **Context Window:** It calculates the maximum chunk size based on `(Context Length / 2) - Margin`. This leaves enough "room" in the model's memory for the system instructions and the generated translation.
* **Split Logic:** Instead of cutting text at a random character count, it splits by double newlines (`\n\n`). This ensures that paragraphs, list items, and tables are sent as whole units whenever possible.

### 3. Translation with Sliding Context
The tool uses a "sliding window" approach to help the LLM understand the flow of the document:
* **Prompting:** It uses the prompt template expected by `translategemma`, including the source/target language codes (e.g., `fi`, `en`).
* **Context Overlap:** For every chunk after the first, the script prepends a "Previous Context" snippet (the end of the previous chunk). This allows the model to see how the previous sentence ended so it can maintain consistent terminology and tone.
* **Markdown Formatting:** The model is prompted to respect the Markdown formatting and not to break any links etc.

### 4. Recursive Output & Resumption
* **Directory Mirroring:** The tool recreates the exact sub-folder structure of the input directory inside the output directory. Great e.g. for translating Obsidian vaults.
* **Naming Convention:** Files are saved with an ISO suffix (e.g., `manual.md` → `manual_en.md`).
* **Retry Logic:** If the Ollama API fails (or the user cancels), the tool tracks the current file index. Users can click **"Resume / Retry"** to pick up exactly where they left off without re-translating completed files.

## Key Features
* **Live Status Indicator:** A dynamic status symbol (`⚪ Idle`, `🟡 Translating`, `🔴 Error`) and progress bar provide real-time feedback on the API's activity.
* **Optional Model Tuning:**
* **Temperature:** Control the "creativity" or determinism of the translation.
* **Repetition Penalty:** Prevent the model from getting stuck in loops or repeating phrases.
* **Context Overlap:** Adjustable token overlap to ensure seamless transitions between translated chunks.

## Prerequisites
1. **Ollama Installed:** Ensure [Ollama](https://ollama.com/) is running on your machine.
2. **CORS Enabled:** Since this is a browser-based tool, Ollama must be configured to allow "Cross-Origin Resource Sharing."
3. **The Model:** Download the wanted model via terminal, e.g.:
```bash
ollama pull translategemma:12b
```

## Usage
1. Open the `mdtranslategemma.html` file in a modern browser (Chrome or Edge recommended for File System API support).
2. Enter your **Ollama URL** and **Model Name**.
3. Set your **Source** and **Target** languages.
4. Select your **Input Folder** (where your original `.md` files are).
5. Select your **Output Folder** (where the translations will be saved).
6. Click **Start Translation**.
