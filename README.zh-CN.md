# JsonCraft - JSON 格式化工具

[English](./README.md) | [简体中文](./README.zh-CN.md)

一款轻量、美观的在线 JSON 格式化工具，基于 Vue 3 + CodeMirror 6 构建，支持实时格式化、压缩、校验与多主题切换。

## 功能特性

- **JSON 格式化** — 一键美化 JSON，2 空格缩进，结构清晰
- **JSON 压缩** — 去除所有空白字符，生成最小化 JSON
- **实时校验** — 输入即校验，错误信息精准定位
- **多主题切换** — 内置 Default（浅色）、MDN-like（浅色）、One Dark（深色）三套主题，偏好自动保存
- **复制到剪贴板** — 一键复制格式化结果
- **下载为文件** — 将格式化后的 JSON 导出为 `.json` 文件
- **统计信息** — 实时显示 JSON 字符数与处理耗时
- **响应式布局** — 适配桌面端与移动端
- **本地历史记录** — 通过 IndexedDB 保存与加载常用 JSON 片段

## 技术栈

| 类别 | 技术 |
|------|------|
| 前端框架 | Vue 3.5 (`<script setup>`) |
| 构建工具 | Vite 8 |
| 代码编辑器 | CodeMirror 6 + vue-codemirror |
| 样式 | CSS 3（Scoped Styles） |
| 本地存储 | IndexedDB（原生 API） |

## 快速开始

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览生产构建
npm run preview
```

## 项目结构

```
src/
├── main.js                  # 应用入口
├── App.vue                  # 根组件
├── style.css                # 全局样式
└── components/
    └── JsonFormatter.vue    # 核心格式化组件
```

## 预览

双栏布局：左侧输入原始 JSON，右侧实时展示格式化结果。顶部工具栏提供格式化、压缩、清空、复制、下载等操作，底部显示错误提示与统计信息。

## License

MIT
