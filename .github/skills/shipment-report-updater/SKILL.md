---
name: shipment-report-updater
description: Update shipment report workbooks from a user-provided configuration matrix and fallback owner.
---

# Shipment Report Updater

Use this skill when a colleague provides an Excel configuration matrix, a current shipment report, and prior shipment report workbooks and asks you to allocate shipment rows to owners while maintaining cumulative capacity. Prior reports are required for cumulative tracking; if they are not supplied, explain that cumulative allocation cannot be calculated reliably and ask the user to provide them.

## Inputs

Ask for or identify:

- The configuration matrix workbook and shipment report workbook.
- One or more prior shipment report workbooks covering shipments already assigned against the matrix capacity.
- The fallback owner to use when a row cannot be matched or all matching capacity is exhausted. Never assume or hardcode a particular person's name.
- Any sheet names or column mappings that are not unambiguous from the workbooks.

## Allocation procedure

1. Inspect the workbooks and identify the configuration-name and shipping-location columns in both files. Preserve the report's existing fields and layout unless an allocation requires changing the `Attn To` or `Allocation` fields.
2. Normalize shipping locations before matching:
   - `RMD` and `MSRedmond` -> `MS Redmond`
   - `MSFinlandEspoo` -> `MS Finland Espoo`
   - `JDM1TPE` -> `JDM1 Taipei`
   Preserve other location values after applying equivalent whitespace/case normalization where appropriate.
3. Match rows using both normalized configuration name and normalized shipping location (configuration plus distribution). Do not match on configuration alone when distribution differs.
4. Build cumulative history before allocating the current report:
   - For every prior workbook, match each prior assignment by configuration name + normalized shipping location + selected owner. Use the prior row's selected owner from `Attn To`, normalizing an email to its account-only alias before comparison.
   - Count the prior assigned units for each owner/configuration/distribution combination. Count the units represented by the report's quantity field when one is clearly present; otherwise count one unit per assigned row.
   - If multiple prior workbooks are provided, process each distinct file once. Do not count the current report as history, and do not double-count duplicate prior files (for example, the same path or the same workbook content under a different filename).
   - Subtract each cumulative prior count from the corresponding owner's numeric matrix capacity. Treat negative remaining capacity as zero and retain only positive remaining capacity for new allocations.
5. Interpret each numeric matrix cell as that owner's per-owner unit capacity for the corresponding configuration and distribution. Treat blank, nonnumeric, and nonpositive values as no available capacity before history subtraction.
6. Allocate current matching rows using remaining capacity in this priority order:
   - `Quality`
   - `SW Dev`
   - Other organizations in the matrix
   
   Within an organization, use the matrix's owner ordering consistently. Never allocate more current units to an owner than that owner's remaining capacity. Decrease remaining capacity after every assignment.
7. If there is no match or every matching owner's remaining capacity is exhausted, assign the fallback owner supplied by the user.
8. Write exactly one `Attn To` value per current report row. Use the account-only alias before `@` when the owner is an email address (for example, `alex.chen` from `alex.chen@example.com`).
9. Write the owner's organization/team in `Allocation`. Do not put an email address in `Allocation`; for fallback rows, use the fallback owner's organization/team when supplied, otherwise preserve the report's applicable existing organization/team value.
10. Preserve all unrelated current-report fields, row order, formulas where possible, and workbook formatting. Do not overwrite the source files.
11. Save a new updated workbook with a distinct output filename. Reopen or otherwise inspect the saved workbook to verify that every current-report row has exactly one `Attn To`, cumulative prior counts were applied once, allocations obey remaining capacity and priority rules, and unrelated fields remain intact. Report the output path clearly.

## Suggested usage prompt

> Using the attached current configuration matrix, current shipment report, and prior shipment reports, identify the configuration-name and shipping-location columns, normalize locations, and match prior assignments by configuration + distribution + selected owner. Count prior assigned units once, subtract them from each owner's numeric capacity, and allocate current rows by remaining capacity with Quality then SW Dev then other organizations. Do not count the current report or duplicate prior files twice. Use **[fallback owner]** for unmatched or exhausted rows. Put one account-only alias in `Attn To`, the organization/team in `Allocation`, preserve all other fields, save a new workbook, verify it, and report the output path.
