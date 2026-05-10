# Plan: Create Unified Quran-Hadith JSON by Verse

## TL;DR
Create 114 separate JSON files (one per Quran surah) in `data/`, where each file contains the surah metadata and verses, with hadiths that reference each verse nested inside. Use all hadith source files under `db/by_book/`, including `db/by_book/forties/`, `db/by_book/other_books/`, and `db/by_book/the_9_books/`. Every piece of implementation code for fetching, parsing, linking, validating, and writing output must be placed within [build_quran_hadith_by_verse.ipynb](build_quran_hadith_by_verse.ipynb) so the full workflow is easy to inspect, validate, and adjust in one place.

## Code Location Rule

All project code for this workflow belongs in [build_quran_hadith_by_verse.ipynb](build_quran_hadith_by_verse.ipynb). Do not create standalone Python scripts, TypeScript files, helper modules, or alternate implementation files for the Quran-hadith build process unless this plan is explicitly changed later.

## Setup: Python Environment

0. **Create Python Virtual Environment & Install Dependencies**
   - Create venv: `python3 -m venv venv`
   - Activate: `source venv/bin/activate` (macOS/Linux) or `venv\Scripts\activate` (Windows)
   - Install required packages from `pyproject.toml`:
     ```bash
     pip install -e .
     ```
   - Required packages:
     - `jupyter` + `notebook` — Jupyter notebook environment
     - `ipython` — interactive Python shell for notebook
     - `pandas` — read/parse JSON files, data manipulation
     - `requests` — fetch Quran data from Al-Quran Cloud API
     - `numpy` — optional, data processing
     - `python-Levenshtein` + `fuzzywuzzy` — fuzzy string matching for linking hadith references to Quran verses
   - Launch Jupyter notebook: `jupyter notebook` and open `build_quran_hadith_by_verse.ipynb`

## Steps

1. **Fetch Quran Data**
   - Fetch complete Quran data (114 surahs, all verses, Arabic/English text) from Al-Quran Cloud API
   - Extract surah metadata (ID, name Arabic/English, verse count)

2. **Parse Hadith References**
   - Iterate through all hadith JSON files in [by_book](http://_vscodecontentref_/2)
   - Include [forties](http://_vscodecontentref_/3), [other_books](http://_vscodecontentref_/4), and [the_9_books](http://_vscodecontentref_/5)
   - Extract Quran references using regex patterns (e.g., "Surah X Verse Y" in English, verse patterns in Arabic text)
   - Handle both Arabic and English reference formats

3. **Link References to Verses** (*parallel with step 2*)
   - Use fuzzy text matching (string-similarity library) to link extracted references to actual Quran verses
   - Handle paraphrases and partial quotes
   - Map each reference to exact surah and verse number

4. **Build Per-Surah JSON Files**
   - Create directory [data](http://_vscodecontentref_/6) with 114 files: `1-alfatiha.json`, `2-albaqarah.json`, ..., `114-alnass.json`
   - Each file structure:
     ```json
     {
       "surah": {
         "id": 1,
         "name_arabic": "الفاتحة",
         "name_english": "Al-Fatiha",
         "verses_count": 7
       },
       "verses": [
         {
           "verse_number": 1,
           "text_arabic": "...",
           "text_english": "...",
           "hadiths": [{ hadith_object }, ...]
         }
       ]
     }
     ```
   - Only include verses that have at least one hadith reference
   - Preserve all hadith metadata (id, narrator, book, chapter, etc.)
   - If a verse is missing from a surah output file, it is because there are no hadith references for that verse

5. **Update Project Structure**
   - Ensure [data](http://_vscodecontentref_/7) directory is used for output files
   - Keep [build_quran_hadith_by_verse.ipynb](build_quran_hadith_by_verse.ipynb) as the only implementation and validation environment
   - Keep package/dependency metadata in `pyproject.toml`
   - Do not add implementation code to `index.ts`, `scrapeData.ts`, helper scripts, or standalone `.py`/`.ts` files

## Relevant Files

- [by_book](http://_vscodecontentref_/10) — source hadith data for parsing references, including `forties/`, `other_books/`, and `the_9_books/`
- [build_quran_hadith_by_verse.ipynb](build_quran_hadith_by_verse.ipynb) — the only place for implementation code, validation code, and workflow notes
- `pyproject.toml` — Python dependency and project metadata
- [data](http://_vscodecontentref_/13) — output directory for 114 generated surah files (e.g., `1-alfatiha.json`, `2-albaqarah.json`)

## Verification

1. Verify all 114 surah files are created with correct naming (1-alfatiha.json through 114-alnass.json)
2. Spot-check 5 surahs (e.g., 1, 2, 65, 100, 114) for:
   - Correct surah metadata (ID, names, verse count)
   - At least one hadith linked to verses
   - Complete hadith metadata preserved
3. Count total verses with hadiths across all surahs (expect 1000+ linked hadiths)
4. Validate JSON schema for each file

## Decisions

- Modular structure: 114 separate files instead of one combined file (better performance, maintainability)
- Output directory: [data](http://_vscodecontentref_/14) to keep generated JSON separate from raw [db](http://_vscodecontentref_/15) source files
- Use Al-Quran Cloud API for Quran data (free, structured, includes translations)
- Include only explicit hadith references to Quran verses (skip implicit ones to avoid noise)
- Handle Arabic/English references separately for better matching accuracy
- Exclude verses with zero hadith links from output to reduce file clutter
- Surah file naming: `{id}-{english_name_lowercase}.json` (e.g., `1-alfatiha.json`, `2-albaqarah.json`)
- Notebook-only implementation: all workflow code stays in [build_quran_hadith_by_verse.ipynb](build_quran_hadith_by_verse.ipynb)

## Further Considerations

1. **Parallel Processing**: Can fetch and link each surah independently (114 operations can run in parallel for speed)
2. **Reference Accuracy**: Some hadiths reference multiple verses or surah ranges—decide whether to link to all or primary verse only
3. **Future Enhancement**: Could add verse topics/themes, tafsir (interpretation), or cross-references between surahs

## Current Findings (from exploration)

- No standalone Quran data in project currently
- Quran references are embedded in hadith text (not structured)
- ~40-60% of hadiths likely reference Quran directly
- English translations have more explicit verse numbers than Arabic originals
- Multiple reference formats exist (explicit/paraphrased, direct/implied)
