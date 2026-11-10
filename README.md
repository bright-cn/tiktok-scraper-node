# 适用于 Node.js 和 JavaScript 的 TikTok 爬虫工具

使用 Node.js 和 JavaScript 将 TikTok 个人资料、视频、评论和 TikTok Shop 商品抓取为 JSON。无需登录 TikTok，也无需浏览器。基于 [Bright Data TikTok 爬虫工具 API](https://www.bright.cn/products/web-scraper/tiktok?utm_source=github&utm_medium=referral&utm_content=tiktok-scraper-node) 构建。新账户每月可获得 5,000 个免费积分。

[![实时检查](https://github.com/bright-cn/tiktok-scraper-node/actions/workflows/live.yml/badge.svg)](https://github.com/bright-cn/tiktok-scraper-node/actions/workflows/live.yml) [![最后验证](https://img.shields.io/badge/last%20verified-9%20Nov%202026-brightgreen)](https://github.com/bright-cn/tiktok-scraper-node/actions/workflows/live.yml) <!-- verified: rewritten by the daily run -->

[快速开始](#如何使用-nodejs-抓取-tiktok-数据) · [无需登录](#无需登录或访问令牌即可抓取-tiktok) · [CLI](#批量-tiktok-抓取-cli) · [端点](#抓取-tiktok-帖子个人资料评论和-tiktok-shop) · [数据](#tiktok-帖子数据字段json) · [错误](#tiktok-爬虫错误) · [MCP 和 Claude Code](#tiktok-mcp-服务器和-claude-code-技能) · [免费积分](#这个-tiktok-爬虫是免费的吗) · [文档](https://docs.brightdata.com/products/scrapers/tiktok/introduction) · [支持](#支持)

使用 JavaScript 将 TikTok 个人资料、帖子、评论和 TikTok Shop 商品抓取为 JSON。无需登录 TikTok，也无需浏览器。基于 [Bright Data TikTok 爬虫工具 API](https://www.bright.cn/products/web-scraper/tiktok?utm_source=github&utm_medium=referral&utm_content=tiktok-scraper-node) 构建。

使用 [Bright Data JavaScript SDK](https://github.com/bright-cn/bright-data-sdk-js)。完整 API 文档：[TikTok 爬虫工具 API](https://docs.brightdata.com/products/scrapers/tiktok/introduction)。

此处还提供：一条命令即可获取近期帖子的 CLI，以及完全不需要 JavaScript 的 [Bright Data CLI](#tiktok-mcp-服务器和-claude-code-技能)。

## 如何使用 Node.js 抓取 TikTok 数据

需要 Node 20 或更高版本。此软件包使用 ESM，因此请使用 `import`，不要使用 `require`。

```bash
npm install @brightdata/sdk
export BRIGHTDATA_API_TOKEN=YOUR_API_KEY
```

从 [Bright Data 控制面板](https://www.bright.cn/cp/setting/users)获取令牌。此 SDK 不会读取 `.env` 文件；Node 可通过 `node --env-file=.env yourscript.mjs` 为其加载该文件。

你也可以跳过令牌。运行一次 `npx -p @brightdata/cli bdata login`：它会打开浏览器，此后 SDK 会自动找到已存储的凭据，供你以及在该终端中工作的任何编程智能体使用。智能体无法点击完成登录，因此请先自行登录。同一个 CLI 也能直接抓取 TikTok；请参阅 [TikTok MCP 服务器和 Claude Code 技能](#tiktok-mcp-服务器和-claude-code-技能)。

还没有账户？[创建账户](https://www.bright.cn/cp/start)；新账户每月可获得 [5,000 个免费积分](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。

```javascript
import { bdclient } from "@brightdata/sdk";

const client = new bdclient({ autoCreateZones: false });
const job = await client.scrape.tiktok.collectPostsByProfileFast(
  [{ url: "https://www.tiktok.com/@tiktok", num_of_posts: 5 }],
  { async: true, includeErrors: true },
);
const result = await job.toResult({ pollTimeout: 900_000 });
if (!result.success) throw new Error(`${result.status}: ${result.error}`);
for (const post of result.data) {
  console.log(post.digg_count, post.comment_count, post.url);
}
await client.close();
```

```
12000 5090 https://www.tiktok.com/@tiktok/video/7694797676306615583
14800 3612 https://www.tiktok.com/@tiktok/video/7693184538704416031
12300 3222 https://www.tiktok.com/@tiktok/video/7692114317151423775
11200 1821 https://www.tiktok.com/@tiktok/video/7691450773619625247
9428 1391 https://www.tiktok.com/@tiktok/video/7691013160425327903
```

预计需要一到两分钟：API 会运行一个作业，而 `toResult` 会等待其完成。每个帖子消耗一个[积分](https://www.bright.cn/pricing/web-scraper)。`digg_count` 是 TikTok 对点赞的命名。

该代码片段中的四项设置均不可省略。

`autoCreateZones: false` 会阻止 SDK 在启动时创建区域。这些区域用于网络解锁器和 SERP，即该爬虫从不涉及的另外两款 Bright Data 产品。未绑定付款方式的账户无法创建区域。

`num_of_posts` 会限制运行规模。如果没有它，Fast API 会返回该账户的所有帖子，并对每个帖子计费。编排式 `postsByProfileFast([url])` 无法传递此参数，因此请使用 `collect` 形式。

`async: true` 让调用返回一个作业。`includeErrors: true` 让 API 将已失效账户报告为一行。如果没有它，该行会被丢弃，你将得不到任何结果。

`pollTimeout` 的单位是毫秒，而不是秒。从 Python 示例复制的数值会在首次状态检查前就超时。

## 无需登录或访问令牌即可抓取 TikTok

TikTok 爬虫工具 API 不需要 TikTok 账户、TikTok 登录信息或 TikTok Research API 访问权限。唯一的凭据是 Bright Data API 密钥，可将其设为 `BRIGHTDATA_API_TOKEN` 或通过 `bdata login` 存储。

## 批量 TikTok 抓取 CLI

此仓库中的命令会同时对多个账户运行 Node.js 快速开始示例，并写入一个 JSON 文件。

```bash
npm install -g github:bright-cn/tiktok-scraper-node
tiktok-scraper tiktok khaby.lame
```

```
Fetching up to 5 recent posts per account, for: tiktok, khaby.lame
Usually one to three minutes each. One credit per post, 5,000 free per month.
asking  @tiktok...
got     @tiktok: 5 posts
asking  @khaby.lame...
got     @khaby.lame: 5 posts

Saved 10 posts as JSON to tiktok.json (36 fields per post)
```

在终端中，`asking` 行会被原地更新的旋转指示器和计时器替代，以便你确认任务正在运行。

```
--limit N    posts per account, default 5, minimum 1
--out PATH   output file, default tiktok.json
```

也可以导入它而不是直接运行，从而按账户获得 `ok`、`note` 和 `error`，而不是原始行。`scrape` 不会因为某个错误账户而被 reject；读取 `posts` 前请先检查 `ok`：

```javascript
import { scrape } from "@brightdata/tiktok-scraper-node";

for (const outcome of await scrape(["tiktok", "zz_not_a_real_account_zz"], { limit: 1 })) {
  if (outcome.ok) {
    console.log(`${outcome.handle}: ${outcome.posts.length} posts`);
  } else {
    console.log(`${outcome.handle} failed: ${outcome.error}`);
  }
}
```

```
tiktok: 1 posts
zz_not_a_real_account_zz failed: User is banned
```

不存在的账户会返回 "User is banned"。没有近期内容的账户会成功返回，但不包含帖子。

## 抓取 TikTok 帖子、个人资料、评论和 TikTok Shop

Bright Data TikTok 爬虫工具 API 有 12 个端点，每个端点在 [API 参考](https://docs.brightdata.com/api-reference/scrapers/social-media-apis/tiktok-profiles-collect-by-url)中都有独立页面。此 SDK 封装了其中四个，位于 `client.scrape.tiktok`。其余八个端点用于发现和 TikTok Shop，需使用原始 HTTP；[最后一节](#通过原始-http-使用-tiktok-shop-爬虫和发现功能)展示了它们共用的调用。

| 你拥有 | 你想要 | 使用方式 |
| --- | --- | --- |
| 个人资料 URL | 近期帖子，36 个字段 | `collectPostsByProfileFast([{ url, num_of_posts: 5 }], { async: true, includeErrors: true })` |
| 个人资料 URL | 近期帖子，44 个字段 | 原始 HTTP，帖子 `discover_by=profile_url` |
| 个人资料 URL | 个人资料 | `profiles([url])` |
| TikTok 搜索 URL | 匹配的个人资料 | 原始 HTTP，个人资料 `discover_by=search_url` |
| 帖子 URL | 这些帖子，44 个字段 | `posts([url, url])` |
| 关键词 | 匹配的帖子 | 原始 HTTP，帖子 `discover_by=keyword` |
| 标签、音乐或探索页面 URL | 该页面上的帖子 | 原始 HTTP，帖子 `discover_by=url` |
| 帖子 URL | 其评论 | `comments([url])`，每条评论消耗一个积分 |
| TikTok Shop 商品 URL | 该商品 | 原始 HTTP，商店采集 |
| 商店分类 URL、关键词或店铺 URL | 商品 | 原始 HTTP，商店 `discover_by=category`、`keyword` 或 `shop` |

以上每项操作都是异步作业。API 会触发作业，SDK 会进行轮询，调用会在作业就绪时返回。下面的每个代码片段都完整可用，只需要 `@brightdata/sdk`；原始 HTTP 则什么都不需要：按原样粘贴即可。它们每周一都会在 Actions 中运行，每隔一天还会执行一次较小规模的检查，顶部徽章显示最近一次结果。

三个短名称 `profiles`、`posts` 和 `comments` 会在一次调用中触发、轮询和获取，并返回 `ScrapeResult`。当你传递 `async: true` 时，名称为 `collect` 的方法会返回需要自行轮询的 `ScrapeJob`。只有第二组方法可以传递 `includeErrors`。

### 在一个作业中抓取多个 TikTok 账户

输入数组只会创建一个作业，而不是每个账户一个作业。

```javascript
import { bdclient } from "@brightdata/sdk";

const client = new bdclient({ autoCreateZones: false });
const job = await client.scrape.tiktok.collectPostsByProfileFast(
  [
    { url: "https://www.tiktok.com/@tiktok", num_of_posts: 1 },
    { url: "https://www.tiktok.com/@khaby.lame", num_of_posts: 1 },
  ],
  { async: true, includeErrors: true },
);
const result = await job.toResult({ pollTimeout: 900_000 });
if (!result.success) throw new Error(`${result.status}: ${result.error}`);
for (const post of result.data) {
  console.log(post.account_id, post.url);
}
await client.close();
```

```
khaby.lame https://www.tiktok.com/@khaby.lame/video/7697307919943060767
tiktok https://www.tiktok.com/@tiktok/video/7694797676306615583
```

`account_id` 是用户名。`profile_username` 是显示名称，例如 "TikTok" 或 "Khabane lame"，而不是用户名。

### 立即触发，稍后获取

如果账户数量超过几个，不要让进程阻塞一小时。触发作业，保存快照 ID，并在就绪后获取。快照可供下载 30 天。

```javascript
import { bdclient } from "@brightdata/sdk";

const client = new bdclient({ autoCreateZones: false });
const job = await client.scrape.tiktok.collectPosts(
  ["https://www.tiktok.com/@tiktok/video/7680721699171601694"],
  { async: true, includeErrors: true },
);
console.log("snapshot:", job.snapshotId);
await job.wait({ pollInterval: 5_000, pollTimeout: 900_000 });
console.log("status:", await job.status());
const [record] = await job.fetch();
console.log("fetched:", record.url, "likes:", record.digg_count);
await client.close();
```

```
snapshot: sd_mvgd8dubxoh0ti15z
status: ready
fetched: https://www.tiktok.com/@tiktok/video/7680721699171601694 likes: 22900
```

`ScrapeJob` 还提供 `download()` 用于将快照写入磁盘，以及 `cancel()` 用于停止作业。

### 按日期筛选 TikTok 帖子

```javascript
import { bdclient } from "@brightdata/sdk";

const client = new bdclient({ autoCreateZones: false });
const job = await client.scrape.tiktok.collectPostsByProfileFast(
  [
    {
      url: "https://www.tiktok.com/@tiktok",
      num_of_posts: 3,
      start_date: "09-01-2026", // MM-DD-YYYY
      end_date: "09-30-2026",
    },
  ],
  { async: true, includeErrors: true },
);
const result = await job.toResult({ pollTimeout: 900_000 });
if (!result.success) throw new Error(`${result.status}: ${result.error}`);
for (const post of result.data) {
  console.log(post.create_time, post.play_count, post.url);
}
await client.close();
```

```
2026-09-30T21:57:30.000Z 346500 https://www.tiktok.com/@tiktok/video/7691450773619625247
2026-09-29T17:39:33.000Z 264700 https://www.tiktok.com/@tiktok/video/7691013160425327903
2026-09-25T20:13:19.000Z 449600 https://www.tiktok.com/@tiktok/video/7689568478570843422
```

### 通过 URL 抓取 TikTok 个人资料

```javascript
import { bdclient } from "@brightdata/sdk";

const client = new bdclient({ autoCreateZones: false });
const result = await client.scrape.tiktok.profiles(["https://www.tiktok.com/@tiktok"]);
if (!result.success) throw new Error(`${result.status}: ${result.error}`);
const [profile] = result.data;
console.log(profile.account_id, "followers:", profile.followers, "videos:", profile.videos_count);
await client.close();
```

```
tiktok followers: 96400000 videos: 1510
```

### 抓取 TikTok 视频的评论

每条评论消耗一个积分，而 `comments()` 无法减少请求数量。请先检查帖子上的 `comment_count`：此 README 其他位置使用的 TikTok 帖子有超过 5,000 条评论。下面的帖子只有两条。若要限制大型帖子的评论数量，请使用带 `limit_per_input` 的原始 HTTP；在一个有 344 条评论的帖子上设置 3，会返回 3 行。

```javascript
import { bdclient } from "@brightdata/sdk";

const client = new bdclient({ autoCreateZones: false });
const result = await client.scrape.tiktok.comments([
  "https://www.tiktok.com/@cooking__alone/video/7565981443512339742",
]);
if (!result.success) throw new Error(`${result.status}: ${result.error}`);
console.log(result.data.length, "comments; first:", JSON.stringify(result.data[0].comment_text.slice(0, 60)));
await client.close();
```

```
2 comments; first: "GfcgUZxolnnnmmmmmmmm mmm."
```

### 通过原始 HTTP 使用 TikTok Shop 爬虫和发现功能

SDK 没有 TikTok Shop 方法，也没有 TikTok 发现方法，因此这些请求会直接发送到 API。无需安装任何内容，但这种方式只读取 `BRIGHTDATA_API_TOKEN`，不读取 CLI 登录信息。

```javascript
const API = "https://api.brightdata.com/datasets/v3";
const headers = {
  Authorization: `Bearer ${process.env.BRIGHTDATA_API_TOKEN}`,
  "Content-Type": "application/json",
};

async function call(path, body) {
  const init = body === undefined ? { headers } : { method: "POST", headers, body: JSON.stringify(body) };
  const response = await fetch(API + path, init);
  if (!response.ok) throw new Error(`${response.status}: ${await response.text()}`);
  return response.json();
}

const product =
  "https://shop.tiktok.com/us/pdp/wireless-bluetooth-earphones-low-latency-gaming-gift/1732339489808880487";
const job = await call("/trigger?dataset_id=gd_m45m1u911dsa4274pi&include_errors=true", [{ url: product }]);
while (!["ready", "failed"].includes((await call(`/progress/${job.snapshot_id}`)).status)) {
  await new Promise((resolve) => setTimeout(resolve, 10_000));
}
const [record] = await call(`/snapshot/${job.snapshot_id}?format=json`);
if (record.error) throw new Error(record.error);
console.log(record.title, record.final_price, record.currency, record.store_name);
```

```
Wireless Bluetooth Earphones, Black TWS Gaming Headset 13.99 USD Saeme
```

其他所有端点都使用相同的 `/trigger` 调用，只需更换数据集、`type=discover_new&discover_by=...` 对和输入。添加 `&limit_per_input=N` 可限制发现结果；如果没有它，关键词或分类会返回所有匹配项，并逐项计费。以下每项输入均已于 2026-10-20 获实时 API 接受。多处内容与 API 参考页面不同，应以实时 API 为准：

| 端点 | dataset_id | discover_by | 输入 |
| --- | --- | --- | --- |
| 商店商品 | `gd_m45m1u911dsa4274pi` | 无 | `{ url }`，一个 `shop.tiktok.com/us/pdp/...` URL |
| 按关键词发现商店商品 | `gd_m45m1u911dsa4274pi` | `keyword` | `{ keyword }`。`num_of_products` 会被拒绝 |
| 按分类发现商店商品 | `gd_m45m1u911dsa4274pi` | `category`，而不是 `category_url` | `{ category_url }` |
| 按店铺发现商品 | `gd_m45m1u911dsa4274pi` | `shop`，而不是 `shop_url` | `{ url }`，而不是 `shop_url` |
| 按个人资料获取帖子，44 个字段 | `gd_lu702nij2f790tmv9h` | `profile_url` | `{ url, num_of_posts }` |
| 按关键词获取帖子 | `gd_lu702nij2f790tmv9h` | `keyword` | `{ search_keyword, num_of_posts }` |
| 标签或音乐页面上的帖子 | `gd_lu702nij2f790tmv9h` | `url` | `{ URL }`，大写。`url` 会被拒绝 |
| 从搜索中获取个人资料 | `gd_l1villgoiiidt09ci` | `search_url` | `{ search_url, country }`，两者均为必填项 |

`bdata pipelines info <dataset_id> <method>` 会输出 API 验证的输入字段，上表正是通过这种方式检查的。发现功能速度较慢：按关键词和分类发现商店商品时，获取两个商品需要 18 到 20 分钟。

## TikTok 帖子数据字段（JSON）

大多数人需要的字段：

```
url  create_time  description  hashtags  digg_count  comment_count  play_count  account_id
```

`digg_count` 表示点赞数，`collect_count` 表示收藏数，而 `share_count` 会以字符串形式返回。代码不会硬编码任何字段列表。API 返回的所有内容都会写入 `result.data` 和命令生成的文件中。

<!-- fields:start -->
<details>
<summary>全部 44 个字段及其类型和说明</summary>

每天通过 `client.datasets.tiktokPosts.getMetadata()` 从帖子数据集架构重新生成，因此不会过时。命令调用的 Fast API 仅返回其中一部分；示例文件包含这 44 个字段中的 36 个。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `url` | url | 帖子的唯一链接 |
| `post_id` | 文本 | 每个帖子的标识 |
| `description` | 文本 | 帖子附带的文本 |
| `create_time` | date | 帖子创建时间戳 |
| `digg_count` | 数字 | 收到的点赞数 |
| `share_count` | 文本 | 分享次数 |
| `collect_count` | 数字 | 收藏次数 |
| `comment_count` | 数字 | 评论数 |
| `play_count` | 数字 | 视频播放次数 |
| `video_duration` | 数字 | 视频时长 |
| `hashtags` | 数组 | 包含的主题标签 |
| `original_sound` | 文本 | 使用的声音（如有） |
| `profile_id` | 文本 | 唯一用户标识符 |
| `profile_username` | 文本 | 用户的用户名 |
| `profile_url` | url | 用户的个人资料链接 |
| `profile_avatar` | url | 用户的个人资料图片 |
| `profile_biography` | 文本 | 用户的个人简介 |
| `preview_image` | url | 帖子预览图 |
| `post_type` | 文本 | 帖子类型（视频、图片、文本） |
| `discovery_input` | 对象 | 发现输入值 |
| `offical_item` | 布尔值 | 表示视频是否为官方内容 |
| `secu_id` | 文本 | 个人资料的安全标识符 |
| `original_item` | 布尔值 | 表示视频是否为原创 |
| `shortcode` | 文本 | 视频短代码 |
| `width` | 数字 | 视频宽度 |
| `ratio` | 文本 | 宽高比 |
| `video_url` | url | 视频 URL；生成的 URL 仅在当前会话中有效 |
| `music` | 对象 | 视频所用音乐的信息 |
| `cdn_url` | url | 媒体 URL 链接 |
| `is_verified` | 布尔值 | 表示 TikTok 账户是否已认证（布尔值：True/False） |
| `account_id` | 文本 | 账户 ID |
| `carousel_images` | 数组 | “轮播”类型 TikTok 帖子的图片列表 |
| `tagged_user` | 数组 | 被标记的用户 |
| `profile_followers` | 数字 | 帖子作者的粉丝数 |
| `tt_chain_token` | 文本 | TikTok 用于跟踪和验证用户请求的会话相关令牌 |
| `region` | 文本 | 地区列表示视频的来源国家/地区（例如 US、UK） |
| `commerce_info` | 文本 | 表示帖子是否包含购物、推广或联盟营销内容的信息 |
| `subtitle_url` | url | 字幕 URL |
| `subtitle_format` | 文本 | 字幕格式 |
| `country` | 文本 | 某些个人资料受地区限制。请选择一个国家/地区以继续（来自发现功能） |
| `subtitle_info` | 数组 | 字幕信息 |
| `cdn_link` | 文本 | 媒体 URL 链接 |
| `num_share_count` | 数字 | 分享次数 |
| `comments` | 数组 | 帖子的评论 |

</details>
<!-- fields:end -->

<details>
<summary>来自 <code>tiktok-scraper tiktok --limit 1</code> 的真实输出文件开头</summary>

```json
{
  "generated_at": "2026-10-20T07:33:23.874Z",
  "handles": [
    {
      "handle": "tiktok",
      "posts": [
        {
          "url": "https://www.tiktok.com/@tiktok/video/7694797676306615583",
          "post_id": "7694797676306615583",
          "description": "this summer gave us pop songs, glam routines, and a few main-character habits. @McCall Mirabella breaks down this summer's souvenirs 🐬☀️",
          "create_time": "2026-10-09T22:25:11.000Z",
          "digg_count": 12000,
          "share_count": "1260",
          "collect_count": 1228,
          "comment_count": 5088,
          "play_count": 1400000,
          "video_duration": 49,
          "original_sound": "TikTok: original sound",
          "profile_id": "107955",
  ...
```

包含一个帖子全部字段的完整文件位于 [examples/sample_output.json](examples/sample_output.json)。

</details>

## TikTok 爬虫错误

| 你看到的内容 | 含义 |
| --- | --- |
| `API token required but not found.` | 在发出任何请求前以状态码 2 退出。请设置令牌。 |
| `failed  @name: User is banned` | 以状态码 1 退出。没有该账户，通常是拼写错误。对于不存在的账户，TikTok 会显示 "banned"。 |
| `got     @name: 0 posts` | 以状态码 0 退出，这是正确结果。搜索期间内没有公开内容。 |
| `failed  @name: Polling timed out after ...` | 以状态码 1 退出。请求在 600 秒后放弃。请重新运行。 |

任何失败都会以状态码 1 退出，因此可以安全地使用运行结果作为脚本的执行条件。

在 SDK 中，相同情况如下所示：

| 你看到的内容 | 含义 |
| --- | --- |
| `AuthenticationError: No API token found.` | 任何位置都没有令牌：选项、环境或 CLI 登录信息中均不存在。 |
| `APIError`，状态为 401 | 令牌已设置，但不正确。 |
| `result.success` 为 `false`，`result.status` 为 `"timeout"` | SDK 已放弃等待。请提高 `pollTimeout`（单位为毫秒），或重新运行。 |
| `result.data` 中包含 `error` 键的一行 | 当你请求 `includeErrors` 时，API 针对某个输入返回的结果。其他行不受影响。 |
| 在预期出现错误时返回零行 | 你没有传递 `includeErrors: true`，因此 API 丢弃了原本用于说明错误的行。 |
| 原始 HTTP 返回 HTTP 400 `validation_error` | API 不接受某个输入字段。消息中会指出该字段；请与上表进行比较。 |

## TikTok MCP 服务器和 Claude Code 技能

无需 JavaScript，也无需安装任何内容。粘贴这两行；第一行会打开一次浏览器，也可以在 SSH 和 CI 中使用 `bdata login --device`：

```bash
npx -p @brightdata/cli bdata login
npx -p @brightdata/cli bdata pipelines run gd_m7n5v2gq296pex2f5m collect_by_url --url "https://www.tiktok.com/@tiktok" --num_of_posts 2 --pretty
```

CLI 可通过数据集 ID 运行爬虫工具，包括所有 TikTok 抓取工具及商店和发现功能。`bdata pipelines search tiktok` 会列出它们的 ID 和方法，`--limit-per-input N` 可限制发现结果，默认值为 10。

`npx skills add brightdata/skills` 会让 Claude Code、Cursor 和 Codex 学会这些命令和文档，之后即可使用自然语言操作。完整指南：[面向编程智能体的 Bright Data](https://docs.brightdata.com/quickstart-coding-agent)。

对于没有终端的托管助手，可以使用 [Bright Data MCP 服务器](https://github.com/bright-cn/brightdata-mcp#which-tool-to-use)。它通过 `search_scrapers` 和 `run_scraper` 等六种工具运行所有 Bright Data 爬虫，其中也包括 TikTok。这些工具默认启用；若只想加载这些工具，请添加 `scrapers` 组：

    https://mcp.brightdata.com/mcp?token=YOUR_API_TOKEN&groups=scrapers

智能体也可以自行创建账户，无需填写注册表单：[智能体注册](https://brightdata.com/auth.md)。Bright Data 支持的其他所有连接，从 LangChain 到 Zapier 和 n8n：[集成](https://docs.brightdata.com/integrations/introduction)。

<details>
<summary>CLI 的 TikTok 命令，每条均于 2026 年 10 月 20 日运行</summary>

在每条命令前添加 `npx -p @brightdata/cli`，或使用 `npm i -g @brightdata/cli` 一次性安装。每条命令都会输出 JSON，并按每条记录消耗一个积分。

五个抓取工具，每个均使用 `bdata pipelines run <dataset_id> <method>` 运行：个人资料 `gd_l1villgoiiidt09ci`、帖子 `gd_lu702nij2f790tmv9h`、评论 `gd_lkf2st302ap89utw5k`、按个人资料获取帖子的 Fast API `gd_m7n5v2gq296pex2f5m`，以及 TikTok Shop `gd_m45m1u911dsa4274pi`。

```bash
bdata pipelines run gd_l1villgoiiidt09ci collect_by_url "https://www.tiktok.com/@tiktok" --pretty
```

1 条记录，35 个字段，33 秒。`"account_id": "tiktok"`、`"followers": 96300000`。

```bash
bdata pipelines run gd_m7n5v2gq296pex2f5m collect_by_url --url "https://www.tiktok.com/@tiktok" --num_of_posts 2 --pretty
```

2 条记录，34 个字段，70 秒。如果没有 `--num_of_posts`，它会返回该账户的所有帖子，并对每个帖子计费。

```bash
bdata pipelines run gd_lu702nij2f790tmv9h collect_by_url "https://www.tiktok.com/@tiktok/video/7680721699171601694" --format csv -o posts.csv
```

11 分钟。CSV 标头包含与上表相同的字段名称。

```bash
bdata pipelines run gd_lu702nij2f790tmv9h discover_by_keyword --search_keyword "street food" --limit-per-input 2 --pretty
```

2 条记录，41 个字段，6.5 分钟。

```bash
bdata pipelines run gd_lkf2st302ap89utw5k collect_by_url "https://www.tiktok.com/@cooking__alone/video/7565981443512339742" --timeout 900 --pretty
```

2 条记录，16 个字段，2 分钟。每条评论消耗一个积分，因此请先检查帖子的 `comment_count`。不带 `--timeout 900` 的同一命令曾在达到 CLI 默认的 600 秒后放弃。

```bash
bdata pipelines run gd_m45m1u911dsa4274pi collect_by_url "https://shop.tiktok.com/us/pdp/wireless-bluetooth-earphones-low-latency-gaming-gift/1732339489808880487" --pretty
```

1 个商品，44 个字段，2 分钟。商店发现功能同样可使用 `discover_by_keyword --keyword`、`discover_by_category --category_url` 或 `discover_by_shop`，但获取两个商品需要 18 到 20 分钟，超过 600 秒默认值：请添加 `--timeout 1500`，或者使用 `--async`，稍后再通过 `bdata pipelines results <snapshot_id>` 获取。

`bdata pipelines info <dataset_id> <method>` 会显示抓取工具的输入字段。`bdata pipelines run --help` 会显示 `--format json|csv|ndjson|jsonl`、`-o FILE`、`--timeout` 和 `--async`。

</details>

## 这个 TikTok 爬虫是免费的吗？

新的 Bright Data 账户每月可获得 [5,000 个免费积分](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。帖子、个人资料和 TikTok Shop 商品每条记录消耗一个积分，评论则每条消耗一个积分，因此免费积分每月可覆盖 5,000 个帖子。后续费率请参阅[定价](https://www.bright.cn/pricing/web-scraper)。

## 支持

此仓库中的错误：[提交 issue](https://github.com/bright-cn/tiktok-scraper-node/issues)，[CONTRIBUTING.md](CONTRIBUTING.md) 中说明了应包含哪些信息。任何与 API、你的账户或积分有关的问题，请联系 [Bright Data 支持团队](https://brightdata.zendesk.com/hc/en-us/requests/new)。

## 许可证

[Apache 2.0](LICENSE)。

## 适用于 Python 的 TikTok 爬虫工具

同一个 TikTok 爬虫也提供 Python 版本：[brightdata/tiktok-scraper](https://github.com/brightdata/tiktok-scraper)。

## Bright Data 提供的更多开源爬虫工具

其他以相同方式构建并包含实时验证示例的 Node.js 抓取工具：

- [适用于 Node.js 的 Instagram 爬虫工具](https://github.com/bright-cn/instagram-scraper-node)
- [适用于 Node.js 的 YouTube 爬虫工具](https://github.com/brightdata/youtube-scraper-node)
- [适用于 Node.js 的 Facebook 爬虫工具](https://github.com/brightdata/facebook-scraper-node)
