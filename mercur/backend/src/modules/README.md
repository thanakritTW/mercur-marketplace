# Custom Module

A module is a package of reusable functionalities. It can be integrated into your Medusa application without affecting the overall system. You can create a module as part of a plugin.

> Learn more about modules in [this documentation](https://docs.medusajs.com/learn/fundamentals/modules).

To create a module:

## 1. Create a Data Model

A data model represents a table in the database. You create a data model in a TypeScript or JavaScript file under the `models` directory of a module.

For example, create the file `src/modules/blog/models/post.ts` with the following content:

```ts
import { model } from "@medusajs/framework/utils"

const Post = model.define("post", {
  id: model.id().primaryKey(),
  title: model.text(),
})

export default Post
```

## 2. Create a Service

A module must define a service. A service is a TypeScript or JavaScript class holding methods related to a business logic or commerce functionality.

For example, create the file `src/modules/blog/service.ts` with the following content:

```ts
import { MedusaService } from "@medusajs/framework/utils"
import Post from "./models/post"

class BlogModuleService extends MedusaService({
  Post,
}){
}

export default BlogModuleService
```

## 3. Export Module Definition

A module must have an `index.ts` file in its root directory that exports its definition. The definition specifies the main service of the module.

For example, create the file `src/modules/blog/index.ts` with the following content:

```ts
import BlogModuleService from "./service"
import { Module } from "@medusajs/framework/utils"

export const BLOG_MODULE = "blog"

export default Module(BLOG_MODULE, {
  service: BlogModuleService,
})
```

## 4. Add Module to Medusa's Configurations

To start using the module, add it to `medusa-config.ts`:

```ts
module.exports = defineConfig({
  projectConfig: {
    // ...
  },
  modules: [
    {
      resolve: "./src/modules/blog",
    },
  ],
})
```

## 5. Generate and Run Migrations

To generate migrations for your module, run the following command:

```bash
npx medusa db:generate blog
```

Then, to run migrations, run the following command:

```bash
npx medusa db:migrate
```

## Use Module

You can use the module in customizations within the Medusa application, such as workflows and API routes.

For example, to use the module in an API route:

```ts
import { MedusaRequest, MedusaResponse } from "@medusajs/framework"
import BlogModuleService from "../../../modules/blog/service"
import { BLOG_MODULE } from "../../../modules/blog"

export async function GET(
  req: MedusaRequest,
  res: MedusaResponse
): Promise<void> {
  const blogModuleService: BlogModuleService = req.scope.resolve(
    BLOG_MODULE
  )

  const posts = await blogModuleService.listPosts()

  res.json({
    posts
  })
}
```

---

## Extending an Existing Model

You **cannot** change core or plugin models directly (they live in `node_modules`). To add custom fields to an existing entity (e.g. Customer, Product), you **extend** it by linking a custom model.

### 1. Create a custom model with the extra fields

In your module, define a new model that holds only the fields you want to add:

```ts
// src/modules/hello/models/custom.ts
import { model } from "@medusajs/framework/utils"

export const Custom = model.define("custom", {
  id: model.id().primaryKey(),
  custom_name: model.text(),
})
export default Custom
```

### 2. Define a link to the existing entity

Create a **module link** in `src/links/` so the custom model is related to the core entity (e.g. Customer):

```ts
// src/links/customer-custom.ts
import { defineLink } from "@medusajs/framework/utils"
import HelloModule from "../modules/hello"
import CustomerModule from "@medusajs/medusa/customer"

export default defineLink(
  CustomerModule.linkable.customer,
  HelloModule.linkable.custom
)
```

Then run `npx medusa db:migrate` so the link table is created.

### 3. Use workflow hooks to keep data in sync

When the core entity is created or updated, create/update the linked custom record by consuming workflow hooks (e.g. `createCustomersWorkflow.hooks.customersCreated`, `updateCustomersWorkflow.hooks.customersUpdated`). Pass your extra data via the request body in `additional_data`.

### 4. Validate and accept custom data on API routes

Use `defineMiddlewares` in `src/api/middlewares.ts` so existing routes accept your fields in `additional_data`:

```ts
additionalDataValidator: {
  custom_name: z.string().optional(),
}
```

### 5. Load extended data when querying

Use Query with the link name to fetch the entity plus your custom fields:

```ts
const { data } = await query.graph({
  entity: "customer",
  fields: ["*", "custom.*"],
  filters: { id: customer_id },
})
```

Full step-by-step (including workflows and hooks) is in the [Medusa docs: Extend Customer Data Model](https://docs.medusajs.com/resources/commerce-modules/customer/extend). The same pattern applies to other entities (Product, Order, etc.) using their workflows and linkables.
```
