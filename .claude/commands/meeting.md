---
description: Capture a meeting — create note, extract actions, update person files
---

# /meeting

Capture a meeting and automatically file everything it produces.

## Usage

```
/meeting Haven Park Q2 planning with Sarah and Tom
/meeting 1:1 with Mike about the renovation budget
```

## Steps

1. **Create** meeting note at `work/meetings/YYYY-MM-DD-topic-kebab.md` from `templates/meeting.md` with frontmatter:
   ```yaml
   ---
   date: YYYY-MM-DD
   attendees: []
   project:
   tags: [meeting]
   links: []
   ---
   ```

2. **Ask** the user for meeting content (or accept it inline). Capture:
   - Attendees (fill frontmatter `attendees:` array)
   - Key discussion points
   - Decisions made
   - Action items with owners and deadlines

3. **For each attendee**, run the `/person` logic:
   - Create or update `work/people/firstname-lastname.md`
   - Add a back-link from the person file to this meeting note

4. **For each decision**, create or append to `work/decisions/YYYY-MM-DD-topic.md`:
   ```yaml
   ---
   date: YYYY-MM-DD
   decision:
   context:
   participants: []
   project:
   tags: [decision]
   ---
   ```

5. **For each action item**, add it to the relevant project's `## Open actions` section with:
   - `- [ ] Action description — @owner, due YYYY-MM-DD (from [[YYYY-MM-DD-meeting-slug]])`

6. **Cross-link** everything:
   - Meeting note links to attendee person files, project, and decision files
   - Today's daily note gets a `## Meetings` entry linking to the meeting note
   - Project files get back-links to the meeting

7. **Append** to `log.md`:
   ```
   ## [YYYY-MM-DD HH:MM] meeting | topic — N attendees, N decisions, N actions
   ```
