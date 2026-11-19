<%*
const archivedFolders = app.vault.getAbstractFileByPath("Archive/Archived Folders");

if (!archivedFolders || !archivedFolders.children) {
    new Notice("Folder not found");
    return;
}

// Lista pastas arquivadas
const folders = archivedFolders.children.filter(f => f.children !== undefined);
const folderNames = folders.map(f => f.name);

if (folderNames.length === 0) {
    new Notice("No archived folder found");
    return;
}

const selected = await tp.system.suggester(folderNames, folders);

if (!selected) {
    new Notice("Canceled");
    return;
}

// Lê metadados
const metadataFile = app.vault.getAbstractFileByPath(`${selected.path}/_metadata.md`);

if (!metadataFile) {
    new Notice("Metadata not found");
    return;
}

const cache = app.metadataCache.getFileCache(metadataFile);
const originalPath = cache?.frontmatter?.archived_from;

if (!originalPath) {
    new Notice("Original path not found");
    return;
}

// Remove arquivo de metadados
await app.vault.delete(metadataFile);

// Move pasta de volta
await app.vault.rename(selected, originalPath);

new Notice(`Folder restored: ${originalPath}`);

// Deleta a nota temporária usada para executar o template
await app.vault.delete(tp.config.target_file);
_%>