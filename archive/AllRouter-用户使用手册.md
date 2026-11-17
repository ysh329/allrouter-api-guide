# All Router（allrouter.cc）用户使用手册

版本：v1.0（面向终端用户）
编写日期：2026-11-16
信息来源：对 allrouter.cc 线上站点、`/api/status`、`/api/about`、`/api/user/groups`、`/api/ratio_config`、`/v1/*` 接口与《用户服务协议》的实际抓取与实测

---

## 一、这个产品到底是什么

All Router 是由 TECHFRONTIER PTE. LTD.（新加坡，UEN 202512973K，注册地址 36 Robinson Road #20-01, City House）运营的 AI API 聚合与转发平台。它对外提供两样东西：

1. 一个统一的 API 入口（`https://allrouter.cc/v1`）。用一个账号、一把 API Key、一套 OpenAI 兼容的接口格式，就能调用多家模型厂商的模型，并在同一个控制台里做计量、计费与密钥管理。
2. 七个"AI 生产工具"：Deck（演示文稿）、Counsel（法务起草）、Code（编码助手）、Digest（文档与会议摘要）、Lingua（翻译）、Insight（数据分析）、Copy（营销文案）。官网明确写着：这些工具当前统一通过 API 提供，可视化应用正在逐个上线。

理解这个产品有三条关键认知：

- 平台自己不做模型。按《用户服务协议》1.2 条，平台"本身不开发或运营任何大语言模型"，只做协议适配、统一接口、计量计费与结算；模型的推理能力、输出质量和可用性由上游供应商决定。所以你买到的本质是"通道 + 额度 + 统一账单"，而不是某一家模型的原厂账号。
- 平台可以随时调整模型清单。协议 1.3 条保留增删、调整、下线模型与接口的权利，重大变更通过站内公告通知。
- 计费是预付制。协议 4.2 条写明采用"预付费充值扣费"模式，按公开价格对每次调用计量，从余额或套餐额度中扣除，用量以后台记录为准。

---

## 二、先搞清楚计费单位：credits

全站统一使用自定义货币单位 credits 计价，规则非常直白：

| 项目 | 数值 |
|---|---|
| 兑换关系 | 1 美元 = 10 credits（即 1 credit ≈ 0.1 美元） |
| 余额展示 | 控制台以 credits 显示余额与扣费 |
| 结算货币 | 美元（USD） |

credits 有两种来源，二者不能混用：

1. 充值余额（Top-up credits）：你充多少用多少，长期有效，可跨工具、跨模型使用。
2. 套餐额度（Plan allowance）：只能在该套餐有效期内、且只能用于该套餐对应的工具；官网明确说明"套餐额度永远不会并入充值余额"。

计费口径分两类：

- 文本类模型：按 token 计费，输入（prompt）、输出（completion）、缓存命中（cache read / cache write）分别按不同倍率折算，输出通常远贵于输入（例如部分模型输出倍率为 4 倍左右，缓存命中倍率可低至 0.02~0.2）。
- 图像、音频、视频等按次计费的模型：直接按每次调用固定价格扣费（例如部分图像模型 0.03~0.04 美元/次）。

官网列出的七个工具典型成本（标准费率下的估算值，实际按真实用量结算）：

| 工具 | 场景 | 典型单价 |
|---|---|---|
| Copy 营销文案 | 广告、社媒、邮件 | ≈ 0.1 credits / 篇（≤800 字符） |
| Counsel 法务起草 | 合同、催告函、合规备忘 | ≈ 0.2 credits / 1,000 字符 |
| Lingua 翻译 | 产品文档、商品页、字幕 | ≈ 0.3 credits / 1,000 字符 |
| Deck 演示文稿 | 路演、培训、周报 | ≈ 0.5 credits / 12 页 |
| Digest 摘要 | 会议纪要、研究笔记 | ≈ 0.6 credits / 10,000 字符原文 |
| Code 编码助手 | 功能开发、重构、写测试 | ≈ 1.2 credits / 次请求 |
| Insight 数据分析 | 销售复盘、问卷分析 | ≈ 2 credits / 次分析（≤500 行） |

记住一句话：官网数字是"典型估算"，真实账单以控制台每次请求的明细为准。

---

