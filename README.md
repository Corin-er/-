<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Simple Pong Game</title>
  <style>
    body {
      background: #222;
      margin: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }
    #pongCanvas {
      background: #111;
      display: block;
      border: 2px solid #27ae60;
      border-radius: 8px;
      box-shadow: 0 0 20px #27ae60;
    }
    #score {
      color: #fff;
      font-family: monospace;
      text-align: center;
      font-size: 28px;
      margin-bottom: 15px;
      letter-spacing: 2px;
    }
  </style>
</head>
<body>
  <div>
    <div id="score">0 : 0</div>
    <canvas id="pongCanvas" width="700" height="400"></canvas>
  </div>
  <script>
    // ====== Setup
    const canvas = document.getElementById('pongCanvas');
    const ctx = canvas.getContext('2d');
    const scoreDiv = document.getElementById('score');

    // Game Constants
    const WIDTH = canvas.width;
    const HEIGHT = canvas.height;
    const PADDLE_WIDTH = 10, PADDLE_HEIGHT = 75, BALL_RADIUS = 8;
    const PADDLE_MARGIN = 10;
    const BALL_SPEED = 5;
    let leftScore = 0, rightScore = 0;

    // Paddles & Ball
    let leftPaddle = {
      x: PADDLE_MARGIN,
      y: HEIGHT/2 - PADDLE_HEIGHT/2,
      dy: 0
    };
    let rightPaddle = {
      x: WIDTH-PADDLE_WIDTH-PADDLE_MARGIN,
      y: HEIGHT/2 - PADDLE_HEIGHT/2,
      dy: 0
    };
    let ball = {
      x: WIDTH/2,
      y: HEIGHT/2,
      dx: BALL_SPEED * (Math.random()>0.5?1:-1),
      dy: (Math.random() * 2 - 1) * BALL_SPEED
    };
    let upPressed = false, downPressed = false;

    // ====== Input handling
    document.addEventListener('keydown', e => {
      if (e.key === "ArrowUp") upPressed = true;
      if (e.key === "ArrowDown") downPressed = true;
    });
    document.addEventListener('keyup', e => {
      if (e.key === "ArrowUp") upPressed = false;
      if (e.key === "ArrowDown") downPressed = false;
    });
    // Mouse control
    canvas.addEventListener('mousemove', function(e) {
      let rect = canvas.getBoundingClientRect();
      let mouseY = e.clientY - rect.top;
      leftPaddle.y = mouseY - PADDLE_HEIGHT/2;
      // Clamp within canvas
      leftPaddle.y = Math.max(0, Math.min(HEIGHT-PADDLE_HEIGHT, leftPaddle.y));
    });

    // ====== Game logic
    function resetBall(direction) {
      ball.x = WIDTH/2;
      ball.y = HEIGHT/2;
      ball.dx = BALL_SPEED * direction;
      ball.dy = (Math.random() * 2 - 1) * BALL_SPEED;
    }

    function drawPaddle(p) {
      ctx.fillStyle = '#27ae60';
      ctx.fillRect(p.x, p.y, PADDLE_WIDTH, PADDLE_HEIGHT);
    }
    function drawBall() {
      ctx.beginPath();
      ctx.arc(ball.x, ball.y, BALL_RADIUS, 0, 2*Math.PI);
      ctx.fillStyle = '#fff';
      ctx.fill();
      ctx.closePath();
    }
    function drawNet() {
      ctx.strokeStyle = "#27ae60";
      ctx.setLineDash([8, 12]);
      ctx.beginPath();
      ctx.moveTo(WIDTH/2, 0);
      ctx.lineTo(WIDTH/2, HEIGHT);
      ctx.stroke();
      ctx.setLineDash([]);
    }
    function draw() {
      ctx.clearRect(0, 0, WIDTH, HEIGHT);
      drawNet();
      drawPaddle(leftPaddle);
      drawPaddle(rightPaddle);
      drawBall();
    }

    function moveLeftPaddle() {
      if (upPressed) {
        leftPaddle.y -= 6;
      }
      if (downPressed) {
        leftPaddle.y += 6;
      }
      // Clamp
      leftPaddle.y = Math.max(0, Math.min(HEIGHT-PADDLE_HEIGHT, leftPaddle.y));
    }
    function moveRightPaddle() {
      // Simple AI: follows ball with max speed
      let target = ball.y - PADDLE_HEIGHT/2;
      if (rightPaddle.y < target - 4) rightPaddle.y += 4;
      else if (rightPaddle.y > target + 4) rightPaddle.y -= 4;
      // Clamp
      rightPaddle.y = Math.max(0, Math.min(HEIGHT-PADDLE_HEIGHT, rightPaddle.y));
    }

    function updateBall() {
      ball.x += ball.dx;
      ball.y += ball.dy;

      // Collide top/bottom wall
      if (ball.y - BALL_RADIUS <= 0 || ball.y + BALL_RADIUS >= HEIGHT) {
        ball.dy *= -1;
      }

      // Collide left paddle
      if (ball.x - BALL_RADIUS <= leftPaddle.x + PADDLE_WIDTH) {
        if (ball.y > leftPaddle.y && ball.y < leftPaddle.y + PADDLE_HEIGHT) {
          ball.dx *= -1;
          // Add "angle"
          let deltaY = ball.y - (leftPaddle.y + PADDLE_HEIGHT/2);
          ball.dy = deltaY * 0.2;
        } else if (ball.x - BALL_RADIUS < 0) {
          // Score for right/computer
          rightScore++; updateScore();
          resetBall(1);
        }
      }

      // Collide right paddle
      if (ball.x + BALL_RADIUS >= rightPaddle.x) {
        if (ball.y > rightPaddle.y && ball.y < rightPaddle.y + PADDLE_HEIGHT) {
          ball.dx *= -1;
          // Add "angle"
          let deltaY = ball.y - (rightPaddle.y + PADDLE_HEIGHT/2);
          ball.dy = deltaY * 0.2;
        } else if (ball.x + BALL_RADIUS > WIDTH) {
          // Score for player
          leftScore++; updateScore();
          resetBall(-1);
        }
      }
    }
    function updateScore() {
      scoreDiv.textContent = leftScore + " : " + rightScore;
    }

    // ====== Main game loop
    function gameLoop() {
      moveLeftPaddle();
      moveRightPaddle();
      updateBall();
      draw();
      requestAnimationFrame(gameLoop);
    }
    updateScore();
    gameLoop();
  </script>
</body>
</html>
