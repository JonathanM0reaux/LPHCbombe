<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Compte à Rebours - Bombe</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap');

        body {
            font-family: 'Orbitron', sans-serif;
            text-align: center;
            background: black url('https://media.giphy.com/media/xT9IgzoKnwFNmISR8I/giphy.gif') no-repeat center center fixed;
            background-size: cover;
            color: red;
        }
        #timer {
            font-size: 3rem;
            margin: 20px;
        }
        #status {
            font-size: 2rem;
            margin: 10px;
            color: white;
        }
        .progress-bar {
            width: 100%;
            height: 20px;
            background: red;
            transition: width 1s linear;
        }
        .container {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .keypad {
            display: grid;
            grid-template-columns: repeat(3, 80px);
            gap: 10px;
            justify-content: center;
            margin-top: 20px;
        }
        .keypad button {
            width: 80px;
            height: 80px;
            font-size: 2rem;
            background: red;
            color: white;
            border: none;
            cursor: pointer;
        }
        #inputCode {
            font-size: 2rem;
            text-align: center;
            width: 220px;
            margin-top: 20px;
            background: white;
            color: black;
            padding: 10px;
        }
        #clear {
            background: darkred;
        }
        #enter {
            background: green;
        }
        #startButton {
            margin-top: 20px;
            padding: 15px;
            font-size: 1.5rem;
            background: green;
            color: white;
            border: none;
            cursor: pointer;
        }
        #message {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 4rem;
            font-weight: bold;
            color: yellow;
            background: rgba(0, 0, 0, 0.8);
            padding: 20px;
            display: none;
        }
        .flash {
            animation: flash 0.5s;
        }
        @keyframes flash {
            0%, 100% { background-color: black; }
            50% { background-color: red; }
        }
        .final-countdown {
            animation: finalCountdown 0.5s infinite alternate;
        }
        @keyframes finalCountdown {
            0% { background-color: red; }
            100% { background-color: black; }
        }
    </style>
</head>
<body>

    <h1>⚠ Bombe Active ! ⚠</h1>
    <div class="progress-bar" id="progressBar"></div>
    <div id="timer">25:00</div>
    <div id="status">Appuyez sur "Démarrer" pour commencer</div>

    <button id="startButton" onclick="startTimer()">▶️ Démarrer</button>

    <div id="message"></div>

    <div class="container">
        <input type="text" id="inputCode" disabled>

        <div class="keypad">
            <button onclick="pressKey(1)">1</button>
            <button onclick="pressKey(2)">2</button>
            <button onclick="pressKey(3)">3</button>
            <button onclick="pressKey(4)">4</button>
            <button onclick="pressKey(5)">5</button>
            <button onclick="pressKey(6)">6</button>
            <button onclick="pressKey(7)">7</button>
            <button onclick="pressKey(8)">8</button>
            <button onclick="pressKey(9)">9</button>
            <button id="clear" onclick="clearCode()">C</button>
            <button id="enter" onclick="checkCode()">✔</button>
            <button onclick="pressKey(0)">0</button>
        </div>
    </div>

    <audio id="clickSound" src="https://www.soundjay.com/button/sounds/button-16.mp3" preload="auto"></audio>
    <audio id="errorSound" src="https://www.soundjay.com/button/sounds/beep-10.mp3" preload="auto"></audio>
    <audio id="winSound" src="https://www.soundjay.com/misc/sounds/success-fanfare-trumpets.mp3" preload="auto"></audio>
    <audio id="tickTockSound" src="https://www.soundjay.com/clock/sounds/clock-ticking-5.mp3" preload="auto" loop></audio>
    <audio id="fastTickSound" src="https://www.soundjay.com/clock/sounds/clock-ticking-2.mp3" preload="auto" loop></audio>

    <script>
        let timeLeft = 25 * 60;
        let timerDisplay = document.getElementById("timer");
        let statusDisplay = document.getElementById("status");
        let inputCode = document.getElementById("inputCode");
        let progressBar = document.getElementById("progressBar");
        let messageBox = document.getElementById("message");
        const correctCode = "7542";
        let timer;
        let isRunning = false;

        function updateTimer() {
            let minutes = Math.floor(timeLeft / 60);
            let seconds = timeLeft % 60;
            timerDisplay.textContent = `${minutes}:${seconds.toString().padStart(2, "0")}`;
            
            let percentage = (timeLeft / (25 * 60)) * 100;
            progressBar.style.width = percentage + "%";

            if (timeLeft === 0) {
                clearInterval(timer);
                explode();
            }

            if (timeLeft === 60) {
                document.getElementById("tickTockSound").pause();
                document.getElementById("fastTickSound").play();
            }

            if (timeLeft <= 10) {
                document.body.classList.add("final-countdown");
            }

            timeLeft--;
        }

        function startTimer() {
            if (!isRunning) {
                timer = setInterval(updateTimer, 1000);
                document.getElementById("tickTockSound").play();
                document.getElementById("startButton").style.display = "none";
                statusDisplay.textContent = "Entrez le code pour désamorcer";
                isRunning = true;
            }
        }

        function pressKey(num) {
            if (inputCode.value.length < 4) {
                inputCode.value += num;
                let sound = document.getElementById("clickSound");
                sound.volume = 1.0;
                sound.play();
            }
        }

        function clearCode() {
            inputCode.value = "";
        }

        function checkCode() {
            let soundError = document.getElementById("errorSound");
            let soundWin = document.getElementById("winSound");

            if (inputCode.value === correctCode) {
                clearInterval(timer);
                document.getElementById("tickTockSound").pause();
                document.getElementById("fastTickSound").pause();
                soundWin.play();
                showMessage("🎉 Bombe désamorcée !");
                document.body.style.backgroundColor = "green";
                timerDisplay.textContent = "✅ SAFE";
            } else {
                soundError.play();
                showMessage("❌ Code incorrect !");
                document.body.classList.add("flash");
                setTimeout(() => {
                    document.body.classList.remove("flash");
                }, 500);
                clearCode();
            }
        }

        function showMessage(text) {
            messageBox.innerHTML = text;
            messageBox.style.display = "block";
            setTimeout(() => {
                messageBox.style.display = "none";
            }, 2000);
        }
    </script>

</body>
</html>
