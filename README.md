# ScopeTranspiler

**ScopeTranspiler** is a professional Caido plugin designed to bridge the gap between HackerOne program definitions and Caido's target scope configuration. It automates the conversion of complex Regex-based scopes into clean, native Glob patterns.

## Key Features

- **Regex-to-Glob Transpilation**: Automatically cleans and converts HackerOne's specific Regex format into Caido-compatible Glob patterns.
- **One-Click Preset Creation**: Seamlessly creates new Scope Presets directly in your Caido instance without manual copying.
- **Asset Summary**: Provides a real-time count of included and excluded hosts before importing.
- **Native Integration**: Features a UI that matches Caido's aesthetic, built with Tailwind CSS.
## 📦 Installation

1. Navigate to the **Releases** section of this repository.
2. Download the latest `.zip` (or `.caido`) package.
3. Open your **Caido** instance.
4. Go to **Plugins** and click on **Install Plugin**.
5. Upload the downloaded file.
6. The **ScopeTranspiler** icon will appear in your sidebar.
## Usage Guide

## Demo
![ScopeTranspiler Tutorial](media/Tutorial.gif)

### 1. Exporting from HackerOne

1. Go to the HackerOne program page you are hacking on.
2. Find the **Scope** section.
3. Look for the **Download Burp Suite Project Configuration File** or **Download** option (usually a JSON export button).
4. Download the `.json` file containing the program's scope.
### 2. Importing to Caido

1. Open **ScopeTranspiler** from the Caido sidebar.
2. Click **Choose H1 JSON** and select your exported file.
3. Review the **In Scope** and **Out of Scope** previews.
4. Define a **Preset Name** for your new scope.
5. Click **Create Caido Preset**.

##  Technical Details

The plugin implements a specialized cleaning logic to ensure 100% compatibility with Caido's scope engine:

- Removes Regex anchors (`^`, `$`).
- Escapes and sanitizes dots and wildcards (`\.` to `.`, `.*` to `*`).
- Deduplicates hosts automatically using `Set`.

## 🤝 Contributing

This plugin is developed using the **Caido SDK**.

- **Frontend**: Vue.js + Tailwind CSS.
- **Build Tool**: Vite + pnpm.

Bash

```
# Clone the repository
git clone https://github.com/flexinzop/Scope-Transpiler.git

# Install dependencies
pnpm install

# Run in development mode
pnpm dev
```

---

**Author**: fl3xinz
