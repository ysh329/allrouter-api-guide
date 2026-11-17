# All Router 新手使用手册

版本 v2.0 · 2026-11-16
信息来源：allrouter.cc 线上页面、`/api/status`、`/api/about`、`/api/user/groups`、`/api/ratio_config` 与 `/v1/*` 接口实测

## 目录

1. 这是什么
2. 能用哪些模型
3. 三步跑通
4. 钱怎么算
5. 接 AI Coding Agent
6. 接桌面客户端
7. 出问题先看这里
8. 联系我们

## 一、这是什么

All Router 是一个 AI 模型的 API 中转站。你在一个网站上注册一次，拿到一把 API Key，把代码里的 base_url 改成 `https://allrouter.cc/v1`，就能调用 OpenAI、Claude、Gemini、DeepSeek、通义千问、GLM、Kimi、Grok 等几十个系列的模型，花多少扣多少。

它卖两样东西：

- 充值余额：充多少用多少，按每次调用扣费，长期有效。
- 套餐：30 天一次性额度，价格和额度都固定，不自动续费。

运营方是 TECHFRONTIER PTE. LTD.（新加坡，UEN 202512973K）。

官网首页还挂了七个成品工具（Deck 演示、Counsel 法务、Code 编码、Digest 摘要、Lingua 翻译、Insight 分析、Copy 文案），目前都只能通过 API 调，可视化应用还在陆续上线。新手先把 API 跑通，工具那部分以后再说。

有三件事建议先知道：

- 模型不是这家做的。平台负责转发、计量和结算，模型本身的能力和稳定性来自各家供应商。
- 先付后用。余额或套餐额度用完就调不动，没有欠费额度。
- 模型清单会变。上新和下架都可能出现在站内公告里，代码里别写死「某个模型永远在」。

## 二、能用哪些模型

截至 2026-11-16，站点公开的价格配置里一共有 563 个可调用的模型 ID：486 个按 token 计费，29 个按次计费（图像、视频、语音），其中 77 个按上下文长度阶梯计价。

按系列归并，大致是这样：

- OpenAI（GPT、o 系列、Codex）：127 个，代表模型 gpt-5、gpt-5-mini、gpt-4.1、o3、o3-pro、gpt-5.3-codex
- 通义千问 Qwen：68 个，代表模型 qwen3-max、qwen3-coder-plus、qwen3-235b-a22b
- 其他国产（文心、星火、零一、MiniMax、阶跃、混元、Mimo）：56 个，代表模型 ERNIE-4.0-8K、SparkDesk-v4.0、yi-large、MiniMax-M2.7、step-3.7-flash
- Claude：45 个，代表模型 claude-sonnet-5、claude-opus-4-5、claude-haiku-4-5
- 图像与视频：39 个，代表模型 gpt-image-2、flux-1.1-pro、dall-e-3、imagen-3.0、mj_imagine、gemini-3-pro-image
- Gemini：38 个，代表模型 gemini-2.5-pro、gemini-2.5-flash、gemini-3-flash-preview
- GLM 智谱：33 个，代表模型 glm-4.6、glm-4.5、glm-4-flash
- Mistral 系：30 个，代表模型 mistral-large-2512、devstral-2512、magistral-medium-latest
- DeepSeek：23 个，代表模型 deepseek-chat、deepseek-reasoner、deepseek-v4-pro
- 语音与向量：21 个，代表模型 tts-1、whisper-1、text-embedding-3-large、gemini-embedding-001
- Grok：16 个，代表模型 grok-4.7、grok-3-fast-beta、grok-2-vision
- Cohere：15 个，代表模型 command-a-03-2025、command-r-plus、command-a-vision
- Perplexity：8 个，代表模型 sonar-pro、sonar-reasoning-pro、sonar-deep-research
- Kimi：6 个，代表模型 kimi-k2.6、kimi-k3.1-pro、kimi-k2.7-code
- 其他（旧模型与社区模型）：38 个，代表模型 PaLM-2、BLOOMZ-7B、solar-pro4、mercury-2、v0-1.5-md

