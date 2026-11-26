---
name: cm-import
description: Import Martim's Revolut statements from Google Drive into the C&M Finance app (Supabase). Use when Martim says "importa", "import", "importa o Martim/C&M" or drops a statement in Drive › money › 01. Imports.
---

# C&M Finance — import from Drive

Replies in PT-PT, short. Never write to the database before Martim says "go".

## Where things are

- Drive folders (Google Drive connector):
  - Martim → `money/01. Imports/1.Martim` — folder id `19j2Ct3JEq-ZE6kIdrNkA1vyvl5MlIpEH`
  - C&M → `money/01. Imports/2.C&M` — folder id `1c36w4WAiStmlxiQH1SgNj-3u5zfPYgD3`
  - If an id fails, search by folder name.
- Supabase project `zltpscoxwfiwcbnfvnec` (Supabase connector, `execute_sql`).

| Profile | Transactions table | Settings table | Balance account name (always in `cm_accounts`) |
|---|---|---|---|
| Martim | `martim_transactions` | `martim_settings` | `Martim - Personal` |
| C&M | `cm_transactions` | `cm_settings` | `C&M Joint` |

`*_transactions` columns: `id bigint, date date, description text, amount numeric, type 'income'|'expense', category text, subcategory text, person text, ignored bool, flagged bool`.

## Steps

1. **Which profile.** "importa" alone = both folders. "importa Martim" / "importa C&M" = that one.
2. **Find new files.** List the folder's CSV files, newest first. Skip file ids already listed in the profile's settings row `key='drive_imports'` (`value` = `{"<fileId>": "<date imported>"}`). Nothing new → say so and stop.
3. **Read the CSV** (download the file). Revolut format: `Type,Product,Started Date,Completed Date,Description,Amount,Fee,Currency,State,Balance`. Keep rows with `State=COMPLETED`, a `Completed Date`, `Product=Current` and `Amount≠0`. Fields can be quoted and contain commas.
   - `date` = first 10 chars of `Completed Date`; `description` = Description trimmed; `amount` = Amount.
   - **Sign decides the type, always:** Amount > 0 → `income`, < 0 → `expense`. Stored `amount` keeps the sign.
4. **Drop duplicates.** Skip a row if the table already has the same `date`, `description`, `abs(amount)` and `type`.
5. **Categorise** (per profile, in SQL against that profile's table):
   - **Rules first:** settings row `key='rules'` is a JSON list of `{match, type, category, subcategory}`. Normalise the description (lower case, remove numbers of 4+ digits, collapse spaces); a rule applies when the normalised description contains `match` and `type` matches (or rule type `any`).
   - **Then history:** past rows with the same normalised description. Take the most common `category/subcategory`. Use it only with **3 or more past rows and ≥ 90 % agreement**. If those past rows were all `ignored`, mark the new one `ignored=true`.
   - **Otherwise leave category and subcategory empty.** The app shows these in the Inbox as "without category", and Martim reviews them there. Do not guess.
   - A refund keeps the merchant's category (it nets against it); do not move it to Income.
6. **Show the summary and wait for "go":**
   ```
   Martim · statement 01–26 Nov
   62 new · 48 categorised · 14 to review in the app · 9 duplicates skipped
   Balance 26 Nov: 1.450,10 €
   ```
   List the 14 to review in one line each (date · description · amount) only if Martim asks.
7. **On "go", write:**
   - `insert` the new rows with `id = <epoch ms now> + row index` (so the app's "Last import" chip finds the batch), `person=null`, `flagged=false`.
   - Balance: last kept row by `Completed Date` → its `Balance`. Insert into `cm_accounts` `(id, name, balance, date)` = `(<epoch ms>, '<account name>', balance, date)` unless that name already has a row for that date.
   - Add the file id to the profile's `drive_imports` settings row (upsert on `key`).
8. **Close:** one line — what was imported, and "abre a app → Inbox → Review para as N por rever".

## Do not

- Delete or update existing rows.
- Import a file whose header does not match the Revolut format; say what you found instead.
- Write to Carlota's tables (her bank is not Revolut).
