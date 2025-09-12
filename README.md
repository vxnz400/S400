<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>رادار النمل</title>
  <style>
    body {
      margin: 0;
      background: #000;
      color: #0f0;
      font-family: monospace;
      overflow: hidden;
    }
    #radarCanvas {
      position: absolute;
      top: 0; left: 0;
      width: 100%;
      height: 100%;
    }
    #video {
      position: absolute;
      top: 0; left: 0;
      width: 100%;
      height: 100%;
      object-fit: cover;
      z-index: -1;
      display: none;
    }
    .controls {
      position: absolute;
      bottom: 10px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 15px;
    }
    .btn {
      background: rgba(0, 255, 0, 0.2);
      border: 1px solid #0f0;
      border-radius: 50%;
      width: 50px;
      height: 50px;
      font-size: 22px;
      color: #0f0;
      display: flex;
      justify-content: center;
      align-items: center;
      cursor: pointer;
      transition: 0.3s;
    }
    .btn:hover { background: rgba(0, 255, 0, 0.4); }
    .night { background: #000; color: #0f0; }
    .day { background: #fff; color: #000; }
  </style>
</head>
<body>
  <video id="video" autoplay playsinline></video>
  <canvas id="radarCanvas"></canvas>

  <div class="controls">
    <div class="btn" id="toggleCam">📷</div>
    <div class="btn" id="toggleNight">🌙</div>
  </div>

  <script>
    const canvas = document.getElementById('radarCanvas');
    const ctx = canvas.getContext('2d');
    const video = document.getElementById('video');
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    let angle = 0;
    let nightMode = true;

    function drawRadar() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // خلفية
      ctx.fillStyle = nightMode ? "black" : "white";
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // شبكة (خطوط أفقية وعمودية)
      ctx.strokeStyle = nightMode ? "#0f02" : "#0003";
      for (let x = 0; x < canvas.width; x += 50) {
        ctx.beginPath();
        ctx.moveTo(x, 0);
        ctx.lineTo(x, canvas.height);
        ctx.stroke();
      }
      for (let y = 0; y < canvas.height; y += 50) {
        ctx.beginPath();
        ctx.moveTo(0, y);
        ctx.lineTo(canvas.width, y);
        ctx.stroke();
      }

      // دائرة الرادار
      ctx.strokeStyle = nightMode ? "#0f0" : "#000";
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.arc(canvas.width/2, canvas.height/2, 200, 0, Math.PI*2);
      ctx.stroke();

      // خط الدوران
      ctx.save();
      ctx.translate(canvas.width/2, canvas.height/2);
      ctx.rotate(angle);
      let grad = ctx.createLinearGradient(0,0,200,0);
      grad.addColorStop(0,"rgba(0,255,0,0.7)");
      grad.addColorStop(1,"transparent");
      ctx.fillStyle = grad;
      ctx.beginPath();
      ctx.moveTo(0,0);
      ctx.arc(0,0,200,0,0.1);
      ctx.closePath();
      ctx.fill();
      ctx.restore();

      angle += 0.02;
      requestAnimationFrame(drawRadar);
    }
    drawRadar();

    // زر الكاميرا
    document.getElementById('toggleCam').onclick = async () => {
      if (video.style.display === "none") {
        video.style.display = "block";
        try {
          let stream = await navigator.mediaDevices.getUserMedia({ video: true });
          video.srcObject = stream;
        } catch (e) {
          alert("الكاميرا غير مدعومة!");
        }
      } else {
        video.style.display = "none";
        let tracks = video.srcObject?.getTracks();
        tracks?.forEach(t => t.stop());
      }
    };

    // زر الوضع الليلي
    document.getElementById('toggleNight').onclick = () => {
      nightMode = !nightMode;
    };

    window.onresize = () => {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    };
  </script>
</body>
</html>
