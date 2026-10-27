# 第 4 章 Git 与 GitHub 协作

本章为[贡献指南](https://github.com/NingXia-XinQiCyberClub/.github/blob/main/CONTRIBUTING.md)的教学版本，侧重操作流程与原理说明。规范性问题以贡献指南为准。

## 4.1 版本控制模型

Git 采用分布式模型，每个克隆均为完整仓库，包含全部提交历史。提交记录由三部分构成：

```
提交对象 = 文件树快照 + 父提交引用 + 作者与时间元数据
```

由此产生的两个性质：

1. 提交内容不可变，修改历史须重写后续全部提交
2. 同一提交在不同克隆中具有相同的哈希值，便于校验一致性

## 4.2 工作区与暂存区

```
工作区 ──git add──▶ 暂存区 ──git commit──▶ 本地仓库 ──git push──▶ 远程仓库
```

暂存区的存在使单次提交可精确控制内容范围。提交前应通过 `git status` 与 `git diff --staged` 确认暂存内容。

## 4.3 常用操作

| 操作 | 命令 | 说明 |
| :-- | :-- | :-- |
| 克隆 | `git clone <地址>` | 默认克隆默认分支的完整历史。成员克隆自己的 fork |
| 浅克隆 | `git clone --depth 1 <地址>` | 仅取最近一次提交，适用于只读场景 |
| 查看远端 | `git remote -v` | 确认 `origin` 指向本人 fork、`upstream` 指向组织仓库 |
| 登记上游 | `git remote add upstream <组织仓库地址>` | 每个 fork 只需执行一次 |
| 拉取上游 | `git fetch upstream` | 获取组织仓库的最新提交 |
| 查看状态 | `git status` | 显示暂存区与工作区差异 |
| 查看差异 | `git diff` | 工作区与暂存区比较 |
| 查看已暂存差异 | `git diff --staged` | 暂存区与最近提交比较 |
| 暂存指定文件 | `git add <文件>` | 优先使用精确路径而非 `git add .` |
| 提交 | `git commit -m "<信息>"` | 见 4.5 节 |
| 查看历史 | `git log --oneline --graph -15` | 图形化显示分支结构 |
| 推送 | `git push` | 首次推送需 `-u origin <分支名>`，推送到本人 fork |
| 快进至上游 | `git merge --ff-only upstream/main` | 拉取合入结果时使用，避免多余合并提交 |
| 同步 fork | `git push origin main` | 本地与上游对齐后推送至本人 fork |
| 同步 | `git pull --rebase` | 将本地提交变基至远程最新提交之上 |

## 4.4 分支模型

组织采用简化的功能分支模型：

```
main ──●──●──●──────────●──●──▶
            \          /
             ●──●──●──●
            feat/xxx（合并后删除）
```

| 分支 | 用途 | 保护策略 |
| :-- | :-- | :-- |
| `main` | 可发布状态 | 受保护，禁止直接推送，需评审通过 |
| `feat/*` | 功能开发 | 合并后自动删除 |
| `fix/*` | 缺陷修复 | 同上 |
| `docs/*` | 文档变更 | 同上 |

分支命名规则见贡献指南第 4 节。成员对组织仓库只有读权限，因此分支建在本人 fork 中，且始终基于 `upstream/main` 创建：

```bash
git fetch upstream
git switch main
git merge --ff-only upstream/main
git switch -c feat/description
```

## 4.5 提交信息

采用 Conventional Commits 简化形式：

```
<类型>(<范围>): <摘要>

<正文>
```

类型取值：`feat`、`fix`、`docs`、`refactor`、`chore`、`test`。

判定标准：一次提交应对应一个可独立回滚的逻辑变更。若摘要中需使用"并且"连接两个动作，说明应拆分为两次提交。

示例对比：

| 不推荐 | 推荐 |
| :-- | :-- |
| `update` | `docs: 补充 Windows 虚拟环境激活步骤` |
| `修改了一下代码` | `fix(flag_handler): 修正端口校验的边界判断` |
| `修复bug和加了新功能` | 拆分为 `fix: ...` 与 `feat: ...` 两次提交 |

## 4.6 变更请求流程

组织默认仓库权限为 `read`：成员账号可读、可克隆，但不能向组织仓库推送分支。因此变更统一经 **fork** 提交——fork 是组织仓库在本人账号下的副本，推送与分支操作都在副本中进行，再通过变更请求合回组织仓库。

### 4.6.1 完整流程

```
0. fork 组织仓库到本人账号，克隆 fork 并登记 upstream
1. 同步 upstream/main
2. 创建功能分支
3. 修改并提交
4. 推送分支至 fork
5. 从 fork 发起变更请求（填写模板）
6. 持续集成检查（首次来自 fork 的请求需维护者批准后运行）
7. 评审与修订
8. 合并（Squash）
9. 同步 fork 与本地，删除分支
```

### 4.6.2 操作示例

```bash
# 0. 首次：fork 后克隆本人副本（fork 在网页上点 Fork 完成）
git clone git@github.com:<本人用户名>/guide.git
cd guide
git remote add upstream https://github.com/NingXia-XinQiCyberClub/guide.git
git remote -v

# 1-3. 每次开工：同步上游、建分支、修改提交
git fetch upstream
git switch main
git merge --ff-only upstream/main
git switch -c docs/fix-typo
# 编辑文件
git add <文件>
git commit -m "docs: 修正术语表述"

# 4. 推送到本人 fork
git push -u origin docs/fix-typo
```

推送后于 fork 页面点击 **Compare & pull request**，确认 base 为组织仓库的 `main`、head 为本人 fork 的分支，按模板填写说明。

### 4.6.3 评审要求

| 项目 | 要求 |
| :-- | :-- |
| 批准人数 | 至少 1 名维护者 |
| 持续集成 | 全部检查通过；首次来自 fork 的变更请求需维护者批准后检查才会运行 |
| 冲突 | 须解决后方可合并 |
| 合并方式 | Squash and merge，保持 `main` 线性历史 |

评审意见的处理方式为在原分支追加提交并推送至 fork，变更请求自动更新。不应关闭后重新发起。

### 4.6.4 合并后清理

合并发生在组织仓库，本人 fork 与本地需各自同步：

```bash
git switch main
git fetch upstream
git merge --ff-only upstream/main   # 拉取组织仓库的合入结果
git push origin main                # 同步到本人 fork
git branch -d <分支名>
```

## 4.7 冲突处理

冲突产生于两个分支修改同一文件的相邻行。处理步骤：

1. 执行 `git status` 确认冲突文件
2. 打开文件，定位冲突标记

```
<<<<<<< HEAD
当前分支内容
=======
合入分支内容
>>>>>>> feat/xxx
```

3. 删除三行标记，整合为最终内容
4. 执行 `git add <文件>`
5. 执行 `git commit` 或 `git rebase --continue`

放弃本次合并可执行 `git merge --abort` 或 `git rebase --abort` 回到操作前状态。

无法判断取舍时，应联系该文件的最近修改者确认。

## 4.8 撤销操作对照

| 目的 | 命令 | 是否改写历史 |
| :-- | :-- | :-- |
| 撤销工作区改动 | `git restore <文件>` | 否 |
| 取消暂存 | `git restore --staged <文件>` | 否 |
| 撤销最近一次提交并保留改动 | `git reset --soft HEAD~1` | 是（仅本地） |
| 撤销最近一次提交并丢弃改动 | `git reset --hard HEAD~1` | 是（仅本地） |
| 已推送的提交需撤销 | `git revert <提交>` | 否，生成反向提交 |

已推送至远程的提交不应使用 `reset` 改写，应使用 `revert`。

## 4.9 组织级配置

| 配置项 | 状态 | 说明 |
| :-- | :-- | :-- |
| `main` 分支保护 | 已启用 | 需评审通过 |
| 合并方式 | 仅 Squash | 保持线性历史 |
| 合并后删除分支 | 已启用 | 减少分支堆积 |
| 组织默认权限 | read | 成员对组织仓库只读，变更经 fork 提交，见 4.6 |

## 4.10 仓库模板与标签

组织级工单模板由 `.github` 仓库提供，对各仓库生效需满足两个条件。

### 4.10.1 模板目录位置

工单模板与 `config.yml` 必须位于仓库内的 `.github/ISSUE_TEMPLATE/` 目录：

```
<仓库>/
└── .github/
    └── ISSUE_TEMPLATE/
        ├── bug_report.yml
        ├── feature_request.yml
        ├── join.yml
        └── config.yml
```

放置于仓库根目录的 `ISSUE_TEMPLATE/` 不会被识别，表现为访问 `?template=<名称>.yml` 时回落到空白工单编辑器。

### 4.10.2 标签依赖

模板 `labels` 字段声明的标签须在当前仓库中存在。GitHub 默认提供 `bug`、`documentation`、`enhancement`、`good first issue`、`help wanted` 等标签，但组织模板额外依赖以下两个非默认标签：

| 标签 | 用途 |
| :-- | :-- |
| `triage` | 待分类，等待维护者评估优先级与归属 |
| `membership` | 加入社团申请与成员登记 |
| `needs-info` | 信息不完整或格式不符，需补充 |

新建仓库时须手动创建这三个标签：

```powershell
foreach ($repo in @('<仓库名>')) {
  gh api -X POST "repos/NingXia-XinQiCyberClub/$repo/labels" -f name="triage" -f color="fbca04" -f description="待分类，等待维护者评估优先级与归属"
  gh api -X POST "repos/NingXia-XinQiCyberClub/$repo/labels" -f name="membership" -f color="0e8a16" -f description="加入社团申请与成员登记"
  gh api -X POST "repos/NingXia-XinQiCyberClub/$repo/labels" -f name="needs-info" -f color="d93f0b" -f description="信息不完整或格式不符，需补充"
}
```

### 4.10.3 验证方式

访问 `https://github.com/NingXia-XinQiCyberClub/<仓库>/issues/new/choose`，页面应列出全部表单选项。

需注意，`community/profile` 接口的 `issue_template` 字段与 GraphQL 的 `repository.issueTemplates` 字段均只反映旧版 `.md` 模板，对 YAML 表单返回空值，不能作为判断依据。

## 4.11 验收清单

- [ ] 可独立完成一次完整变更请求流程
- [ ] 提交信息符合第 4.5 节要求
- [ ] 可独立解决一次合并冲突
- [ ] 理解 `reset` 与 `revert` 的适用场景差异
- [ ] 可为新仓库正确放置工单模板并创建 `triage`、`membership`、`needs-info` 标签

---

适用环境：通用
最近校订：2026-10
