# drawio-obsidian 开发指南

## 构建环境

- **Node 版本**：使用 Node 16（`nvm use 16`），不要用 Node 18+
- **构建命令**：`npm run build`
- **构建工具**：rollup v2 + rollup-plugin-typescript2（不用 @rollup/plugin-typescript，8.x 版本在非 watch 模式下有 bug，load hook 不工作）

## 构建教训

- `@rollup/plugin-typescript` 8.x 在首次构建（非 watch 模式）时，`load` hook 会直接返回 null，导致 rollup 把 `.ts` 文件当 JS 解析报错。改用 `rollup-plugin-typescript2`（有 `transform` hook）。
- `rollup-plugin-output-as-module` 的 `resolveId` 必须返回虚拟模块 ID（`\0bundle:xxx`），不能返回真实 `.ts` 路径，否则 rollup 会去分析原文件 export 并报错。

## 部署到 Obsidian

构建产物在 `dist/main.js`，需要手动复制到 Obsidian vault 的插件目录才能生效。

**步骤：**

1. 询问用户 vault 路径（例如 `/Users/xxx/Library/Mobile Documents/iCloud~md~obsidian/Documents/wzr-base`）
2. 询问用户是否需要备份原文件（建议备份）：
   ```bash
   cp "<vault>/.obsidian/plugins/drawio-obsidian/main.js" \
      "<vault>/.obsidian/plugins/drawio-obsidian/main.js.bak"
   ```
3. 复制新文件：
   ```bash
   cp dist/main.js "<vault>/.obsidian/plugins/drawio-obsidian/main.js"
   ```
4. 在 Obsidian 里执行 `Cmd+P` → "Reload app without saving"

## Plugin.ts 改动说明

`getSvg()` 方法使用 `graph.getSvg()` 原生导出管道，关键参数：
- `ignoreSelection` 必须为 `true`，否则只渲染选中元素，图形不完整
- 此方法支持 flowAnimation、shadow、sketch 等特性，原来的 `mxSvgCanvas2D` 方案不支持
