---
layout: base
title: Snake
permalink: /snake/
---

<style>
        body {
            margin: 0;
            /* display: flex; */
            /* justify-content: center; */
            /* align-items: center; */
            height: 100vh;
            background: #2c3e50;
            font-family: Arial, sans-serif;
        }
        #gameContainer {
            position: relative;
            display: none;  /* Hidden initially */
        }
        #menuScreen {
            position: absolute;
            width: 500px;
            height: 500px;
            background: #2c3e50;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            border-radius: 10px;
        }
        .title {
            color: #2ecc71;
            font-size: 48px;
            margin-bottom: 30px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }
        .menu-container {
            display: flex;
            flex-direction: column;
            gap: 15px;
            align-items: center;
        }
        button {
            background: #2ecc71;
            border: none;
            padding: 12px 30px;
            margin: 5px;
            border-radius: 8px;
            color: white;
            cursor: pointer;
            font-size: 18px;
            transition: all 0.3s;
            min-width: 200px;
        }
        button:hover {
            background: #27ae60;
            transform: scale(1.05);
        }
        .settings {
            background: #2c3e50;
            padding: 20px;
            border-radius: 10px;
            color: white;
            text-align: center;
        }
        .settings label {
            display: block;
            margin: 15px 0;
            font-size: 18px;
        }
        .settings select {
            margin-left: 10px;
            padding: 5px;
            border-radius: 5px;
            font-size: 16px;
        }
        #gameScore {
            position: fixed;
            top: 60px;
            /* right: 400px; */
            color: white;
            font-size: 24px;
        }
        .pause-button {
            position: fixed;
            top: 200px;
            right: 400px;
            background: #2ecc71;
            border: none;
            padding: 8px 20px;
            border-radius: 5px;
            color: white;
            cursor: pointer;
            font-size: 16px;
        }
        .score-display {
            font-size: 32px;
            color: #2ecc71;
            margin: 20px 0;
        }
    </style>

