<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <title>Rhythm Number Game</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <style>
        body{
            background:#111;
            color:#fff;
            font-family:Arial,sans-serif;
            text-align:center;
        }

        h1{
            margin-top:20px;
        }

        #mapping{
            margin:10px auto;
        }

        .key-box{
            display:inline-block;
            margin:5px;
            padding:10px 15px;
            border:1px solid #555;
            border-radius:5px;
        }

        #sequence{
            font-size:42px;
            margin:20px;
            letter-spacing:10px;
        }

        #status{
            margin:10px;
            font-size:22px;
        }

        #score{
            font-size:24px;
            color:#2ecc71;
            margin:15px;
        }

        button{
            margin:10px;
            padding:10px 20px;
            font-size:18px;
        }

        #game{
            display:none;
        }

        /* mobilne przyciski */
        #mobile-controls{
            margin-top:20px;
        }

        .mob-btn{
            margin:5px;
            padding:10px 15px;
            font-size:20px;
        }
    </style>
</head>
<body>

<div id="menu">
    <h1>Rhythm Number Game</h1>
    <button onclick="startGame()">GRAJ</button>
    <button onclick="startBattle()">WALKA</button>
    <button onclick="startRhythm()">RHYTHM</button>
</div>

<div id="game">
    <h1>Rhythm Number Game</h1>

    <div id="mapping">
        <div class="key-box">1 = W</div>
        <div class="key-box">2 = E</div>
        <div class="key-box">3 = A</div>
        <div class="key-box">4 = S</div>
        <div class="key-box">5 = D</div>
    </div>

    <div id="score">Punkty: 0</div>

    <div id="sequence"></div>
    <div id="status"></div>

    <button onclick="backToMenu()">MENU</button>

    <!-- mobilne przyciski pod iOS -->
    <div id="mobile-controls">
        <div>GAME / BATTLE (litery):</div>
        <button class="mob-btn" onclick="virtualKey('w')">W</button>
        <button class="mob-btn" onclick="virtualKey('e')">E</button>
        <button class="mob-btn" onclick="virtualKey('a')">A</button>
        <button class="mob-btn" onclick="virtualKey('s')">S</button>
        <button class="mob-btn" onclick="virtualKey('d')">D</button>

        <div style="margin-top:15px;">RHYTHM (cyfry):</div>
        <button class="mob-btn" onclick="virtualDigit('1')">1</button>
        <button class="mob-btn" onclick="virtualDigit('2')">2</button>
        <button class="mob-btn" onclick="virtualDigit('3')">3</button>
        <button class="mob-btn" onclick="virtualDigit('4')">4</button>
        <button class="mob-btn" onclick="virtualDigit('5')">5</button>
    </div>
</div>

<script>
var mode = "";
var score = 0;

var seqDiv = document.getElementById("sequence");
var statusDiv = document.getElementById("status");

function updateScore(){
    document.getElementById("score").innerHTML = "Punkty: " + score;
}

/* ======================
   MENU
====================== */

function startGame(){
    mode = "GAME";
    document.getElementById("menu").style.display = "none";
    document.getElementById("game").style.display = "block";
    score = 0;
    updateScore();
    generateSequence(4);
}

function startBattle(){
    mode = "BATTLE";
    document.getElementById("menu").style.display = "none";
    document.getElementById("game").style.display = "block";
    score = 0;
    updateScore();
    nextEnemyRound();
}

function startRhythm(){
    mode = "RHYTHM";
    document.getElementById("menu").style.display = "none";
    document.getElementById("game").style.display = "block";
    score = 0;
    updateScore();
    newRhythmRound();
}

function backToMenu(){
    document.getElementById("menu").style.display = "block";
    document.getElementById("game").style.display = "none";
    clearInterval(battleTimer);
}

/* ======================
   GAME (SEKWENCJE)
====================== */

var keyToNumber = { "w":1, "e":2, "a":3, "s":4, "d":5 };
var numberToLetter = { 1:"W", 2:"E", 3:"A", 4:"S", 5:"D" };

var currentSequence = [];
var currentIndex = 0;

function flashSequence(index,good){
    var html = "";
    for(var i=0;i<currentSequence.length;i++){
        var letter = numberToLetter[currentSequence[i]];
        if(i==index){
            html += "<span style='color:"+(good?"#2ecc71":"#e74c3c")+";font-weight:bold;'>"+letter+"</span> ";
        } else {
            html += letter + " ";
        }
    }
    seqDiv.innerHTML = html;
}

function generateSequence(length){
    currentSequence = [];
    for(var i=0;i<length;i++){
        currentSequence.push(Math.floor(Math.random()*5)+1);
    }
    currentIndex = 0;
    flashSequence(-1,true);
    statusDiv.innerHTML = "Powtórz sekwencję.";
}

