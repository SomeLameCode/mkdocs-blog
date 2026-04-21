---
title: Diagram Examples
description: Mermaid diagram examples — flowcharts and sequence diagrams rendered in MkDocs Material.
date: 2026-01-16
tags:
  - mkdocs
  - mermaid
  - diagrams
  - examples
status: published
---

# Diagram Examples

??? note "How to enable this"

    Install the plugin and add the following to `mkdocs.yml`:

    ```bash
    pip install mkdocs-mermaid2-plugin
    ```

    ```yaml
    plugins:
      - mermaid2

    markdown_extensions:
      - pymdownx.superfences:
          custom_fences:
            - name: mermaid
              class: mermaid
              format: !!python/name:pymdownx.superfences.fence_code_format
    ```

    Then wrap diagram code in a fenced block with the `mermaid` language tag.

## Flowcharts

```mermaid
graph LR
  A[Start] --> B{Failure?};
  B -->|Yes| C[Investigate...];
  C --> D[Debug];
  D --> B;
  B ---->|No| E[Success!];
```


## Sequence Diagrams

```mermaid
sequenceDiagram
  autonumber
  Server->>Terminal: Send request
  loop Health
      Terminal->>Terminal: Check for health
  end
  Note right of Terminal: System online
  Terminal-->>Server: Everything is OK
  Terminal->>Database: Request customer data
  Database-->>Terminal: Customer data
```

[diagrams documentation](https://squidfunk.github.io/mkdocs-material/reference/diagrams/#using-state-diagrams)
