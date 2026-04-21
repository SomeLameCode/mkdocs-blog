---
title: Admonitions
description: Examples of MkDocs Material admonition callout boxes and collapsible details blocks.
date: 2026-01-16
tags:
  - mkdocs
  - material-theme
  - examples
status: published
---

??? note "How to enable this"

    Add the following to `mkdocs.yml`:

    ```yaml
    markdown_extensions:
      - admonition
      - pymdownx.details
    ```

    `admonition` enables the `!!!` callout syntax. `pymdownx.details` adds the `???` collapsible variant.

This is an example of an adominition with a title:

!!! note "Title of the callout"

    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.


Collapsible callout:

??? info "Collapsible callout"

    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.

[Admonitions documentations](https://squidfunk.github.io/mkdocs-material/reference/admonitions/#supported-types)