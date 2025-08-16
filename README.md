# Snake-Game



<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Snake Game</title>
    <!-- Inter font from Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap" rel="stylesheet">
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #0d1117;
            color: #c9d1d9;
        }

        #gameCanvas {
            border: 2px solid #238636;
            background-color: #010409;
            box-shadow: 0 0 20px rgba(35, 134, 54, 0.5);
            border-radius: 10px;
            touch-action: none; /* Prevents default browser actions like zooming */
        }

        .btn {
            @apply px-6 py-3 font-semibold rounded-lg shadow-lg transition-transform transform hover:scale-105 active:scale-95;
        }

        .btn-green {
            @apply bg-green-700 text-white hover:bg-green-600;
        }

        .btn-blue {
            @apply bg-blue-700 text-white hover:bg-blue-600;
        }
        
        .control-btn {
            @apply text-white bg-gray-800 p-4 rounded-full w-20 h-20 flex items-center justify-center text-4xl shadow-xl transition-transform transform hover:scale-110 active:scale-90;
        }

        @keyframes pulse-score {
            0% { transform: scale(1); }
            50% { transform: scale(1.1); }
            100% { transform: scale(1); }
        }

        .pulse {
            animation: pulse-score 0.5s ease-in-out;
        }
    </style>
</head>
<body class="flex flex-col items-center justify-center min-h-screen p-4">

    <div class="flex flex-col items-center gap-6 w-full max-w-lg">
        <h1 class="text-4xl font-bold text-center text-green-500 mb-4">
            🐍 Snake Game
        </h1>

        <div class="flex flex-col items-center gap-4 w-full">
            <!-- Game canvas will be rendered here -->
            <canvas id="gameCanvas"></canvas>
            
            <!-- Scoreboard and message display -->
            <div id="scoreboard" class="w-full flex justify-between items-center text-xl font-bold p-4 bg-gray-800 rounded-lg shadow-md mt-4">
                <span>Score: <span id="scoreDisplay">0</span></span>
                <span id="messageBox" class="text-green-400">Press Start to Play</span>
            </div>

            <!-- Start button and mobile controls -->
            <div class="flex flex-col items-center gap-4 mt-6 w-full">
                <button id="startButton" class="btn btn-green">Start Game</button>
                
                <div id="mobileControls" class="flex flex-col items-center gap-2 mt-4 md:hidden">
                    <button class="control-btn up-btn">▲</button>
                    <div class="flex gap-2">
                        <button class="control-btn left-btn">◀</button>
                        <button class="control-btn down-btn">▼</button>
                        <button class="control-btn right-btn">►</button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Use an IIFE (Immediately Invoked Function Expression) to avoid global variable pollution
        (function() {
            // Get canvas and its 2D rendering context
            const canvas = document.getElementById('gameCanvas');
            const ctx = canvas.getContext('2d');

            // Set canvas dimensions
            const gridSize = 20;
            const canvasSize = 400;
            canvas.width = canvasSize;
            canvas.height = canvasSize;

            // Game state variables
            let snake;
            let food;
            let direction;
            let score;
            let gameLoopInterval;
            let isGameOver;

            const scoreDisplay = document.getElementById('scoreDisplay');
            const messageBox = document.getElementById('messageBox');
            const startButton = document.getElementById('startButton');
            const mobileControls = document.getElementById('mobileControls');

            // Initialization function
            function init() {
                // Initialize snake to a single segment in the center
                snake = [{ x: 10, y: 10 }];
                // Initialize food to a random position
                generateFood();
                // Initial direction is right
                direction = 'right';
                // Reset score
                score = 0;
                scoreDisplay.textContent = score;
                // Game is not over
                isGameOver = false;
                messageBox.textContent = "Good luck!";
                // Make the start button responsive to start the game
                startButton.textContent = "Start Game";
                startButton.classList.remove('btn-blue');
                startButton.classList.add('btn-green');
            }

            // Function to generate new food at a random position
            function generateFood() {
                food = {
                    x: Math.floor(Math.random() * (canvasSize / gridSize)),
                    y: Math.floor(Math.random() * (canvasSize / gridSize))
                };
            }

            // Function to draw a single snake segment
            function drawSegment(x, y) {
                ctx.fillStyle = '#238636'; // Green color
                ctx.fillRect(x * gridSize, y * gridSize, gridSize, gridSize);
                ctx.strokeStyle = '#010409'; // Dark border for segments
                ctx.strokeRect(x * gridSize, y * gridSize, gridSize, gridSize);
            }

            // Function to draw the food
            function drawFood() {
                ctx.fillStyle = '#db6d28'; // Orange-red color
                ctx.fillRect(food.x * gridSize, food.y * gridSize, gridSize, gridSize);
                ctx.strokeStyle = '#010409';
                ctx.strokeRect(food.x * gridSize, food.y * gridSize, gridSize, gridSize);
            }

            // Main game update and render loop
            function gameLoop() {
                if (isGameOver) {
                    return; // Stop the loop if the game is over
                }

                // Clear the canvas
                ctx.clearRect(0, 0, canvasSize, canvasSize);

                // Update snake position
                const head = { x: snake[0].x, y: snake[0].y };
                switch (direction) {
                    case 'up':
                        head.y--;
                        break;
                    case 'down':
                        head.y++;
                        break;
                    case 'left':
                        head.x--;
                        break;
                    case 'right':
                        head.x++;
                        break;
                }

                // Check for collision with walls
                if (head.x < 0 || head.x >= canvasSize / gridSize || head.y < 0 || head.y >= canvasSize / gridSize) {
                    endGame();
                    return;
                }

                // Check for collision with self
                for (let i = 1; i < snake.length; i++) {
                    if (head.x === snake[i].x && head.y === snake[i].y) {
                        endGame();
                        return;
                    }
                }

                // Add new head to the snake
                snake.unshift(head);

                // Check if snake has eaten the food
                if (head.x === food.x && head.y === food.y) {
                    score++;
                    scoreDisplay.textContent = score;
                    scoreDisplay.classList.add('pulse');
                    setTimeout(() => scoreDisplay.classList.remove('pulse'), 500);
                    generateFood();
                } else {
                    // Remove the tail if no food was eaten
                    snake.pop();
                }

                // Draw everything
                snake.forEach(segment => drawSegment(segment.x, segment.y));
                drawFood();
            }

            // Function to end the game
            function endGame() {
                isGameOver = true;
                clearInterval(gameLoopInterval);
                messageBox.textContent = `Game Over! Final Score: ${score}`;
                startButton.textContent = "Play Again?";
                startButton.classList.remove('btn-green');
                startButton.classList.add('btn-blue');
            }

            // Event listener for keyboard input
            window.addEventListener('keydown', e => {
                const key = e.key;
                if (isGameOver) return; // Don't allow input if game is over

                // Prevent the snake from reversing on itself
                if (key === 'ArrowUp' && direction !== 'down') {
                    direction = 'up';
                } else if (key === 'ArrowDown' && direction !== 'up') {
                    direction = 'down';
                } else if (key === 'ArrowLeft' && direction !== 'right') {
                    direction = 'left';
                } else if (key === 'ArrowRight' && direction !== 'left') {
                    direction = 'right';
                }
            });

            // Event listener for mobile controls
            mobileControls.addEventListener('click', e => {
                if (isGameOver) return;

                const target = e.target.closest('.control-btn');
                if (!target) return;

                if (target.classList.contains('up-btn') && direction !== 'down') {
                    direction = 'up';
                } else if (target.classList.contains('down-btn') && direction !== 'up') {
                    direction = 'down';
                } else if (target.classList.contains('left-btn') && direction !== 'right') {
                    direction = 'left';
                } else if (target.classList.contains('right-btn') && direction !== 'left') {
                    direction = 'right';
                }
            });

            // Start/restart the game when the button is clicked
            startButton.addEventListener('click', () => {
                // Clear any existing game loop to prevent multiple instances
                clearInterval(gameLoopInterval);
                init();
                // Set the game loop to run every 100ms
                gameLoopInterval = setInterval(gameLoop, 100);
            });

            // Initial setup when the page loads
            init();

            // Make canvas responsive by resizing on window load and resize
            function resizeCanvas() {
                const container = canvas.parentElement;
                const size = Math.min(container.clientWidth, container.clientHeight, 400); // Max size of 400px
                canvas.width = size;
                canvas.height = size;
                // Re-draw the game when the canvas size changes
                if (!isGameOver) {
                    gameLoop();
                }
            }
            window.addEventListener('load', resizeCanvas);
            window.addEventListener('resize', resizeCanvas);
        })();
    </script>
</body>
</html>
