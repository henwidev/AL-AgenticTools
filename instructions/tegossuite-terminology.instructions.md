---
description: "Use Weblate MCP for terminology when translating tegossuite or enwis documentation, changelogs, and user-facing text."
applyTo: "**/docs/**/*.md,**/changelogs/*.md"
---

# Weblate terminology

Use the **Weblate MCP server** as the terminology source when translating domain terms.

1. Select the relevant Weblate project and, when known, release category or component.
   Use `list_projects_and_components` to discover them if needed; do not guess a release.
2. Look up non-sensitive terms with `get_term_translation`, specifying the source and
   target languages. Batch related terms where practical. Use `search_translations` or
   `lookup_translation_memory` for context if the result is ambiguous or incomplete.
3. Check that the selected translation fits the business context and product version.
   Preserve exact quoted UI captions, product names, technical identifiers, and
   placeholders unless their translation is explicitly required and verified.
4. If Weblate is unavailable, or a material term remains unresolved, report this and ask
   before using an unverified translation.

Send only non-sensitive terms or generic phrases to Weblate, not task descriptions,
source code, customer data, credentials, or other confidential content.
