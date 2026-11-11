---
layout: post
title: "前端 TypeScript 开发基本概念"
date: 2026-11-10 21:00:00 +0800
tags: FrontEnd
---

- Monorepo
  一种软件开发实践，将项目的所有代码集中在单个存储库中，而不是为每个组件或模块使用独立存储库。Monorepo 有助于提升代码共享效率、简化版本控制，并缩短开发周期。

- Rush Monorepo
  全名 Rush Stack，一个面向 JavaScript/TypeScript 单体大仓库的工程管理工具，monorepo 的 “包管理器 + 构建编排 + 发版流水线 + 仓库规范” 四合一，由微软开源。
  - 常用命令：
    ```bash
    rush install      # 安装依赖（受锁文件约束）
    rush update       # 更新依赖并刷新锁文件
    rush build        # 按依赖图增量构建所有项目
    rush build -t @scope/pkg   # 只构建该包及其上游依赖
    rush change       # 生成变更说明文件
    rush publish      # 统一发版
    ```

- Rush subspaces
  让同一个 Rush 大仓库内部同时存在多份 pnpm 锁文件，适用于超大代码库或有历史遗留的代码库。
  - 子空间名字必须在 `common/config/rush/subspaces.json` 里集中注册；项目靠 `rush.json` 里的 `"subspaceName": "xxx"` 归属到某个子空间，**不写就落到 `default` 子空间**
  - 每个子空间一个目录 `common/config/subspaces/<subspace-name>/`，可放：
    - `common-versions.json`（版本覆盖）
    - `pnpm-config.json`（pnpm 配置）
    - `pnpm-lock.yaml`（锁文件）
    - `repo-state.json`（Rush 生成，防止手改锁文件）
    - `.npmrc`、`.pnpmfile.cjs`
  - 相关命令：
    - `rush install` 默认把所有子空间都装一遍
    - `rush install --to my-project`（只装某项目的依赖）
    - `rush install --subspace my-subspace`（只装某子空间）
    - `rush install --to subspace:my-subspace`（用 project selector）

- pnpm
  非官方类似 npm 的包管理工具，严格声明依赖，全局保存一份包。

- Biome
  用 Rust 语言编写的 Web 开发完整工具链，支持 JSON / JS / TS / CSS 等多种 Web 开发常用的语言，并且提供了 Linter / Prettier / VSCode 插件等丰富的功能，可以有效提升开发体验。适用于构建前端应用、管理项目代码质量以及统一团队开发规范的场景。
