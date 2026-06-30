# Metadata Management

Welcome to the central repository for platform metadata. This page serves as the single source of truth for defining, managing, and tracking all official project identities, ownership, and structural configurations across our engineering ecosystem.

> _Standardizing project names and short identifiers (often called "slugs") is a foundational step toward building a seamless **Internal Developer Platform (IDP)**. Without this central source of truth, organizations quickly devolve into "naming chaos," where the same project is named one thing in GitHub, another in Jira, and something entirely different in Datadog, making automated tracing and governance nearly impossible._

## Project Slugs

### What is a Project Slug?

A **Project Slug** is a unique, 2-to-5 character alphanumeric identifier assigned to a project at the time of its creation (for example, `pymt` for a payment gateway or `auth` for an authentication service).

#### What is it used for?

The slug acts as a permanent, immutable short-code that ties all distributed resources of a project together. By serving as the universal anchor for your project, it is automatically injected into:
*   **Infrastructure:** Cloud resource names, Kubernetes namespaces, and storage buckets.

*   **Repositories & Artifacts:** GitHub repository names and Docker image tags.

*   **Observability:** Metric tags, log filters, and dashboard definitions in monitoring tools.

Using a standardized, short identifier ensures a consistent developer experience and enables seamless end-to-end tracing across our entire platform.

> **Note**
> Legacy projects are exempt from having their infrastructure resource names altered as this is a potentially hazardous action. Greenfield platform projects MUST adhere to the platform rules

## Projects

|Project Name               |Project Slug    |Notes |
|---------------------------|----------------|------|
|Internal Developer Platform|idp             |N/A   |
|Proof of concept           |poc             |N/A   |
|Operations                 |ops             |N/A   |
|AI                         |ai              |N/A   |
|Villy                      |villy           |N/A   |


