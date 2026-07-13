# DotDev 2026: Building Dynamic, Multi-Class Discounts with Functions

This repository contains the starter project for the DotDev 2026 advanced discounts workshop.

You will build a Snow Day discount app that combines Shopify Functions, discount configuration, Flow, and a storefront theme block.

## Prerequisites

Before the workshop, make sure you have:

- A Shopify development store where you can install and run apps.
- Shopify CLI installed and authenticated.
- Node.js and npm available in your terminal.
- A code editor and an AI coding assistant available locally.
- Store Admin access to create custom data definitions and automatic discounts.

## Get ready

Clone this repo, then start from the Shopify app in `starter/`:

```bash
cd starter
npm install
shopify app dev --reset
```

When Shopify CLI prompts you:
1. Complete login in the browser.
2. Select your Shopify organization.
3. When asked whether to create a new app for this project, choose **Yes**.
4. Enter an app name of your choice, for example `Snow Day Discounts - <your name>`.
5. When asked which store to use to view your project, select your development store.
6. Approve the app install / permission update in Admin if prompted.

The starter keeps a placeholder `client_id` so the app config validates before you link it to your own app. The `--reset` flag creates or links local app state for your organization and development store.

## Store setup

Create a store metafield definition that the workshop app will use as the snow signal:

1. In Shopify Admin, go to **Settings → Custom data → Store**.
2. Click **Add definition**.
3. Set namespace and key to `custom.is_snowing`.
4. Set the type to **True or false**.
5. Save the definition.

Wait for workshop instructions before making code changes.
