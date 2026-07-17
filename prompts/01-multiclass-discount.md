We are implementing Stage 1 of a Shopify Functions discount workshop app.

Goal: make the generated starter app into a working multi-class automatic discount while keeping the diff as small as possible.

Important constraint: do not redesign or rewrite the settings UI. The generated UI may already save the desired `$app.function-configuration` JSON shape. Inspect it first. If it already saves cartLinePercentage, orderPercentage, deliveryPercentage, and collectionIds, leave the UI file unchanged.

Make only the changes required below.

1. Update the active Shopify app config.

Always update `shopify.app.toml`. If Shopify CLI created a local generated config such as `shopify.app.<name>.toml`, apply the same config changes there too, because that may be the app config used by `shopify app dev`. Do not commit generated local config files.

Set access scopes to:

  write_discounts,read_products,read_markets

Reason:
- `write_discounts` lets the app manage discount Functions.
- `read_products` supports the generated collection picker in the settings UI.
- `read_markets` supports the workshop’s market-specific discount setup.

Add the discount metafield definition:

  [discount.metafields.app.function-configuration]
  type = "json"
  access.admin = "merchant_read_write"

Reason:
The settings UI saves merchant configuration into `$app.function-configuration`, and both Function targets need to read it.

2. Update `extensions/snow-day-discount/src/cart_lines_discounts_generate_run.graphql`.

Add this under `discount`:

  metafield(namespace: "$app", key: "function-configuration") {
    jsonValue
  }

Keep only fields the Function actually needs. Since product discounts should target every cart line, cart line `id` is needed. Cart line subtotal is not needed unless your implementation still uses it.

3. Update `extensions/snow-day-discount/src/cart_delivery_options_discounts_generate_run.graphql`.

Add this under `discount`:

  metafield(namespace: "$app", key: "function-configuration") {
    jsonValue
  }

4. Add minimal shared config parsing for the Function code.

Preferred: create a small file at:

  extensions/snow-day-discount/src/config.ts

It should parse `discount.metafield?.jsonValue` into:

  {
    cartLinePercentage: number,
    orderPercentage: number,
    deliveryPercentage: number
  }

Rules:
- Missing config means all percentages are 0.
- Invalid, non-finite, negative, or 0 values mean no discount.
- Clamp values above 100 to 100.
- Ignore `collectionIds` for Stage 1. The UI can keep saving it, but the Function should not implement collection filtering yet.

5. Update `extensions/snow-day-discount/src/cart_lines_discounts_generate_run.ts`.

Behavior:
- If cart has no lines, return no operations.
- If product class is enabled and `cartLinePercentage > 0`, apply that percentage to every cart line.
- If order class is enabled and `orderPercentage > 0`, apply that percentage to the order subtotal.
- Product and order discount messages should be `❄️ Snow Day - <N>% off`.
- If a class is disabled or its percentage is 0, do not create that operation.
- Do not choose only the highest-priced cart line.
- Keep the implementation simple and close to the generated starter style.

6. Update `extensions/snow-day-discount/src/cart_delivery_options_discounts_generate_run.ts`.

Behavior:
- If there is no delivery group, return no operations.
- If shipping class is enabled and `deliveryPercentage > 0`, apply that percentage to the first delivery group.
- Shipping discount message should be `❄️ Snow Day - <N>% off delivery`.
- 100 should work as free shipping.
- Do not hardcode all shipping discounts to 100.
- If shipping class is disabled or percentage is 0, return no operations.

7. Update only the necessary test fixtures.

The fixture-driven tests should pass. Add or update a fixture that proves product discount targets multiple cart lines. Add zero-percentage coverage only if it fits the existing fixture style.

Do not:
- Rewrite `DiscountFunctionSettings.jsx` unless absolutely necessary.
- Remove the generated collection picker.
- Change locale files unless needed.
- Add unrelated abstractions.
- Add new workshop stages or documentation.
- Run `shopify app dev`.

After changes, run:

  shopify app build
  cd extensions/snow-day-discount && npm test -- --run
  cd ../.. && git diff --check

Report:
- exact files changed
- why each file changed
- validation results
