# 日内交易技能

[English](README.md)

11 个用于日内交易的 Claude 技能：扫描成千上万张图表、跟踪突发新闻、为每一次进场和出场计时，并在你承担一美元风险之前回放每一种可能的结果。

免费、MIT 许可、无需注册、无需 API 密钥。工具是普通 Python，读取免费的公开数据：Yahoo Finance 图表、Nasdaq Trader 的代码列表、SEC EDGAR，以及众议院书记官处。

一个技能扫描成千上万张图表，寻找形态。另一个跟踪你关注的每只股票的突发新闻。一个逐根 K 线盯着形态，告诉你进场的时刻；另一个在你进场之前定好止损和目标。最特别的是时间机器：在你承担一美元风险之前，它用这只股票自己最近的 K 线，把当天剩余的交易时段重放 10,000 次，再加上过去每一个相同形态出现的交易日，然后告诉你诚实的概率。

任何一笔交易都要等你说 yes 才会发出。这一整套里没有任何东西能自行下单。默认是模拟交易。

> **在找原来的 Trader 技能？** 复制投资者的 13F、股票筛选、税批和再平衡，见
> [github.com/Jakeschincariol/trader-skill](https://github.com/Jakeschincariol/trader-skill)。
> 本仓库是它的日内交易续作：Trader 技能 v2。

## 安装

把下面这段贴进 Claude：

```
https://github.com/Jakeschincariol/day-trader-skill

install skill
```

或者在 Claude Code 里当插件安装：

```
/plugin marketplace add Jakeschincariol/day-trader-skill
/plugin install day-trader-skill@day-trader-skill
```

Claude Code 会给插件技能加命名空间，所以装成插件后，它们显示为 `/day-trader-skill:trader-scanner` 这样的名字。如果想要原来的 `/trader-scanner`，直接复制文件夹：

```bash
git clone https://github.com/Jakeschincariol/day-trader-skill.git
cp -r day-trader-skill/trader-* ~/.claude/skills/
```

只给当前项目用、不装到全局：把同样的文件夹复制到仓库的 `.claude/skills/`。完全不用 Claude Code？把任意一份 `SKILL.md` 贴在对话开头，它会按这个模式运行。这样会失去 Python 工具，但方法本身仍然有效。

工具需要 Python 3.8 或更新版本。不用 pip 安装任何包。SEC 相关工具（新闻里的申报文件、分析师）要求你表明身份：把姓名和邮箱写进 `trader/config.json`（`"sec_user_agent": "Your Name you@example.com"`），或设在环境变量 `SEC_USER_AGENT` 里。

## 十一个技能

| 命令 | 职责 | 做什么 |
| --- | --- | --- |
| `/trader-scanner` | 图表扫描 | 在成千上万张图表里找形态：开盘区间突破、旗形、超卖反弹。 |
| `/trader-news` | 新闻监控 | 跟踪你关注的每只股票的突发新闻，并标出是什么在推动它。 |
| `/trader-entry` | 进场计时 | 逐根 K 线盯着一个形态，告诉你进场的时刻。 |
| `/trader-exit` | 出场计划 | 在进场之前定好止损和目标，让你知道何时退出。 |
| `/trader-timemachine` | 时间机器 | 在承担一美元风险之前回放一笔交易的每一种可能结果：10,000 次模拟，加上这个形态过去每一次出现。 |
| `/trader-risk` | 风险管理 | 用你的限额检查每一笔交易，超出限额的一律拦住。 |
| `/trader-budget` | 预算管理 | 按你的预算和单笔最大亏损来决定仓位大小。 |
| `/trader-broker` | 订单执行 | 把已通过的订单发给券商。默认模拟交易，没有你的 yes 绝不发送。 |
| `/trader-alerts` | 价格提醒 | 价格、止损或开盘铃到达时响一次。 |
| `/trader-analyst` | 财报分析 | 阅读财报，总结这一季变了什么。 |
| `/trader-pelosi` | 佩洛西追踪 | 国会议员的股票交易披露一提交就读出来。 |

## 怎么用

所有结果都落在项目里的 `trader/` 文件夹，每个技能读取上一个技能写下的内容：

```
scanner -> news -> entry -> exit -> timemachine -> budget -> risk -> broker   （一笔交易）
alerts: stop, target, flat by the close                                       （持仓期间）
news / analyst / pelosi                                                        （按计划运行）
```

例子是演示里的那笔交易：特斯拉，2026 年 10 月 19 日星期一，真实数据。

1. `/trader-scanner`，美东时间上午 10:15：演示扫描的 231 张图表里，有 42 张收在 15 分钟开盘区间之上。TSLA 是其中之一：区间 354.47 到 362.18，10:10 这根 K 线收在 364.18，高出区间上沿 0.55%。
2. `/trader-news`：当天上午 TSLA 的标题，时间为纽约时间（"Tesla-SpaceX Merger Mania Strikes Wall Street Ahead of Earnings"，8:04，标记为财报和交易）。特斯拉本周公布财报：这笔日内交易在收盘前退出，所以不会暴露在财报上。
3. `/trader-entry` 逐根 K 线看着 TSLA。10:12 它说还在等（10:10 那根还在形成）。10:15 它说已触发：第一根收在区间之上的 5 分钟 K 线，364.18。
4. `/trader-exit` 在进场前定好计划：止损放在区间中点 358.32，每股风险 5.86；目标是这份风险的 1 倍，370.04；无论怎样，下午 3:55 前平仓。
5. `/trader-timemachine` 把当天剩余时段重放了 10,000 次。每一根 5 分钟 K 线都从 TSLA 自己过去 59 个交易日、同一时段的 K 线里抽取（种子 19，任何人都能重复）：先到目标 29.4%，先到止损 25.5%，收盘时仍未了结 45.1%。它还在这 59 个真实交易日上重跑了同一条规则：触发 33 次，13 次先到目标，15 次先到止损，5 次都没到。它的结论：接近抛硬币。在承担一美元风险之前先看一眼，意义就在这里。
6. `/trader-budget`，账户 5,000 美元，单笔最大亏损 50 美元：50 / 5.86 = 8 股（2,913.44 美元）。若止损在 358.32 成交，亏损是 46.88 美元。它写出一张括号单：买入 8 股 TSLA，止损 358.32，止盈 370.04。
7. `/trader-risk` 用你的限额检查它：单笔最大亏损、当日亏损上限、每日交易次数、盈亏比、进场时间窗口、仓位大小和现金。PASS 或 BLOCK，并给出原因。
8. `/trader-broker` 用白话把订单票给你看。你为这张确切的订单输入 yes。Claude 调用你的券商连接器（Alpaca 的 MCP 服务器，默认模拟账户），Claude 自己的 “Allow once / Deny” 卡片会再问你一次。
9. `/trader-alerts` 在止损、目标和下午 3:55 响起，用来平掉当时还开着的仓位。

## 工具

全部使用 Python 标准库。读取数据的工具访问公开接口，并且请求之间会停顿。

```bash
python3 trader-scanner/scan.py universe                                # 约 5,900 只美股，Nasdaq Trader
python3 trader-scanner/scan.py daily                                   # 盘前形态，整个股票池
python3 trader-scanner/scan.py orb --at 10:15                          # 截至此时的开盘区间突破
python3 trader-news/news.py --tickers TSLA --hours 12                  # 标题、8-K、当天涨跌
python3 trader-entry/entry.py check TSLA --setup orb                   # 等待 / 已触发 / 已过期
python3 trader-entry/entry.py watch TSLA --every 30                    # 逐根 K 线，直到触发
python3 trader-exit/exit.py plan --stop or-mid --target 1R --write     # -> trader/plan.json
python3 trader-timemachine/timemachine.py run --write                  # 10,000 次回放 + 历史
python3 trader-budget/size.py --write                                  # -> trader/orders.json
python3 trader-risk/check.py                                           # 每张订单 PASS / BLOCK
python3 trader-broker/ticket.py show                                   # 订单票，READY 或未就绪
python3 trader-alerts/alerts.py add --from-plan                        # 止损、目标、收盘前平仓
python3 trader-alerts/alerts.py check                                  # 到达的响一次
python3 trader-analyst/facts.py TSLA                                   # 各季、增长、EPS
python3 trader-pelosi/ptr.py index --member pelosi                     # 她今年的 PTR
```

```bash
python3 -m unittest discover -s tests -v
```

测试离线运行，用的是从真实数据截下来的样本：截至 2026 年 10 月 19 日的 60 天 TSLA 5 分钟 K 线、上午 10:15 的开盘区间扫描、那天上午 Yahoo 的 TSLA 标题、众议院书记官处 2026 年索引、真实的 PTR 文本和真实的 XBRL 数据。上面例子里的每一个数字，测试都会复现。

## 细则

**这不是投资建议。** 这是读取公开数据并做算术的软件。它不了解你的处境，也看不见未来。大多数做日内交易的人会亏钱。你也可能亏钱。自己做决定；需要建议时，去问持牌专业人士。

**没有你的 yes，交易不会发出。** 只有 `/trader-broker` 能通向一张订单，而且必须先由 `/trader-risk` 通过这张确切的订单，并且你在对话里为它输入 yes。在 Claude 里把券商工具设为 “Needs approval”（Customize、Connectors、工具权限），不要设成 “Always allow”，这样 Claude 自己的 “Allow once / Deny” 卡片才是第二道检查。期权、保证金、卖空或杠杆，除非你自己打开，否则不会使用。

**先用模拟账户。** `config.example.json` 里 `"paper": true` 是默认值，Alpaca 的 MCP 服务器也默认连到它的模拟账户。密钥放在环境变量里，不要放进对话。

**时间机器是模拟，不是预测。** 它用的是这只股票自己最近 60 天的 5 分钟 K 线。未来可能做出这 60 天里从未出现过的事：一份财报、一次停牌、一条标题。把它的概率当成一次现实核对，不要当成承诺。

**止损是计划，不是保证。** 在快速行情里，或跨过一个缺口时，止损单的成交价可以远差于它的价格。显示的最大亏损，是按止损价计算的亏损。

**数据免费，也不是官方的。** Yahoo Finance 的图表接口不是官方 API：它可能延迟、改版，或对你限流。扫描器和进场计时读的是 1 分钟或 5 分钟 K 线，不是每一笔成交。动手之前，把要紧的数字拿到你的券商那里核对。

**日内交易规则。** FINRA 已于 2026 年 6 月 4 日取消 25,000 美元的典型日内交易者（PDT）最低资金要求（监管通知 26-10）。券商可以把这项变更分阶段实施到 2027 年 10 月 20 日，所以你的券商可能仍在用旧规则。如果是这样，把 `"pdt_guard": true` 打开，`/trader-risk` 会计入你的日内交易次数。

**国会议员的交易是公开信息，不是内幕消息。** 议员必须在 45 天内披露超过 1,000 美元的交易（《STOCK Act》）。佩洛西技能在这些公开申报一发布就读取它们。

**税务。** 日内交易的盈利是短期收益，频繁交易很容易触发洗售。这里没有任何税务建议。

## 文件

```
trader-scanner/     SKILL.md, scan.py
trader-news/        SKILL.md, news.py
trader-entry/       SKILL.md, entry.py
trader-exit/        SKILL.md, exit.py
trader-timemachine/ SKILL.md, timemachine.py
trader-risk/        SKILL.md, check.py
trader-budget/      SKILL.md, size.py, config.example.json, targets.example.json
trader-broker/      SKILL.md, ticket.py, claude_desktop_config.example.json
trader-alerts/      SKILL.md, alerts.py, alerts.example.json
trader-analyst/     SKILL.md, facts.py, metrics.example.json
trader-pelosi/      SKILL.md, ptr.py
tests/              unittest 测试套件 + 离线样本
```

MIT。作者 [Jake Schincariol](https://opusjake.ai)。