想看完整清单有两个办法：登录后进控制台的模型广场，或者带上 Key 请求 `GET https://allrouter.cc/v1/models`。

模型 ID 必须和列表里一字不差，大小写、后缀都算数，比如 `-thinking`、`[1m]`、`-openai-compact` 都会改变 ID。建议直接从模型广场复制，别手敲。

## 三、三步跑通

注册、建 Key、发请求，三步。不用买服务器，不用装中间件，也不用等人工开通。

### 3.1 注册

打开 https://allrouter.cc/sign-up ，填用户名、密码、邮箱，点「发送验证码」，收邮件填进去，勾选服务条款，提交即可。

站点开了人机校验（Turnstile），如果一直卡在验证环节，换个网络或者关掉代理再试。也可以用 GitHub 或 LinuxDO 账号直接登录。

### 3.2 建一把 Key

登录后进控制台，左侧「令牌」，点「添加令牌」。几个字段值得认真填：

- 名称：按用途起名，比如 `prod-web`、`dev-local`，以后看日志能分清
- 额度上限：生产环境一定要填，避免一把 Key 被刷爆整个账户
- 过期时间：临时测试给 1~7 天，长期服务可以设长一点
- 分组：决定这把 Key 能调哪些模型、按什么价扣费，见第四章
- 模型限制：生产环境建议只放开实际要用的几个模型
- IP 限制：服务器有固定出口 IP 的话建议填上

创建成功后会弹窗显示完整 Key，`sk-` 开头，只显示这一次，马上复制保存。之后列表里只剩掩码，丢了只能重建，重建后旧 Key 立即失效。

### 3.3 发第一个请求

Base URL 是 `https://allrouter.cc/v1`，鉴权用 `Authorization: Bearer sk-xxx`。curl 最小示例：

```bash
curl https://allrouter.cc/v1/chat/completions \
  -H "Authorization: Bearer sk-你的密钥" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5-mini",
    "messages": [{"role": "user", "content": "用三句话解释什么是 API 网关"}]
  }'
```

用 Python 的话，装好 `openai` 包，改两个参数就行：

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-你的密钥",
    base_url="https://allrouter.cc/v1",
)

