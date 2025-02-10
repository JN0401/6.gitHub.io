<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>智能抢答系统</title>
    <style>
        :root {
            --primary-color: #6C5CE7;
            --accent-color: #A66EFA;
            --glass-bg: rgba(255, 255, 255, 0.1);
        }

        body {
            margin: 0;
            height: 100vh;
            background: linear-gradient(135deg, #2D2A6E 0%, #413B8F 100%);
            font-family: 'Microsoft Yahei', sans-serif;
            color: white;
            overflow: hidden;
            position: relative;
        }

        .page {
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            transition: opacity 0.6s ease;
        }

        #startPage {
            z-index: 2;
        }

        #startPage h1 {
            font-size: 80px;
            margin-bottom: 150px;
        }

        #startPage .start-btn {
            font-size: 40px;
            padding: 20px 60px;
            border-radius: 40px;
            background: linear-gradient(45deg, var(--primary-color), var(--accent-color));
            cursor: pointer;
            transition: transform 0.3s ease, background 0.3s ease;
        }

        #startPage .start-btn:hover {
            transform: scale(1.1);
            background: linear-gradient(45deg, var(--accent-color), var(--primary-color));
        }

        #quizPage {
            position: absolute;
            top: 0;
            left: 0;
            opacity: 0;
            pointer-events: none;
            z-index: 1;
        }

        #quizPage.active {
            opacity: 1;
            pointer-events: auto;
        }

        .podium {
            width: 100%;
            height: 100%;
            object-fit: cover;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 0;
        }

        #timer {
            font-size: 6em;
            text-align: center;
            margin: 20px 0;
            font-family: 'Digital-7', monospace;
            text-shadow: 0 0 15px var(--accent-color);
            z-index: 2;
            position: absolute;
            top: 10%; /* 计时器向上移动 */
        }

        .winner {
            text-align: center;
            opacity: 0;
            transition: opacity 0.5s;
            z-index: 4;
            margin-top: 50px; /* 名字和图片向下移动30像素 */
        }

        .avatar {
            width: 150px;
            height: 150px;
            border-radius: 50%;
            border: 3px solid white;
            box-shadow: 0 0 30px var(--accent-color);
            margin-bottom: 15px;
        }

        .winner h2 {
            font-size: 6em;
            margin-top: 20px;
        }

        .score-panel {
            position: fixed;
            top: 20px; /* 移动到右上角 */
            right: 20px; /* 移动到右上角 */
            display: flex;
            gap: 20px;
            background: var(--glass-bg);
            padding: 20px 40px;
            border-radius: 50px;
            backdrop-filter: blur(5px);
            border: 1px solid rgba(255,255,255,0.2);
            z-index: 4;
        }

        .score-panel button {
            font-size: 1.2em;
            padding: 10px 20px;
            border-radius: 30px; /* 更圆润的边角 */
            background: linear-gradient(45deg, var(--primary-color), var(--accent-color)); /* 渐变背景色 */
            color: white;
            border: none;
            cursor: pointer;
            transition: transform 0.3s ease, background 0.3s ease; /* 添加过渡效果 */
        }

        .score-panel button:hover {
            transform: scale(1.1); /* 点击时放大 */
            background: linear-gradient(45deg, var(--accent-color), var(--primary-color)); /* 悬停时交换渐变色 */
        }

        .score-panel button:active {
            transform: scale(0.9); /* 按下时缩小 */
            box-shadow: 0 3px 6px rgba(0, 0, 0, 0.3); /* 按下时的阴影效果 */
        }

        .score-input button {
            font-size: 1.2em;
            padding: 10px 20px;
            border-radius: 30px; /* 更圆润的边角 */
            background: linear-gradient(45deg, #4CAF50, #45a049); /* 绿色渐变背景色 */
            color: white;
            border: none;
            cursor: pointer;
            transition: transform 0.3s ease, background 0.3s ease; /* 添加过渡效果 */
        }

        .score-input button:hover {
            transform: scale(1.1); /* 点击时放大 */
            background: linear-gradient(45deg, #45a049, #4CAF50); /* 悬停时交换渐变色 */
        }

        .score-input button:active {
            transform: scale(0.9); /* 按下时缩小 */
            box-shadow: 0 3px 6px rgba(0, 0, 0, 0.3); /* 按下时的阴影效果 */
        }

        #submitScoreBtn {
            background: linear-gradient(45deg, #FF9800, #ff5722); /* 橙色渐变背景色 */
        }

        #submitScoreBtn:hover {
            background: linear-gradient(45deg, #ff5722, #FF9800); /* 悬停时交换渐变色 */
        }

        #continueQuizBtn {
            background: linear-gradient(45deg, #008CBA, #007bb5); /* 青色渐变背景色 */
        }

        #continueQuizBtn:hover {
            background: linear-gradient(45deg, #007bb5, #008CBA); /* 悬停时交换渐变色 */
        }

        #continueQuizBtn:active {
            transform: scale(0.9); /* 按下时缩小 */
            box-shadow: 0 3px 6px rgba(0, 0, 0, 0.3); /* 按下时的阴影效果 */
        }

        .score-input input {
            width: 60px;
            padding: 5px;
            font-size: 1.2em;
            text-align: center;
            border-radius: 10px;
            border: 1px solid rgba(255,255,255,0.2);
            background: var(--glass-bg);
            color: white;
        }

        #fireworksCanvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 6; /* 确保烟花在最顶层 */
        }
    </style>
