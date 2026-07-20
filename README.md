# Wine Organizer

A single-file browser tool for filtering and sorting a wine spreadsheet using plain-English prompts. 

**Status: prototype.** It ships with a 55-row sample wine list so it's usable out of the box. It hasn't been wired up to a real wine spreadsheet yet — the column-matching will need a pass.

## How to use it

1. Open `Wine_Prompt_Sorter.html` (double-click it, or drag it onto a Chrome/Edge/Safari window).
2. It loads with sample data already in the table. Drop your own `.xlsx` or `.csv` onto the page to replace it, or click "Load sample data" to go back.
3. Type a prompt and click Apply (or hit Enter). Example prompts:
   - `red wines from France under $30 sorted by price`
   - `white wines from Italy or Spain sorted by rating`
   - `sparkling, top rated first`
   - `malbec over 90 points, cheapest first`
   - `anything from Argentina after 2018`
4. The table updates live, and a line above it shows exactly what it understood from your prompt (so misreads are obvious, not silent).
5. Click "Download results as Excel" to save the current filtered/sorted view as a new `.xlsx` file.

## How it works


- **Reading the file** — [SheetJS](https://sheetjs.com/) (loaded from a CDN) parses the uploaded `.xlsx`/`.csv` into a plain array of row objects.
- **The prompt parser** — a JavaScript function lowercases the prompt and runs it through a series of regex checks:
  - Country / Type / Grape / Region — matched against the *actual unique values* found in your data, not a hardcoded list. Color synonyms are recognized too (e.g. "rosé", "pink", "bubbly", "sweet").
  - Price — patterns like "under $30", "over $50", "between $20 and $40".
  - Rating — patterns like "rated above 95", "over 90 points", "90+ points".
  - Vintage — patterns like "after 2018", "before 2015", or a bare 4-digit year.
  - Sort — patterns like "sorted by price", "cheapest first", "highest rated first", "newest first".
- **Rendering + export** — matching rows are redrawn into the table; the download button hands the current filtered array back to SheetJS to write a new `.xlsx`.

It's rule-based, not a language model — it only recognizes the patterns above. Anything phrased very differently (e.g. "something food-friendly for a dinner party") won't be understood.

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

If a real spreadsheet uses different headers, or has extra columns (e.g. bottle size, importer, tasting notes), the detection logic and possibly the filter patterns will need a small update. 

