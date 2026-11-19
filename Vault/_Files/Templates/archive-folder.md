<%*
// Detecta a pasta onde o arquivo temporário foi criado
const currentFolder = tp.config.target_file.parent;
const folderPath = currentFolder.path;

// Se estiver na pasta raiz ou em .temp, pede o caminho manualmente
if (folderPath === "/" || folderPath === "" || folderPath === "." || folderPath.includes(".temp")) {
    const manualPath = await tp.system.prompt("Insert folder path:");
    
    if (!manualPath) {
        new Notice("Canceled");
        await app.vault.delete(tp.config.target_file);
        return;
    }
    
    const folder = app.vault.getAbstractFileByPath(manualPath);
    
    if (!folder || !folder.children) {
        new Notice("Folder not found");
        await app.vault.delete(tp.config.target_file);
        return;
    }
    
    await archiveFolder(folder, manualPath);
} else {
    // Confirma se quer arquivar a pasta detectada
    const confirm = await tp.system.prompt(`Archive folder "${currentFolder.name}"? (type 'y' to confirm)`);
    
    if (confirm?.toLowerCase() !== 'y') {
        new Notice("Canceled");
        await app.vault.delete(tp.config.target_file);
        return;
    }
    
    await archiveFolder(currentFolder, folderPath);
}

async function archiveFolder(folder, folderPath) {
    // Adiciona metadados em arquivo especial
    const metadataContent = `---
archived_from: "${folderPath}"
archived_date: ${tp.date.now("YYYY-MM-DD HH:mm")}
folder_name: "${folder.name}"
---

# Archived Folder: ${folder.name}

**Origin:** \`${folderPath}\`
**Date:** ${tp.date.now("YYYY-MM-DD HH:mm")}

Use the template "restore-folder" to restore.
`;

    const metadataPath = `Archive/Archived Folders/${folder.name}/_metadata.md`;
    const newPath = `Archive/Archived Folders/${folder.name}`;
    
    await app.vault.rename(folder, newPath);
    await app.vault.create(metadataPath, metadataContent);
    
    new Notice(`Folder archived: ${folder.name}`);
}

// Deleta a nota temporária
await app.vault.delete(tp.config.target_file);
_%>