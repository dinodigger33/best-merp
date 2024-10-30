# better-merp
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Duck Avoidance Game</title>
    <style>
        body { margin: 0; overflow: hidden; }
        canvas { display: block; margin: auto; }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="1310" height="640"></canvas>
    <script>
        // Canvas and Context Setup
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        // Constants and Variables
        const SCREEN_WIDTH = 1310, SCREEN_HEIGHT = 640;
        const DUCK_WIDTH = 50, DUCK_HEIGHT = 50, OBJECT_SIZE = 50;
        const GRAVITY = 10, JUMP_STRENGTH = 15, MAX_SPEED = 20, FRICTION = 0.7;
        const INITIAL_OBJECT_FALL_SPEED = 5, MAX_FALL_SPEED = 50;
        let SPEED_MULTIPLIER = 1, spawnProbability = 0.03;

        let useBackgroundImage = false, gameOver = false, score = 0;
        let duck, objects = [], keysPressed = {};

        // Assets
        const backgroundMusic = new Audio('https://dl.sndup.net/tz582/Sparta%20Remix%20Extended%20-%20Instrumental%20version.mp3');
        backgroundMusic.loop = true;
        backgroundMusic.volume = 0.4;

        const frankfurtMusic = new Audio('https://dl.sndup.net/pw254/frak.mp3');
        frankfurtMusic.loop = true;
        frankfurtMusic.volume = 1.0;

        const backgroundImage = new Image();
        backgroundImage.src = 'https://raw.githubusercontent.com/Frankfurt-debug/Duck-Run/refs/heads/main/frankfurtbutwierd.jpg';

        const rainbowImage = new Image();
        rainbowImage.src = 'https://raw.githubusercontent.com/Frankfurt-debug/Duck-Run/refs/heads/main/frankfurtbox.jpg';

        // Game Initialization
        function initGame() {
            duck = { x: SCREEN_WIDTH / 2, y: SCREEN_HEIGHT - DUCK_HEIGHT - 10, velocityX: 0, isJumping: false, jumpVelocity: 0 };
            objects = [];
            score = 0;
            gameOver = false;
            useBackgroundImage = false;
            SPEED_MULTIPLIER = 1;
            spawnProbability = 0.03;
            keysPressed = {};
            backgroundMusic.currentTime = 0;
            backgroundMusic.play();
        }

        // Event Listeners
        document.addEventListener("keydown", (event) => {
            keysPressed[event.key] = true;
            if (event.key === " " && !duck.isJumping) {
                duck.isJumping = true;
                duck.jumpVelocity = JUMP_STRENGTH;
            }
        });

        document.addEventListener("keyup", (event) => keysPressed[event.key] = false);

        // Object Creation
        function addFallingObject() {
            const objectX = Math.random() * (SCREEN_WIDTH - OBJECT_SIZE);
            const isSpecial = Math.random() < 0.1;
            const isRainbow = Math.random() < 0.04;
            objects.push({ x: objectX, y: 0, width: OBJECT_SIZE, height: OBJECT_SIZE, isSpecial, isRainbow });
        }

        function triggerFrankfurtEffect() {
            if (backgroundImage.complete && frankfurtMusic.readyState === 4) {
                setTimeout(() => {
                    alert('frankfurt');
                    triggerFrankfurtEffect();
                }, 100);
            }
        }

        // Speed and Spawn Rate Adjustments
        function adjustGameDifficulty() {
            SPEED_MULTIPLIER += 0.0009;
            spawnProbability = Math.min(0.03 + SPEED_MULTIPLIER * 0.01, 0.1);
        }

        // Game Update
        function updateGame() {
            handleDuckMovement();
            handleObjectInteractions();
            adjustGameDifficulty();
        }

        // Handle Duck Movement and Boundaries
        function handleDuckMovement() {
            // Horizontal movement and friction
            if (keysPressed["ArrowLeft"] || keysPressed["a"]) duck.velocityX -= 0.5;
            if (keysPressed["ArrowRight"] || keysPressed["d"]) duck.velocityX += 0.5;

            duck.velocityX = Math.max(Math.min(duck.velocityX, MAX_SPEED), -MAX_SPEED);

            if (!(keysPressed["ArrowLeft"] || keysPressed["ArrowRight"] || keysPressed["a"] || keysPressed["d"])) {
                duck.velocityX *= FRICTION;
            }

            duck.x = Math.min(Math.max(duck.x + duck.velocityX, 0), SCREEN_WIDTH - DUCK_WIDTH);

            // Vertical movement and jumping
            if (duck.isJumping) {
                duck.y -= duck.jumpVelocity;
                duck.jumpVelocity -= GRAVITY;
                if (duck.y >= SCREEN_HEIGHT - DUCK_HEIGHT - 10) {
                    duck.y = SCREEN_HEIGHT - DUCK_HEIGHT - 10;
                    duck.isJumping = false;
                }
            }
        }

        // Handle Objects
        function handleObjectInteractions() {
            for (let i = objects.length - 1; i >= 0; i--) {
                const obj = objects[i];
                obj.y += Math.min(INITIAL_OBJECT_FALL_SPEED * SPEED_MULTIPLIER, MAX_FALL_SPEED);

                if (obj.y > SCREEN_HEIGHT) {
                    objects.splice(i, 1);
                    score++;
                } else if (isColliding(duck, obj)) {
                    if (obj.isSpecial) {
                        score += 10;
                        objects.splice(i, 1);
                    } else if (obj.isRainbow) {
                        useBackgroundImage = true;
                        backgroundMusic.pause();
                        frankfurtMusic.play();
                        triggerFrankfurtEffect();
                    } else {
                        endGame();
                    }
                }
            }
        }

        function isColliding(duck, obj) {
            return duck.x < obj.x + obj.width &&
                duck.x + DUCK_WIDTH > obj.x &&
                duck.y < obj.y + obj.height &&
                duck.y + DUCK_HEIGHT > obj.y;
        }

        // Game End and Reset
        function endGame() {
            gameOver = true;
            backgroundMusic.pause();
            frankfurtMusic.pause();
            alert("Game Over! Your score: " + score);
            initGame();
        }

        // Game Draw
        function drawGame() {
            ctx.clearRect(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT);
            if (useBackgroundImage) {
                ctx.drawImage(backgroundImage, 0, 0, SCREEN_WIDTH, SCREEN_HEIGHT);
            } else {
                ctx.fillStyle = "lightblue";
                ctx.fillRect(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT);
            }

            ctx.fillStyle = "yellow";
            ctx.fillRect(duck.x, duck.y, DUCK_WIDTH, DUCK_HEIGHT);

            objects.forEach(obj => {
                ctx.fillStyle = obj.isRainbow ? "purple" : obj.isSpecial ? "green" : "black";
                ctx.drawImage(obj.isRainbow ? rainbowImage : null, obj.x, obj.y, obj.width, obj.height);
            });

            ctx.fillStyle = "black";
            ctx.font = "20px Arial";
            ctx.fillText("Score: " + score, 10, 20);
        }

        // Game Loop
        function gameLoop() {
            if (!gameOver) {
                updateGame();
                drawGame();
                if (Math.random() < spawnProbability) addFallingObject();
                requestAnimationFrame(gameLoop);
            }
        }

        // Start Game
        initGame();
        gameLoop();
    </script>
</body>
</html>
