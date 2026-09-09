# Serial Import Compatibility

## Purpose

This branch contains changes required to make existing Metropolitan Sales applications compatible with the new Dynamics GP serial-number import process.

The new process stores serial numbers in the legitimate Dynamics GP serial-number table, `SOP10201`, without requiring the serial numbers to first be stored as comma-delimited text in Sales Comment Entry (`SOP10202`).

The objective of these changes is to make `SOP10201` the authoritative source for serial numbers while maintaining compatibility with existing Metropolitan Sales workflows.

## Source Branch

Development branch:

`serial-import-compatibility`

Created from:

`adjit/Metro_Tools` branch `2026`

Baseline commit:

`9cf4840` - Merge branch 'master' into 2026

No changes will be made directly to Aaron's `2026` branch.

## Existing Serial Number Workflow

The existing warehouse workflow enters serial numbers into Sales Comment Entry.

Serial numbers are initially stored as comma-delimited text in:

`SOP10202.CMMTTEXT`

During Sales Order to Invoice transfer, the existing `AZsopSerialLot` SQL trigger parses the comment and creates the individual GP serial-number records in:

`SOP10201`

Existing applications were found to continue reading serial numbers from `SOP10202` rather than from the resulting GP serial-number records.

## New Serial Import Workflow

The new drop-ship serial import process will insert validated serial-number records into `SOP10201` after Sales Order to Invoice transfer and before invoice posting.

This avoids the Sales Comment field-size limitation while retaining the normal GP posting process.

The existing warehouse process will remain unchanged.

Both workflows ultimately create serial-number records in `SOP10201`.

## Compatibility Changes

### Epson POS Report

Current behavior:

The Epson POS Report obtains its Serial No. data from:

`SOP10202.CMMTTEXT`

Required change:

Obtain serial numbers from `SOP10201.SERLTNUM`.

The existing report behavior and output should otherwise remain unchanged.

### Metro Tools Serial Number Lookup

Current behavior:

The Serial Number Lookup searches:

`SOP10202.CMMTTEXT`

Required change:

Search the actual GP serial-number field:

`SOP10201.SERLTNUM`

Existing lookup functionality should otherwise remain unchanged.

## Initial Test Transactions

### Existing Warehouse Workflow

Invoice: `I704448`

Item: `TM-T88VII-UES`

Expected serial:

`X9XL354508`

This invoice was processed through the existing Sales Comment / AZsopSerialLot workflow and currently appears correctly in the Epson report.

### New Serial Import Method

Invoice: `I704611`

Item: `TMU220BII-E`

Quantity: 4

Expected serials:

- XD7B034791
- XD7B034796
- XD7B034786
- XD7B034819

These serial numbers were inserted directly into `SOP10201` after Sales Order to Invoice transfer.

They survived normal GP posting and appear correctly in the existing GP Serial Number Search.

They currently do not appear in the Epson POS Report because that application reads serial numbers from `SOP10202`.

## Acceptance Criteria

Changes are considered successful only if:

1. Existing warehouse-created serial numbers continue to appear correctly.
2. New directly imported serial numbers appear correctly.
3. Epson report quantity and serial-count validation continues to function.
4. Existing Epson report layout and business logic remain unchanged except for the serial-number source.
5. Metro Tools Serial Number Lookup finds serial numbers created by either workflow.
6. No changes are required to the existing warehouse serial-entry process.
7. No serial-number data needs to be duplicated into `SOP10202` solely for compatibility.

## Change Control

All source-code changes made for this project will be:

- Made only in the `serial-import-compatibility` branch.
- Limited to functionality required for serial-import compatibility.
- Documented in this file.
- Committed to Git with descriptive commit messages.
- Tested against both existing and newly imported serial-number transactions.
- Available for Aaron to review before any merge into the `2026` branch.

Unrelated code cleanup or modernization will not be included in this branch.

## Change Log

### 2026-09-09 - Project Baseline

- Created `serial-import-compatibility` from the current `2026` branch.
- Baseline commit: `9cf4840`.
- Added project documentation.
- No application source code changed.
