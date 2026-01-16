# Ekilex Database Model Reference

This document provides an overview of the Ekilex data model for generating SQL queries from natural language descriptions.

## Core Concepts

Ekilex is a dictionary and terminology database. The fundamental hierarchy is:

```
WORD → LEXEME → MEANING → DEFINITION
```

| Concept | Estonian | Description |
|---------|----------|-------------|
| **Word** | keelend | A morphological form (the actual text, e.g., "koer") |
| **Lexeme** | ilmik | A word-meaning pair within a specific dataset (dictionary entry) |
| **Meaning** | tähendus | A semantic concept that can have multiple definitions |
| **Definition** | seletus/definitsioon | Textual explanation of a meaning |

### Important Relationships

- A **word** can have multiple **lexemes** (one per dataset/meaning combination)
- A **lexeme** connects exactly one word to one meaning within one dataset
- A **meaning** can have multiple **definitions** (in different languages)
- A **meaning** can be shared across multiple lexemes/words (synonyms)

---

## Main Tables

### word
Stores word forms (the actual text entries).

| Column | Type | Description |
|--------|------|-------------|
| id | bigint | Primary key |
| value | text | Word text (e.g., "koer", "maja") |
| value_prese | text | Formatted/presentation value |
| lang | char(3) | Language code (e.g., 'est', 'eng', 'rus') |
| homonym_nr | integer | Distinguishes homonyms (words spelled the same) |
| gender_code | varchar | Grammatical gender (m/f/n) |
| aspect_code | varchar | Verbal aspect (for Russian verbs) |
| is_public | boolean | Whether publicly visible |
| reg_year | integer | Registration year (for OS/standard dictionary) |

### lexeme
Links words to meanings within datasets. This is the central junction table.

| Column | Type | Description |
|--------|------|-------------|
| id | bigint | Primary key |
| word_id | bigint | FK → word |
| meaning_id | bigint | FK → meaning |
| dataset_code | varchar(10) | FK → dataset (e.g., 'ss1', 'eki', 'psv') |
| level1, level2 | integer | Hierarchical sense numbers |
| value_state_code | varchar | Data quality state |
| proficiency_level_code | varchar | CEFR level (A1-C2) |
| is_public | boolean | Whether publicly visible |
| weight | numeric | Importance ranking |

**Unique constraint**: (word_id, meaning_id, dataset_code)

### meaning
Stores semantic concepts.

| Column | Type | Description |
|--------|------|-------------|
| id | bigint | Primary key |
| manual_event_on | timestamp | Last manual edit |

Meanings are linked to definitions, domains, semantic types, and other meanings via separate tables.

### definition
Textual explanations of meanings.

| Column | Type | Description |
|--------|------|-------------|
| id | bigint | Primary key |
| meaning_id | bigint | FK → meaning |
| value | text | Definition text |
| value_prese | text | Formatted definition |
| lang | char(3) | Definition language |
| definition_type_code | varchar | Type: 'definitsioon', 'selgitus', etc. |
| is_public | boolean | Whether publicly visible |

### dataset
Dictionary/terminology collections.

| Column | Type | Description |
|--------|------|-------------|
| code | varchar(10) | Primary key (e.g., 'ss1', 'eki', 'qq2') |
| name | text | Dataset name |
| type | varchar | Dataset type |
| is_public | boolean | Public visibility |
| is_superior | boolean | Has precedence over others |

---

## Word Forms and Morphology

### paradigm
Morphological paradigms (declension/conjugation patterns).

| Column | Type | Description |
|--------|------|-------------|
| id | bigint | Primary key |
| word_id | bigint | FK → word |
| word_class | varchar | Morphological class |
| inflection_type | varchar | Inflection type name |

### form
Individual word forms with morphological markers.

| Column | Type | Description |
|--------|------|-------------|
| id | bigint | Primary key |
| value | text | Form text (e.g., "koera", "koeraga") |
| morph_code | varchar | Morphological marker (e.g., 'SgG', 'PlN') |

### paradigm_form
Links paradigms to forms.

