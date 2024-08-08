# Notiz-Manager index.html
Notiz-Manager/
├── index.html
├── style.css
└── script.js
<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Notiz-Manager</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f0f0f0;
            display: flex;
            height: 100vh;
        }
        .sidebar {
            width: 250px;
            background-color: #333;
            color: white;
            padding: 20px;
            display: flex;
            flex-direction: column;
        }
        .main-content {
            flex-grow: 1;
            padding: 20px;
            display: flex;
            flex-direction: column;
        }
        h1 {
            margin-top: 0;
        }
        button {
            background-color: #4CAF50;
            border: none;
            color: white;
            padding: 10px 20px;
            text-align: center;
            text-decoration: none;
            display: inline-block;
            font-size: 16px;
            margin: 4px 2px;
            cursor: pointer;
            border-radius: 4px;
        }
        input[type="text"] {
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            box-sizing: border-box;
        }
        textarea {
            width: 100%;
            height: 200px;
            padding: 10px;
            margin: 10px 0;
            box-sizing: border-box;
            resize: vertical;
        }
        .folder-list {
            list-style-type: none;
            padding: 0;
            margin-top: 20px;
        }
        .folder-list li {
            cursor: pointer;
            padding: 10px;
            background-color: #444;
            margin-bottom: 5px;
            border-radius: 4px;
        }
        .folder-list li:hover {
            background-color: #555;
        }
        .note-list {
            list-style-type: none;
            padding: 0;
            margin-top: 20px;
            flex-grow: 1;
            overflow-y: auto;
        }
        .note-list li {
            cursor: pointer;
            padding: 10px;
            background-color: white;
            margin-bottom: 5px;
            border-radius: 4px;
        }
        .note-list li:hover {
            background-color: #f0f0f0;
        }
        .dark-mode {
            background-color: #222;
            color: #fff;
        }
        .dark-mode .sidebar {
            background-color: #111;
        }
        .dark-mode .folder-list li {
            background-color: #333;
        }
        .dark-mode .folder-list li:hover {
            background-color: #444;
        }
        .dark-mode .note-list li {
            background-color: #444;
            color: #fff;
        }
        .dark-mode .note-list li:hover {
            background-color: #555;
        }
        .dark-mode input[type="text"],
        .dark-mode textarea {
            background-color: #333;
            color: #fff;
            border: 1px solid #555;
        }
        .dark-mode button {
            background-color: #555;
        }
    </style>
</head>
<body>
    <div class="sidebar">
        <h1>Notiz-Manager</h1>
        <button id="newFolderBtn">Neuer Ordner</button>
        <button id="newNoteBtn">Neue Notiz</button>
        <button id="darkModeBtn">Dunkelmodus</button>
        <ul class="folder-list" id="folderList"></ul>
    </div>
    <div class="main-content">
        <input type="text" id="noteTitle" placeholder="Notiz-Titel" required>
        <textarea id="noteContent" placeholder="Notiz-Inhalt" required></textarea>
        <button id="saveNoteBtn">Notiz speichern</button>
        <ul class="note-list" id="noteList"></ul>
    </div>

    <script>
        let folders = [];
        let notes = [];
        let currentFolder = null;
        let currentNote = null;

        const folderList = document.getElementById('folderList');
        const noteList = document.getElementById('noteList');
        const noteTitle = document.getElementById('noteTitle');
        const noteContent = document.getElementById('noteContent');
        const newFolderBtn = document.getElementById('newFolderBtn');
        const newNoteBtn = document.getElementById('newNoteBtn');
        const saveNoteBtn = document.getElementById('saveNoteBtn');
        const darkModeBtn = document.getElementById('darkModeBtn');

        function createFolder(name) {
            const folder = { id: Date.now(), name: name };
            folders.push(folder);
            renderFolders();
            saveFolders();
        }

        function createNote(title, content) {
            if (!currentFolder) return;
            const note = { id: Date.now(), folderId: currentFolder.id, title: title, content: content };
            notes.push(note);
            renderNotes();
            saveNotes();
        }

        function renderFolders() {
            folderList.innerHTML = '';
            folders.forEach(folder => {
                const li = document.createElement('li');
                li.textContent = folder.name;
                li.onclick = () => selectFolder(folder);
                folderList.appendChild(li);
            });
        }

        function renderNotes() {
            noteList.innerHTML = '';
            if (!currentFolder) return;
            const folderNotes = notes.filter(note => note.folderId === currentFolder.id);
            folderNotes.forEach(note => {
                const li = document.createElement('li');
                li.textContent = note.title;
                li.onclick = () => selectNote(note);
                noteList.appendChild(li);
            });
        }

        function selectFolder(folder) {
            currentFolder = folder;
            currentNote = null;
            renderNotes();
            clearNoteForm();
        }

        function selectNote(note) {
            currentNote = note;
            noteTitle.value = note.title;
            noteContent.value = note.content;
        }

        function clearNoteForm() {
            noteTitle.value = '';
            noteContent.value = '';
            currentNote = null;
        }

        function saveFolders() {
            localStorage.setItem('folders', JSON.stringify(folders));
        }

        function saveNotes() {
            localStorage.setItem('notes', JSON.stringify(notes));
        }

        function loadFolders() {
            const storedFolders = localStorage.getItem('folders');
            if (storedFolders) {
                folders = JSON.parse(storedFolders);
                renderFolders();
            }
        }

        function loadNotes() {
            const storedNotes = localStorage.getItem('notes');
            if (storedNotes) {
                notes = JSON.parse(storedNotes);
            }
        }

        newFolderBtn.onclick = () => {
            const folderName = prompt('Geben Sie den Ordnernamen ein:');
            if (folderName) createFolder(folderName);
        };

        newNoteBtn.onclick = () => {
            if (!currentFolder) {
                alert('Bitte wählen Sie zuerst einen Ordner aus.');
                return;
            }
            clearNoteForm();
        };

        saveNoteBtn.onclick = () => {
            if (!currentFolder) {
                alert('Bitte wählen Sie zuerst einen Ordner aus.');
                return;
            }
            const title = noteTitle.value.trim();
            const content = noteContent.value.trim();
            if (!title || !content) {
                alert('Bitte geben Sie einen Titel und Inhalt für die Notiz ein.');
                return;
            }
            if (currentNote) {
                currentNote.title = title;
                currentNote.content = content;
            } else {
                createNote(title, content);
            }
            renderNotes();
            saveNotes();
            clearNoteForm();
        };

        darkModeBtn.onclick = () => {
            document.body.classList.toggle('dark-mode');
        };

        loadFolders();
        loadNotes();
    </script>
</body>
</html>
