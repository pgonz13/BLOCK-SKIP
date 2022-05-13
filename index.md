## PROJECT OUTLINE

This project was born of an idea to create a game based on Mario Bros mechanics combined with Spelunky characters.
This game has been develop with P5.js (for graphics), Tone.js (for sound) and Arduino (for hardware).

### Markdown

Here's the JavaScript code for the skeleton of the game.

```markdown
var character = "playersprite.png";
var girl;
var gameState = "start";
var startButton;
var win;
var lives = 3;
var gameTime = 60;
var currObst = false;
var obst;
var cooldown;
var coolStart;
var themeMusic;
var winMusic;
var theEndMusic;

var serial;
var xcursval = 0;
var ycursval = 0;
var buttonCheck = false;
var outMessage = 1;

function preload() {
  girl = new Walker(character, 100, 220);
  obst = new Obstacle();
  
  themeMusic = new Tone.Player("audio files/gameMusic.mp3").toMaster();
  theEndMusic = new Tone.Player("audio files/theEndTheme.mp3").toDestination();
  winMusic = new Tone.Player("audio files/winnerTheme.mp3").toDestination();
}

function setup() {
  createCanvas(500, 300);
  imageMode(CENTER);
  textAlign(CENTER);
  setInterval(timer, 1000);

  serial = new p5.SerialPort();
  serial.open("COM3");
  serial.onData(gotData);

  serial.write(outMessage);
}

function gotData() {
  var currentString = trim(serial.readStringUntil("\r\n"));
  if (!currentString) {
    return;
  }
  console.log(currentString);
  var data = split(trim(currentString), ',');
  if (data.length < 3) {
    return;
  }

  xcursval = parseInt(map(data[0], 0, 1023, 0, width));
  ycursval = parseInt(map(data[1], 0, 1023, 0, height));

  if (data[2] == 0) {
    buttonCheck = true;
  } else {
    buttonCheck = false;
  }
}

function buttonClicked() {
  if (gameState == "playing") {
    girl.jump();
  }
  else if (gameState == "start") {
    startGame();
  }
}

function timer() {
  if (gameTime > 0 && gameState == "playing") {
    gameTime--;
  }
  obstProb();
}

function obstProb() {
  if (!currObst) {
    var prob = Math.floor(Math.random() * 10);
    if (prob >= 4 && gameTime < 58) {
      currObst = true;
    }
  }
}

function startGame() {
  themeMusic.start();
  gameState = "playing";
}

function endGame() {
  themeMusic.stop();
}

function draw() {

  if (buttonCheck) {
    buttonClicked();
  }

  if (gameState == "start") {
    background(50, 25, 25);

    textFont('Showcard Gothic');
    textSize(15);
    text('Press button to start', width/2, 195);

    fill(235, 155, 66);
    noStroke();
    circle(xcursval, ycursval, 7);

    textSize(40);
    fill(255, 235, 166);
    text('Block Skip!', width / 2, 100);
  }

  else if (gameState == "playing") {
    background(255, 235, 166);
    fill(50, 25, 25);
    rect(0, 250, width, 50);

    textSize(15);

    if (gameTime >= 60) {
      text('Lives: ' + lives + ' ♥ ' , 50, 20);
      text('try not to step into the blocks', width / 2, 45);
      text('you got 3 lives and 1 minute!', width / 2, 60);
      text(gameTime / 60 + ":" + gameTime % 60 + "0", 460, 20);
    }
    if (gameTime < 60 && gameTime >= 50) {
      text('Lives: ' + lives + ' ♥ ', 50, 20);
      text('try not to step into the blocks', width / 2, 45);
      text('you got 3 lives and 1 minute!', width / 2, 60);
      text("0:" + gameTime, 460, 20);
    }
    if (gameTime < 50 && gameTime >= 10) {
      text('Lives: ' + lives + ' ♥ ', 50, 20);
      text('try not to step into the blocks', width / 2, 45);
      text('you got 3 lives and 1 minute!', width / 2, 60);
      text("0:" + gameTime, 460, 20);
    }
    if (gameTime < 10) {
      text('Lives: ' + lives + ' ♥ ', 50, 20);
      text('try not to step into the blocks', width / 2, 45);
      text('you got 3 lives and 1 minute!', width / 2, 60);
      text('0:0' + gameTime, 460, 20);
    }
    if (gameTime == 0) {
      win = true;
      gameState = "over";
    }

    if (lives == 0) {
      win = false;
      gameState = "over";
    }

    if (cooldown) {
      outMessage = 0
      serial.write(outMessage)
      if (coolStart - 1 > gameTime) {
        cooldown = false;
        outMessage = 1
        serial.write(outMessage)
      }
    }

    girl.draw();

    if (currObst) {
      obst.draw();
      obst.checkCollision(girl.x, girl.y);
    }

  }

  else if (gameState == "over") {
    if(outMessage != 3) {
      endGame();
    }
    if (!win) {
      background(255, 166, 166);
      textFont('Showcard Gothic');

      theEndMusic.start();

      textSize(40);
      fill(50, 25, 25);
      text('You Loose :(', width / 2, 145);

      theEndMusic.start();
    }
    else {
      background(166, 255, 166);
      textFont('Showcard Gothic');

      winMusic.start();

      textSize(40);
      fill(50, 25, 25);
      text('You Win! :D', width / 2, 145);

      winMusic.start();
    }
  }
}

function Walker(imageName, x, y) {
  this.spriteSheet = loadImage(imageName);
  this.frame = 0;
  this.x = x;
  this.y = y;
  this.moving = 0;

  this.draw = function () {

    push();
    translate(this.x, this.y);

    if (this.moving == 0) {
      image(this.spriteSheet, 0, 0, 80, 80, 0, 0, 80, 80);
      console.log(this.y);
    }

    else if (this.moving == 1) {
      console.log("jumping up");
      console.log(this.y);

      for (var i = 0; i < 8; i++) {
        if (this.frame == i) {
          image(this.spriteSheet, 0, 0, 80, 80, 80 * i, 0, 80, 80);
        }
      }

      if (this.y <= 220 && this.y > 190) {
        this.frame = 1;
        this.y -= 3;
      }
      else if (this.y <= 190 && this.y > 160) {
        this.frame = 2;
        this.y -= 3;
      }
      else if (this.y <= 160 && this.y > 120) {
        this.frame = 3;
        this.y -= 3;
      }
      else if (this.y <= 120) {
        this.frame = 4;
        this.moving = 2;
      }
    }

    else if (this.moving == 2) {
      console.log("jumping down");
      console.log(this.y);

      for (var i = 0; i < 8; i++) {
        if (this.frame == i) {
          image(this.spriteSheet, 0, 0, 80, 80, 80 * i, 0, 80, 80);
        }
      }

      if (this.y >= 118 && this.y < 150) {
        this.frame = 4;
        this.y += 3;
      }
      else if (this.y >= 150 && this.y < 180) {
        this.frame = 5;
        this.y += 3;
      }
      else if (this.y >= 180 && this.y < 220) {
        this.frame = 6;
        this.y += 3;
      }
      else if (this.y >= 220) {
        this.frame = 7;
        this.moving = 0;
      }

    }

    pop();

    this.jump = function () {
      this.moving = 1;
    }

  }
}

function Obstacle() {
  this.x = 500;
  this.y = 230;
  this.width = 30;

  this.draw = function () {

    if (this.x > -30) {
      this.x -= 6;
      fill(255, 150, 0);
      rect(this.x, this.y, this.width, this.width);
    }
    else if (this.x < -30) {
      currObst = false;
      this.x = 500;
    }

  }

  this.checkCollision = function (x, y) {

    if (this.x < x + 30 && x - 30 < this.x && this.y < y + 40 && y - 40 < this.y && !cooldown) {
      cooldown = true;
      coolStart = gameTime;
      lives--;
    }
  }

}
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/pgonz13/Block-Skip/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