## 三、五分钟跑通：最短路径

如果你只想尽快发出第一个请求，按下面五步走：

1. 注册账号：打开 https://allrouter.cc/sign-up ，用邮箱 + 用户名 + 密码注册，收邮箱验证码完成验证。
2. 拿额度：登录后进钱包页（/wallet）充值，或购买套餐（/plans）。
3. 建密钥：进"令牌 / API 密钥"页（/keys）新建一把 Key，复制 `sk-` 开头的字符串（只显示一次）。
4. 选模型：在控制台"模型广场"（/models）里挑一个模型 ID，并确认它属于你这把 Key 绑定的分组。
5. 发请求：把 base_url 设为 `https://allrouter.cc/v1`，api_key 设为刚才的 Key，发一次 chat/completions。

---

## 四、注册与登录

### 4.1 邮箱注册

注册页需要填四项：用户名、密码、确认密码、邮箱；邮箱旁有"发送验证码"，说明该站开启了邮箱验证（`email_verification` 为 true），未验证无法完成注册。页面还挂有人机校验（Turnstile），网络环境异常时可能卡在验证环节，换网络或关闭代理重试。

注册前必须勾选同意《服务条款》与《隐私政策》。协议 2.1 条要求使用者年满 18 周岁。

### 4.2 第三方登录

支持两种 OAuth：GitHub 与 LinuxDO（Linux.do 社区账号，站点配置了最低信任等级 0，即新号也可登录）。第三方登录同样要受"一人一号、不得多开规避风控"约束（协议 3.4 条）。

### 4.3 登录后的落点

登录成功后进入控制台（/dashboard）。未登录访问控制台任意页面都会跳回 `/sign-in?redirect=...`，登录后自动回到原页面。

### 4.4 找回密码

登录页有"Forgot password?"入口，走邮箱重置流程（/forgot-password → /reset）。若账号已开启两步验证，重置流程会额外要求验证。

---

## 五、充值与套餐

### 5.1 充值（Top-up）

入口：控制台钱包页，或首页 "Top up credits" 按钮。充值必须登录后进行，按美元结算，支付由第三方支付服务商处理，平台不保存完整卡号。

到账时间：支付服务商确认后自动入账，通常几分钟内。若付款成功但 24 小时内未到账，按协议 4.5 条联系 support@allrouter.cc。

充值页可选的具体支付渠道随站点配置变化，以你登录后看到的选项为准；不要相信站外任何人提供的"代充""折扣充值"。

### 5.2 套餐（Plans）

站点提供两族套餐，都是一次性购买、有效期 30 天、不自动续费、不自动扣款：

文本工具族（可用 Deck / Counsel / Code / Digest / Lingua / Insight / Copy，底层为 Codex 系列模型）：

| 档位 | 面向人群 | 价格 | 额度 | 每周上限 |
|---|---|---|---|---|
| Starter | 日常探索 AI 工具的个人 | $28 / 30 天 | 基础额度 | 额度的 1/4 |
| Studio（推荐） | 有稳定产出的自由职业者与创作者 | $56 / 30 天 | 基础的 2 倍 | 额度的 1/4 |
| Team | 把 AI 嵌入产品的小团队 | $112 / 30 天 | 基础的 4 倍 | 额度的 1/4 |

编码工具族（面向 Code 工具，从受支持的命令行编码客户端使用，底层为 Claude 系列模型）：

| 档位 | 面向人群 | 价格 | 月度额度 |
|---|---|---|---|
| Lite | 每周写几天代码的开发者 | $41 / 30 天 | 1,960 credits |
| Standard（推荐） | 每天都写代码的开发者 | $82 / 30 天 | 3,960 credits |
| Advanced | 在大型代码库上长时间会话 | $164 / 30 天 | 8,060 credits |

三个必须注意的点：

- 每周上限是硬约束。文本族套餐把额度切成"周窗口"，单周最多用掉总额度的 1/4，额度没用完但周窗口打满也会被拦。
- 套餐额度与充值余额是两套账。套餐额度只在有效期内、只对套餐覆盖的工具生效，过期不折算成余额。
- 买套餐前先确认你用的是哪类客户端。编码族明确写了"从受支持的命令行编码客户端使用"，如果你只用网页/自建应用调 API，买文本族或直接充值更合适。

