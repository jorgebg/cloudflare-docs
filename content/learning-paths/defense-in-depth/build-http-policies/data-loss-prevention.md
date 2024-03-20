---
title: Use Data Loss Prevention (DLP) inspection
pcx_content_type: learning-unit
weight: 1
layout: learning-unit
---

In order to use Data Loss Prevention (DLP) tools within Cloudflare Zero Trust, you first need to define your DLP profiles. [DLP profiles](/cloudflare-one/policies/data-loss-prevention/dlp-profiles/) are complex objects with dictionaries, pre-built detections, and custom logic that you can reference as selectors within your Gateway policies.

## Configure a DLP profile

You may either use DLP profiles predefined by Cloudflare, or create your own custom profiles based on regular expressions (regex), predefined detection entries, and DLP datasets.

### Configure a predefined profile

{{<render file="data-loss-prevention/_predefined-profile.md" productFolder="cloudflare-one">}}

### Build a custom profile

{{<render file="data-loss-prevention/_custom-profile.md" productFolder="cloudflare-one">}}

## Build effective DLP profiles

For many Cloudflare users, Cloudflare Zero Trust is often one of the only measures in tackling preventing the loss of sensitive data. For other users, Zero Trust may be the one of the early in-line measures of a complex orchestration of a defense-in-depth strategy. No matter which journey you most resemble, developing effective and appropriate DLP policies and practices starts with first-principles definitions.

### Define your sensitive data

#### Existing data patterns