<body>
    <div id="menuScreen">
        <div class="menu-container" id="mainMenu">
            <h1 class="title">Snake Game</h1>
            <button onclick="startGame()">Play</button>
            <button onclick="showSettings()">Settings</button>
        </div>
    </div>
    <div id="gameContainer">
        <div id="gameScore">Score: 0</div>
        <button class="pause-button" onclick="togglePause()" style="background: #2ecc71;">Pause</button>
    </div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pixi.js/7.3.2/pixi.min.js"></script>
    <script>
    const GRID_SIZE = 20;
    const CELL_SIZE = 25;
    let MOVE_INTERVAL = 150;
    const BASE_MOVE_INTERVAL = 150;
    const SMOOTHNESS = 0.15;

    let gameState = 'menu';
    let wallsEnabled = true;
    let speedMultiplier = 1;

    // Zone variables
    let zones = [];
    const ZONE_TYPES = {
        SLOW: { color: 0x3498db, multiplier: 0.5 },
        FAST: { color: 0xf1c40f, multiplier: 2 },
    };

    const menuScreen = document.getElementById('menuScreen');
    const gameContainer = document.getElementById('gameContainer');

    function showMainMenu() {
        gameContainer.style.display = 'none';
        menuScreen.style.display = 'flex';
        menuScreen.innerHTML = `
            <div class="menu-container">
                <h1 class="title">Snake Game</h1>
                <button onclick="startGame()">Play</button>
                <button onclick="showSettings()">Settings</button>
            </div>
        `;
        gameState = 'menu';
    }

    function showSettings() {
        menuScreen.innerHTML = `
            <div class="menu-container">
                <h1 class="title">Settings</h1>
                <div class="settings">
                    <label>
                        Game Speed:
                        <select onchange="updateSpeed(this.value)">
                            <option value="0.5" ${speedMultiplier === 0.5 ? 'selected' : ''}>0.5x</option>
                            <option value="1" ${speedMultiplier === 1 ? 'selected' : ''}>Normal</option>
                            <option value="2" ${speedMultiplier === 2 ? 'selected' : ''}>2x</option>
                        </select>
                    </label>
                    <label>
                        Walls:
                        <select onchange="updateWalls(this.value)">
                            <option value="false" ${wallsEnabled ? 'selected' : ''}>On</option>
                            <option value="true" ${!wallsEnabled ? 'selected' : ''}>Off</option>
                        </select>
                    </label>
                </div>
                <button onclick="showMainMenu()">Back</button>
            </div>
        `;
    }

    function showGameOverMenu() {
        gameContainer.style.display = 'none';
        menuScreen.style.display = 'flex';
        menuScreen.innerHTML = `
            <div class="menu-container">
                <h1 class="title">Game Over!</h1>
                <div class="score-display">Score: ${score}</div>
                <button onclick="startGame()">Play Again</button>
                <button onclick="showSettings()">Settings</button>
                <button onclick="showMainMenu()">Main Menu</button>
            </div>
        `;
        gameState = 'gameOver';
    }

    function updateSpeed(value) {
        speedMultiplier = parseFloat(value);
        MOVE_INTERVAL = BASE_MOVE_INTERVAL / speedMultiplier;
    }

    function updateWalls(value) {
        wallsEnabled = value === 'true';
    }

    function togglePause() {
        if (gameState === 'playing') {
            gameState = 'paused';
            document.querySelector('.pause-button').textContent = 'Resume';
            menuScreen.style.display = 'flex';
            menuScreen.innerHTML = `
                <div class="menu-container">
                    <h1 class="title">Paused</h1>
                    <button onclick="togglePause()">Resume</button>
                    <button onclick="showSettings()">Settings</button>
                    <button onclick="showMainMenu()">Main Menu</button>
                </div>
            `;
        } else if (gameState === 'paused') {
            gameState = 'playing';
            document.querySelector('.pause-button').textContent = 'Pause';
            menuScreen.style.display = 'none';
        }
    }

    // Create PIXI Application
    const app = new PIXI.Application({
        width: GRID_SIZE * CELL_SIZE,
        height: GRID_SIZE * CELL_SIZE,
        backgroundColor: 0x34495e,
        antialias: true
    });
    gameContainer.appendChild(app.view);

    // Game state variables
    let snake = [{x: 10, y: 10}];
    let snakeSegments = [];
    let direction = {x: 1, y: 0};
    let nextDirection = {x: 1, y: 0};
    let food = {x: 15, y: 10};
    let lastMoveTime = 0;
    let score = 0;
    const scoreElement = document.getElementById('gameScore');

    function startGame() {
        // Reset game state
        snake = [{x: 10, y: 10}];
        direction = {x: 1, y: 0};
        nextDirection = {x: 1, y: 0};
        score = 0;
        scoreElement.textContent = `Score: ${score}`;

        // Remove existing segments
        snakeSegments.forEach(segment => app.stage.removeChild(segment));
        snakeSegments = [];

        // Create initial snake segment
        snakeSegments.push(createSnakeSegment(snake[0].x, snake[0].y));

        generateFood();
        generateZones(); // Generate initial zones
        gameState = 'playing';
        menuScreen.style.display = 'none';
        gameContainer.style.display = 'block';
        document.querySelector('.pause-button').textContent = 'Pause';
    }

    function createSnakeSegment(x, y) {
        const segment = new PIXI.Graphics();
        segment.beginFill(0x2ecc71);
        segment.drawRoundedRect(0, 0, CELL_SIZE - 2, CELL_SIZE - 2, 8);
        segment.endFill();
        segment.x = x * CELL_SIZE;
        segment.y = y * CELL_SIZE;
        segment.actualX = x;
        segment.actualY = y;
        app.stage.addChild(segment);
        return segment;
    }

    // Create food
    const foodGraphic = new PIXI.Graphics();
    foodGraphic.beginFill(0xe74c3c);
    foodGraphic.drawCircle(CELL_SIZE/2, CELL_SIZE/2, CELL_SIZE/2 - 2);
    foodGraphic.endFill();
    app.stage.addChild(foodGraphic);

    function updateFoodPosition() {
        foodGraphic.x = food.x * CELL_SIZE;
        foodGraphic.y = food.y * CELL_SIZE;
    }

    function generateFood() {
        while (true) {
            const newFood = {
                x: Math.floor(Math.random() * GRID_SIZE),
                y: Math.floor(Math.random() * GRID_SIZE)
            };
            if (!snake.some(segment => segment.x === newFood.x && segment.y === newFood.y)) {
                food = newFood;
                updateFoodPosition();
                break;
            }
        }
    }

    // Create zones
    function createZone(x, y, type) {
        const zoneGraphic = new PIXI.Graphics();
        zoneGraphic.beginFill(type.color);
        zoneGraphic.drawRect(0, 0, CELL_SIZE - 2, CELL_SIZE - 2);
        zoneGraphic.endFill();
        zoneGraphic.x = x * CELL_SIZE;
        zoneGraphic.y = y * CELL_SIZE;
        zoneGraphic.actualX = x;
        zoneGraphic.actualY = y;
        app.stage.addChild(zoneGraphic);
        return { x, y, type, graphic: zoneGraphic };
    }

    function generateZones() {
        zones.forEach(zone => app.stage.removeChild(zone.graphic));
        zones = [];

        const numZones = Math.floor(Math.random() * 4) + 2; // 2-5 zones
        for (let i = 0; i < numZones; i++) {
            const x = Math.floor(Math.random() * GRID_SIZE);
            const y = Math.floor(Math.random() * GRID_SIZE);
            const type = Math.random() < 0.5 ? ZONE_TYPES.SLOW : ZONE_TYPES.FAST;

            if (!snake.some(segment => segment.x === x && segment.y === y) && (x !== food.x || y !== food.y)) {
                zones.push(createZone(x, y, type));
            }
        }
    }

