<%*
const file = tp.config.target_file;
if (!file) {
    new Notice("No file selected");
    return;
}

// Lê metadados do frontmatter
const cache = app.metadataCache.getFileCache(file);
const frontmatter = cache?.frontmatter;

if (!frontmatter?.archived_from) {
    new Notice("This file has no origin data");
    return;
}

const originalPath = `${frontmatter.archived_from}/${file.name}`;

// Remove metadados de arquivamento do frontmatter
let content = await app.vault.read(file);
content = content.replace(/archived_from:.*\n/, '');
content = content.replace(/archived_date:.*\n/, '');
await app.vault.modify(file, content);

// Move de volta
await app.vault.rename(file, originalPath);

new Notice(`File restored: ${frontmatter.archived_from}`);
_%>