---
name: subagent-deploy
description: 工作流阶段7：构建 & 部署。执行 Docker 镜像构建、推送到 Registry，再部署到 K8s，三步作为单一阶段处理。由 workflow-orchestrator 派发。
---

# 阶段 7：构建 & 部署

本阶段将 Docker 构建和 K8s 部署合并为一个原子操作，避免镜像构建成功但部署未跟进的中间状态。

## 前置条件

- 已接收：阶段 6 的 commit hash、目标部署环境（dev / staging / prod）
- Git 提交已成功

## 执行步骤

### Step 1：Docker 构建

调用 `/docker-build` 命令：
1. 读取项目 `Dockerfile`，验证基础镜像和构建参数
2. 执行镜像构建
3. 构建成功后，按命名规范打 tag：`{registry}/{image}:{version}-{commit-short}`
4. 推送镜像到 Registry

### Step 2：K8s 部署

调用 `/k8s-deploy` 命令：
1. 确认目标命名空间和 Deployment 名称
2. 更新 Deployment 镜像 tag
3. 验证 Deployment 规范（必须包含）：
   - `resources.limits`
   - `readinessProbe`
   - `livenessProbe`
   - 敏感配置通过 Secret 注入，不写在 ConfigMap 里
4. Apply 变更，等待 rollout 完成
5. 验证 Pod 状态（Running + Ready）

### Step 3：验证

- 检查新 Pod 是否全部 Running
- 检查 readinessProbe 通过
- 记录最终镜像 tag 和 Pod 名称

## 产出物

- Docker 镜像（tag + digest）
- K8s Deployment 更新记录
- 部署验证报告

## Gate 条件

- 镜像推送成功 AND Pod 全部 Running AND readinessProbe 通过

## 阶段摘要

完成后输出：

```
## 阶段摘要：构建 & 部署
- 状态：PASS / FAIL
- 镜像：{registry}/{image}:{tag}
- 部署环境：{namespace}/{deployment}
- Pod 状态：{N}/{N} Running
- Gate 条件：{满足 / 未满足}
- 待确认事项：工作流全部完成 🎉 / [FAIL] 请查看错误详情
```

**工作流结束。如失败，输出错误详情等待用户指令。**
