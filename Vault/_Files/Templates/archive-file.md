<%*
const file = tp.config.target_file;
if (!file) {
    new Notice("No file selected");
    return;
}

const originalPath = file.parent.path;
const fileContent = await app.vault.read(file);

let newContent = fileContent;
if (!fileContent.startsWith('---')) {
    newContent = `---\narchived_from: "${originalPath}"\narchived_date: ${tp.date.now("YYYY-MM-DD HH:mm")}\n---\n\n${fileContent}`;
} else {
    const frontmatterEnd = fileContent.indexOf('---', 3);
    const frontmatter = fileContent.substring(0, frontmatterEnd);
    const content = fileContent.substring(frontmatterEnd);
    newContent = `${frontmatter}archived_from: "${originalPath}"\narchived_date: ${tp.date.now("YYYY-MM-DD HH:mm")}\n${content}`;
}

await app.vault.modify(file, newContent);

const newPath = `Archive/Archived Files/${file.name}`;
await app.vault.rename(file, newPath);

new Notice(`Archived: ${file.name}`);
_%>