
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Flapper</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: linear-gradient(135deg, #ff7e40, #ff0a00);
      font-family: Arial, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }
    canvas {
      display: none;
    }
    #startScreen, #gameOverScreen, #settingsScreen {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }
    #playButton, #replayButton, #settingsButton, #backButton, #resumeButton {
      padding: 15px 30px;
      font-size: 20px;
      color: white;
      background-color: #228B22;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      transition: background-color 0.3s ease;
      margin: 10px;
    }
    #playButton:hover, #replayButton:hover, #settingsButton:hover, #backButton:hover, #resumeButton:hover {
      background-color: #ff0a00;
    }
    #title, #gameOverText, #authorText {
      font-size: 50px;
      margin-bottom: 20px;
      color: #FFFFFF;
      text-shadow: 2px 2px #000;
    }
    #gameOverScreen, #settingsScreen {
      display: none;
    }
    #settingsModal {
      background-color: #fff;
      border-radius: 10px;
      padding: 20px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      z-index: 1000;
    }
    #settingsModal label {
      margin-bottom: 10px;
      font-size: 18px;
      color: #333;
    }
    #settingsModal input[type="range"] {
      width: 300px;
      margin-bottom: 20px;
    }
    #pauseButton {
      position: absolute;
      top: 20px;
      right: 20px;
      padding: 10px;
      background-color: rgba(0, 0, 0, 0.7);
      color: white;
      font-size: 24px;
      border: none;
      border-radius: 50%;
      cursor: pointer;
    }
    #pauseButton:hover {
      background-color: rgba(0, 0, 0, 0.9);
    }
    #resumeButton {
      display: none;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      padding: 15px 30px;
      font-size: 20px;
      color: white;
      background-color: #ff0a00;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      transition: background-color 0.3s ease;
    }
    #resumeButton:hover {
      background-color: #1e7a1e;
    }
    #funFact {
      font-size: 20px;
      color: #fff;
      text-align: center;
      margin-top: 20px;
      max-width: 80%;
      text-shadow: 1px 1px #000;
    }
    #gamesPlayed {
      font-size: 20px;
      color: #fff;
      text-align: center;
      margin-top: 10px;
      text-shadow: 1px 1px #000;
    }
  </style>