### 5.3 退款与对账

按协议 4.6~4.11 条：

- 数字服务即时交付，交付后原则上不退款，未用完的余额与套餐额度也不退。
- 只有四类情形可退：重复扣款、付款成功但 3 个工作日内未交付、非因你原因导致服务终止（退未消费的付费余额，扣必要成本）、法律强制要求。
- 申请方式：付款后 30 天内发邮件到 support@allrouter.cc，写明账号邮箱或用户 ID、订单号、支付日期与金额、退款原因及凭证；3 个工作日内确认受理，退款原路原币退回，通常 5~10 个工作日到账。
- 对账单有异议必须在费用发生后 10 天内提出并举证，逾期视为认可。

---

## 六、创建与管理 API Key

### 6.1 创建

路径：控制台 → 令牌 / API 密钥（/keys）→ 新建。站点 FAQ 的原话是"登录控制台 → 令牌页面 → 添加令牌，创建后复制 sk- 开头的 Key（仅显示一次）"。

创建时需要理解并设置这几项：

| 字段 | 作用 | 建议 |
|---|---|---|
| 名称 | 便于区分用途 | 按项目/环境命名，如 `prod-web`、`dev-local` |
| 额度上限 | 这把 Key 最多能花多少 credits | 生产 Key 设上限，避免单 Key 被刷爆全账户 |
| 过期时间 | 到期自动失效 | 临时测试给 1~7 天，长期服务给较长或不过期 |
| 分组 | 决定能调哪些模型、按什么倍率计价 | 见下一节，最容易被忽略也最影响账单 |
| 模型限制 | 限定这把 Key 只能调哪些模型 ID | 生产环境建议收窄到白名单 |
| IP / 访问限制 | 限定来源 IP | 有固定出口 IP 的服务端强烈建议开启 |

创建成功后弹窗会显示完整 Key，之后列表只显示掩码。丢了只能重建——重建会使旧 Key 立即失效，脚本需要同步更新。

### 6.2 保管

- 不要把 Key 写进前端代码、移动端包体、公开仓库、浏览器插件配置。
- 怀疑泄露：立刻在控制台重置或删除该 Key，再去"使用日志"核对异常调用。
- 协议 3.1 条明确：因你保管不当导致 Key 泄露、被盗刷，费用由你承担。

---

## 七、模型与分组：决定"能调什么"和"多贵"

### 7.1 模型规模

抓取站点公开的模型倍率配置，当前可用模型 ID 约 563 个，覆盖 Claude、GPT（含 GPT-4/5 系列与 Codex 变体）、Gemini、Qwen、DeepSeek、GLM、Grok、Kimi、Mistral、Llama、文心、星火、混元、MiniMax、阶跃、百川、Yi、Midjourney、DALL·E、FLUX、Imagen、Veo、Sora、Suno、Kling、TTS、Whisper、Embedding 等数十个系列。

模型 ID 就是请求体里的 `model` 字段，必须与模型广场里显示的字符串完全一致（大小写、后缀如 `-thinking`、`[1m]`、`-openai-compact` 都算在内）。

### 7.2 分组（group）

同一把 Key 绑定一个分组，分组决定三件事：可用模型集合、计价倍率、走哪条上游通道。实测站点公开的分组与倍率如下：

| 分组 | 说明 | 倍率 |
|---|---|---|
| gpt | 自营 codex 号池 | 0.053 |
| grok | 自营 grok 号池 | 0.045 |
| kimi 号池 | 自营 kimi 号池 | 0.06 |
| glm | GLM 系列 | 0.3 |
| claude-max | Claude Max 反代 | 0.25 |
| deepseek | DeepSeek 系列 | 0.4 |
| gemini | Gemini 官方 API | 0.45 |
| gpt-azure-api | 微软 Azure GPT API | 0.6 |
| claude-official-api | Claude 官方 API | 0.65 |
| default | 官方 API | 0.75 |
| claude-fable-api | Fable 5 官方 API | 1.3 |

倍率是价格乘数：倍率越低越便宜，但"号池/反代"类通道的稳定性、限流强度、合规性与官方 API 不同，生产环境要权衡。经验规则：

