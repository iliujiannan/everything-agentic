---
name: docker-build
description: 展示构建命令供 review，确认后构建并推送 Docker 镜像
---

## 执行步骤

1. 读取 pom.xml 中的 artifactId 和 version，读取 Dockerfile
2. 展示将要执行的命令：
   docker build -t {registry}/{artifactId}:{version} .
   docker push {registry}/{artifactId}:{version}

   同时展示 Dockerfile 内容供 review

3. 停止等待确认。
   提示：请确认镜像名称和 registry 地址，确认后输入 [继续] 执行构建。

4. 收到确认后依次执行 build 和 push，实时输出日志
5. 构建完成后停止。
   提示：✅ 阶段 7 完成。镜像 {tag} 已推送，确认后输入 [继续] 进入 K8s 部署阶段。