# Videosviz
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Private P2P Call</title>
    <script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"></script>
    <style>
        :root { --bg: #0f0f0f; --accent: #2488ff; --text: #ffffff; }
        body { 
            margin: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; 
            background: var(--bg); color: var(--text); overflow: hidden;
            display: flex; flex-direction: column; height: 100vh;
        }
        
        /* Видео сетка */
        .video-container { 
            flex-grow: 1; position: relative; display: flex; flex-direction: column; 
            justify-content: center; align-items: center; gap: 10px; padding: 10px;
        }
        video { 
            width: 100%; height: auto; border-radius: 12px; 
            background: #222; object-fit: cover; box-shadow: 0 4px 15px rgba(0,0,0,0.5);
        }
        #local-video { 
            position: absolute; top: 20px; right: 20px; width: 30%; 
            max-width: 120px; border: 2px solid var(--accent); z-index: 10; 
        }
        #remote-video { width: 100%; height: 100%; max-height: 80vh; }

        /* Панель управления (как в мобильных аппах) */
        .bottom-panel {
            background: rgba(30, 30, 30, 0.9); padding: 20px;
            border-top-left-radius: 20px; border-top-right-radius: 20px;
        }
        .id-badge { font-size: 0.8rem; color: #888; margin-bottom: 10px; }
        .controls { display: flex; gap: 10px; flex-direction: column; }
        input { 
            background: #333; border: none; padding: 12px; border-radius: 8px; 
            color: white; font-size: 16px; /* Предотвращает зум на iOS */
        }
        .btn-group { display: flex; gap: 10px; justify-content: center; }
        button { 
            flex: 1; padding: 12px; border-radius: 8px; border: none; 
            font-weight: bold; cursor: pointer; transition: 0.3s;
        }
        .btn-call { background: var(--accent); color: white; }
        .btn-screen { background: #444; color: white; }
    </style>
</head>
<body>

    <div class="video-container">
        <video id="local-video" autoplay muted playsinline></video>
        <video id="remote-video" autoplay playsinline></video>
    </div>

    <div class="bottom-panel">
        <div class="id-badge">Ваш ID: <span id="my-id" style="color:var(--accent)">...</span></div>
        <div class="controls">
            <input type="text" id="peer-id" placeholder="ID собеседника">
            <div class="btn-group">
                <button class="btn-call" onclick="makeCall()">Позвонить</button>
                <button class="btn-screen" onclick="toggleFullscreen()">На весь экран</button>
            </div>
        </div>
    </div>

    <script>
        const peer = new Peer();
        let localStream;

        // Доступ к камере
        navigator.mediaDevices.getUserMedia({ video: true, audio: true }).then(stream => {
            localStream = stream;
            document.getElementById('local-video').srcObject = stream;
        }).catch(err => alert("Нужен доступ к камере: " + err));

        peer.on('open', id => { document.getElementById('my-id').innerText = id; });

        // Входящий звонок с твоим ведома
        peer.on('call', call => {
            if (confirm("Входящий звонок. Принять?")) {
                call.answer(localStream);
                call.on('stream', stream => {
                    document.getElementById('remote-video').srcObject = stream;
                });
            }
        });

        function makeCall() {
            const id = document.getElementById('peer-id').value;
            if(!id) return alert("Введите ID");
            const call = peer.call(id, localStream);
            call.on('stream', stream => {
                document.getElementById('remote-video').srcObject = stream;
            });
        }

        // Вывод на большой экран
        function toggleFullscreen() {
            const video = document.getElementById('remote-video');
            if (video.requestFullscreen) {
                video.requestFullscreen();
            } else if (video.webkitRequestFullscreen) { /* Safari */
                video.webkitRequestFullscreen();
            }
        }
    </script>
</body>
</html>
