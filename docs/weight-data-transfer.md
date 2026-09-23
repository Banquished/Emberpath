# Weight measurement import and export

The weight service owns parsing, unit conversion, validation and persistence.
The web app provides a preview and confirmation. Unit preferences are deferred;
stored measurements and exports remain in kilograms.

## Delimited text

Choose a comma, semicolon or tab delimiter. Imports accept `.csv` and `.txt`
files; the filename does not determine how the service parses the content.
Exports use a `.csv` filename for all three choices. Files use UTF-8 and the
same three headers, separated by the selected delimiter:

```csv
date,weight,unit
2026-09-20,101.3,kg
2026-09-21,220,lb
```

- Dates use `YYYY-MM-DD`.
- Weights are positive decimal numbers with a decimal point, without thousands
  separators or scientific notation.
- Units are `kg` or `lb`; one file can contain both.
- Kilograms support two decimal places; pounds support up to six.
- Pounds are multiplied by exactly `0.45359237` and rounded half up to two
  decimal places before kilogram validation. For example, `220 lb` becomes
  `99.79 kg`.
- Files are limited to 1 MiB and 10,000 data rows.
- Repeated dates within the file are errors, rather than choosing an arbitrary
  measurement. Correct invalid rows before importing.

## Workflow

1. Select a `.csv` or `.txt` file in the weight page's import dialog and choose
   its delimiter. Comma is selected initially; semicolon- and tab-delimited
   files are supported with either extension.
2. Choose how to handle dates already recorded: skip (default), or replace.
3. Preview converted values, row errors and planned actions.
4. Confirm the import. Replacement updates the existing measurement, retaining
   its ID. Goals and their saved starting weights are not changed by import.

Preview does not write measurements. Commit validates the file again and checks
the preview against the current relevant measurements. A stale preview returns
409 and must be refreshed. Invalid files do not partially import.

Export downloads the signed-in user's complete measurement history in date order,
independent of the selected chart period. It includes the same headers and
`unit=kg`, so exported files can be imported again. Skip mode makes that round
trip leave existing dates unchanged.

## API

All endpoints require the same Clerk bearer authentication as weight CRUD.

| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | `/weight-logs/export?delimiter=comma` | Download all measurements; `semicolon` and `tab` are also supported |
| POST | `/weight-logs/import/preview` | Validate text and produce a preview |
| POST | `/weight-logs/import` | Commit a previously reviewed preview |

Preview accepts JSON with `content`, `delimiter` (`comma`, `semicolon` or `tab`) and
`duplicate_policy` (`skip` or `replace`). Commit also requires the returned
`preview_token`. Files and tokens are not stored as import sessions.

Preview returns rows with `row`, `date`, `weight_kg`, `action` and `errors`, plus
counts named `imported`, `replaced`, `skipped` and `errors`. Commit returns the
three action counts. Each operation is scoped to the authenticated internal user.