| Column | Type | Description |
|--------|------|-------------|
| paradigm_id | bigint | FK → paradigm |
| form_id | bigint | FK → form |
| morph_group1/2/3 | varchar | Morphological groupings |
| display_level | integer | Display priority |

---

## Relations Between Entries

### word_relation
Morphological/etymological relations between words.

| Column | Type | Description |
|--------|------|-------------|
| word1_id | bigint | FK → word |
| word2_id | bigint | FK → word |
| word_rel_type_code | varchar | Relation type |

### lex_relation
Relations between lexemes (word senses).

| Column | Type | Description |
|--------|------|-------------|
| lexeme1_id | bigint | FK → lexeme |
| lexeme2_id | bigint | FK → lexeme |
| lex_rel_type_code | varchar | Relation type |

### meaning_relation
Semantic relations (synonyms, antonyms, hypernyms, etc.).

| Column | Type | Description |
|--------|------|-------------|
| meaning1_id | bigint | FK → meaning |
| meaning2_id | bigint | FK → meaning |
| meaning_rel_type_code | varchar | E.g., 'antonüüm', 'sünonüüm', 'hüperonüüm' |
| weight | numeric | Relation strength |

---

## Classification Tables

### lexeme_pos
Parts of speech for lexemes.

| Column | Type | Description |
|--------|------|-------------|
| lexeme_id | bigint | FK → lexeme |
| pos_code | varchar | E.g., 's' (noun), 'v' (verb), 'adj' (adjective) |

### lexeme_register
Style/register markers.

| Column | Type | Description |
|--------|------|-------------|
| lexeme_id | bigint | FK → lexeme |
| register_code | varchar | E.g., 'kõnek' (colloquial), 'van' (archaic) |

### meaning_domain
Thematic domains.

| Column | Type | Description |
|--------|------|-------------|
| meaning_id | bigint | FK → meaning |
| domain_code | varchar | Domain code |
| domain_origin | varchar | Domain classification system |

### meaning_semantic_type
Semantic type classification.

| Column | Type | Description |
|--------|------|-------------|
| meaning_id | bigint | FK → meaning |
| semantic_type_code | varchar | E.g., 'abstraktne', 'konkreetne' |

---

## Usage and Examples

### usage
Usage examples for lexemes.

| Column | Type | Description |
|--------|------|-------------|
| id | bigint | Primary key |
| lexeme_id | bigint | FK → lexeme |
| value | text | Usage example text |
| lang | char(3) | Language |
| is_public | boolean | Public visibility |

### usage_translation
Translations of usage examples.

| Column | Type | Description |
|--------|------|-------------|
| usage_id | bigint | FK → usage |
| value | text | Translated text |
| lang | char(3) | Target language |

### government
Case government patterns (what cases a word governs).

| Column | Type | Description |
|--------|------|-------------|
| lexeme_id | bigint | FK → lexeme |
| value | text | Government pattern |

### grammar
Grammatical notes.

| Column | Type | Description |
|--------|------|-------------|
| lexeme_id | bigint | FK → lexeme |
| value | text | Grammar note |
| lang | char(3) | Language |

---

## Notes and Comments

### lexeme_note, meaning_note, definition_note
Notes attached to various entities.

| Column | Type | Description |
|--------|------|-------------|
| [entity]_id | bigint | FK → parent entity |
| value | text | Note text |
| lang | char(3) | Language |
| is_public | boolean | Public visibility |

### word_forum, meaning_forum
Internal discussion/notes (not public).

| Column | Type | Description |
|--------|------|-------------|
| [entity]_id | bigint | FK → parent entity |
| value | text | Forum comment |
| creator_id | bigint | FK → eki_user |

---

## Tags and Markers

### word_tag, lexeme_tag, meaning_tag
Tags applied to entries for workflow/classification.

| Column | Type | Description |
|--------|------|-------------|
| [entity]_id | bigint | FK → parent entity |
| tag_name | varchar | Tag identifier |
| created_on | timestamp | When tag was applied |

### tag
Tag definitions.

