# Project: Sõnastikuandmete pärimise demo

## Overview
A single-page Estonian-language demo explaining the differences between Database, API, and UI for querying dictionary data. Target audience: lexicographers at the Estonian Language Institute (Eesti Keele Instituut) who want to use dictionary data for LLM operations.

## Key Concept
The page teaches WHEN to use each method:
- **Database (SQL)**: Bulk queries with complex filters → export to Excel → use with LLM
- **API**: Single word lookups, automated scripts, external system integration
- **UI**: Quick manual lookups, no technical skills needed

## Tech Stack
- Single `index.html` file (HTML + CSS + JavaScript)
- **sql.js**: SQLite compiled to WebAssembly, runs in browser
- Hosted on GitHub Pages: https://estakaad.github.io/ekkd_ttuba/
- Repository: git@github.com:estakaad/ekkd_ttuba.git

## File Structure
```
datademo/
├── index.html    # Everything is here (styles, content, JS)
├── CLAUDE.md     # This file
└── .git/
```

## Key Features
1. **Problem-focused intro**: Starts with relatable scenario
2. **Three solution cards**: Database, API, UI with pros/cons
3. **Interactive SQL playground**:
   - Uses sql.js (in-browser SQLite)
   - Tabbed interface showing all table contents
   - Example queries users can click to run
4. **Detailed workflow example**: LLM register checking with step-by-step reasoning
5. **Collapsible sections**: Using `<details>` elements for presentation use
6. **Comparison table**: Quick reference for choosing the right method

## Database Schema (simplified Ekilex)
```sql
dataset (code, name)           -- Dictionaries: eki, esterm, etc.
word (id, value, lang)         -- Words: "mure", "park" in different languages
meaning (id)                   -- Meaning entries
definition (id, meaning_id, value)  -- Definitions/explanations
lexeme (id, word_id, meaning_id, dataset_code)  -- Links word↔meaning↔dictionary
```

Sample data includes Estonian words "mure" (worry/crumbly) and "park" (park/tanning agent/ship) with multiple homonyms to demonstrate the data model.

## Important Context
- **Ekilex**: EKI's dictionary editing system (PostgreSQL database)
- **Sõnaveeb**: Public dictionary portal (materialized view of Ekilex data, NO separate API)
- **Sketch Engine**: Corpus query system with its own API
- APIs discussed are specifically **Ekilex API** and **Sketch Engine API**

## Language
All UI text is in Estonian. Key terms:
- Andmebaas = Database
- Päring = Query
- Sõnastik = Dictionary
- Tähendus = Meaning
- Seletus/Definitsioon = Definition
- Keelend = Lexical item/word

## Deployment
```bash
git add .
git commit -m "Description"
git push
```
Changes appear on GitHub Pages within 1-2 minutes.

## Common Tasks
- **Add new example query**: Find `.example-queries` div, add new `.example-query` element
- **Modify sample data**: Find `INSERT INTO` statements in `initDatabase()` function
- **Add new collapsible section**: Use `<details class="collapsible-section">` pattern
- **Change table tab content**: Modify `loadAllTableContents()` function