- 要稳定、要 SLA、要合规：选 `*-official-api`、`default`、`gpt-azure-api`。
- 要压成本、能容忍偶发 429 或模型替换：选号池类分组，并做好重试与降级。
- 别在代码里写死分组假设：先调 `/v1/models` 或看模型广场，确认该分组下模型 ID 存在。

---

## 八、发起第一个请求

### 8.1 基本参数

| 项 | 值 |
|---|---|
| Base URL | `https://allrouter.cc/v1` |
| 鉴权 | `Authorization: Bearer sk-xxxx` |
| 兼容格式 | OpenAI Chat Completions、OpenAI Responses、Anthropic Messages、Gemini v1beta、Embeddings、Images |

已实测存在的端点：`/v1/chat/completions`、`/v1/messages`、`/v1/responses`、`/v1/embeddings`、`/v1/images/generations`、`/v1/models`、`/v1beta/models`。未携带合法 Key 时统一返回 401 `Invalid token (request id: ...)`，这个 request id 在排查问题时非常有用，报障时请一并提供。

### 8.2 cURL（最小可用示例）

```bash
curl https://allrouter.cc/v1/chat/completions \
  -H "Authorization: Bearer $ALLROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4.1",
    "messages": [
      {"role": "user", "content": "用三句话解释什么是 API 网关"}
    ]
  }'
```

### 8.3 Python（OpenAI SDK）

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-你的密钥",
    base_url="https://allrouter.cc/v1",
)

resp = client.chat.completions.create(
    model="gpt-4.1",
    messages=[{"role": "user", "content": "把这段话压缩成 50 字以内的摘要：……"}],
    temperature=0.3,
)
print(resp.choices[0].message.content)
print(resp.usage)   # 用于核对计费
```

### 8.4 Node.js（OpenAI SDK）

```javascript
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.ALLROUTER_API_KEY,
  baseURL: "https://allrouter.cc/v1",
});

const r = await client.chat.completions.create({
  model: "claude-sonnet-5-5",
  messages: [{ role: "user", content: "写一个 Python 快排，并解释时间复杂度" }],
});
console.log(r.choices[0].message.content);
```

### 8.5 Anthropic 原生格式

```bash
curl https://allrouter.cc/v1/messages \
  -H "x-api-key: $ALLROUTER_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-5-5",
    "max_tokens": 512,
    "messages": [{"role": "user", "content": "列出三个常见的限流退避策略"}]
  }'
```

### 8.6 Gemini 原生格式

```bash
curl "https://allrouter.cc/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $ALLROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"contents":[{"parts":[{"text":"解释一下向量数据库的 HNSW 索引"}]}]}'
```

### 8.7 编码类客户端（Codex / Claude Code 类）

站点同时提供 `/v1/responses`，这是 Codex 类命令行客户端的常用接口。把客户端里的 `OPENAI_BASE_URL` 指向 `https://allrouter.cc/v1`、`OPENAI_API_KEY` 指向你的 Key，模型填控制台里可用的编码模型 ID 即可。若客户端支持 Anthropic 协议，则指向同一域名并走 `/v1/messages`。

---

## 九、七个工具怎么用

官网对"工具"的定位是：把一句清楚的需求变成可直接交付的成品（成品级输出，而不是演示）。当前所有工具都跑在同一套 chat/completions 上——你选引擎、给出工具指令与输出格式，拿回结果。官网给出的 Deck 示例可以直接复制使用：

```bash
curl 'https://allrouter.cc/v1/chat/completions' \
  -H "Authorization: Bearer $ALLROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "ENGINE_ID",
    "messages": [
      {"role": "system", "content": "AllRouter Deck. Return JSON: [{ title, bullets, notes }]"},
      {"role": "user", "content": "12-slide quarterly review for a retail team"}
    ]
  }'
```

`ENGINE_ID` 必须换成你账户可用的引擎 ID（在控制台模型广场查）。逐工具说明：

### 9.1 Deck（演示文稿）

输入：一句话需求，例如"面向零售团队的 12 页季度复盘"。编排逻辑：先出大纲，再逐页生成正文与演讲备注。输出：结构化幻灯片（JSON 或 Markdown），可直接灌进你自己的模板。成本 ≈ 0.5 credits / 12 页。适用：路演、培训、周报。

