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
| 克隆 | `git clone <地址>` | 默认克隆默认分支的完整历史 |
| 浅克隆 | `git clone --depth 1 <地址>` | 仅取最近一次提交，适用于只读场景 |
| 查看状态 | `git status` | 显示暂存区与工作区差异 |
| 查看差异 | `git diff` | 工作区与暂存区比较 |
| 查看已暂存差异 | `git diff --staged` | 暂存区与最近提交比较 |
| 暂存指定文件 | `git add <文件>` | 优先使用精确路径而非 `git add .` |
| 提交 | `git commit -m "<信息>"` | 见 4.5 节 |
| 查看历史 | `git log --oneline --graph -15` | 图形化显示分支结构 |
| 推送 | `git push` | 首次推送需 `-u origin <分支名>` |
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

分支命名规则见贡献指南第 4 节。分支应基于最新 `main` 创建：

```bash
git switch main
git pull --rebase
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

### 4.6.1 完整流程

```
1. 同步 main
2. 创建功能分支
3. 修改并提交
4. 推送分支
5. 发起变更请求（填写模板）
6. 持续集成检查
7. 评审与修订
8. 合并（Squash）
9. 删除分支并同步本地
```

### 4.6.2 操作示例

```bash
git switch main && git pull --rebase
git switch -c docs/fix-typo
# 编辑文件
git add <文件>
git commit -m "docs: 修正术语表述"
git push -u origin docs/fix-typo
```

推送后于 GitHub 点击 **Compare & pull request**，按模板填写说明。

### 4.6.3 评审要求

| 项目 | 要求 |
| :-- | :-- |
| 批准人数 | 至少 1 名维护者 |
| 持续集成 | 全部检查通过 |
| 冲突 | 须解决后方可合并 |
| 合并方式 | Squash and merge，保持 `main` 线性历史 |

评审意见的处理方式为在原分支追加提交并推送，变更请求自动更新。不应关闭后重新发起。

### 4.6.4 合并后清理

```bash
git switch main
git pull
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
| 组织默认权限 | read | 成员默认只读，按仓库授权 |

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
        ├── onboarding.yml
        └── config.yml
```

放置于仓库根目录的 `ISSUE_TEMPLATE/` 不会被识别，表现为访问 `?template=<名称>.yml` 时回落到空白工单编辑器。

### 4.10.2 标签依赖

模板 `labels` 字段声明的标签须在当前仓库中存在。GitHub 默认提供 `bug`、`documentation`、`enhancement`、`good first issue`、`help wanted` 等标签，但组织模板额外依赖以下两个非默认标签：

| 标签 | 用途 |
| :-- | :-- |
| `triage` | 待分类，等待维护者评估优先级与归属 |
| `onboarding` | 新成员入职流程相关 |

新建仓库时须手动创建这两个标签：

```powershell
foreach ($repo in @('<仓库名>')) {
  gh api -X POST "repos/NingXia-XinQiCyberClub/$repo/labels" -f name="triage" -f color="fbca04" -f description="待分类，等待维护者评估优先级与归属"
  gh api -X POST "repos/NingXia-XinQiCyberClub/$repo/labels" -f name="onboarding" -f color="0e8a16" -f description="新成员入职流程相关"
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
- [ ] 可为新仓库正确放置工单模板并创建 `triage`、`onboarding` 标签

---

适用环境：通用
最近校订：2026-10
