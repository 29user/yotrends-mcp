# Tools

Nine tools. Parameters below are the real signatures from the running server.
Anything marked as costing credits returns an upsell message instead of an
error when the balance is empty, and charges nothing in that case.

Access to regions, history depth and the number of topics you can track is set
by your plan. Call `whoami` first if you want to know the limits before
spending anything.

---

## whoami

No parameters.

Returns the connected plan, remaining AI credits and the account's limits —
which regions are available, how far back history goes, how many topics can be
tracked. Costs nothing. Use it as a connection test.

## search_trends

```
region: str = "US"          # US | RU | GR | CY — availability depends on plan
platform: str = "all"       # all | youtube | tiktok
sort: str = "hot"           # hot | new | viral
subcategory: str | None     # optional taxonomy filter
windowDays: int = 14        # 1-30
pageSize: int = 30          # 1-100
```

Trending videos for a region and platform. `sort=viral` favours breakout
velocity over raw view count, `new` favours recency.

## list_top_creators

```
region: str = "US"
platform: str = "youtube"   # youtube | tiktok
windowDays: int = 30        # 1-30
```

Creators ranked by how often their videos appear in trends over the window.

## list_trend_clusters

```
region: str = "US"
since_days: int = 7         # 1-30
limit: int = 30             # 1-100
```

Emerging topics rather than individual videos — clusters that are starting to
rise. This is the early-signal tool; `search_trends` shows what already won.

## list_my_topics

No parameters. The topics this account tracks, with their ids. Call it before
anything that takes a `topic_id`.

## get_topic_feed

```
topic_id: int
```

Videos matching one tracked topic, matched live rather than from a cache.

## create_topic

```
description: str                    # 3-500 chars, what the topic is about
keywords: list[str]                 # one or more search phrases
name: str | None = None
content_language: str | None = None
```

Starts tracking a new topic. Respects the plan's topic cap and returns an
upsell if it is reached.

## get_topic_digest

```
topic_id: int
```

**Costs 2 AI credits.** An AI narrative explaining why the topic is moving,
plus the key videos behind that read.

## generate_text_pack

```
topic_id: int
hook: str                   # the angle or hook line
title: str                  # working title
format: str = "shorts"      # shorts | long
platform: str | None = None
```

**Costs 3 AI credits.** Produces a publish-ready pack: titles, hooks, a script
outline, description and tags for one angle on a topic.

---

## Errors you may see

| Response | Meaning |
|---|---|
| `401 unauthorized` | Token missing, invalid, or the plan no longer includes MCP access |
| Upsell text instead of data | Out of AI credits, or a plan limit reached. Nothing was charged |
| Empty result set | The filter combination has no data — widen `windowDays` or drop `subcategory` |