| Column | Type | Description |
|--------|------|-------------|
| name | varchar | Tag identifier |
| type | varchar | Tag type |
| set_automatically | boolean | Auto-applied by system |
| remove_to_complete | boolean | Must remove to finish editing |

---

## Sources and Citations

### source
Reference sources for definitions, examples, etc.

| Column | Type | Description |
|--------|------|-------------|
| id | bigint | Primary key |
| dataset_code | varchar | FK → dataset |
| type | varchar | Source type |
| name | text | Source name/title |
| value | text | Citation details |

### definition_source_link, lexeme_source_link, usage_source_link
Links entities to their sources.

| Column | Type | Description |
|--------|------|-------------|
| [entity]_id | bigint | FK → parent entity |
| source_id | bigint | FK → source |
| name | varchar | Link type/name |
| value | text | Specific citation |

---

## Classifier/Lookup Tables

These tables define controlled vocabularies. Each has a corresponding `*_label` table with multilingual labels.

| Table | Purpose | Common Codes |
|-------|---------|--------------|
| language | Language codes | est, eng, rus, deu, fra, lat |
| pos | Parts of speech | s (noun), v (verb), adj, adv, konj |
| register | Style registers | kõnek (colloquial), van (archaic), tehn (technical) |
| domain | Thematic domains | med (medicine), jur (law), bot (botany) |
| gender | Grammatical gender | m, f, n, mf |
| aspect | Verb aspect | сов, несов (Russian) |
| value_state | Data quality | verified, uncertain, deprecated |
| word_type | Word categories | Various word classifications |
| definition_type | Definition types | definitsioon, selgitus, lühivihje |
| morph | Morphological tags | SgN, SgG, PlN, PlP, etc. |

---

## Common Query Patterns

### Find a word with its meanings and definitions

```sql
SELECT w.value AS word, w.lang,
       d.value AS definition, d.lang AS def_lang,
       l.dataset_code, l.level1, l.level2
FROM word w
JOIN lexeme l ON l.word_id = w.id
JOIN meaning m ON m.id = l.meaning_id
LEFT JOIN definition d ON d.meaning_id = m.id
WHERE w.value = 'koer'
  AND w.is_public = true
  AND l.is_public = true
ORDER BY l.dataset_code, l.level1, l.level2;
```

### Find synonyms

```sql
SELECT w2.value AS synonym
FROM word w1
JOIN lexeme l1 ON l1.word_id = w1.id
JOIN meaning_relation mr ON mr.meaning1_id = l1.meaning_id
JOIN lexeme l2 ON l2.meaning_id = mr.meaning2_id
JOIN word w2 ON w2.id = l2.word_id
WHERE w1.value = 'ilus'
  AND mr.meaning_rel_type_code = 'sünonüüm'
  AND w2.id != w1.id;
```

### Find words by part of speech

```sql
SELECT DISTINCT w.value
FROM word w
JOIN lexeme l ON l.word_id = w.id
JOIN lexeme_pos lp ON lp.lexeme_id = l.id
WHERE lp.pos_code = 'v'  -- verbs
  AND w.lang = 'est'
  AND w.is_public = true;
```

### Find words in a domain

```sql
SELECT DISTINCT w.value, d.value AS definition
FROM word w
JOIN lexeme l ON l.word_id = w.id
JOIN meaning m ON m.id = l.meaning_id
JOIN meaning_domain md ON md.meaning_id = m.id
LEFT JOIN definition d ON d.meaning_id = m.id AND d.lang = 'est'
WHERE md.domain_code = 'med'
  AND w.is_public = true;
```

### Find word forms (paradigm)

```sql
SELECT f.value AS form, f.morph_code
FROM word w
JOIN paradigm p ON p.word_id = w.id
JOIN paradigm_form pf ON pf.paradigm_id = p.id
JOIN form f ON f.id = pf.form_id
WHERE w.value = 'koer'
ORDER BY pf.order_by;
```

### Find usage examples

