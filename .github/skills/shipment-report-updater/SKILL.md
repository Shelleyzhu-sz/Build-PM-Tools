---
name: shipment-report-updater
description: Update shipment report workbooks from a user-provided configuration matrix and fallback owner.
---

# Shipment Report Updater

Use this skill when a colleague provides an Excel configuration matrix and an Excel shipment report and asks you to allocate shipment rows to owners.

## Inputs

Ask for or identify:

- The configuration matrix workbook and shipment report workbook.
- The fallback owner to use when a row cannot be matched or all matching capacity is exhausted. Never assume or hardcode a particular person's name.
- Any sheet names or column mappings that are not unambiguous from the workbooks.

## Allocation procedure

1. Inspect the workbooks and identify the configuration-name and shipping-location columns in both files. Preserve the report's existing fields and layout unless an allocation requires changing the `Attn To` or `Allocation` fields.
2. Normalize shipping locations before matching:
   - `RMD` and `MSRedmond` -> `MS Redmond`
   - `MSFinlandEspoo` -> `MS Finland Espoo`
   - `JDM1TPE` -> `JDM1 Taipei`
   Preserve other location values after applying equivalent whitespace/case normalization where appropriate.
3. Match each report row to the matrix using both normalized configuration name and normalized shipping location (configuration plus distribution). Do not match on configuration alone when distribution differs.
4. Interpret each numeric matrix cell as that owner's per-owner unit capacity for the corresponding configuration and distribution. Treat blank, nonnumeric, and nonpositive values as no available capacity.
5. Allocate matching rows in this priority order:
   - `Quality`
   - `SW Dev`
   - Other organizations in the matrix
   
   Within an organization, use the matrix's owner ordering consistently. Never allocate more rows to an owner than that owner's capacity. Decrease remaining capacity after every assignment.
6. If there is no match or every matching owner's capacity is exhausted, assign the fallback owner supplied by the user.
7. Write exactly one `Attn To` value per report row. Use the account-only alias before `@` when the owner is an email address (for example, `alex.chen` from `alex.chen@example.com`).
8. Write the owner's organization/team in `Allocation`. Do not put an email address in `Allocation`; for fallback rows, use the fallback owner's organization/team when supplied, otherwise preserve the report's applicable existing organization/team value.
9. Preserve all unrelated report fields, row order, formulas where possible, and workbook formatting. Do not overwrite the source files.
10. Save a new updated workbook with a distinct output filename. Reopen or otherwise inspect the saved workbook to verify that every report row has exactly one `Attn To`, allocations obey capacity and priority rules, and unrelated fields remain intact. Report the output path clearly.

## Suggested usage prompt

> Using the attached configuration matrix and shipment report, identify the configuration-name and shipping-location columns, normalize locations, and allocate each report row by configuration plus distribution. Treat numeric matrix cells as per-owner capacity, prioritize Quality then SW Dev then other organizations, never exceed capacity, and use **[fallback owner]** for unmatched or exhausted rows. Put one account-only alias in `Attn To`, the organization/team in `Allocation`, preserve all other fields, save a new workbook, verify it, and report the output path.
