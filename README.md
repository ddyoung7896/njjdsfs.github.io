<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>러너 게임</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
        }
        #gameCanvas {
            border: 1px solid black;
            background-color: white;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="800" height="400"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        const GROUND_HEIGHT = 50;
        const PLAYER_WIDTH = 40;
        const PLAYER_HEIGHT = 60;

        let player = {
            x: 50,
            y: canvas.height - GROUND_HEIGHT - PLAYER_HEIGHT,
            width: PLAYER_WIDTH,
            height: PLAYER_HEIGHT,
            jumping: false,
            ducking: false,
            jumpCount: 0,
            jumpHeight: 15,
            fallSpeed: 0,
            jumpLimit: 2
        };

        let obstacles = [];
        let score = 0;
        let gameOver = false;
        let gameSpeed = 5;

        function drawPlayer() {
            ctx.fillStyle = 'blue';
            if (player.ducking) {
                ctx.fillRect(player.x, player.y + player.height / 2, player.width, player.height / 2);
            } else {
                ctx.fillRect(player.x, player.y, player.width, player.height);
            }
        }

        function drawObstacles() {
            obstacles.forEach(obstacle => {
                ctx.fillStyle = 'red';
                ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
            });
        }

        function moveObstacles() {
            obstacles.forEach(obstacle => {
                obstacle.x -= gameSpeed;
            });

            obstacles = obstacles.filter(obstacle => obstacle.x + obstacle.width > 0);

            if (Math.random() < 0.02) {
                let obstacleType = Math.random();
                if (obstacleType < 0.4) {
                    // 낮은 장애물 (엎드려서 피하기)
                    obstacles.push({
                        x: canvas.width,
                        y: canvas.height - GROUND_HEIGHT - 30,
                        width: 30,
                        height: 30
                    });
                } else if (obstacleType < 0.8) {
                    // 높은 장애물 (점프해서 피하기)
                    obstacles.push({
                        x: canvas.width,
                        y: canvas.height - GROUND_HEIGHT - 60,
                        width: 30,
                        height: 60
                    });
                } else {
                    // 매우 높은 장애물 (2단 점프로 피하기)
                    obstacles.push({
                        x: canvas.width,
                        y: canvas.height - GROUND_HEIGHT - 90,
                        width: 30,
                        height: 90
                    });
                }
            }
        }

        function checkCollision() {
            return obstacles.some(obstacle => 
                player.x < obstacle.x + obstacle.width &&
                player.x + player.width > obstacle.x &&
                player.y < obstacle.y + obstacle.height &&
                player.y + player.height > obstacle.y
            );
        }

        function drawGround() {
            ctx.fillStyle = 'green';
            ctx.fillRect(0, canvas.height - GROUND_HEIGHT, canvas.width, GROUND_HEIGHT);
        }

        function drawScore() {
            ctx.fillStyle = 'black';
            ctx.font = '20px Arial';
            ctx.fillText(`점수: ${score}`, 10, 30);
        }

        function updatePlayer() {
            if (player.jumping) {
                player.y -= player.jumpHeight;
                player.fallSpeed += 0.8;
                player.y += player.fallSpeed;

                if (player.y > canvas.height - GROUND_HEIGHT - player.height) {
                    player.y = canvas.height - GROUND_HEIGHT - player.height;
                    player.jumping = false;
                    player.jumpCount = 0;
                    player.fallSpeed = 0;
                }
            }
        }

        function gameLoop() {
            if (gameOver) return;

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            drawGround();
            drawPlayer();
            drawObstacles();
            moveObstacles();
            updatePlayer();
            drawScore();

            if (checkCollision()) {
                gameOver = true;
                ctx.fillStyle = 'black';
                ctx.font = '30px Arial';
                ctx.fillText('게임 오버!', canvas.width / 2 - 70, canvas.height / 2);
                ctx.font = '20px Arial';
                ctx.fillText(`최종 점수: ${score}`, canvas.width / 2 - 60, canvas.height / 2 + 30);
                ctx.fillText('Space를 눌러 재시작', canvas.width / 2 - 100, canvas.height / 2 + 60);
            } else {
                score++;
                if (score % 500 === 0) gameSpeed += 0.5;
                requestAnimationFrame(gameLoop);
            }
        }

        document.addEventListener('keydown', (e) => {
            if (gameOver) {
                if (e.code === 'Space') {
                    player.y = canvas.height - GROUND_HEIGHT - player.height;
                    player.jumping = false;
                    player.ducking = false;
                    player.jumpCount = 0;
                    obstacles = [];
                    score = 0;
                    gameSpeed = 5;
                    gameOver = false;
                    gameLoop();
                }
            } else {
                if (e.code === 'Space' && player.jumpCount < player.jumpLimit) {
                    player.jumping = true;
                    player.jumpCount++;
                    player.fallSpeed = -10;
                } else if (e.code === 'ArrowDown') {
                    player.ducking = true;
                    player.height = PLAYER_HEIGHT / 2;
                    player.y = canvas.height - GROUND_HEIGHT - player.height;
                }
            }
        });

        document.addEventListener('keyup', (e) => {
            if (e.code === 'ArrowDown') {
                player.ducking = false;
                player.height = PLAYER_HEIGHT;
                player.y = canvas.height - GROUND_HEIGHT - player.height;
            }
        });

        gameLoop();
    </script>
</body>
</html>
