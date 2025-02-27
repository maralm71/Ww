# Ww
<!DOCTYPE html>
<html lang="fa">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>فال حافظ با تشخیص چهره</title>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
    <script src="https://cdn.jsdelivr.net/npm/face-api.js"></script>
    <style>
        body {
            text-align: center;
            font-family: Arial, sans-serif;
            background-color: black;
            color: white;
        }
        video {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            object-fit: cover;
            opacity: 0.5;
        }
        #fal-text {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 24px;
            font-weight: bold;
            background: rgba(0, 0, 0, 0.7);
            padding: 20px;
            border-radius: 10px;
        }
        #start-button {
            position: absolute;
            bottom: 50px;
            left: 50%;
            transform: translateX(-50%);
            padding: 10px 20px;
            font-size: 18px;
            background-color: #ff5722;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
    </style>
</head>
<body>

    <h2>📷 روی دکمه "شروع" بزنید و لبخند بزنید تا فال شما نمایش داده شود!</h2>
    <button id="start-button">شروع</button>
    <video id="video" autoplay style="display: none;"></video>
    <div id="fal-text">منتظر تشخیص چهره...</div>
    
    <script>
        const falList = {
            happy: ["لبخندت زندگی را زیبا می‌کند!", "روزی پر از شادی در انتظارت است!", "منتظر یک خبر خوش باش!"],
            sad: ["غم‌های امروز، شادی فردا را می‌سازند.", "صبر کن، این روزها هم می‌گذرند.", "دوری از غم، نزدیک‌ترین راه به شادی است."],
            surprised: ["یک اتفاق غیرمنتظره در راه است!", "زندگی پر از شگفتی‌هاست، آماده باش!", "امروز ممکن است چیزی تو را شگفت‌زده کند!"],
            neutral: ["زندگی تعادل است، مسیرت را پیدا کن.", "آرام باش و از لحظات زندگی لذت ببر.", "گاهی هیچ تغییری، بهترین تغییر است!"]
        };

        let video = document.getElementById('video');
        let falText = document.getElementById('fal-text');
        let startButton = document.getElementById('start-button');

        startButton.addEventListener('click', async () => {
            try {
                startButton.style.display = "none"; 
                video.style.display = "block";

                let stream = await navigator.mediaDevices.getUserMedia({ video: true });
                video.srcObject = stream;

                console.log("✅ دوربین فعال شد");

                await faceapi.nets.tinyFaceDetector.loadFromUri('https://cdn.jsdelivr.net/npm/face-api.js/weights/');
                await faceapi.nets.faceExpressionNet.loadFromUri('https://cdn.jsdelivr.net/npm/face-api.js/weights/');

                console.log("✅ مدل‌های تشخیص چهره بارگذاری شد");

                checkFace();
            } catch (error) {
                console.error("🚨 خطا در دریافت دوربین:", error);
                falText.innerText = "🚨 امکان دسترسی به دوربین وجود ندارد! لطفاً مجوز دوربین را بررسی کنید.";
            }
        });

        async function checkFace() {
            setInterval(async () => {
                const detections = await faceapi.detectAllFaces(video, new faceapi.TinyFaceDetectorOptions()).withFaceExpressions();
                
                console.log("🔍 تعداد چهره‌های شناسایی‌شده:", detections.length);

                if (detections.length > 0) {
                    const expressions = detections[0].expressions;
                    console.log("😊 حالات چهره:", expressions);

                    let mood = "neutral";

                    if (expressions.happy > 0.5) mood = "happy";
                    else if (expressions.sad > 0.5) mood = "sad";
                    else if (expressions.surprised > 0.5) mood = "surprised";

                    let selectedFal = falList[mood][Math.floor(Math.random() * falList[mood].length)];
                    falText.innerText = "✨ فال شما: " + selectedFal;
                    
                    console.log("📜 فال انتخاب شده:", selectedFal);

                    speakText(selectedFal);
                } else {
                    falText.innerText = "😕 چهره‌ای شناسایی نشد، لطفاً مستقیم به دوربین نگاه کنید!";
                }
            }, 5000);
        }

        function speakText(text) {
            let msg = new SpeechSynthesisUtterance();
            msg.text = text;
            msg.lang = "fa-IR";
            msg.rate = 0.9;
            speechSynthesis.speak(msg);
        }
    </script>

</body>
</html>
