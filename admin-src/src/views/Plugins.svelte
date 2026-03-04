<script>
  import { plugins, pluginRegistry, addToast, confirm } from '../lib/stores.js';
  import { api } from '../lib/api.js';

  let installSource = $state('');
  let installEnable = $state(true);

  async function togglePlugin(id, active) {
    try {
      await api('/admin/api/plugins/toggle', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ id, active })
      });
      const data = await api('/admin/api/plugins');
      plugins.set(data.plugins);
      addToast(`Plugin ${active ? 'aktiviert' : 'deaktiviert'}.`, 'success');
    } catch (err) {
      addToast(err.message, 'error');
    }
  }

  async function installFromRegistry(id) {
    try {
      await api('/admin/api/plugins/install', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ id, enable: true })
      });
      const data = await api('/admin/api/plugins');
      plugins.set(data.plugins);
      addToast('Plugin installiert.', 'success');
    } catch (err) {
      addToast(err.message, 'error');
    }
  }

  async function installFromSource(event) {
    event.preventDefault();
    try {
      await api('/admin/api/plugins/install', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ source: installSource, enable: installEnable })
      });
      const data = await api('/admin/api/plugins');
      plugins.set(data.plugins);
      installSource = '';
      addToast('Plugin installiert.', 'success');
    } catch (err) {
      addToast(err.message, 'error');
    }
  }
</script>

<div class="plugins-view">
  <div class="page-header">
    <h1>Plugins</h1>
  </div>

  <div class="section-card">
    <div class="section-header">
      <h3>Installierte Plugins</h3>
    </div>
    {#if $plugins.length > 0}
      <div class="table-wrap">
        <table>
          <thead>
            <tr><th>Name</th><th>Version</th><th>Status</th><th>Aktion</th></tr>
          </thead>
          <tbody>
            {#each $plugins as p}
              <tr>
                <td>
                  <div class="plugin-name">{p.name}</div>
                  {#if p.description}<div class="plugin-desc">{p.description}</div>{/if}
                </td>
                <td><code>{p.version}</code></td>
                <td>
                  <span class="badge" class:badge--active={p.active} class:badge--inactive={!p.active}>
                    {p.active ? 'Aktiv' : 'Inaktiv'}
                  </span>
                </td>
                <td>
                  <button class="action-btn" onclick={() => togglePlugin(p.id, !p.active)}>
                    {p.active ? 'Deaktivieren' : 'Aktivieren'}
                  </button>
                </td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
    {:else}
      <div class="empty-msg">Keine Plugins installiert.</div>
    {/if}
  </div>

  {#if $pluginRegistry.length > 0}
    <div class="section-card">
      <div class="section-header">
        <h3>Registry</h3>
      </div>
      <div class="registry-grid">
        {#each $pluginRegistry as p}
          <div class="registry-card">
            <strong>{p.name}</strong>
            {#if p.description}<p class="registry-desc">{p.description}</p>{/if}
            <button class="install-btn" onclick={() => installFromRegistry(p.id)}>Installieren</button>
          </div>
        {/each}
      </div>
    </div>
  {/if}

  <div class="section-card">
    <div class="section-header">
      <h3>Von Source installieren</h3>
    </div>
    <form class="install-form" onsubmit={installFromSource}>
      <input bind:value={installSource} placeholder="/pfad/zu/plugin oder https://...zip" required>
      <label class="checkbox-label">
        <input type="checkbox" bind:checked={installEnable}> Aktivieren
      </label>
      <button type="submit" class="submit-btn">Installieren</button>
    </form>
  </div>
</div>

<style>
  .plugins-view { max-width: 900px; }

  .page-header h1 {
    font-size: 1.5rem;
    font-weight: 700;
    margin: 0 0 1.5rem;
    letter-spacing: -0.02em;
  }

  .section-card {
    background: var(--surface);
    border: 1px solid var(--line);
    border-radius: 14px;
    overflow: hidden;
    margin-bottom: 1rem;
  }

  .section-header {
    padding: 1rem 1.25rem;
    border-bottom: 1px solid var(--line);
  }

  .section-header h3 {
    margin: 0;
    font-size: 0.95rem;
    font-weight: 600;
  }

  .table-wrap { overflow-x: auto; }

  table {
    width: 100%;
    border-collapse: collapse;
  }

  th, td {
    padding: 0.75rem 1.25rem;
    text-align: left;
    font-size: 0.9rem;
  }

  th {
    font-size: 0.75rem;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    color: var(--muted);
    border-bottom: 1px solid var(--line);
  }

  td {
    border-bottom: 1px solid var(--line);
  }

  tr:last-child td { border-bottom: none; }

  .plugin-name { font-weight: 500; }
  .plugin-desc { font-size: 0.8rem; color: var(--muted); margin-top: 0.15rem; }

  code {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.8rem;
    padding: 0.1rem 0.4rem;
    background: var(--bg);
    border-radius: 4px;
  }

  .badge {
    display: inline-block;
    padding: 0.15rem 0.55rem;
    border-radius: 999px;
    font-size: 0.75rem;
    font-weight: 600;
  }

  .badge--active { background: rgba(34, 197, 94, 0.1); color: #22c55e; border: 1px solid rgba(34, 197, 94, 0.25); }
  .badge--inactive { background: rgba(107, 114, 128, 0.1); color: #9ca3af; border: 1px solid rgba(107, 114, 128, 0.25); }

  .action-btn {
    padding: 0.35rem 0.75rem;
    background: transparent;
    border: 1px solid var(--line);
    border-radius: 6px;
    color: var(--text);
    font: inherit;
    font-size: 0.8rem;
    cursor: pointer;
    transition: all 0.15s;
  }

  .action-btn:hover {
    border-color: var(--brand);
    color: var(--brand);
  }

  .registry-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 0.75rem;
    padding: 1rem;
  }

  .registry-card {
    padding: 1rem;
    background: var(--bg);
    border: 1px solid var(--line);
    border-radius: 10px;
  }

  .registry-desc { font-size: 0.8rem; color: var(--muted); margin: 0.25rem 0 0.75rem; }

  .install-btn {
    padding: 0.35rem 0.75rem;
    background: var(--brand);
    border: none;
    border-radius: 6px;
    color: #1a1a1a;
    font: inherit;
    font-size: 0.8rem;
    font-weight: 600;
    cursor: pointer;
  }

  .install-form {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    padding: 1rem 1.25rem;
  }

  .install-form input[type="text"],
  .install-form input:not([type]) {
    flex: 1;
    padding: 0.5rem 0.75rem;
    background: var(--bg);
    border: 1px solid var(--line);
    border-radius: 8px;
    color: var(--text);
    font: inherit;
    font-size: 0.9rem;
  }

  .checkbox-label {
    display: flex;
    align-items: center;
    gap: 0.35rem;
    font-size: 0.85rem;
    white-space: nowrap;
    cursor: pointer;
  }

  .checkbox-label input { width: auto; margin: 0; }

  .submit-btn {
    padding: 0.5rem 1rem;
    background: var(--brand);
    border: none;
    border-radius: 8px;
    color: #1a1a1a;
    font: inherit;
    font-size: 0.85rem;
    font-weight: 600;
    cursor: pointer;
    white-space: nowrap;
  }

  .empty-msg {
    padding: 2rem;
    text-align: center;
    color: var(--muted);
    font-size: 0.9rem;
  }
</style>
