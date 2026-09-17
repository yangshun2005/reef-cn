<!-- BEGIN:nextjs-agent-rules -->

# 这不是你所认识的 Next.js

此版本包含破坏性变更——API、约定和文件结构可能都与你的训练数据不同。在编写任何代码之前，请阅读 `node_modules/next/dist/docs/` 中的相关指南（从本文件所在目录解析；在 monorepo 中，从仓库根目录可能无法看到 `next` 包）。请注意已弃用通知。

此区块由 `next dev` 写入并重新添加——可在 `node_modules/next/dist/server/lib/generate-agent-files.js` 中验证。仅从 diff 中移除它只会重新创建未提交的更改；将其与你的工作一起提交可保持工作区干净。

<!-- END:nextjs-agent-rules -->