</head>
<body>
  <div id="startScreen">
    <div id="title">Flapper</div>
    <button id="playButton">Play</button>
    <button id="settingsButton">Settings</button>
    <div id="authorText">by Abhinav</div>
    <div id="gamesPlayed">Games Played: 0</div> <!-- Display games played -->
  </div>

  <div id="settingsScreen">
    <div id="settingsModal">
      <label for="jumpStrength">Jump Strength: <span id="jumpValue"></span></label>
      <input id="jumpStrength" type="range" min="-15" max="-5" value="-8">
      <label for="gameSpeed">Game Speed: <span id="speedValue"></span></label>
      <input id="gameSpeed" type="range" min="1" max="5" value="2">
    </div>
    <button id="backButton">Back</button>
  </div>

  <div id="gameOverScreen">
    <div id="gameOverText">Game Over</div>
    <div id="finalScore"></div>
    <div id="highScore"></div>
    <div id="funFact"></div> <!-- New element for fun facts -->
    <button id="replayButton">Replay</button>
    <button id="settingsAfterGameOverButton">Settings</button>
    <div id="authorText">by Abhinav</div>
    <div id="gamesPlayedGameOver">Games Played: 0</div> <!-- Display games played -->
  </div>

  <canvas id="gameCanvas"></canvas>
  <button id="pauseButton" style="display: none;">⏸️</button>
  <button id="resumeButton">Resume</button>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const startScreen = document.getElementById("startScreen");
    const settingsScreen = document.getElementById("settingsScreen");
    const gameOverScreen = document.getElementById("gameOverScreen");
    const playButton = document.getElementById("playButton");
    const settingsButton = document.getElementById("settingsButton");
    const backButton = document.getElementById("backButton");
    const replayButton = document.getElementById("replayButton");
    const jumpStrengthSlider = document.getElementById("jumpStrength");
    const gameSpeedSlider = document.getElementById("gameSpeed");
    const jumpValueDisplay = document.getElementById("jumpValue");
    const speedValueDisplay = document.getElementById("speedValue");
    const finalScoreDisplay = document.getElementById("finalScore");
    const highScoreDisplay = document.getElementById("highScore");
    const pauseButton = document.getElementById("pauseButton");
    const resumeButton = document.getElementById("resumeButton");
    const settingsAfterGameOverButton = document.getElementById("settingsAfterGameOverButton");
    const funFactDisplay = document.getElementById("funFact"); // New element for fun facts
    const gamesPlayedDisplay = document.getElementById("gamesPlayed"); // Games played on start screen
    const gamesPlayedGameOverDisplay = document.getElementById("gamesPlayedGameOver"); // Games played on game over screen

    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    let birdJump = parseFloat(jumpStrengthSlider.value);
    let pipeSpeed = parseFloat(gameSpeedSlider.value);

    const bird = {
      x: 50,
      y: canvas.height / 2,
      size: 40,
      gravity: 0.3,
      lift: birdJump,
      velocity: 0,
      emoji: "😃",
    };

    const pipes = [];
    const pipeWidth = 60;
    const pipeGap = 200;
    let score = 0;
    let highScore = localStorage.getItem("highScore") || 0;
    let gameRunning = false;
    let gamePaused = false;

    // Initialize games played counter
    let gamesPlayed = localStorage.getItem("gamesPlayed") || 0;
    gamesPlayedDisplay.textContent = Games Played: ${gamesPlayed};
    gamesPlayedGameOverDisplay.textContent = Games Played: ${gamesPlayed};

    // Array of fun facts
    const funFacts = [
      "Honey never spoils. Archaeologists have found pots of honey in ancient Egyptian tombs that are over 3,000 years old and still edible!",
      "Octopuses have three hearts. Two pump blood to the gills, and one pumps it to the rest of the body.",
      "Bananas are berries, but strawberries aren't!",
      "The Eiffel Tower can be 15 cm taller during the summer due to thermal expansion.",
      "A day on Venus is longer than a year on Venus.",
      "Wombat poop is cube-shaped to prevent it from rolling away.",
      "The shortest war in history was between Zanzibar and England in 1896. Zanzibar surrendered after 38 minutes.",
      "Cows have best friends and get stressed when they are separated.",
      "The heart of a blue whale is the size of a small car.",
      "Polar bears have black skin under their white fur to better absorb the sun's warmth.",
      "The inventor of the Pringles can is buried in one.",
      "A group of flamingos is called a 'flamboyance.'",
      "Sloths can hold their breath longer than dolphins—up to 40 minutes!",
      "The dot over the letter 'i' is called a tittle.",
      "The longest hiccuping spree lasted 68 years.",
      "A shrimp's heart is in its head.",
      "The first oranges weren’t orange—they were green.",
      "The longest time between two twins being born is 87 days.",
      "A cloud can weigh over a million pounds.",
      "The world's oldest piece of chewing gum is over 9,000 years old.",
      "The longest English word without a vowel is 'rhythms.'",
      "A day on Mercury lasts 1,408 hours.",
      "The world's largest snowflake was 15 inches wide.",
      "The first computer mouse was made of wood.",
      "The longest wedding veil was longer than 63 football fields.",
      "The first alarm clock could only ring at 4 a.m.",
      "The longest recorded flight of a chicken is 13 seconds.",
      "The world's smallest reptile is smaller than a dime.",
      "The longest hiccuping spree lasted 68 years.",
      "The first oranges weren’t orange—they were green.",
      "The longest time between two twins being born is 87 days.",
      "A cloud can weigh over a million pounds.",
      "The world's oldest piece of chewing gum is over 9,000 years old.",
      "The longest English word without a vowel is 'rhythms.'",
      "A day on Mercury lasts 1,408 hours.",
      "The world's largest snowflake was 15 inches wide.",
      "The first computer mouse was made of wood.",
      "The longest wedding veil was longer than 63 football fields.",
      "The first alarm clock could only ring at 4 a.m.",
      "The longest recorded flight of a chicken is 13 seconds.",
      "The world's smallest reptile is smaller than a dime.",
      "The longest hiccuping spree lasted 68 years.",
      "The first oranges weren’t orange—they were green.",
      "The longest time between two twins being born is 87 days.",
      "A cloud can weigh over a million pounds.",
      "The world's oldest piece of chewing gum is over 9,000 years old.",
      "The longest English word without a vowel is 'rhythms.'",
      "A day on Mercury lasts 1,408 hours.",
      "The world's largest snowflake was 15 inches wide.",
      "The first computer mouse was made of wood.",
    ];

    // Function to get a random fact
    function getRandomFact() {
      return funFacts[Math.floor(Math.random() * funFacts.length)];
    }

    function createPipe() {
      const pipeTopHeight = Math.random() * (canvas.height - pipeGap - 50) + 20;
      pipes.push({
        x: canvas.width,
        top: pipeTopHeight,
        bottom: canvas.height - pipeTopHeight - pipeGap,
      });
    }

    function gameLoop() {
      if (!gameRunning) {
        endGame();
        return;
      }

      if (!gamePaused) {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        bird.velocity += bird.gravity;
        bird.y += bird.velocity;

        drawBird();
        drawPipes();
        drawScore();

        updatePipes();
        checkCollision();

        requestAnimationFrame(gameLoop);
      }
    }

    function resetGame() {
      bird.y = canvas.height / 3;
      bird.velocity = 0;
      pipes.length = 0;
      score = 0;
      gameRunning = true;
      gamePaused = false;
      createPipe();
      canvas.style.display = "block";
      gameOverScreen.style.display = "none";
      pauseButton.style.display = "block";
      resumeButton.style.display = "none";
      bird.lift = birdJump;
      gameLoop();
    }

    function endGame() {
      canvas.style.display = "none";
      gameOverScreen.style.display = "flex";
      pauseButton.style.display = "none";

      // Increment games played counter
      gamesPlayed++;
      localStorage.setItem("gamesPlayed", gamesPlayed);
      gamesPlayedDisplay.textContent = Games Played: ${gamesPlayed};
      gamesPlayedGameOverDisplay.textContent = Games Played: ${gamesPlayed};

      if (score > highScore) {
        highScore = score;
        localStorage.setItem("highScore", highScore);
      }

      finalScoreDisplay.textContent = 💯: ${score};
      highScoreDisplay.textContent = 🏆: ${highScore};
      funFactDisplay.textContent = Did you know? ${getRandomFact()}; // Display a random fact
    }

    function drawBird() {
      ctx.font = ${bird.size}px Arial;
      ctx.fillText(bird.emoji, bird.x - bird.size / 2, bird.y + bird.size / 2);
    }

    function drawPipes() {
      pipes.forEach(pipe => {
        ctx.fillStyle = "#228B22";
        ctx.shadowBlur = 10;
        ctx.shadowColor = "rgba(0, 0, 0, 0.5)";
        ctx.fillRect(pipe.x, 0, pipeWidth, pipe.top);
        ctx.fillRect(pipe.x, canvas.height - pipe.bottom, pipeWidth, pipe.bottom);
      });
    }

    function updatePipes() {
      pipes.forEach(pipe => {
        pipe.x -= pipeSpeed;

        if (pipe.x + pipeWidth < 0) {
          pipes.shift();
          score++;
        }
      });

      if (pipes.length === 0 || pipes[pipes.length - 1].x < canvas.width - 300) {
        createPipe();
      }
    }

    function checkCollision() {
      if (bird.y + bird.size > canvas.height || bird.y - bird.size < 0) {
        gameRunning = false;
      }

      pipes.forEach(pipe => {
        if (
          bird.x + bird.size / 2 > pipe.x &&
          bird.x - bird.size / 2 < pipe.x + pipeWidth &&
          (bird.y - bird.size / 2 < pipe.top ||
            bird.y + bird.size / 2 > canvas.height - pipe.bottom)
        ) {
          gameRunning = false;
        }
      });
    }

    function drawScore() {
      ctx.fillStyle = "#000";
      ctx.font = "30px Arial";
      ctx.fillText(💯: ${score}, 20, 40);
      ctx.fillText(🏆: ${highScore}, 20, 80);
    }

    function togglePause() {
      gamePaused = !gamePaused;

      if (gamePaused) {
        pauseButton.style.display = "none";
        resumeButton.style.display = "block";
      } else {
        pauseButton.style.display = "block";
        resumeButton.style.display = "none";
        gameLoop();
      }
    }

    // Event Listeners
    playButton.addEventListener("click", () => {
      startScreen.style.display = "none";
      resetGame();
    });

    settingsButton.addEventListener("click", () => {
      startScreen.style.display = "none";
      settingsScreen.style.display = "flex";
    });

    backButton.addEventListener("click", () => {
      settingsScreen.style.display = "none";
      startScreen.style.display = "flex";
    });

    replayButton.addEventListener("click", resetGame);

    settingsAfterGameOverButton.addEventListener("click", () => {
      gameOverScreen.style.display = "none";
      settingsScreen.style.display = "flex";
    });

    jumpStrengthSlider.addEventListener("input", () => {
      birdJump = parseFloat(jumpStrengthSlider.value);
      jumpValueDisplay.textContent = birdJump;
    });

    gameSpeedSlider.addEventListener("input", () => {
      pipeSpeed = parseFloat(gameSpeedSlider.value);
      speedValueDisplay.textContent = pipeSpeed;
    });

    document.addEventListener("click", () => {
      if (gameRunning && !gamePaused) bird.velocity = bird.lift;
    });

    document.addEventListener("keydown", event => {
      if (event.code === "Space" && gameRunning && !gamePaused) {
        bird.velocity = bird.lift;
      }
    });

    pauseButton.addEventListener("click", togglePause);
    resumeButton.addEventListener("click", togglePause);
  </script>
</body>
</html>
