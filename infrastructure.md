# Project infrastructure / Onboarding

This document lists all relevant information about the "cacographie" project so an engineer or Copilot can start quickly and work immediately.

Repository
- Owner: yacine-harbi
- Repo: cacographie
- Description: "Spot the spelling/grammar mistake."
- Primary language: HTML (100%)
- Important files:
  - french_cacography_lessons.json (path: /french_cacography_lessons.json) — main lessons database (JSON object with key "lessons" containing an array of lesson objects).
  - french_cacography_lessons_extra.json (path: /french_cacography_lessons_extra.json) — additional lessons added previously.

Data format (lesson object)
- id: integer (unique within the lessons array)
- phrase: string (the sentence shown to the learner; may contain the mistake)
- errorIndex: integer or null — index (0-based or 1-based depending on product) of the erroneous token/character; the repo uses integers (historically 1-based in examples). Confirm usage in UI code if you automate highlighting.
- correction: string or null — corrected token or text (null if phrase is correct)
- explanation: string — short explanation of the error and correction
- difficulty: string — one of "facile", "moyen", "difficile" (used for filtering)
- categorie: string — category label such as "accents", "accords", "conjugaisons", "homophones", "orthographe", "ponctuation", "style", "aucune"

Character encoding and JSON rules
- Use UTF-8 encoding to preserve French diacritics (é, è, ç, œ, etc.).
- Keep the JSON valid and properly formatted. Use tools like `jq` for validation:
  - jq . french_cacography_lessons.json
- Maintain consistent field names (errorIndex, correction, explanation, difficulty, categorie).

Adding new sentences / contributions
- Ids must be unique. To add N new items, choose ids that continue from the highest current id.
  - Example: if highest id is 140, new items should start at 141.
- Provide full lesson objects following the schema above.
- Keep phrase strings short and focused on a single type of error when possible.
- Include an accurate explanation and categorize by difficulty and category.

Validation steps before committing
1. JSON validate: jq . french_cacography_lessons.json
2. Check encoding: file -I french_cacography_lessons.json (expect charset=utf-8)
3. Lint/format: Prettier or any JSON formatter to keep consistent style.
4. Spot-check a few phrases in the UI (open the HTML pages locally) to confirm special characters display correctly.

Suggested local dev workflow
- Clone repo:
  git clone https://github.com/yacine-harbi/cacographie.git
  cd cacographie
- Create a feature branch:
  git checkout -b add-lessons-<short-description>
- Update or add files (french_cacography_lessons.json or create a new file like french_cacography_lessons_extra.json)
- Validate JSON and encoding (see Validation steps).
- Commit with a clear message:
  git add french_cacography_lessons.json
  git commit -m "Add N French lessons: categories X, Y, Z"
- Push branch and open a Pull Request on GitHub

Pull Request & review
- Base branch: main
- Title suggestion: "Add X French lessons (categories: ... )"
- Description: list changes, how many lessons added, example entries, QA steps
- Request review from repository owner (@yacine-harbi) or collaborators
- Squash or standard merge according to preference (project is small; default merge is fine)

Automating tasks and Copilot instructions
- If you want Copilot to add lessons, give explicit instructions including:
  - Number of lessons to add
  - Difficulty distribution (e.g., 10 facile, 20 moyen, 20 difficile)
  - Categories to include and target counts per category
  - Whether to include corrections in the same file or only mistaken phrase
  - Desired id range or starting id
  - Preferred punctuation style (e.g., keep trailing punctuation, include duplicates with/without punctuation for UI testing)

Example prompt for Copilot to add 50 lessons:
"Add 50 French lesson objects to /french_cacography_lessons.json starting at id 141. Include a mix of categories (accents, accords, conjugaisons, homophones, orthographe, ponctuation) and difficulties (facile/moyen/difficile). Use UTF-8, ensure unique ids, and provide brief explanations. Return the updated JSON content only."

Merging extras into the main file (manual script)
- If you keep an extra file (french_cacography_lessons_extra.json), merge into main with a small Node/Python script that:
  1. Loads both JSON files
  2. Verifies there are no id collisions
  3. Appends lessons from the extra file to the main lessons array
  4. Sorts by id (optional)
  5. Writes updated JSON back to french_cacography_lessons.json

Simple Python merge example:

```python
# merge_lessons.py
import json

with open('french_cacography_lessons.json', 'r', encoding='utf-8') as f:
    main = json.load(f)

with open('french_cacography_lessons_extra.json', 'r', encoding='utf-8') as f:
    extra = json.load(f)

main_ids = {l['id'] for l in main['lessons']}
for l in extra['lessons']:
    if l['id'] in main_ids:
        raise SystemExit(f"ID collision: {l['id']}")
    main['lessons'].append(l)

# optional: sort by id
main['lessons'].sort(key=lambda x: x['id'])

with open('french_cacography_lessons.json', 'w', encoding='utf-8') as f:
    json.dump(main, f, ensure_ascii=False, indent=2)
```

Testing & QA recommendations
- Run the site locally (open the HTML pages) and test the display of sentences and corrections.
- Test edge cases: phrases with ellipses, question marks, apostrophes, multi-word corrections.
- Add unit tests if you have JS code that manipulates the lessons data (not present in the current HTML-only repo; consider adding simple JS tests if functionality is added).

Branching, commit messages and PR etiquette
- Branch naming: feature/add-lessons-<short-desc>
- Commit message template: "Add N lessons: categories: x,y — difficulty distribution: a/b/c"
- PR description: concise summary + list of sample entries + testing steps

Owner & contacts
- Repo owner and contact: yacine-harbi (GitHub handle)
- Commit email shown in recent commits: yacine.harbi@gmail.com

Notes & TODOs
- Consider normalizing category names (e.g., use either 'accents' or 'orthographe' consistently)
- Decide whether errorIndex is 1-based or 0-based and standardize in code and data
- Add a small JSON schema (schema.json) to validate lesson objects automatically with `ajv` or `jsonschema`

----

This file was added by Copilot on user request to centralize onboarding knowledge and make future changes easier and faster.