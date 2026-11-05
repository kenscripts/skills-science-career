---
name: "science-career-job-requirements-html"
description: "Turn a job posting (URL, pasted text, or file) into an interactive HTML page listing what the posting requires of the candidate, in its own wording, with a notes box under every point."
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

Copy the template below to `<company>_<role>_requirements.html` (lowercase, underscores) in the working directory. Replace **only** the `POSTING` object between the `FILL` markers; leave styles and script untouched, and leave `const SAVED_NOTES = {};` exactly as it is (one line, empty) for a new page. Strings go in as plain text — the script inserts them with `textContent`, so no HTML escaping is needed, but escape any `"` or `\` inside JS strings.

- `required: true` for the "Successful candidate" and "Required qualifications" sections.
- `kind`: `"candidate"` for prose sentences, `"ordered"` for required qualifications, `"bullets"` for the rest.
- `retrieved`: today's date written out (e.g. "November 4, 2026").

## 4. Check it

Run it headless with Playwright (preinstalled; `require('playwright')`, `chromium.launch()`):
- no `pageerror` events;
- number of `textarea.notes` = number of leaf points (each string item + each `sub` point + each candidate sentence);
- `document.documentElement.scrollWidth` equals the viewport width at 390px (no sideways scroll);
- type into two boxes (one with a line break), click "Save page with notes (.html)", open the downloaded copy in a **new browser context** (no shared localStorage) and confirm both notes reappear and the copy has exactly one `const SAVED_NOTES` line;
- optionally click "Download notes (.txt)" and confirm the export lists every point.

## 5. Deliver

Send the file with SendUserFile (`display: "render"`). In one or two lines, tell the user:
- notes autosave only in the browser they type in (the in-chat preview may not keep them); to keep them with the file, open it in their own browser and click **Save page with notes (.html)**, which downloads a copy of the page with the notes inside it that reopens with the notes anywhere. **Download notes (.txt)** gives a plain-text copy;
- anything rejoined, truncated, or typo-fixed that they should check against the posting.

If the user's CV or cover letters are in the conversation, you may add one sentence noting which "one or more" requirements they clearly meet — but do not prefill the notes boxes; those are the user's.

When the user later points out a missed sentence or asks for changes, edit the `POSTING` object (or the template) and resend the same file. If they attach a page they saved with notes, edit that file and keep its `SAVED_NOTES` line as is so their notes survive. Notes are matched to points by position, so if you add, remove or reorder points in a saved page, move the affected `"n<index>"` keys in `SAVED_NOTES` to match and tell the user.

How saving works (for troubleshooting): when the page loads, the script snapshots the file's own source before rendering. "Save page with notes" writes that snapshot back out with the `SAVED_NOTES` line replaced by `{"t":<timestamp>,"notes":{...}}`. On load, the page uses whichever is newer, the notes saved in the file or the browser's autosave.

## Template

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Job Requirements Notes</title>
<style>
  :root { --bg:#fbfaf7; --surface:#fff; --text:#1d1d1b; --muted:#6b6a66; --line:#e4e1da; --accent:#b4232a; --accent-soft:#fbeceb; --chip:#f1efea; }
  @media (prefers-color-scheme: dark) { :root:not([data-theme="light"]) { --bg:#161615; --surface:#1f1f1d; --text:#ecebe7; --muted:#a09e98; --line:#34332f; --accent:#ef6b6f; --accent-soft:#3a2021; --chip:#2a2927; } }
  :root[data-theme="dark"] { --bg:#161615; --surface:#1f1f1d; --text:#ecebe7; --muted:#a09e98; --line:#34332f; --accent:#ef6b6f; --accent-soft:#3a2021; --chip:#2a2927; }
  * { box-sizing: border-box; }
  body { margin:0; background:var(--bg); color:var(--text); font:16px/1.55 Georgia,"Times New Roman",serif; }
  main { max-width:760px; margin:0 auto; padding:40px 16px 64px; }
  .eyebrow { font:600 12px/1 system-ui,sans-serif; letter-spacing:.08em; text-transform:uppercase; color:var(--accent); }
  h1 { font-size:30px; line-height:1.2; margin:10px 0 8px; font-weight:600; }
  .meta { font:14px/1.5 system-ui,sans-serif; color:var(--muted); margin:0 0 28px; }
  .summary { border-left:3px solid var(--accent); padding:4px 0 4px 16px; margin:0 0 28px; font-style:italic; }
  section { background:var(--surface); border:1px solid var(--line); border-radius:10px; padding:22px 24px; margin-bottom:20px; }
  h2 { font:600 13px/1 system-ui,sans-serif; letter-spacing:.06em; text-transform:uppercase; margin:0 0 16px; display:flex; align-items:center; gap:10px; }
  h2 .count { background:var(--chip); color:var(--muted); border-radius:999px; padding:4px 9px; font-size:12px; letter-spacing:0; }
  section.required h2 { color:var(--accent); }
  section p.pt { margin:0 0 4px; } section p.pt ~ p.pt { margin-top:12px; }
  ol, ul { margin:0; padding-left:22px; } li { margin:0 0 10px; } li:last-child { margin-bottom:0; }
  ul.sub { margin-top:10px; padding-left:4px; list-style:none; }
  ul.sub li { background:var(--accent-soft); border-radius:6px; padding:8px 12px; margin-bottom:6px; }
  textarea.notes { display:block; width:100%; margin:8px 0 4px; min-height:52px; resize:vertical; font:14px/1.45 system-ui,sans-serif; color:var(--text); background:var(--bg); border:1px dashed var(--line); border-radius:6px; padding:8px 10px; }
  textarea.notes:focus { outline:none; border-style:solid; border-color:var(--accent); }
  textarea.notes::placeholder { color:var(--muted); }
  ul.sub li textarea.notes { background:var(--surface); }
  .toolbar { display:flex; gap:10px; flex-wrap:wrap; margin:0 0 24px; }
  .toolbar button { font:600 13px/1 system-ui,sans-serif; padding:9px 14px; border-radius:6px; cursor:pointer; background:var(--surface); color:var(--text); border:1px solid var(--line); }
  .toolbar button:hover { border-color:var(--accent); }
  .toolbar button.primary { background:var(--accent); border-color:var(--accent); color:#fff; }
  .status { font:13px/32px system-ui,sans-serif; color:var(--muted); }
  .note { font:13px/1.5 system-ui,sans-serif; color:var(--muted); margin-top:28px; } .note a { color:inherit; }
  @media print { body { background:#fff; } section { break-inside:avoid; } .toolbar { display:none; } textarea.notes { border:none; padding:0 0 0 10px; resize:none; } }
</style>
</head>
<body>
<main id="app"></main>
<script>
/* ==== FILL THIS OBJECT ONLY. Every string is quoted verbatim from the posting. ==== */
const POSTING = {
  company: "Company",
  reqId: "",                       // e.g. "176041"; "" if none
  title: "Job Title",
  meta: "Group · City, ST · Term",  // short facts line; "" to omit
  summary: "",                      // one verbatim opening sentence; "" to omit
  sections: [
    // kind: "candidate" = paragraphs; "ordered" = numbered list; "bullets" = bulleted list
    // required: true colors the heading in the accent color
    // item: "text"  or  { text: "Lead-in sentence:", sub: ["sub point", "sub point"] }
    { title: "Successful candidate", kind: "candidate", required: true, items: ["…"] },
    { title: "Required qualifications", kind: "ordered", required: true, items: ["…"] },
    { title: "Preferred qualifications", kind: "bullets", required: false, items: ["…"] },
    { title: "Responsibilities", kind: "bullets", required: false, items: ["…"] }
  ],
  sourceUrl: "",                    // posting URL; "" if pasted
  retrieved: "Month D, YYYY",
  sourceNote: ""                    // e.g. which bullets were rejoined or typos fixed; "" if none
};
/* ==== END FILL ==== */

/* Notes baked in by "Save page with notes". Leave as {} when building a new page. */
const SAVED_NOTES = {};  // one line only; becomes {"t":<time>,"notes":{…}} when saved

(function () {
  // Snapshot of this file as loaded, used to write a copy with notes baked in
  const SOURCE = "<!doctype html>\n" + document.documentElement.outerHTML;
  const P = POSTING, app = document.getElementById("app");
  const el = (tag, cls, text) => { const e = document.createElement(tag); if (cls) e.className = cls; if (text != null) e.textContent = text; return e; };
  document.title = (P.title + " Notes").slice(0, 60);

  app.appendChild(el("div", "eyebrow", P.company + (P.reqId ? " · Req. " + P.reqId : "")));
  app.appendChild(el("h1", null, P.title));
  if (P.meta) app.appendChild(el("p", "meta", P.meta));
  if (P.summary) app.appendChild(el("p", "summary", "“" + P.summary + "”"));

  const bar = el("div", "toolbar");
  const savePage = el("button", "primary", "Save page with notes (.html)"), exp = el("button", null, "Download notes (.txt)"), clr = el("button", null, "Clear all notes"), status = el("span", "status");
  savePage.type = exp.type = clr.type = "button"; bar.append(savePage, exp, clr, status); app.appendChild(bar);

  const KEY = "jobnotes:" + (P.company + "|" + (P.reqId || P.title)).toLowerCase();
  // Notes come from whichever is newer: the copy saved inside this file, or this browser's autosave
  let local = null; try { local = JSON.parse(localStorage.getItem(KEY) || "null"); } catch (e) { local = null; }
  if (local && !local.notes) local = { t: 0, notes: local };  // older autosave format
  const baked = SAVED_NOTES.notes ? SAVED_NOTES : { t: 0, notes: {} };
  let store = Object.assign({}, (local && local.t >= baked.t) ? local.notes : baked.notes);
  const save = () => { status.textContent = ""; try { localStorage.setItem(KEY, JSON.stringify({ t: Date.now(), notes: store })); status.textContent = "Notes saved in this browser"; } catch (e) { status.textContent = "Notes can't be saved here; download them before closing"; } };
  const boxes = []; let n = 0;
  const addBox = (host, label, section, after) => {
    const id = "n" + (n++), ta = el("textarea", "notes");
    ta.placeholder = "Your notes…"; ta.setAttribute("aria-label", "Notes: " + label); ta.value = store[id] || "";
    ta.addEventListener("input", () => { if (ta.value) store[id] = ta.value; else delete store[id]; save(); });
    after ? host.after(ta) : host.appendChild(ta);
    boxes.push({ section, label, ta });
  };

  P.sections.forEach(s => {
    if (!s.items || !s.items.length) return;
    const sec = el("section", s.required ? "required" : null), h = el("h2", null, s.title);
    if (s.kind !== "candidate") h.appendChild(el("span", "count", String(s.items.length)));
    sec.appendChild(h);
    if (s.kind === "candidate") {
      s.items.forEach(t => { const p = el("p", "pt", t); sec.appendChild(p); addBox(p, t, s.title, true); });
    } else {
      const list = el(s.kind === "ordered" ? "ol" : "ul");
      s.items.forEach(it => {
        const li = el("li");
        if (typeof it === "string") { li.textContent = it; addBox(li, it, s.title); }
        else {
          li.appendChild(document.createTextNode(it.text));
          const sub = el("ul", "sub");
          it.sub.forEach(t => { const sl = el("li", null, t); sub.appendChild(sl); addBox(sl, t, s.title); });
          li.appendChild(sub);
        }
        list.appendChild(li);
      });
      sec.appendChild(list);
    }
    app.appendChild(sec);
  });

  const note = el("p", "note");
  note.append("Wording taken from the ");
  if (P.sourceUrl) { const a = el("a", null, "official posting"); a.href = P.sourceUrl; note.appendChild(a); } else note.append("posting");
  note.append(", retrieved " + P.retrieved + "." + (P.sourceNote ? " " + P.sourceNote : ""));
  app.appendChild(note);

  exp.addEventListener("click", () => {
    const out = [P.title + " (" + P.company + (P.reqId ? ", Req. " + P.reqId : "") + ")", "Notes exported " + new Date().toLocaleDateString()];
    let last = "";
    boxes.forEach(b => {
      if (b.section !== last) { out.push("", b.section.toUpperCase()); last = b.section; }
      out.push("- " + b.label, "  Notes: " + (b.ta.value.trim() ? b.ta.value.trim().replace(/\n/g, "\n         ") : "—"));
    });
    const a = el("a"); a.href = URL.createObjectURL(new Blob([out.join("\n")], { type: "text/plain" }));
    a.download = (P.company + "_" + P.title).toLowerCase().replace(/[^a-z0-9]+/g, "_").replace(/^_|_$/g, "") + "_notes.txt";
    document.body.appendChild(a); a.click(); a.remove();
  });
  savePage.addEventListener("click", () => {
    const json = JSON.stringify({ t: Date.now(), notes: store }).replace(/</g, "\\u003c");
    const html = SOURCE.replace(/^const SAVED_NOTES = .*$/m, () => "const SAVED_NOTES = " + json + ";");
    const a = el("a"); a.href = URL.createObjectURL(new Blob([html], { type: "text/html" }));
    a.download = (P.company + "_" + P.title).toLowerCase().replace(/[^a-z0-9]+/g, "_").replace(/^_|_$/g, "") + "_requirements.html";
    document.body.appendChild(a); a.click(); a.remove();
    status.textContent = "Saved a copy with your notes";
  });
  clr.addEventListener("click", () => { boxes.forEach(b => b.ta.value = ""); store = {}; save(); });
})();
</script>
</body>
</html>
```