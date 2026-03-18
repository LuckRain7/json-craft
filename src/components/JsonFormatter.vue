<script setup>
import { ref, onMounted, watch, shallowRef } from 'vue'
import { EditorView, keymap, lineNumbers, highlightActiveLine, highlightActiveLineGutter } from '@codemirror/view'
import { EditorState } from '@codemirror/state'
import { json } from '@codemirror/lang-json'
import { oneDark } from '@codemirror/theme-one-dark'
import { defaultKeymap, history, historyKeymap } from '@codemirror/commands'
import { bracketMatching, foldGutter, foldKeymap } from '@codemirror/language'
import { searchKeymap, highlightSelectionMatches } from '@codemirror/search'
import { closeBrackets, closeBracketsKeymap } from '@codemirror/autocomplete'

const inputContainer = ref(null)
const outputContainer = ref(null)

let inputView = shallowRef(null)
let outputView = shallowRef(null)

const errorMsg = ref('')
const showError = ref(false)
const jsonSize = ref('大小: 0 字符')
const jsonTime = ref('处理时间: 0 ms')
const currentTheme = ref('default')

const commonEditorStyle = {
  '&': {
    height: '100%',
    fontSize: '14px',
    borderRadius: '4px',
  },
  '.cm-scroller': {
    fontFamily: "'Consolas', 'Monaco', 'Courier New', monospace",
    overflow: 'auto',
  },
  '.cm-content': {
    minHeight: '100%',
  },
}

const baseTheme = EditorView.theme({
  ...commonEditorStyle,
  '&': { ...commonEditorStyle['&'], border: '1px solid #e0e0e0' },
})

const darkEditorTheme = EditorView.theme({
  ...commonEditorStyle,
  '&': { ...commonEditorStyle['&'], border: '1px solid #444' },
})

import { HighlightStyle, syntaxHighlighting } from '@codemirror/language'
import { tags } from '@lezer/highlight'

const mdnLikeTheme = EditorView.theme({
  '&': {
    height: '100%',
    fontSize: '14px',
    border: '1px solid #e0e0e0',
    borderRadius: '4px',
    color: '#07a',
    backgroundColor: '#fff',
  },
  '.cm-scroller': {
    fontFamily: "'Consolas', 'Monaco', 'Courier New', monospace",
    overflow: 'auto',
  },
  '.cm-content': {
    minHeight: '100%',
    caretColor: '#222',
  },
  '&.cm-focused .cm-cursor': {
    borderLeftColor: '#222',
    borderLeftWidth: '2px',
  },
  '.cm-gutters': {
    backgroundColor: '#f8f8f8',
    borderLeft: '6px solid rgba(0, 83, 159, 0.65)',
    color: '#aaa',
    border: 'none',
  },
  '.cm-activeLineGutter': {
    backgroundColor: '#efefff',
  },
  '&.cm-focused .cm-selectionBackground, .cm-selectionBackground': {
    backgroundColor: '#cfc',
  },
  '.cm-activeLine': {
    backgroundColor: '#efefff',
  },
  '&.cm-focused .cm-matchingBracket': {
    outline: '1px solid grey',
    color: 'inherit',
  },
})

const mdnLikeHighlight = HighlightStyle.define([
  { tag: tags.keyword, color: '#6262ff' },
  { tag: tags.atom, color: '#f90' },
  { tag: tags.number, color: '#ca7841' },
  { tag: tags.definition(tags.variableName), color: '#8da6ce' },
  { tag: tags.variableName, color: '#07a' },
  { tag: tags.propertyName, color: '#905' },
  { tag: tags.operator, color: '#cda869' },
  { tag: tags.comment, color: '#777' },
  { tag: tags.string, color: '#07a', fontStyle: 'italic' },
  { tag: tags.meta, color: '#000' },
  { tag: tags.tagName, color: '#997643' },
  { tag: tags.attributeName, color: '#d6bb6d' },
  { tag: tags.heading, color: '#ff6400' },
  { tag: tags.link, color: '#ad9361', fontStyle: 'italic' },
  { tag: tags.bool, color: '#f90' },
  { tag: tags.null, color: '#f90' },
])

const themeIsDark = (theme) => theme === 'one-dark'

function createExtensions(readOnly = false, theme = 'default') {
  const exts = [
    lineNumbers(),
    highlightActiveLine(),
    highlightActiveLineGutter(),
    bracketMatching(),
    foldGutter(),
    closeBrackets(),
    highlightSelectionMatches(),
    history(),
    json(),
    keymap.of([
      ...defaultKeymap,
      ...historyKeymap,
      ...foldKeymap,
      ...searchKeymap,
      ...closeBracketsKeymap,
    ]),
    EditorView.lineWrapping,
  ]

  if (theme === 'one-dark') {
    exts.push(oneDark, darkEditorTheme)
  } else if (theme === 'mdn-like') {
    exts.push(mdnLikeTheme, syntaxHighlighting(mdnLikeHighlight))
  } else {
    exts.push(baseTheme)
  }

  if (readOnly) {
    exts.push(EditorState.readOnly.of(true))
  }

  return exts
}

