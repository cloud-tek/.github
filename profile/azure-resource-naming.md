# Azure Resource Naming Convention

Naming rules for cloud-tek Azure resources. This document is language-agnostic and is
the spec for an HCL (Terraform) implementation.

## Design goals

1. **Consistent** — every name is assembled from the same segments in the same order.
2. **Immune to per-resource rules** — resource types that forbid hyphens (Storage
   Account, Container Registry) change only the *rendering* of the name, never its
   structure or information content.
3. **Clash-avoidant** — a bounded free segment (`qualifier`), a `target` segment for
   satellite resources, and a deterministic uniqueness token for globally-unique types.
4. **Simple** — one assembly rule, one join rule, one validation step. No exception
   lists.

Names identify resources; **tags** (`project`, `environment`, `module`, `owner`) are
the machine-readable source of truth. Names are not required to be mechanically
parseable.

---

## Pattern

A name is an **ordered list of segments**:

```
[ project, environment, module, target?, qualifier?, region, type ]
```

The rendered name is:

```
body = {project}{environment}{module}[target][qualifier]{region}
name = lower( body + "-" + type )    # types that allow hyphens (default)
name = lower( body + type )          # no-hyphen types (see table)
```

- The **only** per-type variation is the join character before `type`: `-` or nothing.
- The entire name is lower-cased.
- `target` and `qualifier` are the only optional segments; `target` may be used only
  by **satellite types** (see below). All other segments always appear, for every
  resource type — including resource groups and global resources.

## Segments

| Segment       | Required | Length | Charset    | Rules |
|---------------|----------|--------|------------|-------|
| `project`     | yes      | 3–5    | `[a-z0-9]` | Must start with a letter. |
| `environment` | yes      | 3      | enum       | One of the environment codes below. No other values, ever. |
| `module`      | yes      | 3–5    | `[a-z0-9]` | Business module / workload within the project. |
| `target`      | no       | 2–10   | `[a-z0-9]` | **Satellite types only.** Type code of the associated resource, plus that resource's qualifier if it has one. |
| `qualifier`   | no       | 1–4    | `[a-z0-9]` | Optional disambiguator. Empty by default. |
| `region`      | yes      | 3      | enum       | Region short code. `glb` for global resources. |
| `type`        | yes      | 2–6    | enum       | Resource-type code. Always last. |

`owner`, `tenant id`, and `subscription id` exist in the naming context but are **not**
part of any name. `owner` is applied as a tag; the ids are used for scoping and for
deriving the uniqueness token.

### Qualifier

The qualifier is the degree of freedom for avoiding clashes. Typical uses:

- **Instance numbers / purposes** — `01`, `02`, or semantic: `data`, `logs`.
- **Deployment variants** — `blue`, `grn`, `cnry`.
- **Uniqueness token** — for **globally-unique types** (Storage Account, Container
  Registry, Key Vault, Cosmos DB account, Web App, Function App, Service Bus and
  Event Hub namespaces), the qualifier may carry a deterministic 4-char token derived
  from the subscription id (e.g. first 4 chars of a hash). This resolves cross-tenant
  name collisions without manual invention and is stable across Terraform runs.

A resource uses at most one qualifier value; if both a variant and a token are needed,
the module layout is too crowded — split the module.

### Target (satellite resources)

Some top-level resources exist only in association with another resource: private
endpoints, network interfaces, metric alerts, activity log alerts. These **satellite
types** may carry a `target` segment identifying the resource they point at:

- `target` = the target's **type code**, plus the target's **qualifier** if it has one
  (e.g. `sbns`, `st02`, `cosno`).
- `region` remains the **satellite's own region**, never the target's. This is what
  makes multi-region private endpoints to a single resource unambiguous.
- `target` is permitted only on types marked **satellite** in the type table; on all
  other types its presence is a validation error.

Satellite types all have generous Azure limits (64–260 chars), so the extra segment
never threatens the length budget: worst case `5+3+5+10+4+3` body `+ "-pep"` = 34.
The 24-char types (Storage Account, Key Vault) are never satellites.

## Environment codes

| Code  | Environment           | Purpose |
|-------|-----------------------|---------|
| `poc` | Proof of concept      | Throwaway experiments and spikes; not a promotion target. |
| `ops` | Operations / IaC      | Shared platform and infrastructure-as-code resources not tied to a single app stage. |
| `dev` | Development           | Day-to-day engineering environment. |
| `tst` | Test                  | Automated and integration testing. |
| `prf` | Performance testing   | Load, stress, and performance benchmarking. |
| `stg` | Staging               | Production-like pre-release validation. |
| `prd` | Production            | Live, customer-facing environment. |

The environment code is exactly 3 characters and comes from this closed set. New
environments are added here as 3-char codes before they may appear in a name.

## Region short codes

| Region          | Code  |
|-----------------|-------|
| West Europe     | `euw` |
| North Europe    | `eun` |
| Poland Central  | `plc` |
| Sweden Central  | `swc` |
| Global          | `glb` |