resp = client.chat.completions.create(
    model="gpt-5-mini",
    messages=[{"role": "user", "content": "把这段话压缩成 50 字以内：……"}],
)
print(resp.choices[0].message.content)
print(resp.usage)
```

Node.js 同理，`baseURL` 换成同一个地址即可。

三种原生协议都支持，按你的客户端选：

- OpenAI 格式：`/v1/chat/completions`、`/v1/responses`，用 `Authorization: Bearer <Key>`
- Anthropic 格式：`/v1/messages`，用 `x-api-key`
- Gemini 格式：`/v1beta/models/{model}:generateContent`，用 `x-goog-api-key`

控制台目前没有内置 Playground，验证连通性直接用 SDK 或 curl。

## 四、钱怎么算

### 4.1 计价单位

全站用 credits 计价，1 美元 = 10 credits。控制台的余额、每笔扣费都显示成 credits。

### 4.2 单价公式

文本模型按 token 计费，每 1M 输入 token 的价格按这个公式算：

输入价（美元/1M） = 模型倍率 × 2 × 分组倍率

输出价（美元/1M） = 输入价 × 输出倍率

其中模型倍率由模型本身决定，输出倍率是同一个模型输出相对输入的倍数，分组倍率取决于你的 Key 绑在哪个分组上。缓存命中和缓存写入按另外的缓存倍率折算，一般远低于正常输入价。

举个例子：`gpt-5` 的模型倍率是 0.625，输出倍率是 8，你的 Key 在 default 分组（倍率 0.75）：

- 输入价 = 0.625 × 2 × 0.75 = 0.9375 美元/1M ≈ 9.4 credits/1M
- 输出价 = 9.4 × 8 = 75 credits/1M

也就是说，一次请求输入 2000 token、输出 500 token，大约扣 0.019 + 0.038 = 0.056 credits。

### 4.3 分组倍率

同一把 Key 绑定一个分组，分组决定可用模型集合和价格乘数。站点公开的 11 个分组如下：

- grok：自营 Grok 号池，倍率 0.045
- gpt：自营 Codex 号池，倍率 0.053
- kimi 号池：自营 Kimi 号池，倍率 0.06
- claude-max：Claude Max 反代，倍率 0.25
- glm：GLM 系列，倍率 0.3
- deepseek：DeepSeek 系列，倍率 0.4
- gemini：Gemini 官方 API，倍率 0.45
- gpt-azure-api：微软 Azure GPT，倍率 0.6
- claude-official-api：Claude 官方 API，倍率 0.65
- default：官方 API，倍率 0.75
- claude-fable-api：Fable 5 官方 API，倍率 1.3

倍率越低越便宜，但便宜的那几档走的是号池或反代，稳定性和限流强度跟官方 API 不一样。要稳定交付选 `default`、`claude-official-api`、`gpt-azure-api`；要压成本、能接受偶发 429 和重试，选号池类分组。

### 4.4 常见模型实价

按 default 分组（0.75）换算，每 1M token 的 credits 价格：

- gpt-5：输入 9.4，输出 75
- gpt-5-mini：输入 1.9，输出 15
- gpt-4.1：输入 15，输出 60
- o3：输入 15，输出 60
- claude-sonnet-5：输入 15，输出 75
- claude-opus-4-5：输入 37.5，输出 187.5
- claude-haiku-4-5：输入 7.5，输出 37.5
- gemini-2.5-pro：输入 9.4，输出 9.4
- gemini-2.5-flash：输入 7.5，输出 18.8
- deepseek-chat：输入 2.0，输出 2.0
- deepseek-v4-pro：输入 3.3，输出 6.5
- qwen3-max：输入 9，输出 45
- glm-4.6：输入 4.5，输出 16.5
- kimi-k2.6：输入 7.1，输出 30
- grok-4.7：输入 15，输出 45
- text-embedding-3-small：输入 0.15，只算输入

同样这些模型，换到对应的号池分组会便宜一大截，比如 gpt 组只有 default 的 7%，claude-max 只有三分之一。

### 4.5 三种计价方式

- 按 token：绝大多数文本模型，就是上面那套公式。
- 按次：图像、视频、语音类共 29 个模型，每次调用固定价格。比如 dall-e-3 0.04 美元/次、imagen-3.0 0.03 美元/次、flux-1.1-pro 0.04 美元/次、mj_imagine 0.1 美元/次、mj_video 0.8 美元/次。
- 阶梯加缓存：77 个模型按上下文长度分档计价，长上下文单价更高。比如 claude-sonnet-5-5 在 512K 以内按基础价，超过 512K 输入价翻倍、输出价翻倍。

### 4.6 充值和套餐

充值就是往余额里加 credits，什么时候用、用哪个模型都行。

套餐是另一种账，30 天有效、一次性购买、不自动续费，额度不能并进余额，也不能跨套餐工具使用。目前有两族：

- 编码族 Lite：$41 / 30 天，1,960 credits，适合每周写几天代码
- 编码族 Standard：$82 / 30 天，3,960 credits，适合每天写代码
- 编码族 Advanced：$164 / 30 天，8,060 credits，适合大型代码库长会话
- 文本族 Starter：$28 / 30 天，基础额度，适合偶尔用
- 文本族 Studio：$56 / 30 天，2 倍额度，适合有稳定产出
- 文本族 Team：$112 / 30 天，4 倍额度，适合小团队

文本族有个每周上限，单周最多用掉总额度的四分之一，周窗口打满就停，即使总额度还有剩。编码族只能从受支持的命令行编码客户端使用，纯调 API 的话充值或文本族更划算。

### 4.7 退款与对账

- 数字服务交付后原则上不退款，没用完的余额和套餐额度也不退。
- 能退的只有四类：重复扣款、付款成功但 3 个工作日未交付、平台原因导致服务终止、法律强制要求。
- 申请方式：付款后 30 天内发邮件到 support@allrouter.cc，附账号邮箱、订单号、支付时间金额和原因。
- 对账单有异议要在费用发生后 10 天内提出，逾期视为认可。
- 使用日志页可以按时间、模型、Key 导出明细，最长可查 365 天，对账就靠它。

## 五、接 AI Coding Agent

### 5.1 Claude Code

Claude Code 走 Anthropic 的 Messages 协议，把两个环境变量指过来就行：

```bash
export ANTHROPIC_BASE_URL=https://allrouter.cc
export ANTHROPIC_AUTH_TOKEN=sk-你的密钥
claude
```

注意这里不带 `/v1`，客户端会自己拼 `/v1/messages`。`claude-max` 分组会把 opus、sonnet、haiku 这几个别名映射到组内对应模型，用别的分组则按该分组的模型清单走。

### 5.2 Codex CLI

Codex 走 OpenAI 的 Responses 协议：

```bash
export OPENAI_BASE_URL=https://allrouter.cc/v1
export OPENAI_API_KEY=sk-你的密钥
codex
```

模型选 `gpt` 号池分组里的 codex 系，或者 `default` 分组里的 `gpt-5` 系列。

### 5.3 Cursor、Cline、Roo Code、Kilo Code、Windsurf

这几个都支持自定义 OpenAI 兼容端点，填两个值即可：

- Base URL：`https://allrouter.cc/v1`
- API Key：你的 `sk-` 密钥

