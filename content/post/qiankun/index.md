---
title: "qiankun微前端 + monorepo + pnpm实践"
description: 
date: 2024-12-27
draft: false
tags: [javascript]
categories: [JS]
---

## 技术选型背景
现代前端应用日益复杂，传统的单体架构面临以下挑战：

- 代码库膨胀，维护困难

- 团队协作效率低下

- 技术栈升级困难

- 独立部署能力差

解决方案组合：

- Qiankun：阿里巴巴开源的微前端框架，提供完整的生命周期管理、样式隔离、JS沙箱等能力

- Monorepo：单一仓库管理多个项目，便于代码共享和依赖管理

- PNPM：快速、节省磁盘空间的包管理器，完美支持Monorepo

## 架构设计

### 整体架构图

```bash
monorepo-root/
├── apps/
│   ├── main-app/        # 主应用(基座)
│   ├── micro-app-1/     # 微应用1
│   └── micro-app-2/     # 微应用2
├── packages/
│   ├── shared-utils/    # 共享工具库
│   └── shared-components/ # 共享组件库
└── pnpm-workspace.yaml  # PNPM工作空间配置
```

### 技术特性

1. 依赖共享：通过PNPM workspace提升安装效率

2. 样式隔离：Qiankun提供的沙箱机制 + 自定义前缀策略

3. 状态管理：主应用与子应用通过自定义事件通信

4. 独立开发：每个微应用可单独开发调试

5. 统一构建：通过Monorepo实现统一CI/CD流程

## Monorepo 项目结构

### 项目结构

```bash
monorepo-root/
├── .husky/              # Git hooks配置
├── .github/             # GitHub工作流
├── .vscode/             # 团队统一编辑器配置
├── apps/                # 应用目录
│   ├── main-app/        # 主应用(React/Vue)
│   ├── micro-app-react/ # React微应用
│   └── micro-app-vue/   # Vue微应用
├── packages/            # 共享包
│   ├── shared-utils/    # 通用工具函数
│   ├── shared-config/   # 共享配置(eslint,stylelint等)
│   └── shared-types/    # 共享TypeScript类型定义
├── scripts/             # 自定义脚本
├── package.json         # 根package.json
├── pnpm-workspace.yaml  # PNPM工作空间配置
└── turbo.json           # Turborepo构建配置(可选)
```

### 关键配置文件

pnpm-workspace.yaml

```yaml
packages:
  - 'apps/*'
  - 'packages/*`
```
全局package.json示例

```json
{
  "name": "monorepo-root",
  "private": true,
  "scripts": {
    "prepare": "husky install",
    "lint": "run-p lint:*",
    "lint:eslint": "eslint --fix --ext .js,.jsx,.ts,.tsx apps/ packages/",
    "lint:style": "stylelint \"**/*.{css,less,scss}\"",
    "build": "turbo run build",
    "dev": "turbo run dev --parallel"
  },
  "devDependencies": {
    "eslint": "^8.0.0",
    "husky": "^8.0.0",
    "npm-run-all": "^4.1.5",
    "pnpm": "^7.0.0",
    "stylelint": "^14.0.0",
    "turbo": "^1.0.0"
  }
}
```