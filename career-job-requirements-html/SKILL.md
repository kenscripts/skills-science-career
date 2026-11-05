---
name: career-job-requirements-html
description: Turn a job posting (URL, pasted text, or file) into an interactive HTML page listing what the posting requires of the candidate, in its own wording, with a notes box under every point. Use when the user shares a job posting and wants its requirements laid out, quoted verbatim, to annotate how they match.
---

# Job requirements notes page

Build a single self-contained HTML file that lays out everything a job posting asks of the candidate, quoted in the posting's own words, with a notes text box under each point so the user can record how they match. Notes autosave in the browser, can be saved into a new copy of the HTML file itself ("Save page with notes"), and can be exported as a .txt file.

## 1. Get the posting text

- **URL:** fetch it with WebFetch and ask for each section's bullets *quoted exactly, one per line, with their section heading, no paraphrasing*. Also ask for the opening description paragraph(s) verbatim.
- WebFetch often truncates long quotes (about 125 characters) and marks them with "…". For every truncated item, fetch again asking only for "the remaining text that follows `<last few words>`", then rejoin the pieces. Record which items were rejoined for the page's source note.
- If the site still will not give exact wording, tell the user which items are incomplete and ask them to paste the posting text rather than paraphrasing.
- **Pasted text or attached file:** use it directly (read .pdf/.docx with the usual file tools).

## 2. Extract what is required of the candidate

Pull these, verbatim. Do not summarize, merge, reorder or "improve" wording.

1. **Header facts:** company, requisition/job ID, exact job title, and a short facts line (group/department · location · term or schedule) built only from facts the posting states.
2. **Summary:** one opening sentence from the description, if there is one.
3. **Successful candidate:** every sentence in the description/overview paragraphs that describes what the candidate will specialize in, identify, own, or be expected to do — typically starting "A successful candidate will…", "The candidate will…", "The ideal candidate…", "You will…". These are easy to miss because they sit in prose, not bullets. Read the whole description and collect all of them.
4. **Required qualifications** (also titled Minimum / Basic / Must have).
5. **Preferred qualifications** (also titled Desired / Nice to have / Bonus).
6. **Responsibilities** (also titled Duties / What you'll do / Key activities).

Rules:
- Keep nesting. A lead-in such as "Research experience in one or more of the following areas:" becomes an item with `sub` points, so the "one or more" logic stays visible.
- Use the posting's own section titles where they differ meaningfully; otherwise use the four titles above. Drop any section the posting doesn't have.
- Fix only an obvious typo, and say so in `sourceNote` and in your reply (e.g. "module" → "modulate").
- Skip benefits, salary, EEO statements and company boilerplate unless the user asks for them.

## 3. Build the page

Copy `assets/template.html` (in this skill's folder) to `<company>_<role>_requirements.html` (lowercase, underscores) in the working directory. Replace **only** the `POSTING` object between the `FILL` markers; leave styles and script untouched, and leave `const SAVED_NOTES = {};` exactly as it is (one line, empty) for a new page. Strings go in as plain text — the script inserts them with `textContent`, so no HTML escaping is needed, but escape any `"` or `\` inside JS strings.

- `required: true` for the "Successful candidate" and "Required qualifications" sections.
- `kind`: `"candidate"` for prose sentences, `"ordered"` for required qualifications, `"bullets"` for the rest.
- `retrieved`: today's date written out (e.g. "November 4, 2026").

## 4. Check it

Run it headless with Playwright if available (`require('playwright')`, `chromium.launch()`), or any other headless browser. If none is available, skip the checks and tell the user they were not run:
- no `pageerror` events;
- number of `textarea.notes` = number of leaf points (each string item + each `sub` point + each candidate sentence);
- `document.documentElement.scrollWidth` equals the viewport width at 390px (no sideways scroll);
- type into two boxes (one with a line break), click "Save page with notes (.html)", open the downloaded copy in a **new browser context** (no shared localStorage) and confirm both notes reappear and the copy has exactly one `const SAVED_NOTES` line;
- optionally click "Download notes (.txt)" and confirm the export lists every point.

## 5. Deliver

Deliver the file to the user (use the environment's file-sending or preview tool if there is one; otherwise give its full path). In one or two lines, tell the user:
- notes autosave only in the browser they type in (the in-chat preview may not keep them); to keep them with the file, open it in their own browser and click **Save page with notes (.html)**, which downloads a copy of the page with the notes inside it that reopens with the notes anywhere. **Download notes (.txt)** gives a plain-text copy;
- anything rejoined, truncated, or typo-fixed that they should check against the posting.

If the user's CV or cover letters are in the conversation, you may add one sentence noting which "one or more" requirements they clearly meet — but do not prefill the notes boxes; those are the user's.

When the user later points out a missed sentence or asks for changes, edit the `POSTING` object (or the template) and resend the same file. If they attach a page they saved with notes, edit that file and keep its `SAVED_NOTES` line as is so their notes survive. Notes are matched to points by position, so if you add, remove or reorder points in a saved page, move the affected `"n<index>"` keys in `SAVED_NOTES` to match and tell the user.

How saving works (for troubleshooting): when the page loads, the script snapshots the file's own source before rendering. "Save page with notes" writes that snapshot back out with the `SAVED_NOTES` line replaced by `{"t":<timestamp>,"notes":{...}}`. On load, the page uses whichever is newer, the notes saved in the file or the browser's autosave.
