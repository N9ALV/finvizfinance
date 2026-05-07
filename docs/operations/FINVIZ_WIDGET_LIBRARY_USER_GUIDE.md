# Finviz Widget Library User Guide

This guide covers the standalone Supabase-backed FinvizFinance widget library. It belongs in this repository because the stored widgets are generated from FinvizFinance-style screeners, groups, quotes, forex, crypto, news, insider, and earnings data. The Smartview and Multistep apps can embed or reuse the rendered widget output, but this library is maintained here.

## Repository Location

Canonical GitHub home: `N9ALV/finvizfinance`.

This guide was moved here from `N9ALV/Multistep-TGPT-Nfly` so the widget library, renderer notes, and FinvizFinance data-source code are tracked together.

## Purpose

The library stores reusable FinvizFinance-derived widgets that can be embedded or reused elsewhere:

- stock screeners
- signal widgets
- sector/industry/group widgets
- quote/fundamental widgets
- forex and crypto performance widgets
- config-first rows for news, insider, and earnings widgets

Each widget row contains both:

1. the config needed to regenerate it, and
2. the latest rendered `html_widget` plus `result_rows` data.

## Main Supabase Objects

### `public.finviz_widgets`

Primary table. One row per widget.

Important columns:

- `slug` — stable widget key, e.g. `signal-top-gainers`
- `title` — human-readable title
- `description` — user-facing description
- `widget_type` — `screener`, `quote`, `group`, `forex`, `crypto`, `news`, `insider`, `earnings`, `calendar`, `future`, or `custom`
- `status` — `draft`, `active`, `paused`, `error`, or `archived`
- `is_public` — public embed/read flag
- `embed_token` — optional private embed token
- `screener_view` — e.g. `overview`, `valuation`, `financial`, `ownership`, `performance`, `technical`
- `screener_signal` — Finviz signal, e.g. `Top Gainers`, `Oversold`, `Major News`
- `screener_filters` — JSONB Finviz filters
- `request_config` — JSONB call config for the renderer
- `render_config` — JSONB display config
- `style_option_key` — centralized style preset key
- `sort_option_key` — centralized sort preset key
- `columns_option_key` — centralized column preset key
- `filter_pack_key` — centralized filter preset key
- `result_rows` — latest table rows as JSONB
- `result_summary` — latest render summary
- `html_widget` — latest embeddable HTML
- `row_count` — latest row count
- `last_refreshed_at`, `next_refresh_at` — refresh tracking
- `docs_reference` — FinvizFinance docs reference

### `public.finviz_widget_options`

Centralized options/presets table.

Current option types:

- `style` — visual theme presets
- `sort` — sorting presets
- `columns` — column presets
- `filter_pack` — reusable filter groups

Current seeded counts:

- 5 style presets
- 25 sort presets
- 10 column presets
- 20 filter packs

### `public.finviz_widget_library`

Convenience view for browsing widgets with option labels joined in.

Use this first when inspecting the library.

## Current Library Coverage

Seeded total: **109 widgets**.

Breakdown:

- 79 screener widgets
- 12 group widgets
- 8 quote/fundamental widgets
- 1 forex widget
- 1 crypto widget
- 8 news/insider/earnings config widgets

Rendered now:

- 101 widgets have `result_rows` and `html_widget`
- 8 news/insider/earnings rows are config-first rows for a future dedicated renderer pass

## Quick Browse Queries

### Browse the library

```sql
select
  slug,
  title,
  widget_type,
  status,
  row_count,
  style_label,
  sort_label,
  columns_label,
  filter_pack_label
from public.finviz_widget_library
order by widget_type, slug;
```

### Find active rendered widgets

```sql
select slug, title, widget_type, row_count, last_refreshed_at
from public.finviz_widgets
where status = 'active'
  and coalesce(row_count, 0) > 0
order by widget_type, slug;
```

### Find screener widgets by filter pack

```sql
select slug, title, screener_view, filter_pack_key, sort_option_key, row_count
from public.finviz_widgets
where widget_type = 'screener'
  and filter_pack_key = 'large_cap_growth'
order by screener_view;
```

### Find signal widgets

```sql
select slug, title, screener_signal, row_count, last_refreshed_at
from public.finviz_widgets
where slug like 'signal-%'
order by slug;
```

### Find public embeddable widgets

```sql
select slug, title, widget_type, row_count
from public.finviz_widgets
where is_public = true
  and status = 'active'
order by slug;
```

## Embed Retrieval

Public widgets can be fetched through the RPC:

```sql
select *
from public.finviz_public_widget('large-cap-growth');
```

For token-gated widgets, pass the widget slug and its stored `embed_token` value to the same RPC. Do not store raw private tokens in documentation.

Returned fields:

- `slug`
- `title`
- `description`
- `widget_type`
- `last_refreshed_at`
- `row_count`
- `result_summary`
- `html_widget`

Embed clients should render `html_widget` inside a controlled container/iframe.

## Common Example Slugs

Screeners/signals:

