<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // Three problems in #4471: the day/month swap by the parser, the swap from the
  // booking-v3 hint, and a one-day move (time zone) that is no swap at all.
  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const oneDay = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rest = merged; // counted where filed by tenant, as the desk's view does
  const rows = $derived([
    { problem: 'parser reads day-first dates month first', n: rest.filter((r) => causeOf(r, tenantOf(r)) === 'parser').length },
    { problem: 'booking-v3 form hint says MM/DD to day-first tenants', n: rest.filter((r) => causeOf(r, tenantOf(r)) === 'hint').length },
    { problem: 'of these, read as a one-day move (time zone)', n: oneDay.length },
  ]);
</script>

<p><strong>{merged.length}</strong> reports in #{ticket} · <strong>{rows.length}</strong> problems</p>
<table>
  <thead><tr><th>problem</th><th>reports</th></tr></thead>
  <tbody>
    {#each rows as r (r.problem)}<tr><td>{r.problem}</td><td>{r.n}</td></tr>{/each}
  </tbody>
</table>
