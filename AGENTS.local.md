# AGENTS.local.md — 本仓库定制工作指引

> 本文件给 opencode agent 看,描述这个 fork 的本地定制工作流。
> upstream 自带 `AGENTS.md`,本文件不替换、只补充。

## 背景

- 这是 `router-for-me/Cli-Proxy-API-Management-Center` 的 fork,推在
  `https://github.com/jx3fans/Cli-Proxy-API-Management-Center`
- 定制内容:**去广告、UI 调整**,跟 upstream 设计取向不一致,**不打算 PR 回上游**
- 因此日常只需要把 upstream 的新提交同步进来,自己的定制保留就行

## Git 远端

- `origin` → upstream(`router-for-me/...`)。**只 fetch / pull,从不 push**
- `fork`   → 你自己的 fork(`jx3fans/...`)。**所有 push 都到这里**
- 禁止 `git push origin ...`,禁止 `git push` 不带 remote(默认指 origin)

## 本地定制约定

- 本地所有 UI 定制应该落在**工作树改动 + 1 个 commit**,不要拆成多次小提交
- 当前定制都集中在 **SCSS / 局部 TSX**,主要是:
  - `src/styles/layout.scss`(悬浮按钮、`.top-gradient-blur` 已隐藏、main-content padding)
  - `src/styles/components.scss` / `themes.scss` / `variables.scss` / `reset.scss`
  - `src/features/providers/`(AI Providers 页面布局、隐藏 kimi、移除 QUICK_FILL 面板)
  - `src/pages/LogsPage.module.scss`、`src/components/layout/MainLayout.tsx`
- 提交信息使用**中英双语**,英文在前中文在后,如:
  ```
  chore(ui): retheme header and providers layout / 调整页眉与 Providers 页面样式

  - english line
    中文行
  ```

## 追上游更新

```bash
git fetch origin
git rebase origin/main          # 把"那一个定制 commit"接到上游最新之上
# 有冲突就解决后 git add + git rebase --continue
git push --force-with-lease fork main
```

- **永远用 `--force-with-lease`,不用 `--force`**
- rebase 时只关心自己那 1 个定制 commit,把它在新版上重放即可
- 冲突一般集中在 `src/features/providers/descriptors.ts`、`ProviderCategoryList.tsx`
  之类跟 upstream 同时改的文件,选 upstream 版本 + 重新叠加定制规则

## 部署到本地 CPA 服务

构建产物是 `dist/index.html`(vite-plugin-singlefile 打成单文件),部署到 brew 安装的 CPA:

```bash
# 装依赖(若 bun.lock 变了)
bun install --frozen-lockfile

# 类型检查 + 构建
bun run type-check     # 必跑
bun run build          # 必跑,产 dist/index.html
```

部署路径:`/opt/homebrew/etc/static/management.html`

每次部署前**必须先备份**,**只备份 management.html 本身**,不要备份整个 static 目录(里面是历史 .before-* 残留):

```bash
cp -p /opt/homebrew/etc/static/management.html \
  /opt/homebrew/etc/static/management.html.before-<改动名>-$(date +%H%M%S)
cp dist/index.html /opt/homebrew/etc/static/management.html
```

备份命名规范:`management.html.before-<动作描述>-<时间>`,描述用小写短横线(例
如 `before-thin-glass`、`before-restore-gemini`、`before-deploy-final`)。

`/opt/homebrew/etc/static/` 下既有的 `.before-*` 文件**不要删**,是你之前的回滚点。

## 工作纪律

- 改任何不确定结果的命令前先解释,得到确认再执行
- build 之前先 `type-check`;type-check 不过不要继续 build
- 改 SCSS 时,**先确认目标 className 是不是已经定义在文件别处**,避免重复定义互相覆盖
- 改完 `cliproxyapi.conf` 不要 `brew services restart cliproxyapi`(CPA 会热加载)
- 推送到 fork 后,本地 working tree 最好保持干净(下一次 rebase 才不会乱)

## 快速参考

- 后端源码:`~/gitwork/CLIProxyAPI`(可用作查 `xai-api-key`、`codex-api-key` 等段规则的真相来源)
- 后端配置:`/opt/homebrew/etc/cliproxyapi.conf`
- 前端本地 dev:`bun run dev`(`http://localhost:5173`),但**不要**让 dev server
  常驻,改完就停;线上只信 build 出的 `dist/index.html`