### 9.2 Counsel（法务起草）

输入：用平实语言陈述事实，例如"自由设计师合同，3 个里程碑，尾款付清时知识产权转移"。编排逻辑：把事实映射到条款结构，并把待确认问题单独列出来。输出：条款化草稿 + 待审问题清单。成本 ≈ 0.2 credits / 1,000 字符。官网明确提示：草稿仅供参考、不构成法律意见，使用前须由执业律师审阅。

### 9.3 Code（编码助手）

输入：描述要做的改动，例如"给订单接口加分页并补测试"。编排逻辑：基于你提供的代码做变更规划，返回可直接应用的补丁与说明。输出：unified diff + 解释。成本 ≈ 1.2 credits / 次。可与标准编码工具配合使用。

### 9.4 Digest（文档与会议摘要）

输入：报告、转录稿或研究材料，例如 40 分钟会议转录（约 10,000 字符）。输出：摘要 + 决议 + 负责人 + 截止时间。成本 ≈ 0.6 credits / 10,000 字符原文。

### 9.5 Lingua（翻译）

输入：原文 + 术语表。输出：保留语气、术语与排版结构的目标语言版本。成本 ≈ 0.3 credits / 1,000 字符。官网示例给了中英日三语对照，强调"结构不变"。

### 9.6 Insight（数据分析）

输入：CSV（≤500 行）+ 自然语言问题，例如"哪些区域下滑，为什么"。输出：关键发现 + 汇总表 + 图表规格（JSON）。成本 ≈ 2 credits / 次。注意它是"给你结论和图表规格"，不是替你出图。

### 9.7 Copy（营销文案）

输入：产品事实 + 品牌调性 + 渠道。输出：3 个变体，含标题与 CTA。成本 ≈ 0.1 credits / 篇（≤800 字符）。

使用这七个工具时的通用建议：把"输出格式"写死在 system 里（JSON schema、Markdown 结构、diff 格式），否则每次返回结构都会漂移；把"事实"和"要求"分开写，Counsel 和 Insight 尤其吃这一点。

---

## 十、把 Key 接进常用客户端

控制台的令牌页提供一键导入，点对应图标即可自动填入 base_url 与 Key，已支持（按站点公开配置）：

- Cherry Studio
- Lobe Chat（官方示例链接）
- OpenCat
- DeepChat
- AionUI
- AI as Workspace
- AMA 问天
- CC Switch
- 流畅阅读

手工配置时只需要两个值：Base URL 填 `https://allrouter.cc/v1`，API Key 填 `sk-` 开头的密钥，协议选 OpenAI 兼容；若客户端是 Claude 系工具，协议选 Anthropic。

---

## 十一、用量、日志与账单

- 使用日志（/usage-logs）：按时间、模型、Key 维度查每次调用，包含 token 数与扣费金额；支持导出（站点允许最长 365 天范围，默认按小时聚合）。
- 模型广场（/models）：实时展示各模型输入/输出单价，按每 1M tokens 计费（站点 FAQ 原文）。
- 额度提醒：站点开启了额度提醒（阈值可配），余额接近阈值时会提醒，别把它当唯一保障，生产环境请自建监控。
- 每日签到：站点开启了签到功能，每日可领一次随机额度奖励，属于额外福利，不要计入容量规划。
- 邀请返利：控制台有邀请返利页，返利可转入余额。

对账建议：每周拉一次日志导出，按 Key 和模型做透视，重点看"输出 token 占比"和"缓存命中率"——这两项通常决定账单为什么比预期高。

---

## 十二、账户安全与合规红线

安全侧（控制台可操作）：

- 开启两步验证（2FA），高价值账号必开。
- 如站点启用了 Passkey，可绑定用于免密登录。
- 定期轮换 Key；离职/协作结束时立即删除对应 Key。
- 开启登录会话管理，异常设备及时踢下线。

合规侧（协议第 3、5 条，违反会被封号且费用不退）：

