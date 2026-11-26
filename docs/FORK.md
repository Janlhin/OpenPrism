# Fork 关系与本地差异说明

## 与上游的关系

- 上游仓库：`WLH55/OpenPrism`（GitHub 标记为 parent / source）
- 本仓：`Janlhin/OpenPrism`，公开仓库，默认分支 `main`
- 本仓是上游的 fork，产品代码与文档以上游为准；本批治理改动只落在治理层（`docs/`、`.github/`、根目录治理文件、`tools/`），不改上游核心代码与构建配置。

## 本仓相对上游的差异

比对依据：2026-11-26 在本地以 `--depth 1` 拉取上游 `main` 后执行：

- `git rev-list --left-right --count upstream/main...HEAD` → `0  0`
- `git diff --stat upstream/main HEAD` → 空

结论：快照层面本仓 `main` 与上游 `main` 一致，未检出文件级本地改动。历史提交的先后与分叉情况未核对（浅克隆，完整历史未拉取）。

本批治理分支上新增的内容（仅治理层，不属于上游产品代码）：

- `docs/FORK.md`：本文件。
- `CHANGELOG.md`：记录本仓改动。
- `.github/pull_request_template.md`：PR 填写模板。
- `.github/dependabot.yml`：仅跟踪 `github-actions` 生态的版本更新；产品依赖（npm/pnpm）更新交给上游。
- `.github/workflows/verify.yml`：只检查本仓治理文件是否齐全、结构是否完整。
- `tools/check-governance.py`：治理文件自检脚本（纯标准库）。

## 禁止本地魔改的目录与文件

以下内容来自上游，本地分支不得修改、不得格式化重排：

- `src/`、`web/`、`prototype/`、`test/`、`deploy/`：上游代码、前端、原型、测试与部署目录。
- `docs/`、`mydocs/`：上游既有文档与调研材料。
- `.agents/`、`.claude/`：上游既有代理配置。
- `package.json`、`pnpm-lock.yaml`、`pnpm-workspace.yaml`、`tsconfig.json`、`vitest.config.ts`、`.npmrc`：上游清单、锁文件与构建配置。
- `README.md`、`AGENTS.md`、`CLAUDE.md`、`CONTEXT.md`、`LICENSE`：上游既有正文与许可。

如确需改动上述内容，应先向上游提交并在本仓同步，而不是在 fork 内单独魔改。

## 同步上游的步骤

1. 添加上游远程（本地备份中已添加）：

   ```bash
   git remote add upstream https://github.com/WLH55/OpenPrism.git
   ```

2. 拉取上游最新 `main`：

   ```bash
   git fetch upstream main
   ```

3. 在治理分支上合并或变基上游：

   ```bash
   git merge upstream/main   # 或 git rebase upstream/main
   ```

4. 冲突只允许出现在治理文件上；若上游核心目录出现冲突，先核对是否误改。

5. 本地复现上游检查的命令以上游 `package.json` 脚本为准（`test`、`typecheck`、`build:web`），需要 pnpm 与对应 Node 版本；这些检查不在本仓治理 CI 中运行。

6. 推送前确认 `git diff --stat upstream/main` 中不包含被禁止魔改的目录。

## CI 口径说明

本仓治理工作流**不接入上游测试框架**：不在 CI 中安装 pnpm 依赖、不跑 `vitest`、不跑 `tsc --noEmit`、不跑前端构建。上游测试与类型检查以上游仓库的流水线为准；本仓 CI 只验证治理文件本身齐全且结构完整。
