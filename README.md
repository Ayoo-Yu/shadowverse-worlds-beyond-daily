# 影之诗·今日上分雷达

面向《影之诗：超凡世界》国服的每日环境决策页。首页只回答四个玩家问题：最近哪些卡组有强度、哪套仍有信息差、哪些正在快速扩散、已确认的有利对局指向哪里；具体 40 张牌表和成绩来源分别展示，避免把官网牌表误当成强度证据。

- 在线页面：<https://ayoo-yu.github.io/shadowverse-worlds-beyond-daily/>
- 对局反馈：<https://github.com/Ayoo-Yu/shadowverse-worlds-beyond-daily/issues/new?template=matchup-feedback.yml>

连胜帖数量不是胜率，视频热度不是卡组强度，同职业的不同体系不共享克制结论。当前页面是每日证据快照，不构成总体天梯对局矩阵。

每日固定检索来源、执行顺序和失败语义保留在 [`WORKFLOW.md`](WORKFLOW.md) 与 [`workflow.json`](workflow.json)，不占用面向玩家的首页。只有所有关键检查完整核验后，网页才会输出可执行上分建议。

B站账号监控按固定的 9 个数字 UID 执行：优先读取账号投稿，受限时回退到站内检索并严格校验作者 UID。赛事、点评等非卡组视频不会误计为卡组扩散，未复核候选也不会被当作无更新。

X监控由日文、英文 `Latest` 主题查询，以及全部有效固定／动态账号的联合 `from:` 查询组成。有 Token 时使用官方 Recent Search；无 Token 时默认使用已登录浏览器执行固定有界检索并滚动账号查询到观察窗口起点。来源覆盖与内容复核在底层分开计算；只要仍有待复核帖子，推荐闸门就保持关闭。