If your organization is most concerned about general data patterns that fit existing classifications such as personal identifiable information (PII), protected health information (PHI), financial information, or source code, we recommend using the [default predefined profiles](#configure-a-predefined-profile).

To help this better match the needs of your organization, you can also build a complex profile that matches data to both an existing library and a custom string detection or database. For example:

{{<tabs labels="Dashboard | API">}}
{{<tab label="dashboard" no-code="true">}}

| Selector    | Operator | Value                     | Logic | Action |
| ----------- | -------- | ------------------------- | ----- | ------ |
| DLP Profile | in       | _Credentials and Secrets_ | Or    | Block  |
| DLP Profile | in       | _AWS Key Dataset_         |       |        |

{{</tab>}}
{{<tab label="api" no-code="true">}}

```bash
curl --request POST \
    --url https://api.cloudflare.com/client/v4/accounts/{account_id}/gateway/rules \
    --header 'Content-Type: application/json' \
    --header 'X-Auth-Email: <EMAIL>' \
    --header 'X-Auth-Key: <API_KEY>' \
    --data '{
    "action": "block",
    "description": "Detect secrets and AWS keys",
    "enabled": true,
    "filters": [
      "http"
    ],
    "name": "Secrets and AWS keys",
    "precedence": 0,
    "traffic": "any(dlp.profiles[*] in <CREDENTIALS_DLP_PROFILE_UUID>) or any(dlp.profiles[*] in <AWS_DLP_PROFILE_UUID>)""
    }'
```

{{</tab>}}
{{</tabs>}}

#### Assorted data patterns

If your data patterns take many different forms and contexts, consider building a custom profile using one or multiple regexes.

{{<Aside type="note" header="Rust regular expressions">}}
Cloudflare implements regular expressions with Rust. Make sure you account for this difference when writing expressions or using regular expression builders and generative AI.

To validate your regex, use [Rustexp](https://rustexp.lpil.uk/).
{{</Aside>}}

For example, you can use a custom expression to when your users share product SKUs in the format `CF1234-56789`:

{{<tabs labels="Dashboard | API">}}
{{<tab label="dashboard" no-code="true">}}

1. [Build a custom profile](#build-a-custom-profile) with the following custom entry:

  | Detection entry name | Value                   |
  | -------------------- | ----------------------- |
  | Product SKUs         | `CF[0-9]{1,4}-[0-9]{5}` |

2. Create an HTTP policy with the following expressions:

  | Selector    | Operator      | Value                        | Logic | Action |
  | ----------- | ------------- | ---------------------------- | ----- | ------ |
  | DLP Profile | in            | _Product SKUs_               | And   | Block  |
  | User Email  | matches regex | `[a-z0-9]{0,15}@example.com` |       |        |

{{</tab>}}

{{<tab label="api" no-code="true">}}

```bash
curl --request POST \
    --url https://api.cloudflare.com/client/v4/accounts/{account_id}/gateway/rules \
    --header 'Content-Type: application/json' \
    --header 'X-Auth-Email: <EMAIL>' \
    --header 'X-Auth-Key: <API_KEY>' \
    --data '{
    "action": "block",
    "description": "Detect product SKUs shared by users in organization",
    "enabled": true,
    "filters": [
      "http"
    ],
    "name": "Detect product SKU leaks",
    "precedence": 0,
    "traffic": "any(dlp.profiles[*] in <SKU_DLP_PROFILE_UUID>)",
    "identity": "identity.email matches \"[a-z0-9]{0,15}@example.com\""
    }'
```

{{</tab>}}
{{</tabs>}}

#### DLP datasets

If your data is a distinct [dataset](/cloudflare-one/policies/data-loss-prevention/datasets/) you have defined, you can build a profile by uploading a database to use in an Exact Data Match or Custom Wordlist function. Exact Data Match and Custom Wordlist feature some key differences:

|                     | Exact Data Match                                        | Custom Wordlist                                                    |
| ------------------- | ------------------------------------------------------- | ------------------------------------------------------------------ |
| **Encryption**      | Hashed and compared to encrypted traffic                | Stored as plaintext                                                |
| **Payload logging** | Matches redacted in logs                                | Matches appear in logs                                             |
| **Usage**           | PII (such as names, addresses, and credit card numbers) | Non-sensitive data (such as intellectual property and SKU numbers) |

We recommend using Exact Data Match for highly sensitive datasets and Custom Wordlists for lists of keywords.

As your datasets change and grow, we recommend building a pipeline to update the data source in the Zero Trust dashboard. For more information, contact your account team.

#### Microsoft Information Protection (MIP) labels

If your data already contains Microsoft Information Protection (MIP) labeling schema, Cloudflare can detect those values in-transit automatically. To get started, connect your Microsoft 365 account with a [CASB integration](/cloudflare-one/applications/scan-apps/casb-integrations/microsoft-365/). Cloudflare will automatically pull in your existing MIP definitions into Zero Trust. You can then use the MIP definitions to build DLP profiles for use in Gateway policies.

For more information, refer to [Integration profiles](/cloudflare-one/policies/data-loss-prevention/dlp-profiles/integration-profiles/).

## Build DLP policies

The best way to start applying data loss prevention to your traffic, minimize the chance of false positives, and collect actionable data is to start with the known knowns in your sensitive data policies. Rather than building policies to detect sensitive data like SSNs or financial information across all of your traffic, you should start by building policies that target both sensitive data types and destinations that are known data sources or points of high risk. These sources can be inside or outside your organization.

### Example

Many organizations want to detect and log financial information egressing from user devices to critical SaaS applications. To limit the risk of false positives and to filter out logging noise, Cloudflare recommends building your first series of policies to specify both target data and target destination. For example:

```bash
---
header: Block financial information shared with AI
---
curl --request POST \
    --url https://api.cloudflare.com/client/v4/accounts/{account_id}/gateway/rules \
    --header 'Content-Type: application/json' \
    --header 'X-Auth-Email: <EMAIL>' \
    --header 'X-Auth-Key: <API_KEY>' \
    --data '{
    "action": "block",
    "description": "Prevent financial information from being shared with AI tools",
    "enabled": true,
    "filters": [
      "http"
    ],
    "name": "Block AI financial info",
    "precedence": 0,
    "traffic": "any(dlp.profiles[*] in <FINANCIAL_INFO_DLP_PROFILE_UUID>) and any(http.request.uri.content_category[*] in {184})"
    }'
```

Once you have analyzed the flow and magnitude of data from the known sources, you can begin focusing on more specialized or explicit datasets for more generalized sources. This might look like inspecting for multiple matching data loss deterministic attributes to all external sources. You may want to except sources that are known internal locations where sensitive data is intentionally transferred. After developing a level of confidence from reviewing the logs and evaluating a rate of false positives for both types of policies, you can feel more confident in experimenting more broadly with data loss prevention policies.