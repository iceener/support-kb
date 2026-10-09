<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import tickets from '#lib/data/tickets-4471.json';
  import { clusterReports, reporterKey, uniqueById, type Report } from '#lib/kb/fingerprint';
  import { placeOf } from '#lib/kb/places';

  // One row per issue found in a merged ticket (same rule as ClusterTable), with the ticket each
  // issue has now. The ticket list is data (tickets-<n>.json): fingerprint → its own ticket.
  type Ticket = { number: number; url: string; title: string; state: 'open' | 'closed'; note?: string };

  let { ticket }: { ticket: number } = $props();

  const own = tickets as Record<string, Ticket>;
  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const clusters = $derived(clusterReports(merged, placeOf));
  const ticketed = $derived(clusters.filter((c) => own[c.fingerprint]).length);
  const open = $derived(clusters.filter((c) => own[c.fingerprint]?.state === 'open').length);
  // Rows nobody wrote: no address in `reporter` (the 03:12 job left one). Counted, but not people.
  const unsigned = $derived(merged.filter((r) => !r.reporter.includes('@')));
  const people = (rs: Report[]) => new Set(rs.filter((r) => r.reporter.includes('@')).map((r) => reporterKey(r.reporter))).size;
  const time = (iso: string) => iso.slice(11, 16);
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports in #{ticket} · <strong>{clusters.length}</strong>
  {clusters.length === 1 ? 'issue' : 'issues'} · <strong>{ticketed}</strong> with a ticket of their own ·
  <strong>{open}</strong> still open
</p>

<table>
  <thead>
    <tr><th>place</th><th>code</th><th>reports</th><th>people</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each clusters as c (c.fingerprint)}
      {@const t = own[c.fingerprint]}
      <tr class:missing={!t}>
        <td>{c.place}</td>
        <td><code>{c.code}</code></td>
        <td>{c.reports.length}</td>
        <td>{people(c.reports)}</td>
        <td>{time(c.firstSeen)} → {time(c.lastSeen)}</td>
        <td>
          {#if t}
            <a href={t.url}>#{t.number}</a> <span class="state {t.state}">{t.state}</span>
            {#if t.note}<span class="note">{t.note}</span>{/if}
          {:else}
            no ticket of its own
          {/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

{#if unsigned.length}
  <p class="foot">
    Counted in reports, not in people:
    {#each unsigned as r, i (r.id)}{i ? ', ' : ''}<code>{r.id}</code> at {time(r.receivedAt)} (<code>{r.body}</code>){/each}.
  </p>
{/if}

<style>
  .sum { font-size: 1.1rem; }
  .state { border: 1px solid var(--line); border-radius: 99px; padding: 0 0.5rem; font-size: 0.8em; }
  .state.open { color: var(--warn); border-color: var(--warn); }
  .note { display: block; color: var(--muted); font-size: 0.8em; }
  .missing td { background: var(--warn-bg); }
  .missing td:last-child { color: var(--warn); font-weight: 600; }
  .foot { color: var(--muted); font-size: 0.85rem; }
</style>
