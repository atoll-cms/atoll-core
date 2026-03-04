<script>
  import { addToast } from '../lib/stores.js';
  import { apiUpload } from '../lib/api.js';

  let uploading = $state(false);
  let dragover = $state(false);

  async function handleUpload(event) {
    event.preventDefault();
    const formData = new FormData(event.target);
    uploading = true;

    try {
      const result = await apiUpload('/admin/api/media/upload', formData);
      if (result.ok) {
        addToast(`Upload erfolgreich: ${result.file}`, 'success');
        event.target.reset();
      } else {
        addToast(result.error || 'Upload fehlgeschlagen.', 'error');
      }
    } catch {
      addToast('Upload fehlgeschlagen.', 'error');
    } finally {
      uploading = false;
    }
  }

  function handleDragOver(e) {
    e.preventDefault();
    dragover = true;
  }

  function handleDragLeave() {
    dragover = false;
  }

  function handleDrop(e) {
    e.preventDefault();
    dragover = false;
    const input = document.querySelector('.file-input');
    if (input && e.dataTransfer.files.length) {
      input.files = e.dataTransfer.files;
    }
  }
</script>

<div class="media-view">
  <div class="page-header">
    <h1>Media</h1>
  </div>

  <form onsubmit={handleUpload} class="upload-card">
    <div
      class="drop-zone"
      class:dragover
      ondragover={handleDragOver}
      ondragleave={handleDragLeave}
      ondrop={handleDrop}
      role="presentation"
    >
      <svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
      <p>Datei hierher ziehen oder klicken zum Auswaehlen</p>
      <input class="file-input" name="file" type="file" required>
    </div>
    <button type="submit" class="upload-btn" disabled={uploading}>
      {uploading ? 'Wird hochgeladen...' : 'Upload'}
    </button>
  </form>
</div>

<style>
  .media-view {
    max-width: 600px;
  }

  .page-header h1 {
    font-size: 1.5rem;
    font-weight: 700;
    margin: 0 0 1.5rem;
    letter-spacing: -0.02em;
  }

  .upload-card {
    background: var(--surface);
    border: 1px solid var(--line);
    border-radius: 14px;
    padding: 1.5rem;
  }

  .drop-zone {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 0.75rem;
    padding: 3rem 2rem;
    border: 2px dashed var(--line);
    border-radius: 12px;
    text-align: center;
    color: var(--muted);
    cursor: pointer;
    transition: all 0.2s;
    position: relative;
    margin-bottom: 1rem;
  }

  .drop-zone:hover,
  .drop-zone.dragover {
    border-color: var(--brand);
    background: rgba(245, 158, 11, 0.04);
    color: var(--text);
  }

  .drop-zone p {
    font-size: 0.9rem;
    margin: 0;
  }

  .file-input {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    opacity: 0;
    cursor: pointer;
  }

  .upload-btn {
    width: 100%;
    padding: 0.65rem;
    background: var(--brand);
    border: none;
    border-radius: 8px;
    color: #1a1a1a;
    font: inherit;
    font-weight: 600;
    cursor: pointer;
    transition: opacity 0.15s;
  }

  .upload-btn:hover:not(:disabled) {
    opacity: 0.9;
  }

  .upload-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
</style>
