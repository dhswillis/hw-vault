---
description: Create or update a person file from a conversation mention
---

# /person

Create or update a person file in `work/people/` when someone is mentioned in a meeting, daily note, or conversation.

## Usage

```
/person Sarah Chen — PM on the Haven Park renovation project, met at Q1 review
/person Tom — cache migration deferred to Q2, he owns the auth module
```

## Steps

1. **Parse** the person's name and any context provided.

2. **Check** if `work/people/firstname-lastname.md` already exists.
   - If yes: read it, append new context under a `## Notes` section with today's date.
   - If no: create it from `templates/person.md` with frontmatter:
     ```yaml
     ---
     created: YYYY-MM-DD
     updated: YYYY-MM-DD
     type: person
     role:
     organization:
     tags: [person]
     related: []
     ---
     ```

3. **Extract** structured data from the context:
   - Role / title (if mentioned)
   - Organization (default: Haven Park Communities if work context)
   - Projects they're associated with
   - Key decisions or commitments they've made
   - Relationship to Harrison (colleague, client, vendor, etc.)

4. **Cross-link** the person file:
   - Add `[[wiki-links]]` to any projects, concepts, or entities mentioned
   - Add the person file path to the `related:` frontmatter of linked project files
   - If mentioned in today's daily note, add a `[[work/people/firstname-lastname]]` link there

5. **Update** `index.md` — add the person under a `## People` section if not already present.

6. **Append** to `log.md`:
   ```
   ## [YYYY-MM-DD HH:MM] person | created/updated firstname-lastname
   ```
