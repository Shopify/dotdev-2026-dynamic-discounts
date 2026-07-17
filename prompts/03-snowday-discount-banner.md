We are implementing Stage 3 of the workshop app.

Start from the Stage 2 app. Stage 2 already has:
- a multi-class discount Function
- a settings UI that saves `$app.function-configuration`
- a snow gate using `custom.is_snowing`

A theme app extension has already been generated. Find it by looking under `extensions/*/shopify.extension.toml` for `type = "theme"`. Use that extension's actual folder name below; do not assume a specific extension name.

Goal: add a Snowday Discount Banner storefront block to that theme app extension. The block appears only when `custom.is_snowing` is true and there is a real banner message to show.

Make the smallest possible diff.

Required changes:

1. Update the active Shopify app config.

Always update `shopify.app.toml`. If Shopify CLI created a local generated config such as `shopify.app.<name>.toml`, apply the same config change there too, because that may be the app config used by `shopify app dev`. Do not commit generated local config files.

Add this app-owned shop metafield definition:

  [shop.metafields.app.snow_day_banner_message]
  type = "single_line_text_field"
  access.admin = "merchant_read_write"
  access.storefront = "public_read"

Reason:
The discount settings UI writes the final banner message, and the theme app extension reads it on the storefront.

Do not add `custom.is_snowing` to any app config; it is merchant-owned custom data.

2. In the generated theme app extension folder you found, create a new block file:

  <theme-extension-folder>/blocks/snow_day_banner.liquid

Do not modify or delete generated sample blocks/snippets such as star rating files. They can remain unused; the merchant will add the `Snowday Discount Banner` block in the theme editor.

The new block should:
- have the merchant-facing schema name "Snowday Discount Banner"
- read `shop.metafields.custom.is_snowing.value`
- read `shop.metafields['$app']['snow_day_banner_message'].value`
- render nothing unless `custom.is_snowing` is true and the app-owned banner message is not blank
- display only the app-owned banner message; do not add a hardcoded fallback message or a block setting for message text
- keep styling minimal and self-contained

3. Update `extensions/snow-day-function-ui/src/DiscountFunctionSettings.jsx`.

When the existing settings UI saves `$app.function-configuration`, also write a derived shop metafield:

  namespace: "$app"
  key: "snow_day_banner_message"
  type: "single_line_text_field"

Use Admin GraphQL from the UI extension:
- query the shop ID
- call `metafieldsSet`

Build the message from the currently enabled discount classes and positive percentages:
- product class + product percentage > 0 → "<N>% off products"
- order class + order percentage > 0 → "<N>% off orders"
- shipping class + shipping percentage > 0:
  - if percentage >= 100 → "free shipping"
  - otherwise → "<N>% off shipping"

Build an array of enabled phrases and join them with ` + `.

Examples:
- one phrase: `❄️ Snow Day Sale: 10% off products`
- two phrases: `❄️ Snow Day Sale: 10% off products + free shipping`
- three phrases: `❄️ Snow Day Sale: 10% off products + 20% off orders + free shipping`

If no enabled class has a positive percentage, write a blank banner message so the theme block renders nothing.

Do not change the saved `$app.function-configuration` JSON shape.

4. Do not change:
- discount Function code
- discount Function GraphQL queries
- discount Function tests/fixtures
- `custom.is_snowing` ownership

Report:
- exact files changed
- the theme app extension folder you found
- whether you also updated a generated `shopify.app.<name>.toml` local config
