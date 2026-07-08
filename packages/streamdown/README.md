```markdown
# Streamdown

`react-markdown` 的即插即用替代品，专为 AI 驱动的流式输出设计。

[![npm 版本](https://img.shields.io/npm/v/streamdown)](https://www.npmjs.com/package/streamdown)

## 概述

格式化 Markdown 很简单，但在对它进行分词和流式传输时，会出现新的挑战。Streamdown 专为处理 AI 模型流式传输 Markdown 内容的独特需求而构建，即使面对不完整或未终止的 Markdown 块，也能提供流畅的格式化。

Streamdown 为 [AI Elements Message](https://ai-sdk.dev/elements/components/message) 组件提供支持，但也可以作为独立包安装，用于您自己的流式传输需求。

## 特性

- 🚀 **即插即用替代品** - 可直接替换 `react-markdown`
- 🔄 **流式优化** - 优雅地处理不完整的 Markdown
- 🎨 **未终止块解析** - 结合 `remend` 构建，提供更好的流式质量
- 📊 **GitHub Flavored Markdown** - 支持表格、任务列表和删除线
- 🔢 **数学公式渲染** - 通过 KaTeX 渲染 LaTeX 公式
- 📈 **Mermaid 图表** - 将 Mermaid 图表作为代码块渲染，并提供渲染按钮
- 🎯 **代码语法高亮** - 使用 Shiki 提供美观的代码块
- 🛡️ **安全优先** - 使用 `rehype-harden` 构建，实现安全渲染
- ⚡ **性能优化** - 采用 Memoized 渲染，实现高效更新

## 安装

```bash
npm i streamdown
```

然后，更新您的 Tailwind `globals.css` 文件，加入以下内容，以便 Tailwind 能够检测到 Streamdown 使用的工具类。

```css
@source "../node_modules/streamdown/dist/*.js";
```

路径必须是相对于您的 CSS 文件到包含 `streamdown` 的 `node_modules` 文件夹的相对路径。在标准的 Next.js 项目中（`globals.css` 位于 `app/` 目录），上面的默认路径应该可以正常工作。

如果您安装了 Streamdown 的可选插件，请从相关插件文档（`/docs/plugins/code`、`/docs/plugins/cjk`、`/docs/plugins/math`、`/docs/plugins/mermaid`）中添加对应的 `@source` 行。只为实际安装的包添加插件 `@source` 条目。

示例：

```css
@source "../node_modules/@streamdown/code/dist/*.js";
```

### Monorepo 设置

在 monorepo（npm workspaces、Turbo、pnpm 等）中，依赖通常会被提升（hoisted）到根目录的 `node_modules`。您需要调整相对路径指向那里：

```
monorepo/
├── node_modules/streamdown/  ← 提升到这里
├── apps/
│   └── web/
│       └── app/
│           └── globals.css   ← 您的 CSS 文件
```

```css
/* apps/web/app/globals.css → 向上 3 级到达根 node_modules */
@source "../../../node_modules/streamdown/dist/*.js";
```

根据您的 CSS 文件相对于根 `node_modules` 的位置，调整 `../` 的数量。如果安装了 Streamdown 插件，仅在实际安装这些包时添加对应的 `@source` 条目。

### CSS 自定义属性（设计令牌）

Streamdown 组件基于 shadcn/ui 的设计系统构建，依赖 CSS 自定义属性来处理颜色、圆角和间距。如果未定义这些变量，组件可能会出现背景缺失、边框缺失或间距错误的问题。

如果您已经在使用 shadcn/ui，这些变量会自动设置。如果没有，请将以下最小配置添加到您的全局 CSS 中：

```css
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.145 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.556 0 0);
  --border: oklch(0.922 0 0);
  --input: oklch(0.922 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --radius: 0.625rem;
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --card: oklch(0.205 0 0);
  --card-foreground: oklch(0.985 0 0);
  --muted: oklch(0.269 0 0);
  --muted-foreground: oklch(0.708 0 0);
  --border: oklch(0.269 0 0);
  --input: oklch(0.269 0 0);
  --primary: oklch(0.985 0 0);
  --primary-foreground: oklch(0.205 0 0);
  --radius: 0.625rem;
}
```

您也可以使用 shadcn/ui 的[主题生成器](https://ui.shadcn.com/themes)创建自定义配色，并将生成的 CSS 变量直接复制到您的项目中。

## 使用方法

以下是在 React 应用中使用 Streamdown 结合 AI SDK 的示例：

```tsx
import { useChat } from "@ai-sdk/react";
import { Streamdown } from "streamdown";
import { code } from "@streamdown/code";
import { mermaid } from "@streamdown/mermaid";
import { math } from "@streamdown/math";
import { cjk } from "@streamdown/cjk";
import "katex/dist/katex.min.css";
import "streamdown/styles.css";

export default function Chat() {
  const { messages, status } = useChat();

  return (
    <div>
      {messages.map(message => (
        <div key={message.id}>
          {message.role === 'user' ? 'User: ' : 'AI: '}
          {message.parts.map((part, index) =>
            part.type === 'text' ? (
              <Streamdown
                key={index}
                animated
                plugins={{ code, mermaid, math, cjk }}
                isAnimating={status === 'streaming'}
              >
                {part.text}
              </Streamdown>
            ) : null,
          )}
        </div>
      ))}
    </div>
  );
}
```

更多信息请参阅[文档](https://streamdown.ai/docs)。
```