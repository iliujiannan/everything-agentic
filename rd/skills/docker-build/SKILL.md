---
name: docker-build
description: 展示构建命令供 review，确认后构建并推送 Docker 镜像
origin: ear
---

调用 `/ear:docker-build` 执行以下流程：

1. 读取 `pom.xml` 中的 `artifactId` 和 `version`，读取 `Dockerfile`
2. 展示将要执行的命令供 review：
   - `docker build -t {registry}/{artifactId}:{version} .`
   - `docker push {registry}/{artifactId}:{version}`
3. 停止等待确认（必须确认 registry 地址和镜像名称）
4. 确认后依次执行 build 和 push，实时输出日志
5. 构建完成后输出镜像 tag，停止

> 规则参考：`rd/rules/common/patterns.md` 容器化最佳实践、`rd/rules/common/security.md`
