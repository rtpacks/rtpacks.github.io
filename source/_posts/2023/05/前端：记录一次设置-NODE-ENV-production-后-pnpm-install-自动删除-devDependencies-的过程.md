---
title: 前端：记录一次设置 NODE_ENV=production 后 pnpm install 自动删除 devDependencies 的过程
date: 2023-05-01 15:18:08
categories: pnpm
tags: pnpm
---

### 问题

在 Jenkins Pipeline 中，我为了便于构建生产和开发环境的项目，设置了 NODE_ENV 为参数的参数化构建过程。在构建的过程中，我将 NODE_ENV 设置为 `production`时， `pnpm install` 不会安装 `devDependencies` 并删除所有的 `devDependencies`。具体可以查看这些文档

- [pnpm install | pnpm](https://pnpm.io/zh/cli/install#--prod--p)

- [`yarn` skips dev dependencies if NODE_ENV is set to "production" · Issue #1830 · yarnpkg/yarn (github.com)](https://github.com/yarnpkg/yarn/issues/1830)

### 解决方案

由于设置 NODE_ENV 会导致这个问题，所以在有关于`pnpm install | run ...` 时，不能直接设置 `--mode production`。

个人的 `package.json : scripts`

```json
{
  "scripts": {
    "dev": "vite --config ./config/vite.config.dev.ts",
    "build": "vue-tsc --noEmit && vite build --config ./config/vite.config.prod.ts",
    "report": "cross-env REPORT=true npm run build",
    "preview": "npm run build && vite preview --host",
    "type:check": "vue-tsc --noEmit --skipLibCheck"
  }
}
```

可以在 scripts 中加入 `--mode development | production`，就能避免 `--mode prodution` 作用 `pnpm`

```json
{
  "scripts": {
    "dev": "vite --config ./config/vite.config.dev.ts",
    "build": "vue-tsc --noEmit && vite build --config ./config/vite.config.prod.ts",
+   "build:dev": "vue-tsc --noEmit && vite build --config ./config/vite.config.prod.ts --mode development",
    "report": "cross-env REPORT=true npm run build",
    "preview": "npm run build && vite preview --host",
    "type:check": "vue-tsc --noEmit --skipLibCheck"
  }
}
```

### 其他拓展

升级包最好使用 `pnpm update`，因为 `pnpm add` 在升级安装时可能会自动 `semver range operator` ，如果要避免这种情况，第一个可以使用`pnpm add --save-exact, -E <package>`，第二种就是 `pnpm update <package>`
