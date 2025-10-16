# Organize SQLite Schema and Data API

This document describes the persistent storage layer for the Organize to-do application. The database is created automatically when the stack opens and lives at:

```
<specialFolderPath("documents")>/organize.sqlite
```

If the documents folder is unavailable (for example in the IDE), the stack falls back to `specialFolderPath("resources")` and finally to the user home directory. The parent folder is created on demand so the database can be opened safely on first launch.

## Schema Overview

All list data is stored inside a single table so that date-based lists and topic-based "Pages" share the same storage engine while remaining easy to query.

### `list_items`

| Column        | Type    | Constraints / Details                                                                                          |
|---------------|---------|------------------------------------------------------------------------------------------------------------------|
| `id`          | INTEGER | Primary key. Auto-incrementing row identifier managed by SQLite.                                                |
| `content`     | TEXT    | The visible text of the list item. Required.                                                                    |
| `is_completed`| INTEGER | `0` (incomplete) or `1` (complete). Defaults to `0`.                                                             |
| `position`    | INTEGER | Explicit ordering within a list. Defaults to `0` and is persisted whenever items are reordered.                 |
| `item_date`   | TEXT    | ISO `YYYY-MM-DD` string when the item belongs to a date-based list. `NULL` for topic-based lists.               |
| `topic`       | TEXT    | Name of the topic/page when the item belongs to a topic list. `NULL` for date-based lists.                     |
| `created_at`  | TEXT    | Timestamp (UTC) set to `datetime('now')` on insert.                                                             |
| `updated_at`  | TEXT    | Timestamp (UTC) refreshed to `datetime('now')` by every update command issued by the stack code.                |

A check constraint ensures that an item always belongs to *either* a specific date *or* a topic, but never both simultaneously:

```
CHECK ((item_date IS NOT NULL AND topic IS NULL) OR (item_date IS NULL AND topic IS NOT NULL))
```

### Indexes

To keep list retrieval and drag-and-drop reordering fast, the stack creates the following indexes:

- `idx_list_items_item_date` — accelerates queries by date.
- `idx_list_items_topic` — accelerates queries by topic/page.
- `idx_list_items_date_position` — keeps date-based lists ordered by `position` with minimal sorting cost.
- `idx_list_items_topic_position` — same optimisation for topic/page lists.

### Write Optimisations

During initialisation several SQLite pragmas are applied to favour frequent, incremental writes that come from keystroke-level autosave:

- `PRAGMA journal_mode = WAL` — enables Write-Ahead Logging for better concurrency and crash safety.
- `PRAGMA synchronous = NORMAL` — balances durability with write throughput.
- `PRAGMA cache_size = -8000` — allocates ~8 MB of cache in memory pages.
- `PRAGMA temp_store = MEMORY` — keeps temporary data in RAM.
- `PRAGMA foreign_keys = ON` — future-proofs the schema for related tables.

## Runtime Lifecycle

The stack script in `Organize.livecode` automatically manages the database lifecycle:

- `preOpenStack` → `organizeInitializeDatabase`
- `closeStack` → `organizeCloseDatabase`

The helper `organizeDatabaseConnection()` returns the live connection ID for advanced use-cases, while all public commands/functions automatically call `organizeInitializeDatabase` if the connection is not yet open.

## Public Data API

| Handler / Function | Purpose | Notes |
|--------------------|---------|-------|
| `organizeFetchItemsByDate(pISODate)` | Returns an array of records for a given ISO date (`YYYY-MM-DD`). | Throws if the date is empty. Records include keys `id`, `content`, `isCompleted`, `position`, `itemDate`, `topic`, `createdAt`, `updatedAt`. |
| `organizeFetchItemsByTopic(pTopic)` | Returns an array of records for the supplied topic/page name. | Throws if the topic is empty. |
| `organizeInsertDateItem(pContent, pISODate, pPosition)` | Inserts a new date-based item and returns the new row ID. | If `pPosition` is empty the item is appended (max position + 1). |
| `organizeInsertTopicItem(pContent, pTopic, pPosition)` | Inserts a new topic-based item and returns the new row ID. | Same automatic positioning behaviour as the date variant. |
| `organizeUpdateItemContent(pItemID, pContent)` | Updates the item text while preserving other fields. | Designed for keystroke-level autosave updates. |
| `organizeUpdateItemPosition(pItemID, pPosition)` | Persists a new ordering index for the item. | Caller controls the rebalance strategy. |
| `organizeUpdateItemCompletion(pItemID, pIsCompleted)` | Toggles the completion flag. Accepts booleans, `0/1`, or words such as `true/false`. | |
| `organizeDeleteItem(pItemID)` | Removes an item entirely. | |
| `organizeMoveItemToDate(pItemID, pISODate, pPosition)` | Moves an existing item to a specific date list. Optional `pPosition` controls ordering. | Clears any topic association. |
| `organizeMoveItemToTopic(pItemID, pTopic, pPosition)` | Moves an existing item to a topic/page list. Optional `pPosition` controls ordering. | Clears any date association. |

Helper utilities (used internally but available if needed) include:

- `organizeResolvedPosition(pPosition, pColumn, pValue)` — normalises requested positions.
- `organizeNextPosition(pColumn, pValue)` — calculates the next slot for appending items.
- `organizeBooleanToInteger`, `organizeToInteger`, `organizeQuote` — data sanitisation helpers.

All handlers throw descriptive errors when inputs are invalid or SQLite reports an error. Callers can wrap invocations in LiveCode `try…catch` blocks to surface messages in the UI.

## Usage Examples

### Adding a Topic-Based Item

```livecode
command addGroceriesItem pContent
    try
        put organizeInsertTopicItem(pContent, "Groceries", empty) into tNewID
        -- refresh UI using tNewID as needed
    catch tError
        answer warning "Unable to add item:" && tError
    end try
end addGroceriesItem
```

### Fetching Items for Today

```livecode
command refreshTodayItems
    put the short system date into tSystemDate
    convert tSystemDate from system date to seconds
    convert tSystemDate from seconds to internet date
    put word 4 of tSystemDate into tYear
    put word 2 of tSystemDate into tDay
    put word 3 of tSystemDate into tMonth
    convert tMonth to month
    put format("%04d-%02d-%02d", tYear, tMonth, tDay) into tISODate
    put organizeFetchItemsByDate(tISODate) into gTodayItemsA
end refreshTodayItems
```

*(Always supply ISO-formatted dates — for example `2024-12-24` — to keep indexes effective and comparisons reliable.)*

### Moving an Item Between Dates

```livecode
command rescheduleItem pItemID, pNewISODate
    try
        organizeMoveItemToDate pItemID, pNewISODate, empty
    catch tError
        answer error "Unable to move item:" && tError
    end try
end rescheduleItem
```

## Extending the Schema

The existing design leaves room for future enhancements:

- Additional tables (e.g. reminders, attachments) can link back to `list_items.id` using foreign keys.
- `topic` values can later be normalised into a dedicated `topics` table without changing the item structure.
- Sync metadata can be added via supplementary columns or companion tables; the WAL mode keeps concurrent writers fast enough for future sync services.

By consolidating storage in a single table with expressive helper commands, the stack now has a robust, high-performance persistence layer ready for UI and sync features to build upon.
