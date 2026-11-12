# 供在此代码库中工作的编码智能体使用

更改任何内容前请先阅读本文。以下每一行均已于 2026-10-20 根据实时 API、已安装的 SDK（`@brightdata/sdk` 1.2.0）或 Bright Data CLI 0.4.0 验证。如果 API 参考页面与实时 API 不一致，以此处记录的实时 API 为准。

## 身份验证

- SDK 会依次从环境中读取 `apiKey`、`BRIGHTDATA_API_TOKEN` 或 `BRIGHTDATA_API_KEY`，然后读取已存储的凭据 `bdata login`。设置该变量或运行一次登录，是唯一需要人工完成的步骤；切勿将密钥粘贴到代码中。
- JavaScript SDK 不会读取 `.env` 文件。Node 20 可通过以下方式为其加载：`node --env-file=.env`。
- 始终以 `new bdclient({ autoCreateZones: false })` 的方式构造客户端。如果保持启用状态，SDK 会在首次请求时创建网络解锁器和搜索引擎区域。此代码库从不使用区域，而且没有付款方式的账户无法创建区域。
- README 中的原始 HTTP 代码片段仅读取 `BRIGHTDATA_API_TOKEN`。它们无法使用 CLI 登录信息。

## API 的行为方式

- TikTok 位于 `client.scrape.tiktok`。它有 `profiles`、`posts`、`comments` 和三个 Fast API 调用。它没有发现方法，也没有 TikTok Shop 方法；这些功能使用原始 HTTP。
- 近期帖子来自 `collectPostsByProfileFast([{ url, num_of_posts }], { async: true, includeErrors: true })`。如果没有 `async: true`，它会在同步端点上等待，而不是返回 `ScrapeJob`。编排式 `postsByProfileFast([url])` 无法传入 `num_of_posts`，因此会对该账户的每篇帖子计费。它返回 36 个字段；帖子发现和 `posts([url])` 返回 44 个字段。
- 每次采集调用都要传入 `includeErrors: true`。如果未设置，已失效的账户会返回零行。不存在的用户名会返回一行，其错误为 "User is banned"。
- 轮询选项的单位是毫秒，而不是秒。
- 一次调用中传入多个输入只会创建一个任务。应优先使用这种方式，而不是循环。对于大型任务，请保留 `snapshotId` 并稍后获取；快照可下载 30 天。
- 每条记录消耗一个积分。评论按每条评论一个积分计费，并且 `comments()` 无法限制数量：请先检查帖子上的 `comment_count`。使用 `limit_per_input` 的原始 HTTP 可以限制数量；在一篇有 344 条评论的帖子上设置为 3，返回了 3 行。
- `account_id` 是用户名。`profile_username` 是显示名称。
- 以下是 API 实际验证的输入名称和 `discover_by` 值。多个 API 参考页面有误：
  - 按 URL 发现帖子时使用 `URL`，且必须大写。`url` 会返回 400："This input should not contain a url field"。
  - Shop 发现使用 `discover_by=category`、`keyword` 或 `shop`。页面中所述的 `category_url` 和 `shop_url` 会返回 400："Incorrect discovery collector id Available types: category, keyword, shop"。
  - 按店铺采集 Shop 数据时使用 `url`，而不是 `shop_url`。按分类采集 Shop 数据时使用 `category_url`。按关键词采集 Shop 数据时使用 `keyword`；`num_of_products` 会返回 400。
  - 按搜索 URL 发现个人资料时必须提供 `country`。
  - `bdata pipelines info <dataset_id> <method>` 会输出实时输入架构。
- Shop 发现速度较慢：按关键词或分类发现两个商品需要 18 到 20 分钟。
- 架构可能随时更改，恕不另行通知。切勿硬编码字段列表。使用 `client.datasets.tiktokPosts.getMetadata()` 读取它。该调用返回 `{ id, fields }`，而 `fields` 是一个以字段名为键的对象；截至 2026-10-20，共有 44 个字段。
- 完整文档索引，每个条目对应一个 `.md` 页面：https://docs.brightdata.com/llms.txt

## 在此代码库中工作

- `npm test` 可离线运行且不需要令牌。`npm run lint` 必须通过。
- CI 会在空白机器上使用 README 自身提供的命令进行安装。每周工作流会针对真实 API 执行 README 中的每个围栏代码块，同时还会运行一项规模较小的实时检查。代码片段必须完整且可单独粘贴运行，展示的输出也必须是真实结果。
- README 中的字段表位于 `<!-- fields:start -->` 和 `<!-- fields:end -->` 之间，每周重新生成。请勿手动编辑。
- README 顶部的“最后验证”徽章行会由每周运行任务重写。请勿手动编辑。
- 保持精简。不要添加重试、去重、调度、数据库或并发功能。
