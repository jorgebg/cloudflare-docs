---
_build:
  publishResources: false
  render: never
  list: never
---

When developing your Worker or Pages Function, create a `.dev.vars` file in the root of your project to define secrets that will be used when running `wrangler dev` or `wrangler pages dev`, as opposed to using [environment variables in `wrangler.toml`](/workers/configuration/environment-variables/#compare-secrets-and-environment-variables). This works both in local and remote development modes.

The `.dev.vars` file should be formatted like a `dotenv` file, such as `KEY=VALUE`:

```bash
---
header: .dev.vars 
---
SECRET_KEY=value
API_TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```