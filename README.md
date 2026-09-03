# news-archive · 安全资讯归档数据仓库

东方隐侠安全团队官网（[eastsword.github.io](https://eastsword.github.io)）「安全资讯」板块的**独立数据仓库**：与官网代码仓库彻底解耦，只存数据、按月分片、无上限积累。本仓库通过 GitHub Pages 静态分发 JSON，官网资讯页前端动态加载——数据增长再大，也不会给官网仓库带来任何体积负担。

## 为什么要独立仓库

- 官网仓库若每日 commit 全量资讯（约 250KB/天），git 历史一年膨胀约 90MB；
- 独立仓库按月分片后，每条资讯只新增一次，同月文件每天至多重写一个递增版本，历史体积约为全量模式的 1/6；
- 官网 `news.md` 不再构建期渲染数据，改为运行时 fetch 本仓库，页面产物恒定轻量。

## 数据结构

```
├── index.json        # 月度索引：总量 / 月份列表（倒序）/ facets（分类、级别、来源全量计数）
├── feed.json         # 最近 24 条（首页与资讯页首屏快照用）
└── months/
    ├── 2026-09.json  # {"month": "2026-09", "items": [ …按时间倒序… ]}
    └── …             # 2021-03 起持续积累
```

单条字段：`id`（源前缀 + 哈希，全局唯一）、`title`（优先中文标题）、`url`（原文直链）、`source`、`category`（网络安全 / AI安全）、`priority`（P0/P1/P2）、`published_at`（原始时间戳）、`published_date`、`digest`（摘要清洗版：去 HTML 标签、解码实体、截 160 字）。

## 数据来源与同步

数据来自内网 EchoMind 情报聚合服务（112 渠道，收录 security + ai-security 共 83 源，含 CISA / Mandiant / MSRC / FreeBuf / The Hacker News 等）。

日常同步（每日 10:00 定时执行，脚本在官网仓库）：

```bash
python3 ../blog-site/scripts/sync_news.py            # 拉近 2 天增量 → 归档 → 推送
python3 ../blog-site/scripts/sync_news.py --no-push  # 只写本地不推送
python3 ../blog-site/scripts/sync_news.py --backfill # 全量回填（从服务端拉全部历史）
```

同步只推本仓库；官网仓库不含任何资讯数据，Pages 重建与数据更新互不干扰。

## 官网接入方式

GitHub Pages 地址：`https://eastsword.github.io/news-archive/`

与官网同属 `eastsword.github.io` 域名，fetch **同源无跨域**。官网资讯页加载策略：

1. `index.json`（数 KB）拿月度索引与 facets → 渲染筛选器；
2. 首屏加载最近月份分片；
3. 无限滚动按月懒加载更早数据；
4. 搜索/筛选时后台逐月并发扫描全量归档（带进度提示），命中即实时刷新。

## 历史体积预估

每日约 60-100 条、每条约 300 字节：单年原始数据约 10MB，git 历史（含每日增量重写当月分片）约 20-30MB/年，全部沉淀在本仓库，与官网无关。若未来历史过大，可 `git checkout --orphan` 重建单 commit 快照（数据文件本身就是全量，重建无损）。