function rhythmKey(key){
    if(!(key in keyToNumber)) return;

    var pressed = keyToNumber[key];
    var expected = currentSequence[currentIndex];

    if(pressed == expected){
        flashSequence(currentIndex,true);
        currentIndex++;

        if(currentIndex >= currentSequence.length){
            score++;
            updateScore();
            statusDiv.innerHTML = "Dobrze!";
            setTimeout(function(){ generateSequence(4); },700);
        }

    } else {
        statusDiv.innerHTML = "Błąd!";
        setTimeout(function(){ generateSequence(4); },700);
    }
}

/* ======================
   WALKA
====================== */

var enemyLetter = "";
var battleTimer = null;
var battleTime = 6;

function nextEnemyRound(){

    var letters = ["W","E","A","S","D"];

    enemyLetter = letters[Math.floor(Math.random()*5)];

    battleTime = 6;

    seqDiv.innerHTML = enemyLetter;
    statusDiv.innerHTML = "Masz 6 sekund";

    clearInterval(battleTimer);

    battleTimer = setInterval(function(){

        battleTime--;

        statusDiv.innerHTML = "Pozostało: " + battleTime + " s";

        if(battleTime <= 0){

            clearInterval(battleTimer);

            score--;
            updateScore();

            if(score <= -25){
                seqDiv.innerHTML = "";
                statusDiv.innerHTML = "PRZEGRAŁEŚ!";
                return;
            }

            nextEnemyRound();
        }

    },1000);
}

function battleKey(key){

    if(score <= -25) return;

    if(key.toUpperCase() == enemyLetter){

        clearInterval(battleTimer);

        score++;
        updateScore();

        statusDiv.innerHTML = "Dobrze!";

        setTimeout(nextEnemyRound,500);

    } else {

        clearInterval(battleTimer);

        score--;
        updateScore();

        if(score <= -25){
            seqDiv.innerHTML = "";
            statusDiv.innerHTML = "PRZEGRAŁEŚ!";
            return;
        }

        statusDiv.innerHTML = "Zły klawisz!";

        setTimeout(nextEnemyRound,500);
    }
}

/* ======================
   RHYTHM MODE (4 cyfry + podświetlenie)
====================== */

var rhythmSequence = "";
var rhythmIndex = 0;

function newRhythmRound(){
    rhythmSequence = generateRhythmSequence();
    rhythmIndex = 0;

    drawRhythmSequence();
    statusDiv.innerHTML = "Klikaj cyfry 1–5.";
}

function generateRhythmSequence(){
    var seq = "";

    for(var i=0;i<4;i++){
        seq += (Math.floor(Math.random()*5) + 1);
    }

    return seq;
}

function drawRhythmSequence(){
    var html = "";

    for(var i=0;i<rhythmSequence.length;i++){
        if(i < rhythmIndex){
            html += "<span style='color:#2ecc71;font-weight:bold;'>" + rhythmSequence[i] + "</span> ";
        } else {
            html += "<span>" + rhythmSequence[i] + "</span> ";
        }
    }

    seqDiv.innerHTML = html;
}

function rhythmPress(key){
    var pressed = key;
    var expected = rhythmSequence[rhythmIndex];

    if(pressed == expected){
        rhythmIndex++;
        drawRhythmSequence();

        if(rhythmIndex >= rhythmSequence.length){
            score++;
            updateScore();
            statusDiv.innerHTML = "Dobrze!";
            setTimeout(newRhythmRound,500);
        }

    } else {
        seqDiv.innerHTML = "<span style='color:#e74c3c;font-weight:bold;'>" + rhythmSequence + "</span>";
        statusDiv.innerHTML = "Błąd!";
        setTimeout(newRhythmRound,500);
    }
}

/* ======================
   KLAWISZE – wersja web (event.key)
====================== */

document.addEventListener("keydown", function(e){
    var key = e.key;

    if(mode == "GAME"){
        rhythmKey(key.toLowerCase());
    }
    else if(mode == "BATTLE"){
        battleKey(key);
    }
    else if(mode == "RHYTHM"){
        if("12345".indexOf(key) !== -1){
            rhythmPress(key);
        }
    }
});

/* mobilne przyciski */

function virtualKey(letter){
    if(mode == "GAME"){
        rhythmKey(letter.toLowerCase());
    } else if(mode == "BATTLE"){
        battleKey(letter.toUpperCase());
    }
}

function virtualDigit(d){
    if(mode == "RHYTHM"){
        rhythmPress(d);
    }
}
</script>

</body>
</html>
