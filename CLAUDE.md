# Project: Sõnastikuandmete pärimise demo

## Overview
A single-page Estonian-language demo/presentation explaining how to get dictionary data for use with LLMs. Target audience: lexicographers at the Estonian Language Institute (Eesti Keele Instituut).

**Main question the page answers:** "Nii et sa tahad suurel keelemudelil lasta midagi keeleandmetega teha?"

## Page Structure (Step-by-Step)

### 1. samm: Hangi andmed ühendsõnastikust
Three options compared:
- **Kasutajaliides** (Sõnaveeb/Ekilex) — NOT recommended for bulk data
- **API** (Ekilexi API) — NOT recommended for bulk data
- **Andmebaas** (PostgreSQL) — RECOMMENDED for LLM use cases

Key points about database access:
- Ekilex uses PostgreSQL
- Need DBeaver or similar tool
- Only backup access (not real-time) — data from previous night
- ~15 GB backup file
- Access is restricted, must be justified

### Interactive SQL Section
- Uses sql.js (SQLite in browser via WebAssembly)
- Tabbed interface showing all table contents
- Example queries users can click
- Simplified Ekilex schema with sample data ("mure", "park")

### 2. samm: Rikasta korpuse andmetega
- Corpus data (Sketch Engine) not in our database
- Must use Sketch Engine API
- Requires writing scripts for bulk queries

### Detailed Example: LLM register checking
Complete workflow showing all three steps:
1. Database query for words with "kõnek" register
2. Sketch Engine API for concordances
3. LLM API for evaluation

## Tech Stack
- Single `index.html` file (HTML + CSS + JavaScript)
- **sql.js**: SQLite compiled to WebAssembly, runs in browser
- Hosted on GitHub Pages: https://estakaad.github.io/ekkd_ttuba/
- Repository: git@github.com:estakaad/ekkd_ttuba.git

## Database Schema (simplified Ekilex)
```sql
dataset (code, name)           -- Dictionaries: eki, esterm, etc.
word (id, value, lang)         -- Words in different languages
meaning (id)                   -- Meaning entries
definition (id, meaning_id, value)  -- Definitions/explanations
lexeme (id, word_id, meaning_id, dataset_code)  -- Links word↔meaning↔dictionary
```

## Important Context
- **Ekilex**: EKI's dictionary editing system (PostgreSQL database)
- **Sõnaveeb**: Public dictionary portal (materialized view of Ekilex data, NO separate API)
- **Sketch Engine**: Corpus query system with its own API
- Database backup = yesterday's data, not real-time

## Language
All UI text is in Estonian. Key terms:
- Andmebaas = Database
- Päring = Query
- Sõnastik = Dictionary
- Tähendus = Meaning
- Seletus/Definitsioon = Definition
- Keelend = Lexical item/word
- Ühendsõnastik (ÜS) = Unified dictionary

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
- **Add collapsible section**: Use `<details class="collapsible-section">` pattern
- **Change table tab content**: Modify `loadAllTableContents()` function
