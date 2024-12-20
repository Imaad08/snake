---
layout: base
title: Snake
permalink: /snake/
---

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Snake Game</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/framer-motion/6.5.1/framer-motion.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: black;
            color: white;
            font-family: 'Arial', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
        }

        .container {
            position: relative;
            text-align: center;
        }

        canvas {
            border: 5px solid #fff;
            border-radius: 12px;
            background: linear-gradient(145deg, #1e1e2e, #2e2e4e);
            box-shadow: 0 0 20px rgba(255, 255, 255, 0.5);
        }

        .overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: rgba(0, 0, 0, 0.8);
            z-index: 10;
            visibility: hidden;
            opacity: 0;
            transition: visibility 0s, opacity 0.5s;
        }

        .overlay.visible {
            visibility: visible;
            opacity: 1;
        }

        .overlay h1 {
            font-size: 3rem;
            margin-bottom: 20px;
            color: #fff;
        }

        .overlay button {
            background: linear-gradient(90deg, #ff416c, #ff4b2b);
            border: none;
            padding: 10px 20px;
            color: white;
            font-size: 1.5rem;
            border-radius: 5px;
            cursor: pointer;
            transition: transform 0.2s;
        }

        .overlay button:hover {
            transform: scale(1.1);
        }

    </style>

</head>
<body>
    <div class="container">
        <canvas id="snake" width="400" height="400"></canvas>
        <div class="overlay" id="menu">
            <h1>Welcome to Snake Game</h1>
            <button id="startGame">Start Game</button>
        </div>
        <div class="overlay" id="gameOver">
            <h1>Game Over!</h1>
            <p>Your Score: <span id="finalScore">0</span></p>
            <button id="restartGame">Play Again</button>
        </div>
    </div>

    <script>
    const canvas = document.getElementById("snake");
    const ctx = canvas.getContext("2d");
    const menuOverlay = document.getElementById("menu");
    const gameOverOverlay = document.getElementById("gameOver");
    const startGameButton = document.getElementById("startGame");
    const restartGameButton = document.getElementById("restartGame");
    const finalScoreElement = document.getElementById("finalScore");

    const BLOCK = 20;
    const FPS = 240; // Higher FPS for smoother movement
    const SPEED_ZONES = [
        { x: 5, y: 5, width: 4, height: 4, color: "rgba(255, 0, 0, 0.2)", speedMultiplier: 0.5 },
        { x: 12, y: 12, width: 4, height: 4, color: "rgba(0, 255, 0, 0.2)", speedMultiplier: 2 }
    ];

    let snake = [];
    let food = {};
    let direction = "RIGHT";
    let nextDirection = "RIGHT";
    let score = 0;
    let lastTime = 0;
    let moveInterval = 0; // Track the interval between moves
    let speed = 600; // Snake moves every 200ms (adjustable)

    const resetGame = () => {
        snake = [
            { x: 8, y: 8 },
            { x: 7, y: 8 },
            { x: 6, y: 8 }
        ];
        food = generateFood();
        direction = "RIGHT";
        nextDirection = "RIGHT";
        score = 0;
        moveInterval = 0; // Reset move interval
        menuOverlay.classList.remove("visible");
        gameOverOverlay.classList.remove("visible");
    };

    const drawZone = ({ x, y, width, height, color }) => {
        ctx.fillStyle = color;
        ctx.fillRect(x * BLOCK, y * BLOCK, width * BLOCK, height * BLOCK);
    };

    const drawSnake = () => {
        snake.forEach((segment, index) => {
            ctx.fillStyle = index === 0 ? "gold" : "white";
            ctx.fillRect(segment.x * BLOCK, segment.y * BLOCK, BLOCK, BLOCK);
        });
    };

    const drawFood = () => {
        ctx.fillStyle = "red";
        ctx.beginPath();
        ctx.arc(food.x * BLOCK + BLOCK / 2, food.y * BLOCK + BLOCK / 2, BLOCK / 2, 0, Math.PI * 2);
        ctx.fill();
    };

    const generateFood = () => {
        let newFood;
        do {
            newFood = {
                x: Math.floor(Math.random() * (canvas.width / BLOCK)),
                y: Math.floor(Math.random() * (canvas.height / BLOCK))
            };
        } while (snake.some(segment => segment.x === newFood.x && segment.y === newFood.y));
        return newFood;
    };

    const detectCollision = () => {
        const head = snake[0];

        // Check wall collision
        if (head.x < 0 || head.y < 0 || head.x >= canvas.width / BLOCK || head.y >= canvas.height / BLOCK) {
            return true;
        }

        // Check self collision
        return snake.slice(1).some(segment => segment.x === head.x && segment.y === head.y);
    };

    const checkFoodCollision = () => {
        if (snake[0].x === food.x && snake[0].y === food.y) {
            score++;
            food = generateFood();
            snake.push({}); // Add a new segment
        }
    };

    const moveSnake = () => {
        const head = { ...snake[0] };

        if (nextDirection === "UP") head.y--;
        if (nextDirection === "DOWN") head.y++;
        if (nextDirection === "LEFT") head.x--;
        if (nextDirection === "RIGHT") head.x++;

        snake.unshift(head);
        snake.pop();
        direction = nextDirection; // Update the direction after the move is completed
    };

    const checkSpeedZone = () => {
        const head = snake[0];
        for (const zone of SPEED_ZONES) {
            if (
                head.x >= zone.x &&
                head.x < zone.x + zone.width &&
                head.y >= zone.y &&
                head.y < zone.y + zone.height
            ) {
                return zone.speedMultiplier;
            }
        }
        return 1;
    };

    const gameLoop = (timestamp) => {
        if (!lastTime) lastTime = timestamp;
        const deltaTime = timestamp - lastTime;

        moveInterval += deltaTime;
        if (moveInterval >= speed) {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            SPEED_ZONES.forEach(drawZone);
            drawFood();
            drawSnake();
            moveSnake();
            checkFoodCollision();

            if (detectCollision()) {
                finalScoreElement.textContent = score;
                gameOverOverlay.classList.add("visible");
                return;
            }

            lastTime = timestamp;
            moveInterval = 0; // Reset move interval after each move
        }

        requestAnimationFrame(gameLoop); // Request the next frame
    };

    const startGame = () => {
        resetGame();
        requestAnimationFrame(gameLoop); // Start the game loop
    };

    startGameButton.addEventListener("click", startGame);
    restartGameButton.addEventListener("click", startGame);

    window.addEventListener("keydown", (event) => {
        const allowedKeys = {
            ArrowUp: "UP",
            ArrowDown: "DOWN",
            ArrowLeft: "LEFT",
            ArrowRight: "RIGHT"
        };

        if (
            allowedKeys[event.key] &&
            !(
                (direction === "UP" && allowedKeys[event.key] === "DOWN") ||
                (direction === "DOWN" && allowedKeys[event.key] === "UP") ||
                (direction === "LEFT" && allowedKeys[event.key] === "RIGHT") ||
                (direction === "RIGHT" && allowedKeys[event.key] === "LEFT")
            )
        ) {
            nextDirection = allowedKeys[event.key];
        }
    });

    menuOverlay.classList.add("visible");

</script>

</body>
</html>