- `signal-top-gainers`
- `signal-top-losers`
- `signal-most-active`
- `signal-unusual-volume`
- `signal-oversold`
- `signal-overbought`
- `signal-major-news`
- `large_cap_growth-overview`
- `value_low_pe-overview`
- `short_squeeze-overview`
- `analyst_buy-overview`
- `earnings_week-overview`
- `etf_universe-overview`

Sectors:

- `sector-technology-large-caps`
- `sector-healthcare-large-caps`
- `sector-financial-large-caps`
- `sector-energy-large-caps`
- `sector-basic-materials-large-caps`

Groups:

- `group-overview-sector`
- `group-valuation-sector`
- `group-performance-sector`
- `group-overview-industry`
- `group-performance-industry`

Quotes/fundamentals:

- `quote-aapl-fundamentals`
- `quote-msft-fundamentals`
- `quote-nvda-fundamentals`
- `quote-tsla-fundamentals`
- `quote-jpm-fundamentals`
- `quote-spy-fundamentals`
- `quote-qqq-fundamentals`

Cross-asset:

- `forex-performance`
- `crypto-performance`

## Centralized Sorting

Sort presets live in `public.finviz_widget_options` with `option_type = 'sort'`.

Examples:

- `market_cap_desc` — largest market cap first
- `change_desc` — biggest daily gainers first
- `change_asc` — biggest daily losers first
- `volume_desc` — highest volume first
- `relative_volume_desc` — highest relative volume first
- `pe_asc` — lowest P/E first
- `dividend_desc` — highest dividend yield first
- `roe_desc` — highest return on equity first
- `short_float_desc` — highest short interest first
- `perf_month_desc` — best monthly performance first
- `rsi_asc` — lowest RSI first

To inspect:

```sql
select option_key, label, description, config
from public.finviz_widget_options
where option_type = 'sort'
order by sort_order;
```

To change a widget's sort preset:

```sql
update public.finviz_widgets
set sort_option_key = 'volume_desc',
    screener_order = 'Volume',
    request_config = jsonb_set(coalesce(request_config, '{}'::jsonb), '{ascend}', 'false'::jsonb),
    updated_at = now()
where slug = 'signal-most-active';
```

The renderer should use the preset and then refresh `result_rows` / `html_widget`.

## Centralized Styling

Style presets live in `public.finviz_widget_options` with `option_type = 'style'`.

Current presets:

- `iu_dark` — default dark embed style
- `compact_dark` — dense dark table
- `light_clean` — light table
- `transparent_dark` — transparent dark container
- `headline_cards` — card-style summary layout

To inspect:

```sql
select option_key, label, description, config
from public.finviz_widget_options
where option_type = 'style'
order by sort_order;
```

To apply a style preset:

```sql
update public.finviz_widgets
set style_option_key = 'compact_dark',
    render_config = jsonb_set(coalesce(render_config, '{}'::jsonb), '{theme}', '"compact_dark"'::jsonb),
    updated_at = now()
where slug = 'signal-top-gainers';
```

The current HTML is static. After changing style/sort/columns/config, refresh the widget so `html_widget` matches the config.

## Adding a New Widget

Minimum insert:

```sql
insert into public.finviz_widgets (
  title,
  slug,
  description,
  widget_type,
  status,
  is_public,
  screener_view,
  screener_order,
  screener_filters,
  request_config,
  render_config,
  style_option_key,
  sort_option_key,
  columns_option_key,
  docs_reference
) values (
  'Oversold Large-Cap Technology',
  'oversold-large-cap-technology',
  'Large-cap technology stocks with RSI below 30.',
  'screener',
  'draft',
  true,
  'technical',
  'Relative Strength Index (14)',
  '{"Sector":"Technology","Market Cap.":"+Large (over $10bln)","RSI (14)":"Oversold (30)"}'::jsonb,
  '{"limit":20,"ascend":true}'::jsonb,
  '{"max_rows":20,"theme":"iu_dark"}'::jsonb,
  'iu_dark',
  'rsi_asc',
  'technical_trading',
  'https://finvizfinance.readthedocs.io/en/latest/screener.html'
);
```

Use `draft` until a renderer has successfully populated `result_rows` and `html_widget`; then mark it `active`.

## Updating a Rendered Widget

A renderer should update these fields together:

- `result_rows`
- `result_summary`
- `html_widget`
- `html_hash`
- `row_count`
- `last_refreshed_at`
- `next_refresh_at`
- `error_message`
- `status`
- `updated_at`

Do not update only `html_widget` without updating `result_rows` and timestamps.

## Notes and Guardrails

- Prefer **over-including** useful widgets. Rows can be archived later.
- Keep labels clear and reference the source docs in `docs_reference`.
- `html_widget` should not include the text `Powered by finvizfinance`.
- Chart images are intentionally not stored by default.
- Finviz/FinvizFinance can throttle or change page structures; keep refresh intervals conservative.
- Config-only widgets are acceptable where a reliable renderer pass has not yet been implemented.