</head>
<body>
    <!-- 开始页面 -->
    <div class="page" id="startPage">
        <h1>智能抢答系统</h1>
        <button class="start-btn" onclick="startQuiz()">开始抢答</button>
    </div>

    <!-- 答题页面 -->
    <div class="page" id="quizPage">
        <img src="领奖台.jpg" class="podium" alt="领奖台">
        <div id="timer">00:00</div>
        
        <div class="winner" id="winner">
            <img src="黄嘉妮.jpg" class="avatar" alt="优胜者">
            <h2>黄嘉妮</h2>
        </div>

        <div class="score-panel" id="score-panel">
            <div class="score-input">
                <button onclick="changeScore(-1)">-1 分</button>
                <input type="number" id="score-input" value="0" min="0">
                <button onclick="changeScore(1)">+1 分</button>
            </div>
            <button id="submitScoreBtn" onclick="submitScore()">提交成绩</button>
            <button id="continueQuizBtn" onclick="continueQuiz()">继续抢答</button>
        </div>
    </div>

    <!-- 音频元素 -->
    <audio id="backgroundAudio" src="背景音.mp3" loop></audio>
    <audio id="selectAudio" src="选中.mp3"></audio>

    <!-- 烟花Canvas元素 -->
    <canvas id="fireworksCanvas"></canvas>

    <script>
        let timer;
        let seconds = 0;
        let score = 0;
        let backgroundAudio = document.getElementById('backgroundAudio');
        let selectAudio = document.getElementById('selectAudio');
        let fireworksCanvas = document.getElementById('fireworksCanvas');
        let ctx = fireworksCanvas.getContext('2d');
        let particles = [];

        // 烟花粒子对象
        function Particle(x, y, color, size, lifespan, speed, angle) {
            this.x = x;
            this.y = y;
            this.color = color;
            this.size = size;
            this.maxSize = size;
            this.lifespan = lifespan;
            this.speed = speed;
            this.angle = angle;
        }

        Particle.prototype.update = function () {
            this.lifespan--;
            this.size *= 0.98; // 让粒子逐渐减小
            this.speed *= 0.98; // 让粒子逐渐减速
            this.x += Math.cos(this.angle * Math.PI / 180) * this.speed; // 水平方向移动
            this.y -= Math.sin(this.angle * Math.PI / 180) * this.speed; // 垂直方向移动

            if (this.lifespan <= 0) {
                return false;
            }
            return true;
        };

        Particle.prototype.draw = function () {
            ctx.fillStyle = this.color;
            ctx.beginPath();
            ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
            ctx.fill();
            ctx.closePath();
        };

        // 启动烟花
        function startFireworks(x, y) {
            for (let i = 0; i < 200; i++) {
                setTimeout(createParticle.bind(null, x, y), i * 10);
            }
            animate();
        }

        function createParticle(x, y) {
            let color = `hsl(${Math.random() * 360}, 100%, 50%)`; // 随机颜色
            let size = 5 + Math.random() * 10;
            let lifespan = 100 + Math.random() * 50;
            let speed = 2 + Math.random() * 5;
            let angle = Math.random() * 360; // 随机角度

            particles.push(new Particle(x, y, color, size, lifespan, speed, angle));
        }

        function animate() {
            ctx.clearRect(0, 0, fireworksCanvas.width, fireworksCanvas.height);

            particles = particles.filter(particle => particle.update());
            particles.forEach(particle => particle.draw());

            if (particles.length > 0) {
                requestAnimationFrame(animate);
            }
        }

        function startQuiz() {
            // 播放背景音
            backgroundAudio.play();

            // 隐藏开始页面
            document.getElementById('startPage').style.opacity = '0';
            setTimeout(() => {
                document.getElementById('startPage').classList.add('hidden');
            }, 600);

            // 显示答题页面
            document.getElementById('quizPage').classList.add('active');
            startTimer();

            // 3秒后显示获奖者
            setTimeout(() => {
                showWinner();
            }, 4000);
        }

        function startTimer() {
            seconds = 0;
            timer = setInterval(() => {
                seconds++;
                let m = Math.floor(seconds / 60).toString().padStart(2, '0');
                let s = (seconds % 60).toString().padStart(2, '0');
                document.getElementById('timer').textContent = `${m}:${s}`;
            }, 1000);
        }

        function changeScore(delta) {
            const input = document.getElementById('score-input');
            let currentScore = parseInt(input.value);
            currentScore += delta;
            if (currentScore < 0) currentScore = 0;
            input.value = currentScore;
        }

        function submitScore() {
            const finalScore = document.getElementById('score-input').value;
            alert(`最终成绩: ${finalScore} 分`);
        }

        function continueQuiz() {
            // 重置界面
            document.getElementById('winner').style.opacity = '0';
            document.getElementById('score-input').value = 0;
            clearInterval(timer);
            startTimer();

            // 播放背景音
            backgroundAudio.play();

            // 3秒后显示获奖者
            setTimeout(() => {
                showWinner();
            }, 4000);
        }

        function showWinner() {
            document.getElementById('winner').style.opacity = '1';
            clearInterval(timer); // 停止计时器
            backgroundAudio.pause(); // 停止背景音
            selectAudio.play(); // 播放选中音

            // 获取获奖者位置
            const winnerDiv = document.getElementById('winner');
            const rect = winnerDiv.getBoundingClientRect();
            const x = rect.left + rect.width / 2;
            const y = rect.top + rect.height / 2;

            // 启动烟花
            startFireworks(x, y);
        }

        window.addEventListener('resize', () => {
            fireworksCanvas.width = window.innerWidth;
            fireworksCanvas.height = window.innerHeight;
        });

        fireworksCanvas.width = window.innerWidth;
        fireworksCanvas.height = window.innerHeight;
    </script>
</body>
</html>
