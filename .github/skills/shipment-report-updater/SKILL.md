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
11. Add an `Allocation Summary` worksheet to every output workbook:
   - Include every team/organization represented by a matrix owner and any team/organization represented in shipped rows (including fallback assignments), even if its current shipped quantity or allocation total is zero.
   - Calculate `Total Allocation Qty` for each team by summing every numeric, positive owner quantity in the full configuration matrix, across all configuration names and distributions, grouped by each owner's team/organization. Do not limit this denominator to configurations or distributions present in the current shipment.
   - Calculate shipped quantity by team through the latest shipment date represented in the supplied prior reports and current report. Include each distinct supplied prior report once, plus the current report once; do not count the current report as prior history. Use the report's clearly identified quantity field, or one unit per row if no such field exists. Attribute shipped units to the row's selected team in `Allocation`.
   - De-duplicate repeated prior reports (including copies with different filenames) and overlapping shipment units across reports using a stable unit identity where available, preferably serial number/PPID combined with configuration and normalized shipping location. Do not count a current-report unit again if that same identity is already in prior history. If no reliable unit identifier is available, state that duplicate units across reports cannot be reliably identified and that shipped totals may be uncertain; do not claim exact de-duplication.
   - Calculate shipped percentage as shipped quantity divided by the full-matrix team allocation quantity. Do not cap results at 100%. Explain any overage, including when fallback assignments or other shipped rows exceed the matrix allocation total. For a team with zero allocation, show `N/A` rather than dividing by zero.
   - Include the latest shipment date in the summary, preferably in the shipped-quantity column heading (`Shipped Qty through [latest date]`) and/or a clearly visible note.
   - Present a clear, well-formatted table with columns `Team`, `Total Allocation Qty`, `Shipped Qty through [latest date]`, and `Shipped %`.
   - Add two charts: a clustered column chart comparing `Total Allocation Qty` with `Shipped Qty through [latest date]` by team, and a separate shipped-utilization chart by team. Give both charts clear titles. Store chart source values as numeric cells; store shipped percentages as numeric fractions for teams with a nonzero allocation and apply percentage formatting to the cells, utilization axis, and data labels. Show zero-allocation teams as `N/A` in the table and exclude them from the utilization plot rather than presenting a misleading percentage.
12. Save a new updated workbook with a distinct output filename. Reopen the saved workbook and verify that every current-report row has exactly one `Attn To`, cumulative prior counts were applied once, allocations obey remaining capacity and priority rules, unrelated fields remain intact, and the `Allocation Summary` table and both charts are present and valid. Confirm chart series reference the intended numeric cells and that percentage axis/data labels are percent-formatted. Report the output path clearly.

## Suggested usage prompt

> Using the attached current configuration matrix, current shipment report, and all relevant prior shipment reports, identify the configuration-name and shipping-location columns, normalize locations, and match prior assignments by configuration + distribution + selected owner. Count prior assigned units once, subtract them from each owner's numeric capacity, and allocate current rows by remaining capacity with Quality then SW Dev then other organizations. Do not count the current report as prior history or count duplicate prior reports or repeated shipment units twice; use stable unit identity where available and state any uncertainty when no reliable identifier exists. Use **[fallback owner]** for unmatched or exhausted rows. Put one account-only alias in `Attn To`, the organization/team in `Allocation`, preserve all other fields, and add an `Allocation Summary` worksheet with full-configuration-matrix allocation totals by team, shipped quantities through the latest shipment date, uncapped shipped percentages, and the required allocation-comparison and utilization charts. Save a new workbook without overwriting the sources, reopen it to verify the allocations, summary, and charts, and report the output path.
