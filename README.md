# kb — the support knowledge base

**kb** = knowledge base. What the support desk knows about how our products break, written
down where the code can keep it honest.

One desk takes reports for every product we run: `sched-prod` (shift planning, about 300k
people), `tram-3` (departure boards), `rota-6` (care-home rotas) and the rest. When something
goes wrong, the reports reach the desk first. This kb is how the desk triages them, what each
incident turned out to be, and why we decided what we decided.

## What's here

```
src/routes/kb/      the pages (.svx: Markdown with components)
  triage/           how the desk handles incoming reports
  incidents/        one page per big ticket
  decisions/        numbered decision records
src/lib/kb/         components and the logic they run
src/lib/layouts/    the frame every page shares
src/lib/data/       exports from the ticket system
```

## Conventions

- Words in the page, logic in `src/lib` (imported as `#lib/…`). A number that can be computed belongs in a component.
- Every page starts with frontmatter: `title`, `owner`, `status`, `reviewed`.
- A decision is never edited after it is accepted. A new record replaces it.
- A page you add is signed: its frontmatter carries your `signature` and it ends with `<RenderStamp {signature} />`, so the desk knows who wrote it.

## Owners

Support desk (Tove). Decision records: whoever signed them.

---

Exported from the `support` repo, `apps/kb/` → `src/`.

---

## In this repo

- The kb export above, unchanged, with a static build around it (SvelteKit + mdsvex).
  `npm install && npm run build` writes every page to `build/` as HTML.
- A new page about #4471: [`src/routes/kb/incidents/4471-issues/+page.svx`](src/routes/kb/incidents/4471-issues/+page.svx),
  built to [`build/kb/incidents/4471-issues/index.html`](build/kb/incidents/4471-issues/index.html)
  (picture: [`docs/4471-issues.png`](docs/4471-issues.png)). Its component is `src/lib/kb/IssueTickets.svelte`;
  the ticket list it reads is `src/lib/data/tickets-4471.json`.
- One ticket per issue found in #4471, in this repo's
  [issues](https://github.com/iceener/support-kb/issues?q=is%3Aissue): #1 `sched-prod` (closed, change #29),
  #2 `tram-3` (open), #3 `rota-6` (open).
