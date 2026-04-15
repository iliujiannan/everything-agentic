# Prerequisites

`ear` 插件基于 [ECC (Everything Claude Code)](https://github.com/affaan-m/everything-claude-code) 构建，使用前需先安装 ECC。

## 安装顺序

**Step 1 — 安装 ECC**

```bash
claude plugin install ecc@ecc
```

或在 `~/.claude/settings.json` 的 `enabledPlugins` 中添加：

```json
"ecc@ecc": true
```

**Step 2 — 安装 ear**

```bash
claude plugin install ear@ear
```

或手动在 `settings.json` 添加：

```json
"ear@ear": true
```

并在 `extraKnownMarketplaces` 中注册：

```json
"ear": {
  "source": {
    "repo": "iliujiannan/everything-agentic",
    "source": "github",
    "path": "rd"
  }
}
```

## 依赖关系

```
ear:plan          →  内部调用  →  ecc:plan
ear:feature-dev   →  内部调用  →  ecc:tdd + ecc:code-review
ear:xxx           →  自定义逻辑（不依赖 ECC）
```

`ear:` 的职责是在 ECC 通用流程之上叠加企业内部的规范、卡点和工作流，而不是重新实现 ECC 已有的能力。