function checkZones() {
const head = snake[0];

        // Find the zone the snake's head is in, if any
        zones = zones.filter(zone => {
            if (zone.x === head.x && zone.y === head.y) {
                MOVE_INTERVAL = BASE_MOVE_INTERVAL / zone.type.multiplier;

                // Reset speed after 3 seconds
                setTimeout(() => {
                    MOVE_INTERVAL = BASE_MOVE_INTERVAL / speedMultiplier;
                }, 3000);

                // Remove the zone from the stage
                app.stage.removeChild(zone.graphic);
                return false; // Remove this zone from the array
            }
            return true;
        });
    }

    // Handle keyboard input
    document.addEventListener('keydown', (e) => {
        if (gameState !== 'playing') return;

        const key = e.key;
        if (key === 'ArrowUp' && direction.y !== 1) nextDirection = {x: 0, y: -1};
        if (key === 'ArrowDown' && direction.y !== -1) nextDirection = {x: 0, y: 1};
        if (key === 'ArrowLeft' && direction.x !== 1) nextDirection = {x: -1, y: 0};
        if (key === 'ArrowRight' && direction.x !== -1) nextDirection = {x: 1, y: 0};
        if (key === 'Escape' || key === ' ') togglePause();
    });

    // Game loop
    app.ticker.add((delta) => {
        if (gameState !== 'playing') return;

        const currentTime = Date.now();

        if (currentTime - lastMoveTime >= MOVE_INTERVAL) {
            direction = nextDirection;

            let newHead = {
                x: snake[0].x + direction.x,
                y: snake[0].y + direction.y
            };

            // Handle walls
            if (wallsEnabled) {
                if (newHead.x < 0 || newHead.x >= GRID_SIZE || newHead.y < 0 || newHead.y >= GRID_SIZE) {
                    showGameOverMenu();
                    return;
                }
            } else {
                newHead.x = (newHead.x + GRID_SIZE) % GRID_SIZE;
                newHead.y = (newHead.y + GRID_SIZE) % GRID_SIZE;
            }

            // Check collision with self
            if (snake.some(segment => segment.x === newHead.x && segment.y === newHead.y)) {
                showGameOverMenu();
                return;
            }

            snake.unshift(newHead);

            // Check for food collision
            if (newHead.x === food.x && newHead.y === food.y) {
                snakeSegments.push(createSnakeSegment(snake[snake.length - 1].x, snake[snake.length - 1].y));
                generateFood();
                generateZones(); // Regenerate zones when food is eaten
                score += 10;
                scoreElement.textContent = `Score: ${score}`;
            } else {
                snake.pop();
            }

            snake.forEach((pos, index) => {
                if (snakeSegments[index]) {
                    snakeSegments[index].actualX = pos.x;
                    snakeSegments[index].actualY = pos.y;
                }
            });

            checkZones(); // Check if the snake is in a slow/fast zone

            lastMoveTime = currentTime;
        }

        // Smooth movement interpolation
        snakeSegments.forEach((segment) => {
            segment.x += (segment.actualX * CELL_SIZE - segment.x) * SMOOTHNESS * delta;
            segment.y += (segment.actualY * CELL_SIZE - segment.y) * SMOOTHNESS * delta;
        });
    });

    showMainMenu();

</script>

</body>