如果客户端支持 Anthropic 协议（比如 Cline 的 Claude 模式），Base URL 填 `https://allrouter.cc`，鉴权方式选 x-api-key。

### 5.4 分组和套餐怎么挑

- Claude Code 长时间跑：claude-max（0.25）或编码族套餐
- Codex CLI 日常用：gpt 号池（0.053）或文本族套餐
- 要稳定、要可预期：claude-official-api、gpt-azure-api、default
- 只调 API、不用命令行：直接充值，别买套餐

一个细节：站点提示过，不同模型走的路由不同，会话中途换分组时，正在跑的 Codex 或 Claude Code 会话可能需要重启才能生效。

## 六、接桌面客户端

控制台的令牌页有一键导入，点图标会自动填好 base_url 和 Key，目前已支持 Cherry Studio、Lobe Chat、OpenCat、DeepChat、AionUI、AI as Workspace、AMA 问天、CC Switch、流畅阅读。

手工配置也只有两个值：Base URL 填 `https://allrouter.cc/v1`，API Key 填 `sk-` 密钥，协议选 OpenAI 兼容；Claude 系工具则选 Anthropic，Base URL 去掉 `/v1`。

## 七、出问题先看这里

- 401 Invalid token：Key 拼错、已删除、过期，或鉴权头写错。处理办法：重新复制 Key；Anthropic 协议要用 x-api-key
- 429：额度不足或触发限流。处理办法：充值换套餐，或降低并发、加指数退避
- 404 model not found：模型 ID 不在当前分组，或拼写有偏差。处理办法：去模型广场核对完整 ID，确认 Key 的分组
- 扣费比预期高：输出 token 多、没命中缓存、用了高倍率分组、长上下文进阶梯。处理办法：看日志明细，压 prompt，换低倍率分组
- 注册卡住：人机校验没过、验证码没收到。处理办法：换网络重试，查垃圾邮件箱
- 支付成功没到账：支付回调延迟。处理办法：等几分钟；超过 24 小时带订单号发邮件

当前还有几处不能用的地方，先说清楚免得踩坑：

- 官方文档站没上线，`docs.allrouter.cc` 会跳回首页，`/docs` 是 404。
- 公开价格页没开放，`/api/pricing` 返回 `pricing is disabled`，查价请进控制台模型广场。
- 可视化应用和 Playground 都没上线，功能只能通过 API 调。
- 号池类分组不承诺稳定性，便宜是价格优势，不是 SLA。

报障时请带上：账号邮箱或用户 ID、Key 名称（别发完整 Key）、请求时间、模型 ID，以及报错里那串 `request id`。

## 八、联系我们

- 客服微信：待补充
- 交流群：待补充
- 邮箱：support@allrouter.cc（退款、账单、账号问题首选）
- 电话：+65 3895 5264
- 站内公告：登录后首页公告位
- 协议与政策：/user-agreement（含交付与退款政策）、/privacy-policy、/about
