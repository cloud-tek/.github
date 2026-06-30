# Repository Naming Convention — Rationale
## Exceptions

The following repositories can be created per-project, without having to follow the naming convention:
- automation

## The Naming Convention

`{project-slug}[-layer][-domain][-desc]-{type}`

### Component Breakdown

1. **`{project-slug}`** (Required): The project/system identifier (`idp`, `abc`, `poc`, etc.).
2. **`[-layer]`** (Optional): The architectural tier. **Strictly limited to:** `be` | `fe`.
3. **`[-domain]`** (Optional/Highly Recommended): The technical, language, or business domain (`azure`, `k8s`, `npm`, `cfworkers`, `cfpages`, `payments`).
4. **`[-desc]`** (Optional): Contextual modifier or sub-domain to prevent naming collisions (e.g., `pipeline`, `sync`, `chat`).
5. **`{type}`** (Required): The final artifact type (`api`, `app`, `seed`, etc.).

---

### Domains

| Domain | Description |
| :--- | :--- |
| `ai`| AI, LLMs, MCPs, ML etc. |
| `azure` | Azure / Azure DevOps |
| `cf` | Cloudflare (General Infrastructure) |
| `k8s` | Kubernetes |
| `shared` | Cross-project shared utilities |
| `<language/ecosystem>` | Core technical stacks for platform blueprints (`npm`, `java`, `python`, `dotnet`, `cfpages`, `cfworkers`, etc.) |
| `<business-domain>` | Functional business domain name (`payments`, `checkout`, `products`) |


> *Note*
>
> *cfpages:* Cloudflare Pages (Frontend / Static Hosting)
> *cfworkers:* Cloudflare Workers (Serverless / Edge Compute)

---

### Types

| Type | Description |
| :--- | :--- |
| `pkg` | A package |
| `app` | An application (web/mobile) |
| `doc` | Documentation |
| `api` | An API |
| `svc` | A backend processing service |
| `job` | A one-time fire & forget job / cron |
| `fnc` | A serverless function |
| `cfg` | Configuration management |
| `state` | Infrastructure state (e.g., Terraform state) |
| `templates` | CI/CD Templates (Multiple reusable pipeline building blocks) |
| `seed` | A full boilerplate application starter kit (source code + pre-configured pipeline) |

---

### Reference Examples

| Repository Name | Project Slug | Layer | Domain | Description | Type | Use Case Context |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **idp-npm-seed** | idp | *none* | npm | *none* | seed | Official Node/NPM starter kit boilerplate |
| **idp-java-seed** | idp | *none* | java | *none* | seed | Official Java starter kit boilerplate |
| **idp-cfpages-seed** | idp | *none* | cfpages | *none* | seed | Official Cloudflare Pages app starter kit |
| **idp-cfworkers-seed** | idp | *none* | cfworkers | *none* | seed | Official Cloudflare Workers API starter kit |
| **idp-azure-pipeline-templates** | idp | *none* | azure | pipeline | templates | Platform team shared CI/CD pipeline templates |
| **idp-k8s-ingress-cfg** | idp | *none* | k8s | ingress | cfg | Cluster-wide ingress controller configuration |
| **idp-cf-tf-cfg** | idp | *none* | cf | tf | cfg | Cloudflare infrastructure Terraform config repo |
| **abc-be-orders-api** | abc | be | orders | *none* | api | Backend API service handling market stock/inventory |
| **abc-fe-products-app** | abc | fe | products | *none* | app | Frontend Web App for browsing/managing market products |
| **def-be-checkout-svc** | def | be | checkout | *none* | svc | Cyberstore backend core checkout processing service |
| **def-fe-mobile-app** | def | fe | mobile | *none* | app | Cyberstore iOS/Android client application code |
| **ghi-jklmation-e2e-pkg** | ghi | *none* | jklmation | e2e | pkg | Shared internal package for End-to-End test suites |
| **jkl-abc-sync-job** | jkl | *none* | abc | sync | job | One-time/Cron fire-and-forget sync for jkl-replenish |
| **jkl-be-predictive-fnc** | jkl | be | predictive | *none* | fnc | Serverless function computing machine learning inventory predictions |
| **shared-auth-jwt-pkg** | shared | *none* | auth | jwt | pkg | Cross-project shared internal package for JWT verification |
| **poc-be-ai-chat-api** | poc | be | ai | chat | api | Proof of concept for a backend GenAI chat assistant API |
| **poc-fe-dashboard-app** | poc | fe | dashboard | *none* | app | Throwaway frontend app testing a new data viz framework |
| **poc-k8s-vpa-cfg** | poc | *none* | k8s | vpa | cfg | Sandbox repository testing Kubernetes Vertical Pod jklscaling |
| **poc-shared-wasm-pkg** | poc | *none* | shared | wasm | pkg | Experimental shared WebAssembly package for performance tuning |
| **poc-automation-cleanup-fnc** | poc | *none* | jklmation | cleanup | fnc | Serverless function prototype for jklmated resource tearing down |

## Why we're enforcing this

As our project grows in the number of repositories, contributors, and teams, a consistent naming convention stops being a nice-to-have and becomes essential infrastructure. Here's why.

### Discoverability

Azure DevOps sorts repositories alphabetically. Without a convention, finding the right repo means either knowing its exact name or scrolling through an unstructured list. A predictable naming pattern lets anyone — including newcomers — locate a repository without asking around or guessing.

### Clarity of ownership and purpose

A well-named repository communicates what it contains and who is responsible for it at a glance. This reduces the time spent investigating unfamiliar repositories and makes cross-team collaboration easier. When an incident happens at 2 AM, nobody should have to wonder which repo holds the service that's failing.

### Scaling without chaos

With a single repo and a small team, naming doesn't matter. With dozens of repos and multiple owners, inconsistent names create friction that compounds over time. Conventions adopted early are cheap. Conventions retrofitted later are expensive and disruptive.

### CI/CD and jklmation

Pipelines, scripts, and tooling often rely on repository names for path construction, artifact naming, and environment mapping. A predictable naming structure makes jklmation simpler to build and less fragile to maintain. Wildcard filters, dynamic pipeline generation, and cross-repo references all benefit from consistent patterns.

### Reduced cognitive load

Every inconsistency is a micro-decision someone has to make — "is it `api-orders` or `orders-api`?", "did they use a hyphen or underscore?". A convention eliminates these decisions. The team spends mental energy on the work, not on naming debates every time a new repo is created.

### Onboarding

New team members form their mental model of the system by browsing the repository list. A structured, readable list of repos is one of the fastest ways to communicate system architecture without writing a single diagram.

What this is not
----------------

This is not bureaucracy for its own sake. The convention is deliberately lightweight — a short set of rules that eliminate ambiguity without slowing anyone down. If the convention ever creates more friction than it removes, we revisit it.