- 不得出售、出租、转让、共享账号、Key 或额度，不得用本服务对外转售 API 访问。
- 不得注册/控制多账号规避限流、风控或优惠规则。
- 不得逆向、绕过鉴权与计费、未授权压测或渗透、爬取平台数据。
- 不得伪造请求特征绕过上游区域限制（含用 VPN/代理绕过）。
- 未经书面许可不得做提示注入、越狱等对抗测试；有研究需求须事先申请。
- 输入内容须来源合法、有权处理；不得提交国家秘密、他人商业秘密等依法须保密的信息，敏感数据自行脱敏。
- 各模型还受其上游供应商条款约束，违规导致的封禁与索赔由用户承担。

---

## 十三、故障排查速查表

| 现象 | 常见原因 | 处理 |
|---|---|---|
| 401 `Invalid token` | Key 拼错、已删除、已过期；`Authorization` 头格式不对；把 Key 用在了错误的鉴权头 | 用 `Authorization: Bearer sk-...`；Claude 协议用 `x-api-key`；重新复制 Key |
| 401 但 Key 确定正确 | Key 被重置、账户被封、分组无权 | 控制台看 Key 状态与分组；换分组重试 |
| 429 | 站点 FAQ：通常是额度不足或触发限流 | 充值/换套餐，或降低并发、加指数退避重试 |
| 404 / model not found | 模型 ID 不在当前分组、或拼写/后缀不一致 | 去模型广场核对完整 ID；确认 Key 绑定分组 |
| 403 类报错 | 分组无权限、上游饱和、风控拦截 | 换分组或换模型；带 request id 报障 |
| 流式输出中断 | 上游抖动或客户端超时 | 加超时与重试；长文本改非流式分段 |
| 扣费比预期高 | 输出 token 多、未命中缓存、用了高倍率分组、长上下文触发阶梯价 | 看日志明细；压缩 prompt；换低倍率分组 |
| 注册卡住 | 人机校验（Turnstile）未通过、邮箱验证码未收到 | 换网络环境重试；检查垃圾邮件箱 |
| 支付成功未到账 | 支付回调延迟 | 等几分钟；超 24 小时邮件报障并附订单号 |

报障时请提供：账号邮箱或用户 ID、Key 名称（不要发完整 Key）、请求时间、模型 ID、返回的 `request id`。

---

## 十四、当前状态与已知限制（2026-11-16 实测）

写手册必须说清楚哪些"还不能用"，否则用户会踩坑：

1. 官方文档站尚未上线。导航里的 Docs 指向 docs.allrouter.cc，该域名目前 302 跳回首页；站内 /docs 返回 404。现阶段请以首页说明、控制台 FAQ（站点内置 6 条问答）与本文档为准。
2. 公开价格页未开放。`/pricing` 接口返回 `pricing is disabled`，页脚链接的 "Models & Pricing" 会落到首页。查价请进控制台模型广场。
3. 可视化应用未上线。官网原文："Use them today through our API and developer console; visual apps are rolling out."七个工具目前只能通过 API 调用。
4. 对话 Playground 未开放。控制台侧栏未启用 chat/playground 模块，验证连通性请直接用 SDK 或客户端。
5. 模型清单与分组倍率随时可能调整。协议 1.3 条已保留该权利，请勿在代码中硬编码"某模型永远可用"的假设，务必实现降级链路。
6. 号池类分组的稳定性不由平台承诺。倍率低是价格优势，不是稳定性承诺。

---

## 十五、支持渠道

- 邮箱：support@allrouter.cc（退款、账单、账号问题首选）
- 电话：+65 3895 5264
- 站内公告：登录后首页公告位（例如当前公告指向文档与接入指南）
- 协议与政策：/user-agreement（含交付与退款政策第 4 条）、/privacy-policy、/about

---

## 附：一页速查

```
Base URL      https://allrouter.cc/v1
鉴权          Authorization: Bearer sk-xxx   （Claude 协议：x-api-key）
货币          1 USD = 10 credits
余额 vs 套餐  两套账，套餐额度不并入余额、不跨工具
套餐          文本族 $28/$56/$112（周上限=额度1/4）
              编码族 $41/$82/$164（1,960/3,960/8,060 credits）
Key 位置      /keys（创建后仅显示一次）
模型          /models 模型广场，约 563 个模型 ID
日志          /usage-logs，可导出，最长 365 天
退款          仅重复扣款/未交付/平台终止/法定情形，30 天内邮件申请
对账异议      费用发生后 10 天内提出
```