Every resource carries a region code, including region-agnostic types — `glb` keeps
the shape uniform. New regions are added here as 3-char codes.

## Resource-type codes

Codes follow the Microsoft Cloud Adoption Framework (CAF) abbreviations where CAF
defines one. Codes marked **ext** are cloud-tek extensions in the same style, for
types CAF does not cover. Types marked **no-hyphen** join their code without a
separator. Types marked **satellite** may carry a `target` segment.

| Resource type              | Code     | Source | Notes |
|----------------------------|----------|--------|-------|
| Resource Group             | `rg`     | CAF    | |
| Virtual Network            | `vnet`   | CAF    | |
| Network Security Group     | `nsg`    | CAF    | |
| Route Table                | `rt`     | CAF    | CAF reserves `udr` for routes *within* a table. |
| Network Interface          | `nic`    | CAF    | **Satellite.** |
| Private Endpoint           | `pep`    | CAF    | **Satellite.** |
| Key Vault                  | `kv`     | CAF    | Globally unique. 24-char limit — **design ceiling**, see Validation. |
| User Assigned Identity     | `id`     | CAF    | |
| Container Registry         | `cr`     | CAF    | **No-hyphen.** Globally unique. |
| Storage Account            | `st`     | CAF    | **No-hyphen.** Globally unique. 24-char limit. |
| Log Analytics Workspace    | `log`    | CAF    | |
| Application Insights       | `appi`   | CAF    | |
| Service Bus Namespace      | `sbns`   | CAF    | Globally unique. Requires leading letter. |
| Service Bus Queue          | `sbq`    | CAF    | Scoped to namespace. |
| Service Bus Topic          | `sbt`    | CAF    | Scoped to namespace. |
| Event Hub Namespace        | `evhns`  | CAF    | Globally unique. Requires leading letter. |
| Event Hub                  | `evh`    | CAF    | Scoped to namespace; hyphens legal, standard type. |
| Event Hub Auth Rule        | `evhar`  | ext    | Scoped to parent. |
| Cosmos DB Account (NoSQL)  | `cosno`  | CAF    | Globally unique. CAF distinguishes account kinds. |
| Cosmos SQL Database        | `cosmos` | CAF    | CAF assigns `cosmos` to the *database*, not the account. |
| Cosmos SQL Container       | `coscon` | ext    | Scoped to database. |
| App Service Plan           | `asp`    | CAF    | |
| Web App                    | `app`    | CAF    | Globally unique (default hostname). See caveats. |
| Function App               | `func`   | CAF    | Same ARM type as Web App (`Microsoft.Web/sites`, kind `functionapp`); same rules. Globally unique. See caveats. |
| Container App              | `ca`     | CAF    | 2–32 chars, lowercase/numbers/hyphens, leading letter. Worst case 23 ✓. |
| Container Apps Environment | `cae`    | CAF    | |
| Action Group               | `ag`     | CAF    | |
| Metric Alert               | `mal`    | ext    | Distinct from Activity Log Alert. **Satellite.** |
| Activity Log Alert         | `alal`   | ext    | **Satellite.** |
| Management Lock            | `lck`    | ext    | |

**Role Assignment is intentionally absent.** ARM names role assignments with GUIDs and
Terraform generates them; they are excluded from this policy.

New types: use the CAF code if one exists
(<https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations>);
otherwise define an `ext` code here, 2–6 chars, unique in this table.

### Web App & Function App caveats

- Web App and Function App names form their public URL (`<name>.azurewebsites.net`).
  Azure rejects names of public-endpoint resources containing **reserved words or
  trademarks**. Such a rejection is an Azure-side refusal of an otherwise policy-valid
  name, not a policy bug; choose a different `module` or `qualifier` value.
- **Function Apps (`func`)**: Azure truncates the function app name to 32 characters
  when generating its host ID; apps sharing a storage account whose names agree on
  the first 32 characters can collide. This convention's names stay under 32
  characters, so no collision is possible while segment bounds hold — noted here
  because the safety is incidental, not enforced.

### Virtual Machines (pre-emptive carve-out)

VMs are currently out of scope, but the convention **cannot** produce a valid Windows
VM host name: host names are capped at 15 characters (Linux: 64), the portal and
default Terraform usage set host name = resource name, and this convention's shortest
possible name is 17 characters. When VMs enter scope:

- The **resource name** follows the full convention (resource names allow 64 chars),
  e.g. `{project}devappswc-vm`.
- The **host name** is set explicitly (Terraform: `computer_name`) to a compressed
  form: `{project}{environment}[qualifier]{nn}` ≤ 15 chars, where `nn` is a two-digit
  instance number, e.g. `{project}dev01`.

## Validation

Validation runs in two stages. **An invalid context or an invalid rendered name is a
hard error, never a best-effort name.**

