# web scraping ai：分清抓取层和提取层，用 credits 账本算清真实成本，选对工具少花冤枉钱

搜 web scraping ai 的人，想解决的问题通常很具体：手里有一批要喂给 LLM 或 RAG 管线的页面，要么被 Cloudflare 挡在门外，要么被“每页都过一遍模型”的账单吓到。先给一个能落地的结论——AI 抓取分两段：抓取层负责把页面完整拿到手（反爬、JS 渲染、代理轮换），提取层负责让模型读懂页面。大部分“AI 抓取工具”榜单都在讲第二段，但真正决定项目能不能跑起来、花多少钱的，往往是第一段。ScraperAPI 就是第一段的代表工具：它以 API 形式返回目标页面的 HTML，自带 JS 渲染和 CAPTCHA 处理，月付 49 美元起，注册即得 1,000 免费 credits，另有 7 天 5,000 credits 的试用，不需要信用卡。

> **选购速览**：个人项目和小团队选 Hobby（49 美元/月，10 万 credits，20 并发）基本够用；月抓百万级或常跑 Google/Amazon 这类重防护站，直接看 Business（299 美元/月，300 万 credits）。所有套餐都含 JS 渲染、反爬绕过和无限带宽，随时可取消，7 天内无理由退款。👉 [查看 ScraperAPI 全部套餐价格并领取 5,000 credits 试用](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## AI 抓取和传统爬虫到底差在哪

差在解析方式。传统爬虫按 DOM 路径取数，比如 `div.product-card > span.price`，网站改一次版就废一次，而且失效时往往静默返回空数据。AI 抓取把清洗后的页面文本交给语言模型，用一句指令或一个 JSON Schema 描述字段，模型按语义读页面——所以它能分清促销价和原价，也能在布局变化后继续工作，一个 schema 甚至可以套在几百个结构不同的站点上。

代价也很明确。每一页都要过一遍模型，中档模型每页提取成本大约 0.005 到 0.02 美元，延迟增加 1–5 秒，模型还可能用“看起来合理”的猜测填充缺失字段，输出必须按 schema 校验再抽查。所以工程上更常见的做法是混合方案：稳定主体的页面用选择器（几乎零成本、毫秒级），选择器拿不到或结构混乱的边缘页面才回退到 LLM。判断阈值可以参考：十万量级的同结构页面写选择器，两百个结构各异的站点用 LLM，目标站每季度改版就别跟选择器较劲了。

## ScraperAPI 全套餐价格表

ScraperAPI 按 credits 计费，官网当前展示免费计划加 7 个付费档位，全部列在这里。年付统一九折：

| 套餐 | 月付 | 年付（折合每月） | Credits/月 | 并发 | 购买 |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 | 5 | [ 免费注册](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [ 开通 Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [ 开通 Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [ 开通 Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [ 开通 Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [ 开通 Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [ 开通 Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | 定制 | 定制 | 22,000,000+ | 500+ | [ 咨询定制方案](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

功能上没有按档位拆分的套路：每个套餐都含 JS 渲染、高级代理轮换、JSON 自动解析、CAPTCHA 与反爬绕过（Cloudflare、Datadome、PerimeterX 级别）、自定义 Session、自动重试、无限带宽和 99.9% 在线率保障。有两个差异藏在表格外：Geotargeting 在 Hobby 和 Startup 只能定位美国和欧洲，Business 起才支持国家级定位；分析面板数据低档位只保留 30 天，Business 起不限。年付的差价算下来，Business 一档一年省 358.80 美元，Hobby 一年省 58.80 美元。

## Credits 不等于请求数：一笔大多数文章没算的账

这是选 ScraperAPI 之前最值得搞清楚的一点，也是各家评测最常含糊过去的地方。官方对“一次请求扣几个 credit”有明确定价：普通页面 1 credit，Amazon 页 5 credits，Google 和 Bing 搜索（含子域名）25 credits，LinkedIn 30 credits，触发 Cloudflare、Datadome、PerimeterX 这类防护绕过时每请求再加 10 credits。

把这笔账换算到 Hobby 的 10 万 credits，差距就很直观了：

- 只抓普通网页：约 100,000 页，单页约 0.00049 美元
- 全抓 Amazon：约 20,000 页
- 全跑 Google 搜索：约 4,000 次，单次约 0.012 美元
- 全部叠加重防护绕过：再打对折都不止

也就是说，“10 万 credits”这个数字本身没什么意义，你的目标站点构成才是决定因素。好在官方在 dashboard 里提供了 Domain Cost Estimator，输入任意 URL 就能查出实际扣几个 credit，还能给单次抓取设 `max_cost` 上限防止跑飞。买之前先拿自己的目标站查一遍，比看十篇测评都有用。👉 [用免费试用实测你的目标站扣费标准](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## 它在 AI 抓取管线里的位置，以及两段成本怎么省

把边界说清楚：ScraperAPI 返回的是原始 HTML，字段提取要自己接模型。这个“分工”其实有成本上的好处——一体化 AI 抓取工具每页都收提取费，而你自建管线时，喂给模型的内容可以自己裁剪。

具体省法有几个，都有依据：

1. **剥离噪音再喂模型**。把 HTML 里的 script、style、nav 等标签剥掉再送入模型，token 能减少 70% 以上；LangChain 里用 `SoupStrainer` 只保留目标区块效果类似。
2. **截断控制**。超长页面截取前 8000 字符再连同 schema 发出，要求模型只返回 JSON，用 Pydantic 校验字段类型，解析失败就 re-prompt。
3. **混合管线**。抓取层统一走 ScraperAPI（JS 渲染、IP 轮换在服务端完成），提取层对大头页面用选择器，LLM 只处理脏数据。

顺带提一个实操陷阱：LangChain 的 `WebBaseLoader` 默认 User-Agent 标识是 Python，不少站点会直接拒绝，需要手动设真实 UA；而且它不执行 JavaScript，遇到 JS 渲染的内容会漏数据——这正是需要先把页面交给带渲染能力的抓取 API 的原因。

## 和同类工具的横向对比

把搜索结果里常被一起讨论的几款放在一起（信息仅供决策参考，购买入口只在 ScraperAPI）：

| 工具 | 起步价 | 计费特点 | 适合场景 |
| --- | --- | --- | --- |
| ScraperAPI | $49/月 | 按 credits，普通页 1 credit，搜索引擎页 25 | 重防护站点、电商与搜索页、规模化抓取 |
| Firecrawl | $16/月 | 一次调用返回 Markdown 或按 schema 提取 JSON，JSON 模式每页加 4 credits | LLM/RAG 数据供给，轻量提取 |
| Crawl4AI | 免费开源 | Apache 2.0，本地运行，无 API key，输出干净 Markdown | 不设防站点、自托管 |
| ScrapingBee | $49/月 | stealth 代理+渲染组合单次消耗 75 credits，约 14.70 美元/千页 | 中等防护的 JS 站点 |
| ZenRows | $16/月 | 反爬绕过为主，另有 AI 提取功能 | 防护站点、RAG 管线 |
| Bright Data | $499/产品/月 | 每个产品单独承诺最低消费，Web Unlocker 与 Scraping Browser 合用约 998 美元/月 | 企业级、最难抓的目标 |

两个细节值得展开。一是表面同价的差距：ScrapingBee 和 ScraperAPI 都从 49 美元起，但前者隐身模式叠渲染后的千页成本约 14.70 美元，后者标准组合低一个数量级——轻量场景下这个差距比起价看起来大得多。二是不花钱的选项确实存在：Crawl4AI 本地跑就能解决不设防静态站，Jina Reader 免费额度够低量试用。ScraperAPI 的价值集中在“页面拿不到”这个问题上，页面好拿的项目没必要为它付费。

第三方对各家在真实防护站点上的成功率做过基准测试，但不同评测的口径和结论差异很大，参考价值有限。免费计划每月 1,000 credits 长期可用，用自己的目标站实测比任何数字都可靠。

## 下单前该知道的几条规则

这些内容写在官方文档和 FAQ 里，但直接影响实际花费：

- **超并发直接拒绝，不排队**。超出套餐并发上限的请求返回 429，不会等待，也不收费。任务调度时把并发控制在限额内，免得白丢请求。
- **credits 用完默认暂停**。额度耗尽后抓取停到下个计费周期，不会自动扣钱。想继续有两 条路：Hobby 到 Professional 可以开 auto-upgrade（自动升到下一档，单 credit 价格通常更优，需要主动开启）；Scaling 及以上内置 pay-as-you-go，按固定费率继续抓，可设月度消费上限。
- **取消随时可以，不收费**。在 dashboard 的 Billing 页面自助操作即可。
- **7 天无理由退款**。通过联系支持办理。
- **按量计费，不按带宽**。所有套餐按请求数计费，不提供单独购买代理 IP 的选项。

这最后两条对犹豫的人挺重要：试用不需要信用卡，付费后一周内反悔也能全额退。👉 [开通 ScraperAPI 并在 7 天内免费验证](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## 常见问题

**ScraperAPI 能直接输出结构化数据吗？** 部分能。所有套餐带 JSON 自动解析，付费功能还有 Parsing & Structured Data APIs、DataPipeline 和完整爬虫入口，但字段级的 LLM 语义提取仍需自己接模型完成。

**免费试用需要信用卡吗？** 不需要。注册即有 1,000 credits 的长期免费计划和前 7 天共 5,000 次免费请求；测试不够用还可以联系支持申请追加。

**月付和年付差多少？** 年付统一九折：Hobby 每月 49 美元变 44.10 美元，Business 每月 299 美元变 269.10 美元，其余档位同比例。

**抓 Google 和抓普通网页一个价吗？** 差很多。普通页 1 credit，Google/Bing 25 credits，Amazon 5 credits，LinkedIn 30 credits，高级反爬绕过每请求加 10 credits。先在 dashboard 用 Domain Cost Estimator 查自己的目标站。

**AI 抓取的合规红线在哪里？** 用 LLM 解析页面不改变法律图景，抓取的合规要求与普通抓取相同：公开可访问不等于没有版权和隐私约束，优先抓公开非个人数据、遵守 robots.txt 和网站条款、不绕过登录和访问控制、控制请求频率。

## 按你的情况对号入座

- 目标站不设防、静态页、给本地模型用：Crawl4AI 免费解决，一分钱不用花。
- 想让 LLM 直接读页面、月量几百到几千页：Firecrawl 这类按页计费的一体化工具更省事，起价也更低。
- 目标站有 Cloudflare 级防护、要抓 Amazon/Google/LinkedIn、或者量大到自建代理池不划算：这是 ScraperAPI 的主场，先用免费 credits 实测成功率和扣费，再按目标站构成选档——单人项目选 Hobby，月量百万级或常跑搜索引擎选 Business 及以上。

下单前就确认两件事：用 Domain Cost Estimator 查清目标站的单次扣费，然后估算月 credits 消耗别低于套餐量的一半，否则年付的九折不如先月付观察一个周期。👉 [领取 5,000 credits 试用，实测后再决定套餐](https://www.scraperapi.com/pricing/?fp_ref=coupons)
