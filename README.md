<html>
<head>
    <title>Stick Figure Fighting Game</title>
    <style>
        body { margin: 0; overflow: hidden; background: #222; }
        canvas { display: block; background: #111; }
        .instructions {
            position: absolute;
            top: 10px;
            left: 10px;
            color: white;
            font-family: Arial, sans-serif;
            background: rgba(0,0,0,0.5);
            padding: 10px;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div class="instructions">
        Player 1: WASD to move, F to attack<br>
        Player 2: OKL; to move, J to attack
    </div>
    <canvas id="gameCanvas"></canvas>
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        canvas.width = 800;
        canvas.height = 600;

        class Player {
            constructor(x, y, color, keys) {
                this.x = x;
                this.y = y;
                this.width = 20;
                this.height = 40;
                this.health = 100;
                this.maxHealth = 100;
                this.color = color;
                this.keys = keys;
                this.speed = 5;
                this.attackCooldown = 500;
                this.attackTimer = 0;
                this.attackActive = false;
                this.attackDuration = 100;
            }

            update() {
                // Movement
                if (this.keys.w) this.y -= this.speed;
                if (this.keys.s) this.y += this.speed;
                if (this.keys.a) this.x -= this.speed;
                if (this.keys.d) this.x += this.speed;

                // Attack
                if (this.keys.f && !this.attackActive && this.attackTimer <= 0) {
                    this.attackActive = true;
                    this.attackTimer = this.attackCooldown;
                }

                // Reset attack timer
                if (this.attackTimer > 0) {
                    this.attackTimer -= 1;
                    if (this.attackTimer === 0) {
                        this.attackActive = false;
                    }
                }
            }

            draw() {
                // Head
                ctx.beginPath();
                ctx.arc(this.x, this.y - 20, 10, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.strokeStyle = 'black';
                ctx.stroke();

                // Body
                ctx.beginPath();
                ctx.moveTo(this.x, this.y - 20);
                ctx.lineTo(this.x, this.y + 20);
                ctx.stroke();

                // Arms (neutral)
                ctx.beginPath();
                ctx.moveTo(this.x - 10, this.y - 10);
                ctx.lineTo(this.x - 20, this.y - 5);
                ctx.stroke();
                ctx.beginPath();
                ctx.moveTo(this.x + 10, this.y - 10);
                ctx.lineTo(this.x + 20, this.y - 5);
                ctx.stroke();

                // Legs
                ctx.beginPath();
                ctx.moveTo(this.x, this.y + 20);
                ctx.lineTo(this.x - 10, this.y + 40);
                ctx.stroke();
                ctx.beginPath();
                ctx.moveTo(this.x, this.y + 20);
                ctx.lineTo(this.x + 10, this.y + 40);
                ctx.stroke();

                // Attack animation (extended arm)
                if (this.attackActive) {
                    if (this.x < 400) { // Player 1 (left side)
                        ctx.beginPath();
                        ctx.moveTo(this.x + 10, this.y - 10);
                        ctx.lineTo(this.x + 40, this.y - 5);
                        ctx.stroke();
                    } else { // Player 2 (right side)
                        ctx.beginPath();
                        ctx.moveTo(this.x - 10, this.y - 10);
                        ctx.lineTo(this.x - 40, this.y - 5);
                        ctx.stroke();
                    }
                }
            }
        }

        function drawHealthBar(player, x, y, width, height) {
            // Outline
            ctx.strokeStyle = 'black';
            ctx.strokeRect(x, y, width, height);
            
            // Health fill
            const healthWidth = (player.health / player.maxHealth) * width;
            ctx.fillStyle = player.health > 50 ? 'green' : (player.health > 20 ? 'yellow' : 'red');
            ctx.fillRect(x, y, healthWidth, height);
        }

        // Initialize players
        const player1 = new Player(150, 300, 'red', { w: false, a: false, s: false, d: false, f: false });
        const player2 = new Player(650, 300, 'blue', { o: false, k: false, l: false, semicolon: false, j: false });

        // Key event listeners
        document.addEventListener('keydown', (e) => {
            if (e.key === 'w') player1.keys.w = true;
            if (e.key === 'a') player1.keys.a = true;
            if (e.key === 's') player1.keys.s = true;
            if (e.key === 'd') player1.keys.d = true;
            if (e.key === 'f') player1.keys.f = true;
            
            if (e.key === 'o') player2.keys.w = true;
            if (e.key === 'k') player2.keys.a = true;
            if (e.key === 'l') player2.keys.s = true;
            if (e.key === ';') player2.keys.d = true;
            if (e.key === 'j') player2.keys.f = true;
        });

        document.addEventListener('keyup', (e) => {
            if (e.key === 'w') player1.keys.w = false;
            if (e.key === 'a') player1.keys.a = false;
            if (e.key === 's') player1.keys.s = false;
            if (e.key === 'd') player1.keys.d = false;
            if (e.key === 'f') player1.keys.f = false;
            
            if (e.key === 'o') player2.keys.w = false;
            if (e.key === 'k') player2.keys.a = false;
            if (e.key === 'l') player2.keys.s = false;
            if (e.key === ';') player2.keys.d = false;
            if (e.key === 'j') player2.keys.f = false;
        });

        // Game loop
        function gameLoop(timestamp) {
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Update players
            player1.update();
            player2.update();
            
            // Collision detection (attack range)
            const attackRange = 40;
            if (player1.attackActive && Math.abs(player1.x - player2.x) < attackRange) {
                player2.health -= 10;
            }
            if (player2.attackActive && Math.abs(player1.x - player2.x) < attackRange) {
                player1.health -= 10;
            }
            
            // Draw health bars
            drawHealthBar(player1, 50, 20, 200, 20);
            drawHealthBar(player2, 550, 20, 200, 20);
            
            // Draw players
            player1.draw();
            player2.draw();
            
            // Check for game over
            if (player1.health <= 0 || player2.health <= 0) {
                ctx.fillStyle = 'white';
                ctx.font = '48px Arial';
                ctx.textAlign = 'center';
                ctx.fillText(
                    player1.health <= 0 ? 'Player 2 Wins!' : 'Player 1 Wins!',
                    canvas.width / 2,
                    canvas.height / 2
                );
                return;
            }
            
            requestAnimationFrame(gameLoop);
        }
        
        // Start game
        requestAnimationFrame(gameLoop);
    </script>
</body>
</html>
