# JsonCraft - JSON Formatter

[English](./README.md) | [简体中文](./README.zh-CN.md)

A lightweight, polished online JSON formatter built with Vue 3 + CodeMirror 6, supporting real-time formatting, minification, validation, and multi-theme switching.

## Features

- **JSON Formatting** — Beautify JSON in one click, 2-space indentation, clear structure
- **JSON Minification** — Strip all whitespace to produce minified JSON
- **Real-time Validation** — Validate as you type, with precise error positioning
- **Multi-theme** — Built-in Default (light), MDN-like (light), and One Dark (dark) themes; preference auto-saved
- **Copy to Clipboard** — One-click copy of the formatted result
- **Download as File** — Export the formatted JSON as a `.json` file
- **Live Stats** — Real-time character count and processing time
- **Responsive Layout** — Adapts to both desktop and mobile
- **Local History** — Save and reload JSON snippets via IndexedDB

## Tech Stack

| Category | Technology |
|----------|------------|
| Framework | Vue 3.5 (`<script setup>`) |
| Build Tool | Vite 8 |
| Code Editor | CodeMirror 6 + vue-codemirror |
| Styling | CSS 3 (Scoped Styles) |
| Local Storage | IndexedDB (native API) |

## Getting Started

```bash
# Install dependencies
npm install

# Start the dev server
npm run dev

# Build for production
npm run build

# Preview the production build
npm run preview
```

## Project Structure

```
src/
├── main.js                  # App entry
├── App.vue                  # Root component
├── style.css                # Global styles
└── components/
    └── JsonFormatter.vue    # Core formatter component
```

## Preview

Dual-pane layout: raw JSON on the left, formatted result on the right in real time. The top toolbar provides formatting, minification, clearing, copying, and downloading, while the bottom shows error hints and stats.

## License

MIT
