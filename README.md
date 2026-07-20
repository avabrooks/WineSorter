# Wine Organizer

A single-file browser tool for filtering and sorting a wine spreadsheet using plain-English prompts.

**Status: prototype.** It ships with a 55-row sample wine list. It hasn't been connected up to the real wine spreadsheet yet.

## Why build it this way

This tool intentionally doesn't call any AI model at runtime, it's plain JavaScript pattern-matching, not an AI agent. With work computers, unsure if privacy access would allow an active AI agent to make changes to files on computers. Building this as a single local HTML file sidesteps the question entirely, no API key, no AI account, nothing to review or approve, since there's no AI involved once it's built. It also means using it is free and doesn't draw down any Claude/AI license, since everything runs in the browser off hand-written rules. 

## How to use it

1. Open `Wine_Prompt_Sorter.html` (double-click it, or drag it onto a Chrome/Edge/Safari window).
2. It loads with sample data already in the table. Drop your own `.xlsx` or `.csv` onto the page to replace it, or click "Load sample data" to go back.
3. Type a prompt and click Apply (or hit Enter). Example prompts:
   - `red wines from France under $30 sorted by price`
   - `white wines from Italy or Spain sorted by rating`
   - `sparkling, top rated first`
   - `malbec over 90 points, cheapest first`
   - `anything from Argentina after 2018`
   - `something that goes great with white fish`
   - `a bold red for steak night`
   - `something budget-friendly for a casual weeknight`
   - `a sparkling wine for a celebration`
4. The table updates live, and a line above it shows exactly what it understood from your prompt (so misreads are obvious, not silent).
5. Click "Download results as Excel" to save the current filtered/sorted view as a new `.xlsx` file.

## How it works

- **Reading the file** — [SheetJS](https://sheetjs.com/) parses the uploaded `.xlsx`/`.csv` into a plain array of row objects.
- **The prompt parser** — a JavaScript function lowercases the prompt and runs it through a series of checks:
  - Country / Type / Grape / Region — matched against the *actual unique values* found in your data, not a hardcoded list. Color synonyms are recognized too (e.g. "rosé", "pink", "bubbly", "sweet").
  - Price: patterns like "under $30", "over $50", "between $20 and $40".
  - Rating: patterns like "rated above 95", "over 90 points", "90+ points".
  - Vintage: patterns like "after 2018", "before 2015", or a bare 4-digit year.
  - Sort: patterns like "sorted by price", "cheapest first", "highest rated first", "newest first".
- **Rendering + export** — matching rows are redrawn into the table; the download button hands the current filtered array back to SheetJS to write a new `.xlsx`.

### Food pairing / occasion / style dictionary

Some phrases don't map to anything literally in the spreadsheet such as "goes with white fish," "for a celebration," "budget-friendly." For these, there's a hand-written dictionary (`PAIRING_DICTIONARY` in the script) of keyword → suggestion mappings, covering three categories:

- **Food pairings** — fish/seafood, shellfish, sushi, steak/red meat, lamb, pork, chicken, turkey, pasta, pizza, spicy food, BBQ, cheese, chocolate/dessert, salad, mushroom/earthy dishes.
- **Occasions/mood** — celebration, summer/patio, cozy/winter, date night, brunch, gift/splurge, budget-friendly.
- **Style descriptors** — light/crisp, bold/full-bodied, smooth/mellow, sweet, dry, fruity.

Each entry suggests candidate Type/Grape values (or a price ceiling / rating floor), matched against whatever values actually exist in your data. These only fill in gaps, if the prompt already says "red" or names a grape directly, that always wins over an inferred suggestion.

## Column matching

The parser auto-detects columns by header name (case-insensitive), accepting a few synonyms per field:

| Field | Recognized headers |
|---|---|
| Name | Wine Name, Name, Wine, Label, Product |
| Country | Country, Origin, Nation |
| Region | Region, Appellation, Area |
| Type | Type, Color, Colour, Wine Type, Style |
| Grape | Grape, Varietal, Variety |
| Vintage | Vintage, Year |
| Price | Price, Cost |
| Rating | Rating, Score, Points |


