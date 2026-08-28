# 影之诗·每日情报

面向《影之诗：超凡世界》的每日环境决策页。页面把具体 40 张构筑、X 侧强度复现、B站内容扩散和有方向的克制证据分开记录，并明确展示每条建议尚缺哪一环。

- 在线页面：<https://ayoo-yu.github.io/shadowverse-worlds-beyond-daily/>
- 对局反馈：<https://github.com/Ayoo-Yu/shadowverse-worlds-beyond-daily/issues/new?template=matchup-feedback.yml>

连胜帖数量不是胜率，视频热度不是卡组强度，同职业的不同体系不共享克制结论。当前页面是每日证据快照，不构成总体天梯对局矩阵。

每日固定检索来源、执行顺序和失败语义见 [`WORKFLOW.md`](WORKFLOW.md)；机器可读清单见 [`workflow.json`](workflow.json)。只有所有关键检查完整核验后，网页才会输出可执行上分推荐。

B站账号监控按固定的 9 个数字 UID 执行：优先读取账号投稿，受限时回退到站内检索并严格校验作者 UID。页面同时公开“来源读取”和“内容复核”两个状态；赛事、点评等非卡组视频不会误计为卡组扩散，未复核候选也不会被当作无更新。

X监控由官方 Recent Search 的日文、英文主题查询，以及全部有效固定／动态账号的联合 `from:` 查询组成。页面分别展示查询翻页状态、逐账号窗口覆盖和候选内容复核；三项来源即使达到3/3，只要仍有待复核帖子，推荐闸门仍保持关闭。
