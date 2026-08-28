# 每日情报采集工作流

`config/daily-workflow.json` 是机器可读的唯一工作流清单；`config/sources.json` 是账号注册表；`data/daily/YYYY-MM-DD.json` 中的 `collection_audit.checks` 是当日实际执行记录。三者不能互相替代。

## 每日顺序

1. 逐一核验 9 个用户指定的 B站账号。采集器先读取账号投稿接口；遇到 412、-799 等限制时，自动改用站内视频检索，并且只接受完全一致的数字 UID。每个账号都要写入来源状态、内容复核状态、时间、方法、证据链接和最新结果；短时外部阻断可沿用 24 小时内的已核验缓存，但必须明确标记缓存方法。
2. 对观察窗口内的每条 B站候选做内容分类。已映射到具体卡组观察的视频才能计作卡组传播；赛事、全卡点评、直播回放等非卡组内容必须排除；其余候选保持 `pending`，使推荐闸门降级，而不能自动判成“有扩散”或“无更新”。随后执行 B站全站主题检索，对当日 `meta_presence` 与所有候选体系逐一搜索中文名和别名。账号检索用于发现主播推动，关键词检索用于发现名单外扩散，两者缺一不可。
3. 执行 X 日文、英文 Recent Search，随后逐一检查 `config/sources.json` 中未过期的 X 固定与动态账号。聚合页只能发现候选，回链到原帖后才可计作强度或克制证据。
4. 对候选构筑核验可复制的具体 40 张；官网牌组页只证明牌表，不证明连胜、热度或克制。
5. 对“候选 → 主流目标”的每条克制边至少寻找两个独立、同体系范围、同方向的原始来源。没有分对局日志时证据等级封顶 B。
6. 先运行质量校验，再生成网页。来源被风控时允许发布“数据不完整”的情报页，但不得发布可执行上分推荐。

## 状态语义

- `verified`：精确账号/查询成功读取，观察窗口、作者或 UID 可以复核。
- `partial`：查询执行成功但无法证明覆盖完整，例如站内搜索未返回精确作者结果。
- `blocked`：412、-799、验证码、超时等外部阻断；只表示未知。
- `not_run`：本轮没有执行。它与 `blocked` 都不能写成“没有更新”。

只有全部 `critical` 检查为 `verified`，页面才允许输出“优先玩/建议上分”类动作。检查项齐全但部分被阻断时，网页仍可发布，以便明确暴露数据缺口。

## 校验与发布

```powershell
python -m yzs_daily fetch-bili `
  --sources config\sources.json `
  --workflow config\daily-workflow.json `
  --snapshot data\daily\2026-08-28.json `
  --output data\raw\bilibili-2026-08-28-latest.json `
  --since 2026-08-27T00:00:00+08:00 `
  --update-snapshot

python scripts\validate_daily_workflow.py `
  --input data\daily\2026-08-28.json

.\scripts\publish_dashboard.ps1 -Date 2026-08-28 -Push
```

发布脚本默认先运行 B站逐 UID 采集，再执行同一质量校验。只有离线重放一份已经采集完毕的快照时才使用 `-SkipBilibiliCollect`。缺失任何固定检查项、账号 UID 不一致、覆盖数字与逐项记录不一致，都会中止构建；外部来源被阻断不会伪装成成功，而会把推荐模式降为 `provisional`。
