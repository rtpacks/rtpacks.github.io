---
title: 'workflow: 项目工程利用 bumpp + github action 控制版本发包'
date: 2024-08-16 15:11:45
categories: workflow
tags: github
---

## 项目工程利用 bumpp + github action 控制版本发包

学习到 vueuse, mixte 两个库利用 `bumpp + github action` 控制 monorepo 项目版本的具体流程，它共分为三步：

- 项目的正常打包
- commitlint + changelog + bumpp 规范项目的 commit 信息，根据 commit 信息生成更新日志，最后由 bumpp 确定生成版本
- github action 根据 git 事件确定发版和更新 release assets

当然 `bumpp + github action` 这种搭配也是适合非 monorepo 类型项目的版本控制，如果希望每个库都生成对应的 changelog，可以考虑使用 changesets。



在阅读 [基于 pnpm + changesets 的 monorepo 最佳实践最近在设计多组件包管理的 Monorepo 方案 - 掘金 (juejin.cn)](https://juejin.cn/post/7181409989670961207)  后，再参考本文项目流程：[rtpacks/toolkit (github.com)](https://github.com/rtpacks/toolkit)，这里使用 pnpm 作为 workspace 和包管理器。项目根目录新建 `.npmrc` 和 `pnpm-workspace.yml` 两个文件。

```ini
# .npmrc
shamefully-hoist=true
strict-peer-dependencies=false
package-manager-strict=false
```

```yaml
# 
packages:
  - "packages/*"
  - playground
  - docs
```



### 1. 项目的正常打包

采用 monorepo 形式管理多个库，涉及到多个库的打包配置，这里 vueuse 使用手动配置 meta/packages 和 scripts 来完成。它分为三步：

- 配置 meta/packages.ts 中每个 package 的打包信息
- 配置读取 meta/packages.ts 的打包脚本
- 配置使用打包脚本的命令



meta/packages 与 scripts：

```shell
.
├── CHANGELOG.md
├── eslint.config.js
├── meta
│   ├── alias.ts
│   └── packages.ts
├── package.json
├── packages
│   ├── core
│   ├── react
│   └── vue
├── pnpm-lock.yaml
├── pnpm-workspace.yaml
├── scripts
│   ├── build.ts
│   ├── release.ts
│   └── verify-commit.ts
└── tsconfig.json
```



meta/packages 记录每个 package 的打包配置信息，供 scripts 使用：

```typescript
// import type { PackageManifest } from '@vueuse/metadata'

export interface PackageManifest {
  entry: string;
  outDir: string;
  outputFileName?: string;
  /**
   * 是否使用 Vue 及相关插件
   */
  useVue?: boolean;

  /* feature 额外的拓展 */
  author?: string;
  description?: string;
  external?: string[];
  globals?: Record<string, string>;
  manualImport?: boolean;
  deprecated?: boolean;
  submodules?: boolean;
  build?: boolean;
  iife?: boolean;
  cjs?: boolean;
  mjs?: boolean;
  dts?: boolean;
  target?: string;
  utils?: boolean;
  copy?: string[];
}

export const packages: PackageManifest[] = [
  // @rtpackx/core
  {
    entry: "packages/core/index.ts",
    outDir: "packages/core/dist",
  },
  // @rtpackx/vue
  {
    entry: "packages/vue/index.ts",
    outDir: "packages/vue/dist",
    external: ["vue-router"], // 排除打包的库，由外部提供依赖
  },
];
```



scripts 中用 node 读取 meta/packages 信息，然后用 vite 根据配置信息打包生成产物：

```typescript
import { dirname, resolve } from "node:path";
import { fileURLToPath } from "node:url";
import { build } from "vite";
import Vue from "@vitejs/plugin-vue";
import VueJsx from "@vitejs/plugin-vue-jsx";
import fs from "fs-extra";
import { alias } from "../meta/alias";
import { packages } from "../meta/packages";
import configVisualizerPlugin from "../config/plugin/visualizer";

const __dirname = dirname(fileURLToPath(import.meta.url));
fs.removeSync(resolve(__dirname, "../packages/auto-imports.d.ts"));
fs.removeSync(resolve(__dirname, "../packages/.eslintrc-auto-import.json"));

const externals = ["vue", "vue-router", "vue-demi", "@vueuse/core"];
const execFn = async () => {
  for (const manifest of packages) {
    const { useVue, entry, outDir, outputFileName, external } = manifest;
    await build({
      resolve: { alias },
      plugins: [configVisualizerPlugin(), useVue && [Vue(), VueJsx()]],
      build: {
        outDir,
        lib: {
          entry,
          formats: ["es", "cjs"],
          fileName: (format) => `${outputFileName ?? "index"}.${format === "es" ? "mjs" : format}`,
        },
        minify: true,
        emptyOutDir: false,
        rollupOptions: {
          external: externals.concat(external ?? []),
        },
      },
      esbuild: {
        drop: ["console", "debugger"],
        minifyIdentifiers: true,
        minifyWhitespace: true,
        minifySyntax: true,
        treeShaking: true,
      },
    });
  }
};
execFn();
```



最后配置顶部 package.json 的运行命令，这样在 meta/packages.ts 中配置的不同 package 都能生成产物：

```json
{
  "scripts": {
    "build": "pnpm install && tsx scripts/build.ts",
  },
}
```



### 2. commitlint + changelog + bumpp 

如果希望自动生成完整的 changelog，那么规范的 commit 信息必不可少，这里利用 commitlint + changelog + bumpp 规范项目的 commit 信息，然后 changelog 根据 commit 信息生成更新日志，最后由 bumpp 确定生成版本。它分为三步：

- 使用 git-hooks + commitlint 规范 commit 信息
- 使用 changelog 工具生成 CHANGELOG.md
- 使用 bumpp 自动修改顶层的 package.json 和 packages 下各个子包的 package.json 版本，并打上 tag 版本



使用 commitlint 规范提交的 commit 信息：

```shell
pnpm add -Dw simple-git simple-git-hooks @commitlint/cli @commitlint/config-conventional bumpp tsx
```

配置 git-hook 和 changelog，这里的 lint-staged 默认你已经配置好，verify-commit.ts 可以在本项目中获取：

```json
{
    "scripts": {
         "changelog": "npx conventional-changelog -p angular -i CHANGELOG.md -s -r 0",
         "preinstall": "npx only-allow pnpm",
         "postinstall": "simple-git-hooks"
    },
    "simple-git-hooks": {
        "pre-commit": "pnpm lint-staged",
        "commit-msg": "npx tsx scripts/verify-commit.ts"
    },
}
```

现在执行 commit 提交就有规范要求，使用 `pnpm changelog` 即可根据规范的 commit 信息生成 CHANGELOG.md。

最后使用 bumpp 修改版本号并打上对应的 tag：

```json
{
    "scripts": {
        "release": "pnpm changelog && tsx scripts/release.ts"
    }
}
```

```typescript
// scripts/release.ts
import process from "node:process";
import { execSync } from "node:child_process";
import fs from "fs-extra";
// import { readJSONSync } from "fs-extra"; // error, The requested module 'fs-extra' does not provide an export named 'readJSONSync'

const { version: oldVersion } = fs.readJSONSync("package.json");
execSync("bumpp -r --no-commit --no-tag --no-push", { stdio: "inherit" });
const { version } = fs.readJSONSync("package.json");

if (oldVersion === version) {
  console.log("canceled");
  process.exit();
}

execSync("git add .", { stdio: "inherit" });
execSync(`git commit -m "chore: release v${version}"`, { stdio: "inherit" });
execSync(`git tag -a v${version} -m "v${version}"`, { stdio: "inherit" });
```



推送到 github，不要忘记 `--tags` ：

```shell
git push -u
git push -u --tags
```



### 3. github action

到目前位置，配置已经完成了绝大部分：

- 项目可以正常打包
- commit 信息规范，根据 commit 信息可以自动生成 CHANGELOG.md，利用 bumpp 可以快速正确的修改版本

最后利用 github action 完成 npmjs 发包和创建 github release assets，在 `.github/workflows` 中定义 github action：

```yaml
// npm-publish.yaml
name: Node.js Package

on:
  push:
    tags:
      - "v*" # for bumpp

permissions:
  contents: write

jobs:
  publish:
    runs-on: ubuntu-latest

    steps:
      # env
      - name: Setup git
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Setup pnpm
        uses: pnpm/action-setup@v4
      - name: Setup node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          registry-url: https://registry.npmjs.org/
          cache: pnpm
      - name: Setup Install
        run: pnpm install
      # build
      - name: Build
        run: pnpm build
        env:
          NODE_OPTIONS: --max-old-space-size=6144
      # npm-publish
      - name: Publish Beta
        if: contains(github.event.release.name, 'beta') == true
        run: pnpm publish -r --access public --no-git-checks --tag beta
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
      - name: Publish
        if: contains(github.event.release.name, 'beta') == false
        run: pnpm publish -r --access public --no-git-checks
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
      # github release
      - name: Github Release
        uses: softprops/action-gh-release@v2
        with:
          # files: |
          #   CHANGELOG.md
          #   LICENSE
          generate_release_notes: true
        env:
          GITHUB_TOKEN: ${{ secrets.ACCESS_TOKEN }}
```



这里利用 `git` 触发 tags 推送事件，由 bumpp 打上的 tag 推送触发，构建项目并更新 npmjs 和 github release assets。NPM_TOKEN 和 ACCESS_TOKEN 需要自行在项目的 settings/secret and variables 中配置。

> ACCESS_TOKEN 是 GITHUB_TOEKN，由于 secret 中不允许以 GITHUB 开头，所以改成 ACCESS。



最后发版本时，只需要 pnpm release 即可生成对应的 CHANGELOG 和项目版本。如果希望所有的耗费性能的生成都由 github action 实现，那么还可以将 changelog 的生成和未来项目文档的生成都放在 github action。



修订

上述的 changelog 生成需要在打完 tag 后，但是很明显这里的流程是先生成 changlog 然后再打 tag，这样就会导致 changelog 总是落后 tag。

可以选择在生成 tag 后再构建 changelog，不过这样带来的问题是该 tag 携带的并不是最新的 changelog。



### References

- [基于 pnpm + changesets 的 monorepo 最佳实践最近在设计多组件包管理的 Monorepo 方案 - 掘金 (juejin.cn)](https://juejin.cn/post/7181409989670961207)



