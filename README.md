# Mini-project
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mini Git Repository Simulator</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f7f6;
            color: #333;
            max-width: 900px;
            margin: 20px auto;
            padding: 20px;
        }
        h1 { text-align: center; color: #2c3e50; }
        .container {
            display: flex;
            gap: 20px;
            margin-top: 20px;
        }
        .box {
            flex: 1;
            background: #fff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        .box h2 {
            margin-top: 0;
            border-bottom: 2px solid #eee;
            padding-bottom: 10px;
            font-size: 1.2rem;
        }
        .controls {
            background: #fff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            align-items: center;
        }
        input[type="text"] {
            flex: 1;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            min-width: 200px;
        }
        button {
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            color: white;
            transition: background 0.2s;
        }
        .btn-init { background-color: #34495e; }
        .btn-init:hover { background-color: #2c3e50; }
        
        .btn-commit { background-color: #27ae60; }
        .btn-commit:hover { background-color: #219150; }
        
        .btn-push { background-color: #2980b9; }
        .btn-push:hover { background-color: #2471a3; }
        
        .btn-pull { background-color: #e67e22; }
        .btn-pull:hover { background-color: #d35400; }

        .btn-remote-add { background-color: #8e44ad; font-size: 0.85rem; padding: 6px 10px; margin-bottom: 10px;}
        .btn-remote-add:hover { background-color: #732d91; }

        button:disabled {
            background-color: #bdc3c7;
            cursor: not-allowed;
        }
        ul {
            list-style-type: none;
            padding: 0;
        }
        li {
            background: #ecf0f1;
            margin-bottom: 8px;
            padding: 10px;
            border-radius: 4px;
            border-left: 4px solid #3498db;
            font-family: monospace;
            font-size: 0.9rem;
        }
        .status-bar {
            margin-top: 15px;
            padding: 10px;
            background: #2c3e50;
            color: #ecf0f1;
            border-radius: 4px;
            font-family: monospace;
        }
    </style>
</head>
<body>

    <h1>Mini Git Repository Simulator</h1>

    <!-- Action Controls -->
    <div class="controls">
        <button class="btn-init" id="initBtn" onclick="initRepo()">1. Create Repo (Init)</button>
        <input type="text" id="commitMsg" placeholder="Enter file/commit message..." disabled>
        <button class="btn-commit" id="commitBtn" onclick="commitChange()" disabled>2. Commit</button>
        <button class="btn-push" id="pushBtn" onclick="pushChanges()" disabled>3. Push &uarr;</button>
        <button class="btn-pull" id="pullBtn" onclick="pullChanges()" disabled>4. Pull &darr;</button>
    </div>

    <!-- Terminal Status Output -->
    <div class="status-bar" id="statusLog">> Status: Ready. Click "Create Repo (Init)" to start.</div>

    <!-- Local vs Remote View -->
    <div class="container">
        <!-- Local Repo -->
        <div class="box">
            <h2>💻 Local Repository (Your PC)</h2>
            <ul id="localList"></ul>
        </div>

        <!-- Remote Repo -->
        <div class="box">
            <h2>☁️ Remote Repository (GitHub/Server)</h2>
            <button class="btn-remote-add" id="simulateRemoteBtn" onclick="simulateTeammateCommit()" disabled>
                + Simulate Teammate Commit on Remote
            </button>
            <ul id="remoteList"></ul>
        </div>
    </div>

    <script>
        let isInitialized = false;
        let localCommits = [];
        let remoteCommits = [];
        let commitCounter = 1;

        function logStatus(message) {
            document.getElementById('statusLog').innerText = `> ${message}`;
        }

        function generateHash() {
            return Math.random().toString(16).substring(2, 9);
        }

        // 1. Create / Initialize Repository
        function initRepo() {
            isInitialized = true;
            localCommits = [];
            remoteCommits = [];
            commitCounter = 1;

            // Enable controls
            document.getElementById('commitMsg').disabled = false;
            document.getElementById('commitBtn').disabled = false;
            document.getElementById('pushBtn').disabled = false;
            document.getElementById('pullBtn').disabled = false;
            document.getElementById('simulateRemoteBtn').disabled = false;
            document.getElementById('initBtn').innerText = "Reset Repo";

            renderLists();
            logStatus("Initialized empty local and remote Git repositories.");
        }

        // 2. Commit to Local Repository
        function commitChange() {
            const input = document.getElementById('commitMsg');
            const message = input.value.trim();

            if (!message) {
                alert("Please enter a commit message first!");
                return;
            }

            const newCommit = {
                id: commitCounter++,
                hash: generateHash(),
                msg: message
            };

            localCommits.push(newCommit);
            input.value = '';
            renderLists();
            logStatus(`Committed locally: [${newCommit.hash}] "${newCommit.msg}"`);
        }

        // 3. Push Local Commits to Remote
        function pushChanges() {
            if (localCommits.length === remoteCommits.length && 
                JSON.stringify(localCommits) === JSON.stringify(remoteCommits)) {
                logStatus("Everything up-to-date. Nothing to push.");
                return;
            }

            // Copy local commits to remote
            remoteCommits = JSON.parse(JSON.stringify(localCommits));
            renderLists();
            logStatus("Push successful! Remote repository updated with local commits.");
        }

        // 4. Pull Remote Commits to Local
        function pullChanges() {
            if (localCommits.length === remoteCommits.length && 
                JSON.stringify(localCommits) === JSON.stringify(remoteCommits)) {
                logStatus("Already up-to-date. Nothing to pull.");
                return;
            }

            // Merge remote commits into local (avoiding duplicates by hash)
            const localHashes = new Set(localCommits.map(c => c.hash));
            let pulledCount = 0;

            remoteCommits.forEach(remoteCommit => {
                if (!localHashes.has(remoteCommit.hash)) {
                    localCommits.push(remoteCommit);
                    pulledCount++;
                }
            });

            renderLists();
            if (pulledCount > 0) {
                logStatus(`Pull successful! Fetched ${pulledCount} new commit(s) from remote.`);
            } else {
                logStatus("Local is ahead of remote. Use 'Push' instead.");
            }
        }

        // Helper: Simulate someone else pushing to remote so you can test 'Pull'
        function simulateTeammateCommit() {
            const teammateCommit = {
                id: commitCounter++,
                hash: generateHash(),
                msg: "Teammate's update on server"
            };
            remoteCommits.push(teammateCommit);
            renderLists();
            logStatus(`Remote updated by teammate: [${teammateCommit.hash}]. Click "Pull" to sync!`);
        }

        // Update UI Lists
        function renderLists() {
            const localList = document.getElementById('localList');
            const remoteList = document.getElementById('remoteList');

            localList.innerHTML = localCommits.length === 0 
                ? "<li style='border-left-color:#95a5a6; color:#7f8c8d;'>No local commits yet</li>" 
                : localCommits.map(c => `<li><strong>#${c.id} [${c.hash}]</strong>: ${c.msg}</li>`).join('');

            remoteList.innerHTML = remoteCommits.length === 0 
                ? "<li style='border-left-color:#95a5a6; color:#7f8c8d;'>No remote commits yet</li>" 
                : remoteCommits.map(c => `<li><strong>#${c.id} [${c.hash}]</strong>: ${c.msg}</li>`).join('');
        }
    </script>

</body>
</html>
