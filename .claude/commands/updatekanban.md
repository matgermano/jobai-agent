# /updatekanban

## Goal
Update the Status of a card in the "jobAI Roadmap" Notion database.

## Usage
```
/updatekanban <card name or number> <new status>
```

Examples:
- `/updatekanban "Card 1.1" Doing`
- `/updatekanban 2.3 Done`
- `/updatekanban "Card 3.1" Backlog`

## Valid statuses
- `Backlog`
- `To Do`
- `Doing`
- `Done`

## Steps

### 1. Search for the card
Use `notion-search` to find the card by name in the jobAI Roadmap database:
- Query: the card name or number provided by the user
- `query_type`: `internal`
- `page_size`: 5

If multiple results are returned, pick the one whose title most closely matches the input. If ambiguous, list the candidates and ask the user to confirm.

### 2. Validate the status
If the requested status is not one of `Backlog`, `To Do`, `Doing`, `Done` — stop and inform the user of the valid options.

### 3. Update the card
Use `notion-update-page` with:
- `page_id`: the ID of the matched card
- `command`: `update_properties`
- `properties`: `{"Status": "<new status>"}`

### 4. Confirm
Print a confirmation in this format:
```
Card: <full card title>
Status: <old status> → <new status>
URL: <notion page URL>
```
