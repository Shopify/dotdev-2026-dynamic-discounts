We are implementing Stage 2 of a Shopify Functions discount workshop app.

Start from the Stage 1 app. Stage 1 already has:
- a working multi-class discount Function
- a settings UI that saves `$app.function-configuration`
- product/order/shipping percentages
- tests and fixtures passing

Goal: add a shop-level snow gate. The discount should only apply when the shop metafield `custom.is_snowing` is true.

Make the smallest possible diff.

Required behavior:
- If `custom.is_snowing` is true, keep the Stage 1 behavior.
- If `custom.is_snowing` is false or missing, return no operations.
- Apply the snow gate to both Function targets:
  - `cart.lines.discounts.generate.run`
  - `cart.delivery-options.discounts.generate.run`
- Do not change the settings UI.
- Do not change the saved `$app.function-configuration` shape.
- Do not add an app-owned shop metafield definition for `is_snowing`; this is a merchant-owned `custom.*` metafield that will be created manually in Admin.

Implementation requirements:
1. In `extensions/snow-day-discount/src/cart_lines_discounts_generate_run.graphql`, read:

  shop {
    metafield(namespace: "custom", key: "is_snowing") {
      jsonValue
    }
  }

2. In `extensions/snow-day-discount/src/cart_delivery_options_discounts_generate_run.graphql`, read the same shop metafield.

3. Add a small helper if useful, for example in `src/config.ts`, that treats only boolean true as snowing.

4. In both Function implementations, return `{operations: []}` early when the snow gate is not true.

5. Keep existing config parsing and discount percentage behavior unchanged.

6. Update only the necessary test fixtures:
   - positive discount fixtures should include `shop.metafield.jsonValue: true`
   - add or update coverage proving `custom.is_snowing: false` returns no operations
   - keep the existing multiple-cart-line product discount coverage

7. Run:
  shopify app build
  cd extensions/snow-day-discount && npm test -- --run
  cd ../.. && git diff --check

Do not:
- rewrite `DiscountFunctionSettings.jsx`
- change locale files unless required
- add Flow, weather service code, theme code, or docs
- run `shopify app dev`

Report:
- exact files changed
- why each file changed
- validation results
