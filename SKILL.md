---
name: amap-save-places
description: Use when the user asks to 收藏, 保存, 添加, or batch-save a list of places in 高德地图/Amap, including attractions, restaurants, hotels, or itinerary POIs.
---

# Amap Save Places

## Purpose

Save user-specified places to the user's Amap favorites with high confidence. Favor correctness over speed: select the intended place, never toggle an existing favorite off, and report only verified saves.

**REQUIRED TOOL SKILL:** Use `browser:browser` for a visible Amap login and page interaction. Use `chrome:Chrome` only when the user explicitly needs an existing logged-in Chrome session and that page responds reliably.

## Input

Extract a flat list of places from the user's message. Retain useful context such as city, district, day grouping, and POI type only to distinguish similarly named results.

- If the list has a city context, use it for every query unless an item states otherwise.
- If the user provides an itinerary, save only explicitly named places; do not add nearby recommendations.
- If the requested operation includes organizing favorites into folders, treat that as an additional scope and confirm before creating or changing folders.

## Safety

- Ask the user to complete QR code, phone, password, or SMS verification personally in the visible Amap page.
- Never request or transmit credentials or verification codes.
- Clicking `收藏` changes the user's Amap account. The request to save locations authorizes this action for the listed places only.
- Do not guess through ambiguity. Pause and ask when two plausible POIs remain after checking name, city, district, address, and category.

## Workflow

### 1. Open Amap and establish login

1. Open `https://www.amap.com/` in the visible in-app browser.
2. If a login panel appears, leave the page visible and ask the user to log in, then resume after they reply that login is complete.
3. Confirm logged-in state through a visible account/avatar control or access to account features; do not inspect cookies or local storage.
4. If the old map view is displayed and provides `立即体验`, switch to the newer `/ssr/search` interface before bulk saving. It exposes more reliable search results and detail actions.

### 2. Search one place at a time

For each requested place:

1. Search with enough context to disambiguate, usually `<city> <place name>`.
2. When normal typing becomes unresponsive, use visible UI interaction or navigate to the site's focused search result view rather than repeatedly filling the same field.
3. Read the result list and choose the primary POI, not a similarly named hotel, parking lot, gate, shop, or bus stop.
4. Prefer the main attraction/place entry over sub-entrances unless the user explicitly requested an entrance or station.

### 3. Confirm the detail page

Before saving, open the selected POI detail and confirm:

- Detail heading matches the intended place or an unambiguous official name.
- Address/district matches the user's city or itinerary context.
- POI category is appropriate for the requested place.

Record official Amap naming changes for the final summary, for example when `八一好吃街` appears as `八一路美食街`.

### 4. Save without toggling off

The `收藏` action is a toggle. Determine current state before clicking:

- Empty star, such as an icon source containing `star-o`: not yet saved; click `收藏` once.
- Filled star, such as an icon source containing `star-filled`: already saved; do not click.

After any click, re-read the star state and count the item as completed only when the star is filled.

### 5. Recover from slow map interactions

Amap rendering can be slow. Keep actions small and resumable:

1. Search.
2. Read candidate result.
3. Open detail.
4. Read star state.
5. Click only if empty.
6. Verify filled state.

If a call times out after a possible click, re-open or re-read the current detail's star state before doing anything else. Never repeat an uncertain click.

### 6. Report completion

Report:

- Places confirmed as saved or already saved.
- Amap official names that differ from the user's wording.
- Places skipped because identification was ambiguous or the page could not confirm the save.

Do not claim success based only on clicking; require filled-star verification.

## Quick Reference

| Situation | Action |
| --- | --- |
| Login needed | Leave visible page for user authentication, then resume |
| Old homepage input stalls | Switch to the new search experience |
| Multiple same-name results | Check address/category; ask if ambiguity remains |
| Star already filled | Leave it untouched and mark already saved |
| Click timed out | Re-check star state before any retry |
| User interrupts for status | Report verified saves, then continue unless asked to stop |

## Example Trigger

User: `把下面景点收藏到高德：解放碑、洪崖洞、磁器口，都是重庆。`

Expected result: open Amap, obtain user login if needed, save each confirmed Chongqing POI only once, verify each filled star, and summarize saved official place names.
