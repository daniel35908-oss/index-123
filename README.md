<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Anime Character Studio</title>
    <style>
        body { font-family: 'Arial', sans-serif; background: #121212; color: white; text-align: center; margin: 0; }
        .container { padding: 20px; }
        
        /* The Canvas / Stage */
        #stage {
            width: 800px;
            height: 450px;
            margin: 20px auto;
            border: 4px solid #6c5ce7;
            border-radius: 10px;
            position: relative;
            overflow: hidden;
            background: #2d3436;
            background-size: cover;
            background-position: center;
        }

        #character {
            position: absolute;
            cursor: move;
            width: 150px;
            user-select: none;
        }

        .controls {
            background: #1e1e1e;
            padding: 20px;
            display: inline-block;
            border-radius: 10px;
            margin-bottom: 20px;
        }

        input, button { padding: 10px; margin: 5px; border-radius: 5px; border: none; font-weight: bold; }
        button { cursor: pointer; transition: 0.3s; }
        .btn-record { background: #ff4757; color: white; }
        .btn-stop { background: #2f3542; color: white; }
        .btn-upload { background: #00b894; color: white; }
        
        p { color: #a4b0be; font-size: 0.9em; }
    </style>
</head>
<body>

<div class="container">
    <h1>Anime Character Studio</h1>
    <p>Upload a background, upload a character, drag them into place, and record!</p>

    <div id="stage">
        <img src="https://via.placeholder.com/150?text=Character" id="character" draggable="false">
    </div>

    <div class="controls">
        <div>
            <label>1. Background:</label>
            <input type="file" id="bg-upload" accept="image/*" class="btn-upload">
        </div>
        <div>
            <label>2. Character:</label>
            <input type="file" id="char-upload" accept="image/*" class="btn-upload">
        </div>
        <hr style="border: 0.5px solid #444">
        <button id="start-btn" class="btn-record">Start Recording</button>
        <button id="stop-btn" class="btn-stop" disabled>Stop & Download</button>
    </div>
</div>

<script>
    const stage = document.getElementById('stage');
    const character = document.getElementById('character');
    const bgUpload = document.getElementById('bg-upload');
    const charUpload = document.getElementById('char-upload');
    const startBtn = document.getElementById('start-btn');
    const stopBtn = document.getElementById('stop-btn');

    // --- 1. Background Upload ---
    bgUpload.addEventListener('change', function(e) {
        const reader = new FileReader();
        reader.onload = function() {
            stage.style.backgroundImage = `url(${reader.result})`;
        }
        reader.readAsDataURL(e.target.files[0]);
    });

    // --- 2. Character Upload ---
    charUpload.addEventListener('change', function(e) {
        const reader = new FileReader();
        reader.onload = function() {
            character.src = reader.result;
        }
        reader.readAsDataURL(e.target.files[0]);
    });

    // --- 3. Drag and Drop Logic ---
    let isDragging = false;
    let offsetX, offsetY;

    character.onmousedown = (e) => {
        isDragging = true;
        offsetX = e.clientX - character.offsetLeft;
        offsetY = e.clientY - character.offsetTop;
    };

    document.onmousemove = (e) => {
        if (!isDragging) return;
        character.style.left = (e.clientX - offsetX) + 'px';
        character.style.top = (e.clientY - offsetY) + 'px';
    };

    document.onmouseup = () => isDragging = false;

    // --- 4. Recording Logic ---
    let mediaRecorder;
    let chunks = [];

    startBtn.onclick = async () => {
        // Captures the screen
        const stream = await navigator.mediaDevices.getDisplayMedia({ video: true });
        mediaRecorder = new MediaRecorder(stream);
        
        mediaRecorder.ondataavailable = (e) => chunks.push(e.data);
        mediaRecorder.onstop = () => {
            const blob = new Blob(chunks, { type: 'video/webm' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'my-anime-video.webm';
            a.click();
            chunks = [];
        };

        mediaRecorder.start();
        startBtn.disabled = true;
        stopBtn.disabled = false;
    };

    stopBtn.onclick = () => {
        mediaRecorder.stop();
        startBtn.disabled = false;
        stopBtn.disabled = true;
    };
</script>

</body>
</html>
