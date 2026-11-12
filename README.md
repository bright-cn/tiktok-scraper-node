# 适用于 Node.js 和 JavaScript 的 TikTok 抓取工具

使用 Node.js 和 JavaScript 将 TikTok 个人资料、视频、评论和 TikTok Shop 商品抓取为 JSON。无需 TikTok 登录，也无需浏览器。基于 [Bright Data TikTok 爬虫工具 API](https://www.bright.cn/products/web-scraper/tiktok?utm_source=github&utm_medium=referral&utm_content=tiktok-scraper-node) 构建。新账户每月可获 5,000 个免费积分。

[![实时检查](https://github.com/bright-cn/tiktok-scraper-node/actions/workflows/live.yml/badge.svg)](https://github.com/bright-cn/tiktok-scraper-node/actions/workflows/live.yml) [![最后验证](https://img.shields.io/badge/last%20verified-11%20Nov%202026-brightgreen)](https://github.com/bright-cn/tiktok-scraper-node/actions/workflows/live.yml) <!-- verified: rewritten by the weekly run -->

[快速开始](#如何使用-nodejs-抓取-tiktok-数据) · [无需登录](#无需登录或访问令牌即可抓取-tiktok) · [CLI](#批量-tiktok-爬虫-cli) · [端点](#抓取-tiktok-帖子个人资料评论和-tiktok-shop) · [数据](#tiktok-帖子数据字段json) · [错误](#tiktok-爬虫错误) · [MCP 和 Claude Code](#tiktok-mcp-服务器和-claude-code-技能) · [免费积分](#这个-tiktok-爬虫工具免费吗) · [文档](https://docs.brightdata.com/products/scrapers/tiktok/introduction) · [支持](#支持)

使用 JavaScript 将 TikTok 个人资料、帖子、评论和 TikTok Shop 商品获取为 JSON。无需 TikTok 登录，也无需浏览器。基于 [Bright Data TikTok 爬虫工具 API](https://www.bright.cn/products/web-scraper/tiktok?utm_source=github&utm_medium=referral&utm_content=tiktok-scraper-node) 构建。

使用 [Bright Data JavaScript SDK](https://github.com/bright-cn/sdk-js)。完整 API 文档：[TikTok 爬虫工具 API](https://docs.brightdata.com/products/scrapers/tiktok/introduction)。

此外还提供：用于获取近期帖子的单命令 CLI，以及完全不需要 JavaScript 的 [Bright Data CLI](#tiktok-mcp-服务器和-claude-code-技能)。

## 如何使用 Node.js 抓取 TikTok 数据

需要 Node 20 或更高版本。此软件包使用 ESM，因此请使用 `import`，而不是 `require`。

```bash
npm install @brightdata/sdk
export BRIGHTDATA_API_TOKEN=YOUR_API_KEY
```

从 [Bright Data 控制面板](https://www.bright.cn/cp/setting/users)获取令牌。此 SDK 不会读取 `.env` 文件；Node 可通过 `node --env-file=.env yourscript.mjs` 为其加载。

也可以跳过令牌。运行一次 `npx -p @brightdata/cli bdata login`：它会打开浏览器，此后 SDK 会自行找到已存储的凭据，供你以及在该终端中工作的任何编码智能体使用。智能体无法点击完成登录，因此请先自行操作。该 CLI 也可以直接抓取 TikTok；请参阅 [TikTok MCP 服务器和 Claude Code 技能](#tiktok-mcp-服务器和-claude-code-技能)。

还没有账户？[创建账户](https://www.bright.cn/cp/start)；新账户[每月可获 5,000 个免费积分](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。

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

预计需要一到两分钟：API 会运行一个任务，`toResult` 会等待任务完成。每篇帖子消耗一个[积分](https://www.bright.cn/pricing/web-scraper)。`digg_count` 是 TikTok 对点赞数的命名。

该代码片段中的四项设置不是可选项。

`autoCreateZones: false` 可阻止 SDK 在启动时创建区域。这些区域用于网络解锁器和搜索引擎，这是该爬虫从不接触的另外两款 Bright Data 产品。没有付款方式的账户无法创建区域。

`num_of_posts` 用于限制此次运行。如果没有它，Fast API 会返回该账户的每篇帖子并逐篇计费。编排式 `postsByProfileFast([url])` 无法传入该参数，因此请使用 `collect` 形式。

`async: true` 使调用返回一个任务。`includeErrors: true` 使 API 将已失效的账户报告为一行。如果没有它，该行会被丢弃，你将得不到任何结果。

`pollTimeout` 的单位是毫秒，而不是秒。从 Python 示例复制的数值会在首次状态检查前就超时。

## 无需登录或访问令牌即可抓取 TikTok

TikTok 爬虫工具 API 不需要 TikTok 账户、TikTok 登录或 TikTok Research API 访问权限。唯一需要的凭据是 Bright Data API 密钥，可将其设置为 `BRIGHTDATA_API_TOKEN` 或通过 `bdata login` 存储。

## 批量 TikTok 爬虫 CLI

此代码库中的命令可同时为多个账户运行 Node.js 快速开始流程，并写入一个 JSON 文件。

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

在终端中，`asking` 行会被原地更新的加载动画和运行计时器替代，让你可以看到任务正在运行。

```
--limit N    posts per account, default 5, minimum 1
--out PATH   output file, default tiktok.json
```

也可以导入它而不是直接运行，以便按账户获取 `ok`、`note` 和 `error`，而不是原始行。`scrape` 不会因为某个账户无效而拒绝；读取 `posts` 前请检查 `ok`：

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

Bright Data TikTok 爬虫工具 API 有 12 个端点，每个端点都在 [API 参考](https://docs.brightdata.com/api-reference/scrapers/social-media-apis/tiktok-profiles-collect-by-url)中有独立页面。此 SDK 在 `client.scrape.tiktok` 上封装了其中四个。其他八个端点用于发现和 TikTok Shop，采用原始 HTTP；[最后一节](#通过原始-http-使用-tiktok-shop-爬虫和发现功能)展示了它们共用的调用方式。

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
| TikTok Shop 商品 URL | 该商品 | 原始 HTTP，Shop 采集 |
| Shop 分类 URL、关键词或店铺 URL | 商品 | 原始 HTTP，Shop `discover_by=category`、`keyword` 或 `shop` |

以上每项都是异步任务。API 会触发任务，SDK 会进行轮询，并在任务就绪后返回调用结果。以下每个代码片段都完整可用，只需要 `@brightdata/sdk`；原始 HTTP 则完全不需要：可直接按原样粘贴。它们每周一都会在 Actions 中运行，每隔一天还会运行一次规模较小的检查，顶部徽章显示的是最新结果。

三个短名称 `profiles`、`posts` 和 `comments` 会在一次调用中触发、轮询并获取结果，然后返回 `ScrapeResult`。当你传入 `async: true` 时，`collect` 这组名称会返回一个由你自行轮询的 `ScrapeJob`。只有第二组可以传入 `includeErrors`。

### 在一个任务中抓取多个 TikTok 账户

输入数组只会创建一个任务，而不是每个账户创建一个任务。

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

对于超过几个账户的任务，不要阻塞进程一小时。触发任务，保存快照 ID，并在任务就绪后获取。快照可下载 30 天。

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

`ScrapeJob` 还提供 `download()` 用于将快照写入磁盘，并提供 `cancel()` 用于停止任务。

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

### 按 URL 抓取 TikTok 个人资料

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

每条评论消耗一个积分，而且 `comments()` 无法要求返回更少的评论。请先检查帖子上的 `comment_count`：此 README 其他位置使用的 TikTok 帖子有超过 5,000 条评论。下面的帖子有两条评论。要限制大型帖子的评论数量，请使用带有 `limit_per_input` 的原始 HTTP；在一篇有 344 条评论的帖子上设置为 3，返回了 3 行。

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

SDK 没有 TikTok Shop 方法或 TikTok 发现功能，因此这些调用会直接发送到 API。无需安装任何内容，但此方式仅读取 `BRIGHTDATA_API_TOKEN`，不使用 CLI 登录信息。

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

其他每个端点都使用相同的 `/trigger` 调用，只是数据集、`type=discover_new&discover_by=...` 对和输入不同。添加 `&limit_per_input=N` 可限制发现结果：如果不添加，关键词或分类发现会返回所有匹配项并逐条计费。以下每项输入均已于 2026-10-20 被实时 API 接受。多项输入与 API 参考页面不同，应以实时 API 为准：

| 端点 | dataset_id | discover_by | 输入 |
| --- | --- | --- | --- |
| Shop 商品 | `gd_m45m1u911dsa4274pi` | 无 | `{ url }`，一个 `shop.tiktok.com/us/pdp/...` URL |
| 按关键词查找 Shop 商品 | `gd_m45m1u911dsa4274pi` | `keyword` | `{ keyword }`。`num_of_products` 会被拒绝 |
| 按分类查找 Shop 商品 | `gd_m45m1u911dsa4274pi` | `category`，而不是 `category_url` | `{ category_url }` |
| 按店铺查找 Shop 商品 | `gd_m45m1u911dsa4274pi` | `shop`，而不是 `shop_url` | `{ url }`，而不是 `shop_url` |
| 按个人资料查找帖子，44 个字段 | `gd_lu702nij2f790tmv9h` | `profile_url` | `{ url, num_of_posts }` |
| 按关键词查找帖子 | `gd_lu702nij2f790tmv9h` | `keyword` | `{ search_keyword, num_of_posts }` |
| 标签或音乐页面上的帖子 | `gd_lu702nij2f790tmv9h` | `url` | `{ URL }`，必须大写。`url` 会被拒绝 |
| 从搜索结果中获取个人资料 | `gd_l1villgoiiidt09ci` | `search_url` | `{ search_url, country }`，两者都必填 |

`bdata pipelines info <dataset_id> <method>` 会输出 API 实际验证的输入字段，这也是上表的检查方式。发现功能速度较慢：按关键词和分类发现两个 Shop 商品需要 18 到 20 分钟。

## TikTok 帖子数据字段（JSON）

大多数人需要的字段：

```
url  create_time  description  hashtags  digg_count  comment_count  play_count  account_id
```

`digg_count` 表示点赞数，`collect_count` 表示收藏数，而 `share_count` 以字符串形式返回。代码没有硬编码任何字段列表。API 返回的所有内容都会写入 `result.data` 以及命令生成的文件。

<!-- fields:start -->
<details>
<summary>全部 44 个字段及其类型和说明</summary>

每周通过 `client.datasets.tiktokPosts.getMetadata()` 根据帖子数据集架构重新生成，因此不会过时。命令调用的 Fast API 返回其中一部分字段；示例文件包含这 44 个字段中的 36 个。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `url` | url | 唯一的帖子链接 |
| `post_id` | text | 每篇帖子的标识信息 |
| `description` | text | 帖子附带的文本 |
| `create_time` | date | 帖子创建时间戳 |
| `digg_count` | number | 收到的点赞数 |
| `share_count` | text | 分享次数 |
| `collect_count` | number | 收藏次数 |
| `comment_count` | number | 评论数 |
| `play_count` | number | 视频播放次数 |
| `video_duration` | number | 视频时长 |
| `hashtags` | array | 包含的标签 |
| `original_sound` | text | 使用的声音（如有） |
| `profile_id` | text | 唯一用户标识符 |
| `profile_username` | text | 用户的用户名 |
| `profile_url` | url | 用户的个人资料链接 |
| `profile_avatar` | url | 用户的个人资料图片 |
| `profile_biography` | text | 用户的个人简介 |
| `preview_image` | url | 帖子预览图 |
| `post_type` | text | 帖子类型（视频、图片、文本） |
| `discovery_input` | object | 发现输入值 |
| `offical_item` | boolean | 表示视频是否为官方项目 |
| `secu_id` | text | 个人资料的安全标识符 |
| `original_item` | boolean | 表示视频是否为原创 |
| `shortcode` | text | 视频的短代码 |
| `width` | number | 视频宽度 |
| `ratio` | text | 视频宽高比 |
| `video_url` | url | 视频 URL，生成的 URL 仅在会话期间有效 |
| `music` | object | 视频所用音乐的信息 |
| `cdn_url` | url | 媒体 URL 链接 |
| `is_verified` | boolean | 表示 TikTok 账户是否已验证（布尔值：True/False） |
| `account_id` | text | 账户 ID |
| `carousel_images` | array | “轮播”类型 TikTok 帖子的图片列表 |
| `tagged_user` | array | 被标记的用户 |
| `profile_followers` | number | 帖子作者的关注者数量 |
| `tt_chain_token` | text | TikTok 用于跟踪和验证用户请求的会话相关令牌 |
| `region` | text | 地区列表示视频的来源国家/地区（例如 US、UK） |
| `commerce_info` | text | 表示帖子是否包含购物、推广或联盟营销内容的信息 |
| `subtitle_url` | url | 字幕 URL |
| `subtitle_format` | text | 字幕格式 |
| `country` | text | 部分个人资料受地区限制。请选择一个国家/地区以继续（来自发现结果） |
| `subtitle_info` | array | 字幕信息 |
| `cdn_link` | text | 媒体 URL 链接 |
| `num_share_count` | number | 分享次数 |
| `comments` | array | 帖子的评论 |

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

完整文件包含一篇帖子及其所有字段，位于 [examples/sample_output.json](examples/sample_output.json)。

</details>

## TikTok 爬虫错误

| 你看到的内容 | 含义 |
| --- | --- |
| `API token required but not found.` | 在发送任何请求前以状态码 2 退出。请设置令牌。 |
| `failed  @name: User is banned` | 以状态码 1 退出。该账户不存在，通常是拼写错误。TikTok 会将不存在的账户称为 "banned"。 |
| `got     @name: 0 posts` | 以状态码 0 退出，这是正确结果。在搜索的时间段内没有公开内容。 |
| `failed  @name: Polling timed out after ...` | 以状态码 1 退出。请求在 600 秒后放弃。请重新运行。 |

任何失败都会以状态码 1 退出，因此可以安全地根据运行结果控制脚本。

在 SDK 中，相同情况如下所示：

| 你看到的内容 | 含义 |
| --- | --- |
| `AuthenticationError: No API token found.` | 任何位置都没有令牌：选项、环境或 CLI 登录信息中均不存在。 |
| `APIError`，状态码为 401 | 令牌已设置，但不正确。 |
| `result.success` 为 `false`，`result.status` 为 `"timeout"` | SDK 放弃等待。请增大以毫秒为单位的 `pollTimeout`，或重新运行。 |
| `result.data` 中含有 `error` 键的一行 | 当你请求 `includeErrors` 时，API 对其中一个输入的响应。其他行不受影响。 |
| 预期出现错误，但返回零行 | 你没有传入 `includeErrors: true`，因此 API 丢弃了原本会说明问题的那一行。 |
| 原始 HTTP 返回 HTTP 400 `validation_error` | API 不接受某个输入字段。消息中会指出该字段；请与上表比较。 |

## TikTok MCP 服务器和 Claude Code 技能

无需 JavaScript，也无需安装任何内容。粘贴以下两行；第一行会打开一次浏览器，也可通过 SSH 或在 CI 中使用 `bdata login --device`：

```bash
npx -p @brightdata/cli bdata login
npx -p @brightdata/cli bdata pipelines run gd_m7n5v2gq296pex2f5m collect_by_url --url "https://www.tiktok.com/@tiktok" --num_of_posts 2 --pretty
```

CLI 可根据数据集 ID 运行爬虫，涵盖所有 TikTok 爬虫，包括 Shop 和发现功能。`bdata pipelines search tiktok` 会列出它们的 ID 和方法，`--limit-per-input N` 用于限制发现结果，默认值为 10。

`npx skills add brightdata/skills` 会向 Claude Code、Cursor 和 Codex 提供这些命令和文档，之后即可使用自然语言操作。完整指南：[面向编码智能体的 Bright Data](https://docs.brightdata.com/quickstart-coding-agent)。

对于没有终端的托管助手，可以使用 [Bright Data MCP 服务器](https://github.com/bright-cn/brightdata-mcp#which-tool-to-use)。它通过 `search_scrapers` 和 `run_scraper` 等六个工具运行所有 Bright Data 爬虫，包括 TikTok。这些工具默认启用；若只想加载这些工具，请添加 `scrapers` 组：

    https://mcp.brightdata.com/mcp?token=YOUR_API_TOKEN&groups=scrapers

智能体还可以自行开设账户，无需填写注册表单：[智能体注册](https://brightdata.com/auth.md)。Bright Data 支持的其他所有连接，从 LangChain 到 Zapier 和 n8n：[集成](https://docs.brightdata.com/integrations/introduction)。

<details>
<summary>适用于 CLI 的 TikTok 命令，每个命令均于 2026 年 10 月 20 日运行</summary>

在每个命令前添加 `npx -p @brightdata/cli`，或使用 `npm i -g @brightdata/cli` 安装一次。每个命令都会输出 JSON，并按每条记录一个积分计费。

五个爬虫工具，每个均使用 `bdata pipelines run <dataset_id> <method>` 运行：个人资料 `gd_l1villgoiiidt09ci`、帖子 `gd_lu702nij2f790tmv9h`、评论 `gd_lkf2st302ap89utw5k`、按个人资料获取帖子的 Fast API `gd_m7n5v2gq296pex2f5m`，以及 TikTok Shop `gd_m45m1u911dsa4274pi`。

```bash
bdata pipelines run gd_l1villgoiiidt09ci collect_by_url "https://www.tiktok.com/@tiktok" --pretty
```

1 条记录，35 个字段，33 秒。`"account_id": "tiktok"`、`"followers": 96300000`。

```bash
bdata pipelines run gd_m7n5v2gq296pex2f5m collect_by_url --url "https://www.tiktok.com/@tiktok" --num_of_posts 2 --pretty
```

2 条记录，34 个字段，70 秒。如果没有 `--num_of_posts`，它会返回该账户的每篇帖子并逐篇计费。

```bash
bdata pipelines run gd_lu702nij2f790tmv9h collect_by_url "https://www.tiktok.com/@tiktok/video/7680721699171601694" --format csv -o posts.csv
```

11 分钟。CSV 表头使用与上表相同的字段名。

```bash
bdata pipelines run gd_lu702nij2f790tmv9h discover_by_keyword --search_keyword "street food" --limit-per-input 2 --pretty
```

2 条记录，41 个字段，6.5 分钟。

```bash
bdata pipelines run gd_lkf2st302ap89utw5k collect_by_url "https://www.tiktok.com/@cooking__alone/video/7565981443512339742" --timeout 900 --pretty
```

2 条记录，16 个字段，2 分钟。每条评论消耗一个积分，因此请先检查帖子的 `comment_count`。同一命令在未设置 `--timeout 900` 时，曾在达到 CLI 默认的 600 秒后放弃。

```bash
bdata pipelines run gd_m45m1u911dsa4274pi collect_by_url "https://shop.tiktok.com/us/pdp/wireless-bluetooth-earphones-low-latency-gaming-gift/1732339489808880487" --pretty
```

1 个商品，44 个字段，2 分钟。Shop 发现以相同方式使用 `discover_by_keyword --keyword`、`discover_by_category --category_url` 或 `discover_by_shop`，但发现两个商品需要 18 到 20 分钟，超过默认的 600 秒：请添加 `--timeout 1500`，或使用 `--async`，然后通过 `bdata pipelines results <snapshot_id>` 稍后获取。

`bdata pipelines info <dataset_id> <method>` 显示爬虫工具的输入字段。`bdata pipelines run --help` 显示 `--format json|csv|ndjson|jsonl`、`-o FILE`、`--timeout` 和 `--async`。

</details>

## 这个 TikTok 爬虫工具免费吗？

新的 Bright Data 账户[每月可获 5,000 个免费积分](https://docs.brightdata.com/general/account/billing-and-pricing/free-tier)。帖子、个人资料和 TikTok Shop 商品按每条记录一个积分计费，评论按每条评论一个积分计费，因此免费积分每月可覆盖 5,000 篇帖子。之后的费率请参阅[定价](https://www.bright.cn/pricing/web-scraper)。

## 支持

此代码库中的错误：[创建问题](https://github.com/bright-cn/tiktok-scraper-node/issues)，[CONTRIBUTING.md](CONTRIBUTING.md) 中说明了需要提供的内容。有关 API、你的账户或积分的任何问题，请联系 [Bright Data 支持团队](https://brightdata.zendesk.com/hc/en-us/requests/new)。

## 许可证

[Apache 2.0](LICENSE)。

## 适用于 Python 的 TikTok 爬虫工具

同一 TikTok 爬虫工具也提供 Python 版本：[bright-cn/tiktok-scraper](https://github.com/bright-cn/tiktok-scraper)。

## 更多 Bright Data 开源爬虫工具

以相同方式构建并提供实时验证示例的其他 Node.js 爬虫工具：

- [适用于 Node.js 的 Instagram 爬虫工具](https://github.com/bright-cn/instagram-scraper-node)
- [适用于 Node.js 的 YouTube 爬虫工具](https://github.com/bright-cn/youtube-scraper-node)
- [适用于 Node.js 的 Facebook 爬虫工具](https://github.com/bright-cn/facebook-scraper-node)
