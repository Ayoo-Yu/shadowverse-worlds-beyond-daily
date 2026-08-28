# 每日情报采集工作流

`config/daily-workflow.json` 是机器可读的唯一工作流清单；`config/sources.json` 是账号注册表；`data/daily/YYYY-MM-DD.json` 中的 `collection_audit.checks` 是当日实际执行记录。三者不能互相替代。

## 每日顺序

1. 逐一核验 9 个用户指定的 B站账号。采集器先读取账号投稿接口；遇到 412、-799 等限制时，自动改用站内视频检索，并且只接受完全一致的数字 UID。每个账号都要写入来源状态、内容复核状态、时间、方法、证据链接和最新结果；短时外部阻断可沿用 24 小时内的已核验缓存，但必须明确标记缓存方法。
2. 对观察窗口内的每条 B站候选做内容分类。已映射到具体卡组观察的视频才能计作卡组传播；赛事、全卡点评、直播回放等非卡组内容必须排除；其余候选保持 `pending`，使推荐闸门降级，而不能自动判成“有扩散”或“无更新”。随后做两层全站检索：先以“影之诗 超凡世界”按发布时间翻页到观察窗口起点，发现名单外体系；再从当天 `meta_presence` 与候选构筑自动生成逐体系清单，逐项使用固定中文召回词搜索。任何体系缺失、没有翻到截止点或内容未复核，`bili_search_archetypes` 都不得为 `verified`。账号检索用于发现主播推动，全站检索用于测量名单外扩散，两者缺一不可。
3. 执行 X 日文、英文 `Latest` 查询，并把 `config/sources.json` 中未过期的固定与动态账号合并为精确 `from:` 查询。有 Bearer Token 时使用官方 Recent Search，只有翻页到末尾且没有未解决错误才记为 `verified`。无 Token 时默认操控已登录浏览器：记录固定查询、可见结果、帖子 ID，并将账号联合查询滚动到观察窗口起点；这条通道记为“登录态有界检索”，不得冒充 API 全量分页。只有浏览器无法访问或未达到固定截止条件时才记为 `partial/blocked`。
4. 对 X 候选按 Post ID 去重并复核正文、构筑图片、连胜口径和转载链。已映射证据、卡组候选、克制候选、无关内容和转载必须明确分类；任何 `pending` 候选都会继续关闭推荐闸门。
5. 对候选构筑核验可复制的具体 40 张；官网牌组页只证明牌表，不证明连胜、热度或克制。
6. 对“候选 → 主流目标”的每条克制边至少寻找两个独立、同体系范围、同方向的原始来源。没有分对局日志时证据等级封顶 B。
7. 先运行质量校验，再生成网页。来源被风控时允许发布“数据不完整”的情报页，但不得发布可执行上分推荐。

## 状态语义

- `verified`：精确账号/查询成功读取，观察窗口、作者或 UID 可以复核；X API模式必须完整翻页，浏览器模式必须使用 `Latest`、保存帖子 ID并达到预设截止条件。
- `partial`：查询执行成功但无法证明覆盖完整，例如站内搜索未返回精确作者结果。
- `blocked`：412、-799、验证码、超时等外部阻断；只表示未知。
- `not_run`：本轮没有执行。它与 `blocked` 都不能写成“没有更新”。

只有全部 `critical` 检查为 `verified`，并且B站与X候选内容均完成复核，页面才允许输出“优先玩/建议上分”类动作。检查项齐全但部分被阻断或仍有待分类内容时，网页仍可发布，以便明确暴露数据缺口。

## 校验与发布

```powershell
python -m yzs_daily fetch-bili `
  --sources config\sources.json `
  --queries config\queries.json `
  --workflow config\daily-workflow.json `
  --snapshot data\daily\2026-08-28.json `
  --output data\raw\bilibili-2026-08-28-latest.json `
  --since 2026-08-27T00:00:00+08:00 `
  --update-snapshot

# 公共搜索触发风控时，由已登录浏览器完成同一计划并导入
python -m yzs_daily import-bili-browser `
  --capture data\raw\bilibili-browser-2026-08-28-latest.json `
  --queries config\queries.json `
  --workflow config\daily-workflow.json `
  --snapshot data\daily\2026-08-28.json

python -m yzs_daily fetch-x `
  --queries config\queries.json `
  --sources config\sources.json `
  --workflow config\daily-workflow.json `
  --snapshot data\daily\2026-08-28.json `
  --output data\raw\x-2026-08-28-latest.json `
  --start-time 2026-08-27T00:00:00+08:00 `
  --update-snapshot

# 无Token时，由已登录浏览器完成固定搜索并保存采集文件后导入
python -m yzs_daily import-x-browser `
  --capture data\raw\x-browser-2026-08-28-latest.json `
  --queries config\queries.json `
  --sources config\sources.json `
  --workflow config\daily-workflow.json `
  --snapshot data\daily\2026-08-28.json

python scripts\validate_daily_workflow.py `
  --input data\daily\2026-08-28.json

.\scripts\publish_dashboard.ps1 -Date 2026-08-28 -Push
```

发布脚本默认先运行 B站逐 UID、主题宽搜和逐体系采集，再运行三项 X 采集，最后执行同一质量校验。B站公共搜索受限时导入 `data/raw/bilibili-browser-日期-latest.json`；X 有 `X_BEARER_TOKEN` 时走官方 API，没有时导入 `data/raw/x-browser-日期-latest.json`。两份浏览器文件都必须由已登录浏览器按机器生成计划检索。只有离线重放一份已经采集完毕的快照时才使用 `-SkipBilibiliCollect -SkipXCollect`。缺失任何固定检查项、未翻到观察窗口起点、当天体系清单漏项、账号 UID 不一致、覆盖数字与逐项审计不一致，都会阻止推荐或中止构建。
