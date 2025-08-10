<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Catch the Star</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;700&display=swap');

        body {
            font-family: 'Poppins', sans-serif;
            background-color: #121212;
            color: #e0e0e0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            flex-direction: column;
            overflow: hidden;
            text-align: center;
        }

        #game-container {
            position: relative;
            width: 100%;
            max-width: 600px;
            height: 80vh;
            max-height: 800px;
            background-color: #1e1e1e;
            border-radius: 20px;
            box-shadow: 0 0 30px rgba(0, 255, 255, 0.2), 0 0 10px rgba(0, 255, 255, 0.1);
            border: 2px solid #00ffff;
            overflow: hidden;
            touch-action: none;
        }

        #game-area {
            width: 100%;
            height: 100%;
            position: relative;
        }

        .star {
            position: absolute;
            font-size: 3rem;
            cursor: pointer;
            text-shadow: 0 0 10px #ffea00, 0 0 20px #ffea00, 0 0 30px #ffea00;
            transition: transform 0.2s ease-out, opacity 0.2s ease-out;
            pointer-events: auto;
        }

        .star.clicked {
            transform: scale(0.5);
            opacity: 0;
        }

        .score-pop-up {
            position: absolute;
            font-size: 2rem;
            font-weight: bold;
            color: #4CAF50;
            text-shadow: 0 0 5px #00e676;
            animation: fadeOutUp 1s forwards;
            pointer-events: none;
        }
        
        #score-board {
            position: absolute;
            top: 20px;
            left: 20px;
            font-size: 2rem;
            font-weight: bold;
            color: #00ffff;
            text-shadow: 0 0 10px #00ffff;
            z-index: 10;
        }

        #message-box {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 20px 40px;
            border-radius: 15px;
            border: 2px solid #00ffff;
            box-shadow: 0 0 20px rgba(0, 255, 255, 0.5);
            text-align: center;
            display: none;
            z-index: 20;
            font-size: 1.5rem;
        }

        #message-box button {
            margin-top: 20px;
            padding: 10px 25px;
            font-size: 1rem;
            font-weight: bold;
            border-radius: 10px;
            border: none;
            background: linear-gradient(45deg, #00ffff, #00b3ff);
            color: black;
            cursor: pointer;
            box-shadow: 0 5px 15px rgba(0, 179, 255, 0.4);
            transition: all 0.3s ease;
        }

        #message-box button:hover {
            transform: translateY(-3px);
            box-shadow: 0 8px 20px rgba(0, 179, 255, 0.6);
        }

        h1 {
            font-size: 2.5rem;
            font-weight: 700;
            margin-bottom: 20px;
            color: #00ffff;
            text-shadow: 0 0 10px #00ffff;
        }

        #play-button {
            padding: 15px 40px;
            font-size: 1.5rem;
            font-weight: bold;
            border-radius: 15px;
            border: none;
            background: linear-gradient(45deg, #00ffff, #00b3ff);
            color: black;
            cursor: pointer;
            box-shadow: 0 5px 20px rgba(0, 179, 255, 0.4);
            transition: all 0.3s ease;
            margin-top: 20px;
        }

        #play-button:hover {
            transform: scale(1.05);
            box-shadow: 0 8px 25px rgba(0, 179, 255, 0.6);
        }
        
        @keyframes fadeOutUp {
            from {
                opacity: 1;
                transform: translateY(0);
            }
            to {
                opacity: 0;
                transform: translateY(-50px);
            }
        }
    </style>
</head>
<body>
    <h1>Catch the Star</h1>
    <div id="game-container">
        <div id="score-board">Score: 0</div>
        <div id="game-area"></div>
        <div id="message-box">
            <p>Game Over!</p>
            <p id="final-score">Final Score: 0</p>
            <button id="restart-button">Play Again</button>
        </div>
    </div>
    <button id="play-button">Start Game</button>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const gameArea = document.getElementById('game-area');
            const scoreBoard = document.getElementById('score-board');
            const playButton = document.getElementById('play-button');
            const restartButton = document.getElementById('restart-button');
            const messageBox = document.getElementById('message-box');
            const finalScoreElement = document.getElementById('final-score');
            
            let score = 0;
            let gameInterval;
            let gameDuration = 30000; // 30 seconds
            let timeRemaining;
            let gameTimer;
            let isGameRunning = false;

            // Function to generate a new star
            function createStar() {
                if (!isGameRunning) return;

                const star = document.createElement('div');
                star.classList.add('star');
                star.innerHTML = '⭐'; // Using a star emoji
                
                // Get dimensions of game area
                const gameAreaRect = gameArea.getBoundingClientRect();
                const starSize = 48; // Approx size of the star emoji
                
                const x = Math.random() * (gameAreaRect.width - starSize);
                const y = Math.random() * (gameAreaRect.height - starSize);
                
                star.style.left = `${x}px`;
                star.style.top = `${y}px`;
                
                star.addEventListener('pointerdown', (e) => {
                    // Prevent default touch behavior
                    e.preventDefault();
                    if (isGameRunning) {
                        score += 1;
                        scoreBoard.textContent = `Score: ${score}`;
                        popScore(e.clientX, e.clientY);
                        
                        // Add a class for the fade-out effect and remove the element after the transition
                        star.classList.add('clicked');
                        star.addEventListener('transitionend', () => {
                            star.remove();
                        });
                    }
                });

                gameArea.appendChild(star);
            }
            
            // Function to show a score pop-up
            function popScore(x, y) {
                const popUp = document.createElement('div');
                popUp.classList.add('score-pop-up');
                popUp.textContent = '+1';
                popUp.style.left = `${x}px`;
                popUp.style.top = `${y}px`;
                document.body.appendChild(popUp);
                
                popUp.addEventListener('animationend', () => {
                    popUp.remove();
                });
            }

            // Function to start the game
            function startGame() {
                isGameRunning = true;
                score = 0;
                scoreBoard.textContent = 'Score: 0';
                gameArea.innerHTML = ''; // Clear any existing stars
                messageBox.style.display = 'none';
                playButton.style.display = 'none';

                gameInterval = setInterval(createStar, 1000); // Create a new star every second
                
                timeRemaining = gameDuration;
                gameTimer = setTimeout(endGame, gameDuration);
            }

            // Function to end the game
            function endGame() {
                isGameRunning = false;
                clearInterval(gameInterval);
                clearTimeout(gameTimer);
                finalScoreElement.textContent = `Final Score: ${score}`;
                messageBox.style.display = 'block';
            }

            // Event listeners
            playButton.addEventListener('click', startGame);
            restartButton.addEventListener('click', startGame);

        });
    </script>
</body>
</html>