1. **Segment validation** — every segment present (`target`/`qualifier` optional),
   within its length bounds, matching its charset or enum; `target` present only on
   satellite types.
2. **Rendered-name validation** — the assembled name is checked against a per-type
   map of Azure constraints: max length, allowed characters, first-character rules.

With the segment bounds above, the worst cases clear Azure's tightest limits by
design:

- Storage Account (24 max, no hyphens): `5+3+5+4+3+2 = 22` ✓
- Key Vault (24 max, incl. hyphen): `5+3+5+4+3` body `+ "-kv"` `= 23` ✓
- Container App (32 max): `20 + "-ca" = 23` ✓
- Satellites (64–260 max): `5+3+5+10+4+3 + "-pep" = 34` ✓

> **Design ceiling: Key Vault.** At 23 of 24 characters worst case (the hyphen counts
> against the limit), Key Vault — not Storage Account — is the tightest fit in the
> convention, with exactly one character of headroom. **Any change to a segment
> length bound, or to the `kv` type code, must re-verify this arithmetic first.**

Two structural invariants also hold by construction and are relied upon by several
resource types:

- Names never contain consecutive hyphens and never start or end with a hyphen
  (single-join rendering over `[a-z0-9]` segments).
- Every name starts with a letter (the `project` first-character rule — load-bearing
  for Key Vault, Service Bus Namespace, and Event Hub Namespace, which all require a
  leading letter).

Stage 2 should therefore never fire — it exists as the safety net that lets the type
table grow without re-deriving the arithmetic by hand.

## Worked examples

Context: `project={project}`, `environment=dev`, `module=app`, `region=Sweden Central
(swc)`, no target/qualifier unless stated.

### Standard types

| Resource                   | Name                    |
|----------------------------|-------------------------|
| Resource Group             | `{project}devappswc-rg`     |
| Virtual Network            | `{project}devappswc-vnet`   |
| Network Security Group     | `{project}devappswc-nsg`    |
| Key Vault                  | `{project}devappswc-kv`     |
| Log Analytics Workspace    | `{project}devappswc-log`    |
| Application Insights       | `{project}devappswc-appi`   |
| Service Bus Namespace      | `{project}devappswc-sbns`   |
| Service Bus Queue          | `{project}devappswc-sbq`    |
| Event Hub Namespace        | `{project}devappswc-evhns`  |
| Event Hub                  | `{project}devappswc-evh`    |
| Cosmos DB Account (NoSQL)  | `{project}devappswc-cosno`  |
| App Service Plan           | `{project}devappswc-asp`    |
| Web App                    | `{project}devappswc-app`    |
| Function App               | `{project}devappswc-func`   |
| Container App              | `{project}devappswc-ca`     |
| Container Apps Environment | `{project}devappswc-cae`    |
| Action Group               | `{project}devappswc-ag`     |
| Management Lock            | `{project}devappswc-lck`    |

### No-hyphen types

| Resource           | Name               |
|--------------------|--------------------|
| Storage Account    | `{project}devappswcst` |
| Container Registry | `{project}devappswccr` |

### With qualifier

| Resource                       | Qualifier | Name                     |
|--------------------------------|-----------|--------------------------|
| Web App (green variant)        | `grn`     | `{project}devappgrnswc-app`  |
| Storage Account (2nd instance) | `02`      | `{project}devapp02swcst`     |
| Storage Account (unique token) | `x7k2`    | `{project}devappx7k2swcst`   |

### Satellites (with target)

| Resource                          | Target  | Name                       |
|-----------------------------------|---------|----------------------------|
| PE → SB namespace, Sweden Central | `sbns`  | `{project}devappsbnsswc-pep`   |
| PE → SB namespace, West Europe    | `sbns`  | `{project}devappsbnseuw-pep`   |
| PE → Storage `02`, West Europe    | `st02`  | `{project}devappst02euw-pep`   |
| Metric Alert → Cosmos account     | `cosno` | `{project}devappcosnoswc-mal`  |

### Global resources

`module=dns`, region `glb`: `{project}devdnsglb-rt` — the shape never changes.

## Tagging (companion policy)

Because segments concatenate without separators, names are not reliably parseable
back into segments. Every resource therefore carries these tags as the queryable
source of truth:

| Tag           | Value                         |
|---------------|-------------------------------|
| `project`     | full project name             |
| `environment` | one of the environment codes   |
| `module`      | module name                   |
| `owner`       | owning team or person         |

Cost queries, security tooling, and automation key off tags, never off name parsing.

## Open items

- **Child naming** — namespace-scoped children (SB queues/topics, Event Hubs, Cosmos
  databases/containers, auth rules) currently follow the full standard pattern, which
  repeats the parent's context (`{project}devappswc-sbq` inside `{project}devappswc-sbns`). A
  proposed alternative — semantic child names (`orders-sbq`, `catalog-cosmos`) scoped
  to the parent — is under consideration and not yet adopted.