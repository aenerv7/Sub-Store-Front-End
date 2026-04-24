# Cloudflare Pages 部署指南

## 项目概述

本项目是 [sub-store-org/Sub-Store-Front-End](https://github.com/sub-store-org/Sub-Store-Front-End) 的 fork，部署在 Cloudflare Pages 上，项目名为 `sub-store-front-end-cf`。

## 技术栈

- Vue 3 + Vite 3 + TypeScript
- pnpm 7（通过 `package.json` 的 `packageManager` 字段锁定版本）
- 构建输出目录：`dist`

## 部署架构

### 自动化流程

通过 GitHub Actions（`.github/workflows/sync-and-deploy.yml`）实现：

1. 每小时检查上游 `sub-store-org/Sub-Store-Front-End` 是否有新 release
2. 对比本地 `package.json` 版本号与上游最新 release tag
3. 如有新版本，用上游代码完全覆盖项目，然后还原我们自己的文件（workflow、wrangler.toml）
4. 构建并通过 wrangler 部署到 Cloudflare Pages

### 同步策略

项目代码完全跟随上游，我们只维护以下自有文件：

- `.github/workflows/sync-and-deploy.yml` — CI/CD workflow
- `wrangler.toml` — Cloudflare Pages 配置

同步时使用 `git checkout upstream/master -- .` 覆盖全部代码，再还原自有文件，避免 merge 冲突。

### 手动部署

```bash
# 本地构建
pnpm build

# 部署到 Cloudflare Pages
npx wrangler pages deploy dist --project-name=sub-store-front-end-cf
```

## GitHub Secrets 配置

workflow 需要以下 secrets：

| Secret | 说明 |
|--------|------|
| `CLOUDFLARE_API_TOKEN` | Cloudflare API Token，需要 Account / Cloudflare Pages / Edit 权限 |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare Account ID，在 Dashboard 首页右侧获取 |

## 关键配置文件

### wrangler.toml

```toml
name = "sub-store-front-end-cf"
compatibility_date = "2024-09-23"
pages_build_output_dir = "./dist"
```

### package.json 中的关键字段

- `"packageManager": "pnpm@7.33.7"` — 锁定 pnpm 版本，避免 lock 文件格式不兼容

## 踩坑记录

1. **pnpm-lock.yaml 包含淘宝镜像源地址**：Cloudflare 构建环境无法访问 npmmirror，需要用官方 registry 重新生成 lock 文件
2. **pnpm 版本不匹配**：Cloudflare 默认使用 pnpm 10，与本地 pnpm 7 生成的 lock 文件格式不兼容，通过 `packageManager` 字段解决
3. **`wrangler deploy` vs `wrangler pages deploy`**：Pages 项目必须使用 `wrangler pages deploy`，前者是 Workers 的部署命令
4. **上游同步冲突**：不使用 git merge，改用完全覆盖 + 还原自有文件的策略，彻底避免冲突

## 已清理的历史配置

- `.github/workflows/main.yml` — 原 Vercel 部署 workflow（已删除）
- `.github/workflows/update-vercel-project-settings.yml` — 原 Vercel Node.js 版本配置工具（已删除）
