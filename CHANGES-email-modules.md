# Nourish email modules — standardization (hubl-email-module skill)

All 11 modules were rebuilt to the skill's conventions and render-tested with every repeater count:
Buttons 1–2, Cards 1–3, Speaker Grids 1–12 (each speaker checked to appear exactly once), and Nav links 1–4.
Each render was checked for balanced Outlook conditionals, balanced tags in both the browser and Outlook views, and no empty `href`s or blank CSS values.

## Upload

The whole folder goes up in one command:

```
hs cms upload "<local modules folder>" "<Design Manager destination folder>"
```

Test with a **fresh** instance of each module. Existing instances keep their saved values and won't show new defaults.
Delete this file from the modules folder if you don't want it uploaded.

## Changes common to every module

- **Font:** `font_family` is a single `{% set %}` constant at the top of each `module.html`. It is never a field.
- **Field IDs:** every field has a real UUID. The old IDs (`"section_bg"`, `"b2c1a000-…-0001"`, etc.) are gone.
- **Style tab:** module-scope style fields (wrapper/section colour and padding, shared card colours, module-level button and image styling) now live in one `styles` group on the Style tab.
  - Per-item styling inside repeaters stays on the Content tab (known issue #14).
- **Buttons:** every button has the full standard set: alignment, background, text and border colour, font weight, width mode (full/fixed), and fixed width.
  - The fixed width is clamped at render time to what the column can hold.
  - The link's width is the table width minus 52px (2 × 25px padding + 2 × 1px border). Several originals got this wrong by 2px.
- **Images:** every image has the full standard set: optional link URL (no `<a>` is output when it's empty), alignment, and width clamped to the layout.
- **Links:** all link fields are `link` type, read via `.url.href`, with link types URL, File, Email address and Page.
  - Blog was dropped from Cards 1-3, Cards 2 column and Repeatable Items.
- **Repeaters:** every repeater's `default` array lists every child field (known issue #2). `sorting_label_field` always points at a plain text field.
- **`meta.json`:** standard shape. `label` equals the folder name, `host_template_types` is `["EMAIL"]`, and the existing `module_id` is kept.

## ⚠️ Live instances: verify after upload

Every module has a `module_id`, so I assumed they are in use and added `aliases_mapping` for renamed or moved fields.
I used HubSpot's documented format: `{"property_aliases_paths": {"new_name": ["old_name"]}}`.

- **Documented pattern (top-level old field → new field):** these should just work.
- **Untested:** aliases on fields inside a repeater, and aliases from an old *group* path (`style.background_color`, `button.cta_text`, `text_padding.top`).
  - If an upload is rejected, remove the offending `aliases_mapping`.
  - If an upload succeeds, check that one existing email kept its values.
- **Field-type changes HubSpot can't alias.** These have render-time fallbacks in HubL so existing instances keep working:
  - Buttons: `button_url` changed from `url` to `link`. The template falls back to the old `.href`.
  - Cards - CTA: the old `button` group became top-level `button_cta_text` and `button_url`. The template falls back to `module.button.*`.
  - Logo - Nav Bar: `link_url` changed from `url` to `link`. The template falls back to the old `.href`.
  - Repeatable Items: the `button_full_width` boolean became `button_width_mode`. The template falls back to the old boolean.
  - The fallbacks are commented in the templates and can be deleted once no old instances remain.

## Per module

### Nourish - Buttons - 1-2 columns
- Renames:
  - `section_bg` / `padding_top` / `padding_bottom` → `styles.section_*`.
  - `cta_text` → `button_cta_text`, `button_bg` → `button_bg_color`, `button_border` → `button_border_color`.
- Added: font weight, width mode, fixed width (cap 540 for one button, 250 for two).
- Padding sliders step by 1px instead of 5px.

### Nourish - Cards - 1-3 columns
- Wrapper fields moved to `styles`.
- Per-card bg/border colours stay in the repeater, since each card is independent.
- Added: `image_link_url`; full button set (text colour was hardcoded white); `image_alignment` now has a default and applies in the 1-card layout too.
- The card body is shared between the 1-card and multi-card layouts through a macro.
- **Visual change:** the default button width is now 190 everywhere, capped at 137 for 3 cards. The originals used 200 at 2 cards and 110 at 3.

### Nourish - Cards - 2 column
- Wrapper and shared card colours moved to `styles`. Card colours are module-scope because it's one card.
- Added: image link, alignment and width (was hardcoded 200, max 224); full button set.
- **Visual change:** the button link is 148px instead of 150, which fixes a 2px overflow.

### Nourish - Cards - CTA
- `wrapper_bg` / `section_bg` / `section_border` → `styles.wrapper_bg_color` / `card_bg_color` / `card_border_color`.
- New wrapper padding fields, defaulting to the old hardcoded 10/20.
- The `button` group was flattened: CTA text and URL are on the Content tab, button styling on the Style tab. Fixed width is capped at 190.

### Nourish - Cards - Repeatable Items
- Wrapper and shared card colours moved to `styles`. New wrapper padding fields default to 10/20.
- Renames: `button_text` → `button_cta_text`, `button_width` → `button_fixed_width` (max now 488).
- Added: `show_rich_text` toggle, image link, alignment and width (was hardcoded 400), button alignment and font weight.
- Fixes:
  - The default array's `full_width_button` key didn't match the `button_full_width` field.
  - The Outlook card table had a hardcoded `#E0F5F5` background.
- **Visual change:** the default button text colour is now `#ffffff`. It used to be unset, so links rendered in the client's default colour.

### Nourish - List - Single Column
- The `style` group became `styles`, with `section_bg_color`, padding fields (default 0/10), `text_color` and `bottom_border_color`.
- Fixes: a hardcoded `#004038` div background; `host_template_types` was PAGE/BLOG.
- Added SECTION START/END markers.

### Nourish - Logo - Nav Bar
- `bg_color` / `padding_*` → `styles.section_*`.
- `logo` → `image`, plus `image_link_url`, `image_alignment` and `image_width` (max 200).
- `meta.json` label changed from "Logo + Nav Bar" to the module name.
- Nav link URLs are now `link` fields.

### Nourish - Speaker Grid - Card
- Wrapper fields moved to `styles`, with new padding fields defaulting to 10/20.
- `section_bg_color` / `section_border_color` → `card_bg_color` / `card_border_color`.
- `alignment` → `styles.speaker_alignment`.
- Added: photo and company-logo alignment and width (module-scope so the grid stays uniform), and per-speaker optional photo and logo link URLs.
- Removed the unused `style.background_color` group.
- Fixes: `sorting_label_field` pointed at a field ID instead of `speakers.last_name`; the speaker default was missing fields; `host_template_types`.
- **Visual change:** the photo is now capped at 117px. At 132px it overflowed the 117px cell in this layout.

### Nourish - Speaker Grid - Standard
- `style.background_color` → `styles.section_bg_color`.
- New padding fields: top padding applies to the first row and bottom padding to the last. The 20px gap between rows is unchanged.
- The same speaker and photo additions and fixes as the Card version.
- Removed the unused `ghost_content` macro.
- Fixed: the company-name colour referenced an undefined `text_color` variable.

### Nourish - Text - Header Text
- Applied the Text Header standard: `heading_text` → `header_text`, `text_align` → `header_alignment`, `text_color` → `header_text_color`, `font_size` → `header_font_size`.
- All styling, including `text_padding`, is in `styles`.

### Nourish - Text - Rich Text
- `section_text` → `rich_text` (now has a default), `text_align` → `text_alignment`, and styling moved to `styles`.
- Fixes: the font-size fallback was 40 (copied from Header Text) and is now 14; the comments and SECTION markers said "Header text".

## Left as-is: your call

- **Naming:** by the skill's heuristic, the Speaker Grids have a wrapper and would be `Cards`. They weren't renamed because they're live (known issue #7).
- **Formation differences:** the two Speaker Grids lay out 6 and 7 speakers differently. Card uses 4+2 and 4+3 ungrouped; Standard uses 3+3, and 4+3 grouped.
- **List font:** List - Single Column uses Montserrat rather than FS Me. The constant keeps Montserrat.
- **Body text colour:** body text in the card modules is hardcoded `#000000`. There's no standard field for it.
- **Corner radius:** in Repeatable Items, the inner text cell has a 20px radius on the left corners only.
- **Nav Bar menu:** the checkbox ID is hardcoded, which matters only if two nav bars appear in one email.

## Suggested fixes to the skill itself

1. **`aliases_mapping` example:** the skill shows `"aliases_mapping": "old_field_name"`. HubSpot's actual format is the object above.
2. **Fixed-width button pattern:** it sets the link width to the full button width, which makes the button 52px too wide. It should be the table width minus 52.
3. **MSO checklist grep:** it miscounts MJML's `<!--[if !mso]><!-->` pattern, as used in the Nav Bar. It should count all `<!--[if` openers.
4. **Macro guidance:** the advice to avoid macros was overridden at your request.
