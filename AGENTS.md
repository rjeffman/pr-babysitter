# General rules for using AI agents

## Code style

These are common errors that **MUST BE AVOIDED**:

- Prefer putting braces around variable references even when not strictly required.
- You should not use `echo "${VARIABLE}" | command`, prefer `command <<<"${VARIABLE}"`

### Checking code style

To check code style run:

```
shellcheck -x -a -o all -e SC2292 -e SC2310 -e SC2311 pr-babysitter
```

## Managing Git repository

**MANDATORY**: **Never** commit changes automatically, wait for developer explicit request to commit changes.

Follow the restrictions on the file `.gitignore` and **never** commit any of this files:
    - CLAUDE.md
    - GEMINI.md

## Commit message

- **MANDATORY**: No line on the commit message must have more than 72 characters.
- **MANDATORY**: Always add "Assisted-by: AGENT IDENTIFICATION <AGENT E-MAIL>"
- **MANDATORY**: Always commit with '-s/--signoff'
- Each commit should represent a single, logical change
- Use bullet points for multiple changes.
- **ALWAYS** add a bullet list of the prompts used to generate the change being commited.- **NEVER** add promps like "commit changes" to the list of prompts.