```sql
SELECT w.value, u.value AS example, u.lang
FROM word w
JOIN lexeme l ON l.word_id = w.id
JOIN usage u ON u.lexeme_id = l.id
WHERE w.value = 'jooksma'
  AND u.is_public = true;
```

### Find words with a specific tag

```sql
SELECT w.value
FROM word w
JOIN word_tag wt ON wt.word_id = w.id
WHERE wt.tag_name = 'kopimisel'
ORDER BY w.value;
```

### Find words registered in a specific year (OS)

```sql
SELECT w.value, w.reg_year
FROM word w
WHERE w.reg_year = 2023
  AND w.is_public = true
ORDER BY w.value;
```

### Count words per dataset

```sql
SELECT l.dataset_code, COUNT(DISTINCT w.id) AS word_count
FROM word w
JOIN lexeme l ON l.word_id = w.id
WHERE w.is_public = true
  AND l.is_public = true
GROUP BY l.dataset_code
ORDER BY word_count DESC;
```

---

## Key Tips for Query Generation

1. **Always join through lexeme**: To connect words with meanings/definitions, go through the `lexeme` table.

2. **Check public visibility**: Use `is_public = true` filters for user-facing queries.

3. **Dataset filtering**: Most queries should filter by `dataset_code` to get data from a specific dictionary.

4. **Language codes**: Use 3-letter codes: 'est' (Estonian), 'eng' (English), 'rus' (Russian), 'deu' (German).

5. **Homonyms**: Words with the same spelling are distinguished by `homonym_nr`.

6. **Levels**: `level1` and `level2` in lexeme indicate sense hierarchy (1.1, 1.2, 2.1, etc.).

7. **Presentation values**: Columns ending in `_prese` contain formatted text with markup.

8. **Labels**: For user-friendly classifier names, join with `*_label` tables and filter by `lang` and `type`.

9. **Bidirectional relations**: Relations (word, lexeme, meaning) may need to be queried in both directions.

10. **Timestamps**: `manual_event_on` indicates last human edit; activity logs track all changes.

---

## Entity Relationship Summary

```
                    ┌─────────────────────────────────────────────────┐
                    │                   DATASET                       │
                    │  (code, name, is_public, is_superior)           │
                    └──────────────────────┬──────────────────────────┘
                                           │
         ┌─────────────────────────────────┼─────────────────────────────────┐
         │                                 │                                 │
         ▼                                 ▼                                 ▼
┌─────────────────┐              ┌─────────────────┐              ┌─────────────────┐
│      WORD       │              │     LEXEME      │              │    MEANING      │
│  (id, value,    │◄─────────────│  (word_id,      │─────────────►│  (id)           │
│   lang,         │              │   meaning_id,   │              │                 │
│   homonym_nr,   │              │   dataset_code, │              │                 │
│   is_public)    │              │   level1/2,     │              │                 │
└────────┬────────┘              │   is_public)    │              └────────┬────────┘
         │                       └────────┬────────┘                       │
         │                                │                                │
    ┌────┴────┐                    ┌──────┴──────┐                  ┌──────┴──────┐
    ▼         ▼                    ▼             ▼                  ▼             ▼
paradigm  word_tag            lexeme_pos    usage            definition    meaning_domain
    │                         lexeme_reg    grammar          meaning_note  meaning_relation
    ▼                         lexeme_tag    government       meaning_tag   semantic_type
  form                        lex_relation  lexeme_note
```

---

## Frequently Used Dataset Codes

| Code | Name | Description |
|------|------|-------------|
| ss1 | Sõnaveeb | Main unified dictionary |
| eki | EKI | Estonian Language Institute data |
| psv | PSV | Dictionary of standard Estonian |
| qq2 | ÕS | Orthographic dictionary |
| ev | EV | Estonian-Russian dictionary |
| les | LES | Estonian scientific terminology |

---

## Activity and Audit

Changes are tracked in:
- `activity_log` - All changes with diffs
- `word_activity_log`, `meaning_activity_log`, `lexeme_activity_log` - Entity-specific logs
- `*_last_activity_log` - Most recent change per entity

This allows querying change history, finding who modified what and when.
