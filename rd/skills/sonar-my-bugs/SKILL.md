---
name: sonar-my-bugs
description: |
  从 SonarQube 拉取指定用户/项目的未解决 Bug，生成报告并逐一修复。
  每次运行会交互式索取 Cookie，首次运行会要求配置 branch/author/project/sonarUrl，
  后续运行可沿用上次配置。使用此 skill 当用户提到 SonarQube bug、sonar bug、
  代码质量问题、Sonar 扫描结果、或要求修复 SonarQube 告警时。
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - AskUserQuestion
---

# SonarQube Bug 查询与修复

## 第 0 步：加载配置并收集参数

### 0.1 读取历史配置

读取 `.claude/skills/sonar-my-bugs/config.json`。如果文件不存在或字段为空，则视为首次运行。

```json
{
  "sonarUrl": "",
  "author": "",
  "project": "",
  "branch": ""
}
```

### 0.2 收集 Cookie（每次必问）

使用 AskUserQuestion 向用户索取完整的浏览器 Cookie 字符串：

```
问题："请提供 SonarQube 的完整浏览器 Cookie（在浏览器 F12 → Network → 任意请求的 Request Headers 中复制 Cookie 值）："
```

将 Cookie 存入变量，不要持久化保存（安全考虑）。

### 0.3 首次运行：强制收集全部配置

如果是首次运行（config.json 不存在或字段为空），使用 AskUserQuestion 逐一询问：

| 参数 | 提问内容 |
|------|---------|
| `sonarUrl` | "请输入 SonarQube 地址（如 `http://sonar.example.com:9000`）" |
| `author` | "请输入你的 SonarQube 用户名（通常是邮箱）" |
| `project` | "请输入项目 Key" |
| `branch` | "请输入分支名（如 `main`、`develop`、`uat`）" |

收集完成后，将这四个值写入 `config.json` 持久保存，并输出确认：

```
配置已保存：
- SonarQube: <sonarUrl>
- 作者:   <author>
- 项目:   <project>
- 分支:   <branch>
- 级别:   CRITICAL, MAJOR
```

### 0.4 非首次运行：询问是否更改

如果配置文件存在且字段非空，使用 AskUserQuestion：

```
问题："检测到上次配置：author=<author>, project=<project>, branch=<branch>，是否需要修改？"
选项：
  - "沿用上次配置" (Recommended)
  - "修改配置"
```

如果用户选择修改，则重新收集全部四个字段并保存。

---

## 第 1 步：获取 Bug 列表

将 `author` 邮箱做 URL encode（`@` → `%40`），然后用 Bash 执行：

```bash
curl -s \
  -H "Cookie: <COOKIE>" \
  "<SONAR_URL>/api/issues/search?branch=<BRANCH>&componentKeys=<PROJECT>&s=FILE_LINE&author=<AUTHOR_ENCODED>&resolved=false&severities=CRITICAL%2CMAJOR&ps=100&facets=severities%2Ctypes%2Cauthor&additionalFields=_all&timeZone=Asia%2FShanghai"
```

从响应中提取：
- `total`：总数
- `issues[]`：每个 issue 的 `key`、`rule`、`severity`、`message`、`component`、`line`

**错误处理：**
- HTTP 401/403：停止，提示 "Cookie 已失效，请重新提供"
- `total` 为 0：输出 "当前没有待修复的 Bug" 后停止

---

## 第 2 步：获取规则详情

对每个**唯一 rule key** 去重，只请求一次：

```bash
curl -s \
  -H "Cookie: <COOKIE>" \
  "<SONAR_URL>/api/rules/show?key=<RULE_KEY>"
```

提取 `rule.name` 和 `rule.htmlDesc`（去 HTML 标签，取前 300 字）。

---

## 第 3 步：获取代码位置快照

对每个 issue key 请求代码片段：

```bash
curl -s \
  -H "Cookie: <COOKIE>" \
  "<SONAR_URL>/api/sources/issue_snippets?issueKey=<ISSUE_KEY>"
```

提取问题所在的文件路径和相关代码行。

---

## 第 4 步：输出 Bug 总结报告

```
## SonarQube Bug 总结报告

**查询时间：** <当前时间>
**作者：** <author>
**项目：** <project>
**分支：** <branch>
**待修复总数：** <total>（CRITICAL: X，MAJOR: Y）

### Bug 列表

| # | 严重级别 | 规则 | 文件 | 行号 | 问题描述 |
|---|---------|------|------|------|---------|
| 1 | CRITICAL | java:S3776 | src/.../Foo.java | 42 | Refactor this method... |
| 2 | MAJOR    | java:S108   | src/.../Bar.java | 15 | Empty catch block      |
...

### 规则说明

**java:S3776 — Cognitive Complexity of methods should not be too high**
> 方法认知复杂度过高，建议拆分为多个小方法...
```

报告输出后**必须停止，等待用户确认**。使用 AskUserQuestion 询问：

```
问题："以上是本次扫描到的 Bug 列表，是否开始逐一修复？"
选项：
  - "开始修复" (Recommended)
  - "暂不修复"
```

只有用户选择"开始修复"才进入第 5 步。

---

## 第 5 步：逐一修复 Bug

**修复原则：**
- 按文件聚合，整体修复，不逐条孤立处理
- 优先级：CRITICAL > MAJOR
- 遵循项目编码规范（`.claude/rules/`）
- 只改有问题的代码，不做无关重构

**常见修复模式：**

| 规则 | 问题 | 修复方式 |
|------|------|---------|
| java:S3776 | 认知复杂度过高 | 提取私有方法，降低嵌套 |
| java:S1541 | 圈复杂度过高 | 提取私有方法，减少分支 |
| java:S108  | 空代码块 | catch 块加 log.debug，或删除无用块 |
| java:S2259 | 空指针解引用 | 加非空检查或使用 Optional |
| java:S1192 | 字符串重复 | 提取为 `private static final String` 常量 |
| java:S112  | 抛出泛化异常 | 改抛具体业务异常 |
| java:S1874 | 使用废弃 API | 替换为非废弃版本 |
| java:S2095 | 资源未关闭 | 用 try-with-resources |
| java:S106  | System.out.println | 改用 SLF4J log |
| java:S1135 | TODO/FIXME 注释 | 删除或完成实现 |

**修复流程（每个文件）：**
1. Read 读取文件完整内容
2. 分析该文件内所有 issue 的位置与原因
3. Edit 整体修复（一个文件一次 Edit，不要拆成多条）
4. 输出摘要：`已修复：src/.../Foo.java（共修复 N 个 issue：S3776x2, S108x1）`

---

## 第 6 步：修复完成报告

```
## 修复完成报告

| 文件 | 修复数 | 规则 |
|------|--------|------|
| src/.../Foo.java | 3 | S3776, S108 |

**总计修复：** X 个 issue，涉及 Y 个文件

> 建议运行 /git-commit 提交修复。
```

---

## 错误处理

| 情况 | 处理方式 |
|------|---------|
| Cookie 失效（401/403） | 立即停止，提示重新提供 Cookie |
| 接口超时 | 重试 1 次，仍失败则跳过并记录 |
| 文件不在本地仓库 | 跳过，报告中标注"文件不在本地" |
| 无法自动修复的复杂 issue | 标注"需人工处理"，附规则说明和代码位置 |
