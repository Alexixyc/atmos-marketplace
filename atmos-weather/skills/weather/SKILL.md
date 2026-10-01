---
name: weather
description: 使用观晴 · Atmos 打开天气应用，查询当前天气、未来 24 小时或未来 7 天天气，搜索城市、切换城市或查看城市地标背景。适用于“打开天气”“看看杭州天气”“切换到上海”等请求。
---

Use the Atmos MCP tools for real weather. Respond in Chinese unless the user prefers another language.

1. For “打开天气”, call `open_weather_app` with `{}`. The app offers location and city search; do not infer the user's location.
2. For “看看杭州天气” or “切换到上海”, call `open_weather_app` with `city` set to the requested city name. Pass a known stable `cityId` when one has already been returned. The launch result supplies the initial weather and city to the view.
3. For ambiguous city names, call `search_cities`, show country and region, and ask which candidate the user intends. Reuse stable IDs and coordinates, not names alone.
4. For weather questions without an application request, call `get_weather`. It accepts a city name, stable `cityId`, full returned city object, or an explicit latitude/longitude pair. Never fabricate weather, UV, rain probability, time, or data freshness.
5. Use `get_city_landscape` to retrieve scenery provenance. It is a curated wallpaper, never a live camera. A missing local scenery resource should not block weather.

When MCP Apps is supported, `open_weather_app` renders the official HTML resource in fullscreen/side-panel mode. Do not reopen the tool from inside the view: use its launch result. The opener automatically reuses or starts the bundled loopback Web service. If the host does not render MCP Apps, open the returned `webUrl` with an available browser-opening tool. The installed release needs no development checkout or npm startup command. If the opener returns `webError` without `webUrl`, explain that the standalone view could not start, retry when appropriate, and continue providing weather through `get_weather` if needed. Never claim the widget was displayed unless its rendering was verified.

State the observation/model estimate time and whether a response is cached when freshness is relevant. Data comes from Open-Meteo numerical weather models, not an on-site measuring station. Do not describe forecast values as historical observations. Use the selected city's timezone.

Request browser geolocation only when the user presses the app's location action. Do not retrieve system location through unrelated tools. Precise coordinates are sent to the weather and reverse-geocoding providers only to fulfill that explicit action.
