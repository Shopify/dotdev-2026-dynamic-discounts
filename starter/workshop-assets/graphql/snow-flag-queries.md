# Snow flag Admin GraphQL snippets

Use these in Admin GraphiQL during the Snow Day workshop to read or force the merchant-owned Store metafield `custom.is_snowing`.

If `shopify app dev` is running, press `g` in that terminal to open Admin GraphiQL for your development store.

Before running the mutations, create the Store custom data definition in Admin:

- Namespace and key: `custom.is_snowing`
- Type: `True or false`

## 1. Get your shop ID

Copy the full `shop.id` value from the result. It should look like `gid://shopify/Shop/123456789`.

```graphql
query GetShopId {
  shop {
    id
    name
    myshopifyDomain
  }
}
```

## 2. Read the current snow flag

```graphql
query ReadSnowFlag {
  shop {
    id
    name
    myshopifyDomain
    metafield(namespace: "custom", key: "is_snowing") {
      id
      namespace
      key
      type
      value
      jsonValue
    }
  }
}
```

## 3. Set snow to true

Replace `gid://shopify/Shop/YOUR_SHOP_ID` with the full `shop.id` from `GetShopId`.

```graphql
mutation SetSnowFlagTrue {
  metafieldsSet(metafields: [
    {
      ownerId: "gid://shopify/Shop/YOUR_SHOP_ID"
      namespace: "custom"
      key: "is_snowing"
      type: "boolean"
      value: "true"
    }
  ]) {
    metafields {
      id
      namespace
      key
      type
      value
      jsonValue
    }
    userErrors {
      field
      message
    }
  }
}
```

## 4. Set snow to false

Replace `gid://shopify/Shop/YOUR_SHOP_ID` with the full `shop.id` from `GetShopId`.

```graphql
mutation SetSnowFlagFalse {
  metafieldsSet(metafields: [
    {
      ownerId: "gid://shopify/Shop/YOUR_SHOP_ID"
      namespace: "custom"
      key: "is_snowing"
      type: "boolean"
      value: "false"
    }
  ]) {
    metafields {
      id
      namespace
      key
      type
      value
      jsonValue
    }
    userErrors {
      field
      message
    }
  }
}
```
