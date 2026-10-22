---
name: google-dorking
description: Turn a natural-language public research or authorized search goal into focused Google Search operator queries. Use when a user asks for Google dorks or search commands; return queries without running them.
---

# Google Dorking

Convert the user's search goal into concise, copyable Google Search queries. These are search queries to paste into Google, not shell commands. Do not run searches unless separately asked and an appropriate search tool is available.

## Build the queries

1. Identify the subject, desired information, and any site, domain, file type, or phrase the user supplied. Preserve useful terms from the request.
2. Choose only operators that narrow the search usefully, such as `site:`, `filetype:`, `intitle:`, `inurl:`, quoted phrases, `OR`, and exclusions with `-`.
3. Return a small set of distinct query options. Put each query in its own code block and add a short note describing its purpose.
4. If a useful query depends on missing information, use a clear placeholder such as `[domain]`. Ask one brief question only when the goal cannot be translated usefully without the missing detail.
5. Keep syntax readable. Avoid piling on operators, implying that a query guarantees complete results, or including a URL scheme after `site:` unless specifically useful.

## Boundaries

- Help with ordinary public research and passive searches for sites or domains the user owns or is authorized to assess.
- Do not construct queries intended to uncover passwords, authentication tokens, private personal data, or other exposed secrets, or to facilitate unauthorized access or targeting. Briefly say what part you cannot help with and, when useful, offer a safer query for public documentation or an authorized asset inventory.
- Do not infer authorization from a domain name alone. If the request is an exposure assessment and ownership or authorization is unclear, ask a short clarifying question before producing domain-targeted queries.

## Example

User: “Find PDF setup guides on the example.org site.”

Answer with a brief note and queries such as:

```text
site:example.org filetype:pdf setup guide
```

```text
site:example.org ("setup guide" OR "getting started") filetype:pdf
```
