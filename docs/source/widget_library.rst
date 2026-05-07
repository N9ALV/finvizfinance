Supabase Widget Library
=======================

The repository also tracks an operational guide for the Supabase-backed
FinvizFinance widget library. The library stores reusable rendered widgets
for screeners, signal tables, group views, quote fundamentals, forex, crypto,
news, insider, and earnings data.

Canonical guide
---------------

See the full guide in the repository:

``docs/operations/FINVIZ_WIDGET_LIBRARY_USER_GUIDE.md``

Main Supabase objects
---------------------

- ``public.finviz_widgets`` — primary widget table with configuration,
  rendered ``html_widget`` output, ``result_rows`` data, refresh status, and
  public/embed metadata.
- ``public.finviz_widget_options`` — centralized style, sort, column, and
  filter-pack presets.
- ``public.finviz_widget_library`` — convenience browsing view joining widgets
  with option labels.

Embed pattern
-------------

Public widgets can be retrieved through the ``finviz_public_widget`` RPC. Embed
clients should render the returned ``html_widget`` inside a controlled container
or iframe.
