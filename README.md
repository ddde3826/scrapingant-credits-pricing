# scrapingant api：credits 计费规则与每千次请求真实成本一次讲清，附已核验的替代方案选购建议

搜 scrapingant api 的人，大多是两件事：想知道这个 API 怎么计费、到底要花多少钱。先把答案放在最前面：ScrapingAnt 的 API 用 credits 计费，默认请求（Chrome 渲染 + 标准代理）一次扣 10 credits，官方给的区间是 1–25 credits；免费计划每月给 10,000 credits，不需要绑卡。但「10,000 credits」能跑多少次请求，取决于你开什么参数——这正是大多数介绍文章没讲清楚的地方，本文会把这笔账算开。

顺带说明：文中所有可点击的购买链接都指向另一家已核验的抓取 API 服务 ScraperAPI。ScrapingAnt 的信息只做事实呈现，方便你对照选型。

## 选购速览

> 如果你主要抓普通网页、想要每月免费额度跑起来再说：ScrapingAnt 的免费层（每月 10,000 credits，无需信用卡）和 $19 的 Enthusiast 档是它家最低门槛。如果你要高频抓 Google、Bing、Amazon 这类站点，或者需要更大的并发和按量付费的灵活性，可以看看下面已核验过全套餐价格的 ScraperAPI——新用户有 7 天免费试用（含 5,000 credits，同样不用信用卡）：👉 [领取 7 天免费试用，5,000 credits 到账](https://www.scraperapi.com/signup?fp_ref=coupons)

## ScrapingAnt API 是什么，能做什么

它是一个网页抓取 API：你发请求，它负责代理轮换、CAPTCHA 规避、反爬指纹这些脏活，返回页面内容。单次接入只需要两个东西——目标 URL 和 API key，都通过 query parameters 传给 `v2/general` 端点，支持 GET/POST/PUT/DELETE 四种方法。timeout 参数可以设 5 到 60 秒，官方文档的建议是：对成功率要求不高、想更快拿响应的场景用低 timeout。

功能上有几个点值得单独说：

- **JS 渲染**：每个请求跑一个全新的隔离 Chrome 实例（最新 Chromium），SPA、懒加载、无限滚动都能处理，也支持自定义 `js_snippet`、wait-for-selector、自定义 cookies 和 headers。
- **并发无上限**：官方页面写明所有套餐均无并发上限，可以一次并行发 1,000 个请求，不需要管理队列或 worker。这是它和不少按并发线程限流的同类产品差异最大的地方。
- **AI 提取与 Markdown**：`/extract` 端点用自然语言描述需求就能返回结构化 JSON，不用写 CSS 选择器；`/markdown` 端点直接输出 LLM-ready 的 Markdown；`/extended` 返回带 cookies、headers、XHRs 的扩展 JSON。
- **MCP server**：一条安装命令就能在 Claude Code、Cursor、Windsurf 里挂上三个工具（抓 HTML、抓 Markdown、抓纯文本），agent 调用消耗的也是同一池 API credits。

另外它还卖独立的住宅+数据中心代理池，覆盖 100+ 国家，按 GB 计费、与 API credits 分开结算——如果你只想买代理不想用 API，这是另一条产品线。

## ScrapingAnt 的 credits 计费规则：默认请求一次 10 credits

计费单位是 API credits，每个订阅周期给一个固定池子。官方口径是「多数请求消耗 1–25 credits，取决于你开启的选项（代理类型、JS 渲染等）」，而默认请求——Chrome 渲染加标准代理——一次扣 10 credits。完整费用明细在它的官方 docs 里，每次响应还会带一个 `Ant-credits-cost` header，告诉你这次实际扣了多少。

查余量有两种方式：直接看 dashboard，或者调用量接口 `v2/usage`，它会返回套餐名、订阅周期、总 credits 和剩余 credits。请求缺 API key 返回 HTTP 422，key 无效返回 403。

有一个规则对成本影响很大：**失败请求不计费**。官方原话是 "Pay only for successful scrapes — failed requests cost 0"。抓取成功率本来就有波动，这条等于给预算上了道保险。

 credits 用尽之后，月中可以即时重启计划，付款后 credits 立刻刷新，不用干等下一个账期。

## ScrapingAnt 价格：每月免费 10,000 credits，五个付费档

以下五档均为官网当前展示的月付价，币种为美元：

| 套餐 | credits/月 | 价格 | 支持与服务差异 |
| --- | --- | --- | --- |
| Enthusiast | 100,000 | $19/月 | 仅邮件支持，集成帮助只有文档 |
| Startup（官方标 Most Popular） | 500,000 | $49/月 | 优先邮件支持、自定义代码片段、专家协助 |
| Business | 3,000,000 | $249/月 | 调试会话，含自定义代理池与自定义反爬规避 |
| Business Pro | 8,000,000 | $599/月 | 优先调试会话、专属客户经理 |
| Custom | 10M+ | $699+/月 | 完整企业 onboarding、专属支持渠道 |

免费计划独立于这五档存在：每月 10,000 credits，循环发放，注册不要求信用卡，升级时才需要填支付信息。

几个选档时容易被忽略的差异：Enthusiast 和 Startup 都不含自定义代理池和自定义反爬规避，这两项从 Business 档才开始有；专属客户经理要到 Business Pro。如果你只是个人项目跑跑数据，$19 的 Enthusiast 加免费层大概率就够了，往上多花的钱主要买的是支持服务而非额度。

## 每千次请求的真实成本：两家口径不一样，别只比标价

这是网上多数对比文章没算清楚的一笔账。同样写着「100,000 credits」，在不同平台能跑的请求数可能差好几倍，因为每家对「一次请求值多少 credits」的定义不同。

先看 ScrapingAnt：默认请求 10 credits。Enthusiast 档 $19 买 100,000 credits，按默认请求算就是 1 万次，折合 **每千次默认请求约 $1.9**。它自己的宣传页给出的口径是「每 1,000 requests 起价 $0.19」，对应的是消耗最少credits 的轻量请求场景——这个数字来自官方宣传，实际取决于你的参数组合。

再看 ScraperAPI 的口径，差别就出来了：它按目标站点定价，普通页面一次只要 1 credit，但 Amazon 是 5，Google 和 Bing（含子域名）是 25，LinkedIn 是 30；遇到 Cloudflare、Datadome、PerimeterX 这类防护站点，每次绕过加 10 credits。Hobby 档 $49 买 100,000 credits，抓普通页面折合 **每千次 $0.49**——比 ScrapingAnt 默认请求便宜很多；但反过来，抓 Google 每千次要 25,000 credits，折合约 **$12.25**，单次成本反而更高。

结论很直白：你要抓的站点类型，决定了哪家便宜。抓普通网页和低防护站点，ScraperAPI 的 1 credit 口径很占便宜；抓需要渲染的 SPA、且想要失败不扣费和无并发上限，ScrapingAnt 的固定 10 credits 口径更可预期。想精确到自己的目标 URL，两家都有工具：ScrapingAnt 有响应头和用量接口，ScraperAPI 在 dashboard 里有 Domain Cost Estimator，还能设 `max_cost` 给单次请求封顶。

## 想直接注册试用：ScraperAPI 全套餐与价格

ScraperAPI 是本文所有购买链接指向的服务，官方价格页当前展示七个档位，年付统一打九折。以下是月付价与年付折后月均价（美元）：

| 套餐 | 月付（年付月均） | credits/月 | 并发线程 | 关键差异 | 购买 |
| --- | --- | --- | --- | --- | --- |
| Hobby | $49（$44.10） | 100,000 | 20 | Geotargeting 仅限 US & EU | [ 开通 Hobby 并开始试用](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Startup | $149（$134.10） | 1,000,000 | 50 | Geotargeting 仅限 US & EU | [ 开通 Startup 并开始试用](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Business | $299（$269.10） | 3,000,000 | 100 | 国家级 Geotargeting | [ 开通 Business 并开始试用](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Scaling（官方标 Most popular） | $475（$427.50） | 5,000,000 | 200 | 国家级 Geotargeting，支持按量付费 | [ 开通 Scaling 并开始试用](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Professional | $975（$877.50） | 10,500,000 | 300 | 国家级 Geotargeting | [ 开通 Professional 并开始试用](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Advanced | $1,975（$1,777.50） | 21,500,000 | 500 | 国家级 Geotargeting | [ 开通 Advanced 并开始试用](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Enterprise | 定制价 | 22,000,000+ | 500+ | 定制方案，需联系销售 | [ 咨询 Enterprise 定制方案](https://www.scraperapi.com/?fp_ref=coupons) |

所有付费档都包含 JS 渲染、高级住宅与移动 IP、高级反爬绕过、解析与结构化数据 API、DataPipeline、按量付费和 Slack 优先支持。免费计划注册即得 1,000 credits（最多 5 个并发连接），需要更多测试 credits 可以联系支持申请。年付档位比月付每月省 10%，以 Hobby 为例一年省 $58.80。

如果你拿不准额度够不够，先用免费试用跑真实目标站点是最稳的验证方式，量出来再选档：👉 [用 5,000 免费 credits 先测你的目标站点](https://www.scraperapi.com/signup?fp_ref=coupons)

credits 用尽后的处理也值得留意：默认会暂停抓取直到下个周期，Hobby 到 Professional 五档可以开启 auto-upgrade（额度耗尽自动升到下一档，需要主动打开）；Scaling 及以上还能用按量付费继续跑，按固定每 credit 单价计费，上限由你自己设。超并发时请求会返回 429 直接被拒，不排队也不额外收费。

## 怎么选：按你的抓取对象定

- **主要抓 Google SERP、LinkedIn 这类高扣费站点**：ScraperAPI 虽然单次 25–30 credits，但它的定价体系就是为这些站点设计的（有专门的 Google Search、Amazon、Walmart 结构化端点，直接返回 JSON），成功率和解析质量有专门保障。年付 Hobby 档 $44.10/月起步。
- **抓需要 JS 渲染的 SPA、电商商品页、且请求量大但目标多样**：ScrapingAnt 固定 10 credits 的默认请求更好预算，失败不计费加上无并发上限，适合大批量并发跑列表页。$19 起步的门槛也更低。
- **想给 AI agent 接网页能力**：ScrapingAnt 的 MCP server 集成路径最短，一条命令挂三个工具；ScraperAPI 这边也有结构化数据端点可以直接喂 agent。
- **只是个人小项目、每月几千次请求**：两家都有免费层，ScrapingAnt 每月 10,000 credits（无需信用卡），ScraperAPI 注册送 1,000 credits + 7 天试用 5,000 credits，先都跑一下再决定，一分钱不用花。

## 常见问题

**ScrapingAnt 免费计划要绑信用卡吗？** 不要。官方 FAQ 明确写免费计划每月 10,000 credits、无需信用卡，只有升级付费时才填支付信息。

**抓失败了会扣 credits 吗？** ScrapingAnt 明确失败请求成本为 0，只对成功抓取计费。响应里的 `Ant-credits-cost` header 可以核对每次实际扣费。

**并发有限制吗？** 两家规则不同。ScrapingAnt 所有套餐无并发上限，官方示例是一次并行 1,000 个请求；ScraperAPI 按档位给并发线程，从 Hobby 的 20 到 Advanced 的 500，超出返回 429。

**ScrapingAnt 有年付折扣吗？** 官网当前展示的是月付价，页面没有公示年付折扣信息，下单前以其结账页实际显示为准。

**两家有退款政策吗？** 本次调研的官方页面上没有公示退款条款，这里不下结论。介意这一点的读者，建议先用两家的免费额度完成验证再付费，把退款风险降到零。

## 下单前最后两件事

第一，用你自己的目标 URL 算一遍账，别用宣传页的示例数字：ScrapingAnt 看 `Ant-credits-cost` 响应头，ScraperAPI 用 dashboard 里的 Domain Cost Estimator，各自跑一次就知道你的场景每千次到底多少钱。第二，免费额度先跑满——两家的免费层都不用信用卡，成本为零的验证机会没理由跳过。

如果你的目标站点偏 Google、Amazon、LinkedIn 这类有独立定价的结构化端点，或者预算已经到了千元美元级、需要专属支持和 Slack 通道，ScraperAPI 的档位覆盖更宽：👉 [查看 ScraperAPI 全部套餐与当前价格](https://scraperapi.com/pricing?fp_ref=coupons)
