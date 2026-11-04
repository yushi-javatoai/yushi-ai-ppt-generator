# Issue 跟踪：GitHub

本仓库的 Issue 与规格说明都放在 GitHub Issues。所有操作使用 `gh` CLI。

## 约定

- **创建 Issue**：`gh issue create --title "..." --body "..."`。多行正文用 heredoc。
- **读取 Issue**：`gh issue view <number> --json number,title,body,labels,comments`。
- **列出 Issue**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，并按需要加上 `--label`、`--state` 过滤。
- **把 Issue 设为父 Issue 的子 Issue**：`gh issue create --parent <parent> ...`，或事后 `gh issue edit <parent> --add-sub-issue <child>`（`gh` 2.94+）。更早版本：`gh api --method POST repos/<owner>/<repo>/issues/<parent>/sub_issues -F sub_issue_id=<child-db-id>`（数据库 id，见下方 **阻塞关系**）。若不支持子 Issue，在子 Issue 正文顶部写 `Part of #<parent>`。
- **阻塞关系**：使用 GitHub **原生 Issue 依赖**，这是界面上可见的标准表示。添加依赖：`gh issue create --blocked-by <n>,<n> ...`，或 `gh issue edit <child> --add-blocked-by <blocker>`（`gh` 2.94+）。更早版本：`gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`，其中 `<blocker-db-id>` 是阻塞方的数字 **数据库 id**（`gh api repos/<owner>/<repo>/issues/<n> --jq .id`，不是 `#number`，也不是 `node_id`）。若依赖功能不可用，在子 Issue 正文顶部写一行 `Blocked by: #<n>, #<n>`。所有阻塞方都关闭后，该工单才算解除阻塞。
- **统计未关闭的阻塞方**：`gh api repos/<owner>/<repo>/issues/<n> --jq .issue_dependencies_summary.blocked_by`（只计未关闭的阻塞方；0 表示可以开工）。`gh issue view <n> --json blockedBy` 也会列出已关闭的阻塞方，需要过滤：`--jq '[.blockedBy.nodes[] | select(.state == "OPEN")] | length'`。
- **评论 Issue**：`gh issue comment <number> --body "..."`
- **添加 / 移除标签**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **关闭**：先用 `gh issue comment <number> --body "..."` 写明原因，再 `gh issue close <number>`。对已关闭的 Issue，`gh issue close --comment` 会丢掉评论，且退出码仍为 0。

仓库从 `git remote -v` 推断；在克隆目录内运行时，`gh` 会自动完成这一步。

只向用户自己的仓库提 Issue。除非用户本人拥有 `mattpocock/skills`（这些技能的来源仓库），否则不要往那里提。在 fork 中，`gh` 可能默认指向上游父仓库，因此在一次会话里第一次执行 `gh issue create` 之前，用 `gh repo view --json nameWithOwner` 核对目标；若不是用户的仓库，先与用户确认。

## 把 Pull Request 当作分诊入口

**把 PR 当作需求入口：否。**（若本仓库把外部 PR 当作功能需求处理，改为 `yes`；`/triage` 会读取该标记。）

设为 `yes` 时，PR 与 Issue 使用同一套标签和状态，对应命令换成 `gh pr`：

- **读取 PR**：`gh pr view <number>`，然后 `gh pr view <number> --comments`（非 TTY 时 `--comments` 只打印评论，不打印正文），差异用 `gh pr diff <number>`。
- **列出待分诊的外部 PR**：`gh api --paginate 'repos/{owner}/{repo}/pulls?state=open' --jq '.[] | select(.author_association | IN("OWNER","MEMBER","COLLABORATOR") | not) | {number, title, author: .user.login, author_association, labels: [.labels[].name]}'`。
- **评论 / 打标签 / 关闭**：`gh pr comment`、`gh pr edit --add-label` / `--remove-label`、`gh pr close`。

GitHub 的 Issue 与 PR 共用同一套编号，单独的 `#42` 可能是其中任意一种：先 `gh pr view 42`，失败再 `gh issue view 42`。

## 技能要求「发布到 Issue 跟踪器」时

创建一个 GitHub Issue。

## 技能要求「获取相关工单」时

按上文 **读取 Issue** 的方式读取。

## 寻路操作

供 `/wayfinder` 使用。**地图**是一条 Issue，其 **子 Issue** 作为工单。

- **地图**：带标签 `wayfinder:map` 的单条 Issue，正文包含 Notes / Decisions-so-far / Fog。创建：`gh issue create --label wayfinder:map`。
- **子工单**：作为 GitHub 子 Issue 挂到地图上（见 **把 Issue 设为父 Issue 的子 Issue**）。若未启用子 Issue，把子项写进地图正文的任务列表，并在子 Issue 正文顶部写 `Part of #<map>`。标签：`wayfinder:<type>`（`research` / `prototype` / `grilling` / `task`）。认领后，工单指派给正在推进的开发者。
- **阻塞关系**：见约定中的 **阻塞关系**。
- **前沿查询**：列出地图下未关闭的子 Issue（`gh issue list --state open`，范围限定在地图的子 Issue / 任务列表），去掉仍有未关闭阻塞方的项（见 **统计未关闭的阻塞方**，或 `Blocked by` 行里仍有未关闭 Issue）以及已有指派人的项；按地图中的顺序，取第一条。
- **认领**：`gh issue edit <n> --add-assignee @me`，作为本次会话的第一次写入。
- **结案**：`gh issue comment <n> --body "<answer>"`，然后 `gh issue close <n>`，再往地图的 Decisions-so-far 追加一条上下文指针（gist + 链接）。
