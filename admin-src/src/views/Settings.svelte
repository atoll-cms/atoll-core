<script>
  import { settings, themes, addToast, confirm } from '../lib/stores.js';
  import { api } from '../lib/api.js';

  let siteName = $state('');
  let baseUrl = $state('');
  let updaterChannel = $state('stable');
  let updaterManifestUrl = $state('');
  let selectedTheme = $state('default');
  let saving = $state(false);

  $effect(() => {
    siteName = $settings?.name || '';
    baseUrl = $settings?.base_url || '';
    updaterChannel = $settings?.updater?.channel || 'stable';
    updaterManifestUrl = $settings?.updater?.manifest_url || '';
    selectedTheme = $settings?.appearance?.theme || 'default';
  });

  async function saveSettings(event) {
    event.preventDefault();
    saving = true;

    try {
      await api('/admin/api/settings/save', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          settings: {
            name: siteName,
            base_url: baseUrl,
            updater: {
              ...($settings.updater || {}),
              channel: updaterChannel,
              manifest_url: updaterManifestUrl
            },
            appearance: {
              ...($settings.appearance || {}),
              theme: selectedTheme
            },
            smtp: $settings.smtp || {},
            security: $settings.security || {}
          }
        })
      });

      const t = await api('/admin/api/themes');
      themes.set(t.themes);
      addToast('Settings gespeichert.', 'success');
    } catch (err) {
      addToast(err.message, 'error');
    } finally {
      saving = false;
    }
  }

  async function createBackup() {
    try {
      const result = await api('/admin/api/backup/create', { method: 'POST' });
      if (result.ok) {
        addToast(`Backup erstellt: ${result.file}`, 'success');
      } else {
        addToast(result.error || 'Backup fehlgeschlagen.', 'error');
      }
    } catch (err) {
      addToast(err.message, 'error');
    }
  }

  async function clearCache() {
    const confirmed = await confirm('Cache leeren', 'Soll der komplette Cache geleert werden?');
    if (!confirmed) return;

    try {
      await api('/admin/api/cache/clear', { method: 'POST' });
      addToast('Cache geleert.', 'success');
    } catch (err) {
      addToast(err.message, 'error');
    }
  }
</script>

<div class="settings-view">
  <div class="page-header">
    <h1>Settings</h1>
  </div>

  <form class="section-card" onsubmit={saveSettings}>
    <div class="section-header">
      <h3>Allgemein</h3>
    </div>
    <div class="form-body">
      <div class="field">
        <label for="site-name">Site Name</label>
        <input id="site-name" bind:value={siteName}>
      </div>
      <div class="field">
        <label for="base-url">Base URL</label>
        <input id="base-url" bind:value={baseUrl} placeholder="https://example.com">
      </div>
      <div class="field-row">
        <div class="field">
          <label for="updater-channel">Update Channel</label>
          <input id="updater-channel" bind:value={updaterChannel}>
        </div>
        <div class="field">
          <label for="updater-url">Manifest URL</label>
          <input id="updater-url" bind:value={updaterManifestUrl} placeholder="https://...">
        </div>
      </div>
      <div class="field">
        <label for="theme-select">Theme</label>
        <select id="theme-select" bind:value={selectedTheme}>
          {#each $themes as t}
            <option value={t.id}>{t.id}</option>
          {/each}
        </select>
      </div>
      <button type="submit" class="save-btn" disabled={saving}>
        {saving ? 'Speichert...' : 'Speichern'}
      </button>
    </div>
  </form>

  <div class="actions-grid">
    <button class="action-card" onclick={createBackup}>
      <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>
      <span>Backup erstellen</span>
    </button>
    <button class="action-card" onclick={clearCache}>
      <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 4 23 10 17 10"/><polyline points="1 20 1 14 7 14"/><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"/></svg>
      <span>Cache leeren</span>
    </button>
  </div>
</div>

<style>
  .settings-view { max-width: 700px; }

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

  .section-header h3 { margin: 0; font-size: 0.95rem; font-weight: 600; }

  .form-body { padding: 1.25rem; }

  .field {
    margin-bottom: 1rem;
  }

  .field label {
    display: block;
    font-size: 0.8rem;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.04em;
    color: var(--muted);
    margin-bottom: 0.35rem;
  }

  .field input,
  .field select {
    width: 100%;
    padding: 0.6rem 0.75rem;
    background: var(--bg);
    border: 1px solid var(--line);
    border-radius: 8px;
    color: var(--text);
    font: inherit;
    font-size: 0.9rem;
    transition: border-color 0.15s;
  }

  .field input:focus,
  .field select:focus {
    outline: none;
    border-color: var(--brand);
  }

  .field-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 0.75rem;
  }

  .save-btn {
    padding: 0.6rem 1.5rem;
    background: var(--brand);
    border: none;
    border-radius: 8px;
    color: #1a1a1a;
    font: inherit;
    font-size: 0.9rem;
    font-weight: 600;
    cursor: pointer;
    transition: opacity 0.15s;
  }

  .save-btn:hover:not(:disabled) { opacity: 0.9; }
  .save-btn:disabled { opacity: 0.5; cursor: not-allowed; }

  .actions-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 0.75rem;
  }

  .action-card {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 0.75rem;
    padding: 1.5rem;
    background: var(--surface);
    border: 1px solid var(--line);
    border-radius: 14px;
    color: var(--muted);
    font: inherit;
    font-size: 0.9rem;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s;
  }

  .action-card:hover {
    border-color: var(--brand);
    color: var(--text);
    transform: translateY(-1px);
  }

  @media (max-width: 600px) {
    .field-row { grid-template-columns: 1fr; }
    .actions-grid { grid-template-columns: 1fr; }
  }
</style>