let autoFormatTimer = null

function initEditors() {
  inputView.value = new EditorView({
    state: EditorState.create({
      doc: '',
      extensions: [
        ...createExtensions(false, currentTheme.value),
        EditorView.updateListener.of((update) => {
          if (update.docChanged) {
            updateStats(update.state.doc.toString())
            clearTimeout(autoFormatTimer)
            autoFormatTimer = setTimeout(formatJson, 500)
          }
        }),
      ],
    }),
    parent: inputContainer.value,
  })

  outputView.value = new EditorView({
    state: EditorState.create({
      doc: '',
      extensions: createExtensions(true, currentTheme.value),
    }),
    parent: outputContainer.value,
  })
}

function rebuildEditors() {
  const inputDoc = inputView.value ? inputView.value.state.doc.toString() : ''
  const outputDoc = outputView.value ? outputView.value.state.doc.toString() : ''

  if (inputView.value) inputView.value.destroy()
  if (outputView.value) outputView.value.destroy()

  inputView.value = new EditorView({
    state: EditorState.create({
      doc: inputDoc,
      extensions: [
        ...createExtensions(false, currentTheme.value),
        EditorView.updateListener.of((update) => {
          if (update.docChanged) {
            updateStats(update.state.doc.toString())
            clearTimeout(autoFormatTimer)
            autoFormatTimer = setTimeout(formatJson, 500)
          }
        }),
      ],
    }),
    parent: inputContainer.value,
  })

  outputView.value = new EditorView({
    state: EditorState.create({
      doc: outputDoc,
      extensions: createExtensions(true, currentTheme.value),
    }),
    parent: outputContainer.value,
  })
}

watch(currentTheme, () => {
  localStorage.setItem('jsonFormatterTheme', currentTheme.value)
  rebuildEditors()
})

onMounted(() => {
  currentTheme.value = localStorage.getItem('jsonFormatterTheme') || 'default'
  initEditors()
})

function updateStats(content) {
  jsonSize.value = `大小: ${content.length} 字符`
}

function updateTime(startTime) {
  const time = (performance.now() - startTime).toFixed(2)
  jsonTime.value = `处理时间: ${time} ms`
}

function removeSurroundingQuotes(content) {
  if (
    (content.startsWith('"') && content.endsWith('"')) ||
    (content.startsWith("'") && content.endsWith("'"))
  ) {
    return content.substring(1, content.length - 1)
  }
  return content
}

function setOutput(text) {
  outputView.value.dispatch({
    changes: {
      from: 0,
      to: outputView.value.state.doc.length,
      insert: text,
    },
  })
}

function formatJson() {
  const startTime = performance.now()
  let content = inputView.value.state.doc.toString().trim()

  if (!content) {
    errorMsg.value = '请输入JSON内容'
    showError.value = true
    return
  }

  content = removeSurroundingQuotes(content)

  try {
    const parsed = JSON.parse(content)
    const formatted = JSON.stringify(parsed, null, 2)
    setOutput(formatted)
    showError.value = false
    errorMsg.value = ''
    updateStats(formatted)
    updateTime(startTime)
  } catch (e) {
    errorMsg.value = `JSON格式错误: ${e.message}`
    showError.value = true
    updateTime(startTime)
  }
}

function compressJson() {
  const startTime = performance.now()
  let content = inputView.value.state.doc.toString().trim()

  if (!content) {
    errorMsg.value = '请输入JSON内容'
    showError.value = true
    return
  }

  content = removeSurroundingQuotes(content)

  try {
    const parsed = JSON.parse(content)
    const compressed = JSON.stringify(parsed)
    setOutput(compressed)
    showError.value = false
    errorMsg.value = ''
    updateStats(compressed)
    updateTime(startTime)
  } catch (e) {
    errorMsg.value = `JSON格式错误: ${e.message}`
    showError.value = true
    updateTime(startTime)
  }
}

function clearEditor() {
  inputView.value.dispatch({
    changes: { from: 0, to: inputView.value.state.doc.length, insert: '' },
  })
  setOutput('')
  showError.value = false
  errorMsg.value = ''
  jsonSize.value = '大小: 0 字符'
  jsonTime.value = '处理时间: 0 ms'
}

async function copyResult() {
  const result = outputView.value.state.doc.toString()
  if (!result) {
    errorMsg.value = '没有可复制的内容'
    showError.value = true
    return
  }

  try {
    await navigator.clipboard.writeText(result)
    alert('复制成功')
  } catch (err) {
    errorMsg.value = '复制失败: ' + err.message
    showError.value = true
  }
}

function downloadJson() {
  const result = outputView.value.state.doc.toString()
  if (!result) {
    errorMsg.value = '没有可下载的内容'
    showError.value = true
    return
  }

  const blob = new Blob([result], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'formatted.json'
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
}
</script>

<template>
  <div class="json-tool-container" :class="{ dark: themeIsDark(currentTheme) }">
    <div class="json-tool-header">
      <h1>JSON格式化工具</h1>
      <div class="json-tool-buttons">
        <button class="btn" @click="formatJson">格式化</button>
        <button class="btn" @click="compressJson">压缩</button>
        <button class="btn" @click="clearEditor">清空</button>
        <button class="btn" @click="copyResult">复制</button>
        <button class="btn" @click="downloadJson">下载</button>
        <div class="theme-selector">
          <label>主题:</label>
          <select v-model="currentTheme">
            <optgroup label="亮色主题">
              <option value="default">默认</option>
              <option value="mdn-like">MDN Like</option>
            </optgroup>
            <optgroup label="暗色主题">
              <option value="one-dark">One Dark</option>
            </optgroup>
          </select>
        </div>
      </div>
    </div>
    <div class="json-tool-content">
      <div class="json-editor-container" :class="{ dark: themeIsDark(currentTheme) }">
        <div ref="inputContainer" class="editor-wrapper"></div>
      </div>
      <div class="json-result-container" :class="{ dark: themeIsDark(currentTheme) }">
        <div ref="outputContainer" class="editor-wrapper"></div>
      </div>
    </div>
    <div class="json-tool-footer" :class="{ dark: themeIsDark(currentTheme) }">
      <div v-show="showError" class="json-error">{{ errorMsg }}</div>
      <div class="json-stats">
        <span>{{ jsonSize }}</span>
        <span>{{ jsonTime }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.json-tool-container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.json-tool-header {
  background-color: #07a;
  color: white;
  padding: 12px 16px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.json-tool-header h1 {
  font-size: 16px;
  font-weight: 600;
}

.json-tool-buttons {
  display: flex;
  gap: 8px;
  align-items: center;
  flex-wrap: wrap;
}

.theme-selector {
  display: flex;
  align-items: center;
  gap: 5px;
}

.theme-selector label {
  font-size: 13px;
  color: white;
}

.theme-selector select {
  padding: 4px 8px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 4px;
  font-size: 13px;
  background-color: rgba(255, 255, 255, 0.2);
  color: white;
  cursor: pointer;
}

.theme-selector select:hover {
  border-color: rgba(255, 255, 255, 0.5);
}

.theme-selector select:focus {
  outline: none;
  border-color: white;
  box-shadow: 0 0 0 2px rgba(255, 255, 255, 0.3);
}

.btn {
  background-color: rgba(255, 255, 255, 0.2);
  border: none;
  color: white;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 13px;
  transition: background-color 0.3s;
}

.btn:hover {
  background-color: rgba(255, 255, 255, 0.3);
}

.json-tool-content {
  flex: 1;
  display: flex;
  overflow: hidden;
}

.json-editor-container,
.json-result-container {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  padding: 10px;
  background-color: white;
  margin: 10px;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.json-editor-container.dark,
.json-result-container.dark {
  background-color: #1e1e1e;
}

.editor-wrapper {
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.editor-wrapper :deep(.cm-editor) {
  flex: 1;
}

.json-tool-footer {
  background-color: white;
  padding: 10px 20px;
  box-shadow: 0 -2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.json-tool-footer.dark {
  background-color: #1e1e1e;
  color: #ccc;
}

.json-error {
  color: #d32f2f;
  background-color: #ffebee;
  padding: 8px 12px;
  border-radius: 4px;
  font-size: 14px;
  border-left: 4px solid #d32f2f;
  flex: 1;
  margin-right: 20px;
}

.json-stats {
  display: flex;
  gap: 20px;
  font-size: 14px;
  color: #666;
}

.json-tool-footer.dark .json-stats {
  color: #aaa;
}

.json-tool-container.dark {
  background-color: #121212;
}

.json-tool-container.dark .json-tool-header {
  background-color: #1a1a2e;
}

@media (max-width: 768px) {
  .json-tool-content {
    flex-direction: column;
  }

  .json-editor-container,
  .json-result-container {
    margin: 5px 10px;
  }

  .json-tool-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }

  .json-tool-buttons {
    flex-wrap: wrap;
  }

  .json-tool-footer {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }

  .json-error {
    margin-right: 0;
    width: 100%;
  }
}
</style>
