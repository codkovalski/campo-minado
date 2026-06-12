<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Campo Minado</title>

<style>

*{
    margin:0;
    padding:0;
    box-sizing:border-box;
}

body{
    height:100vh;
    display:flex;
    justify-content:center;
    align-items:center;
    background:#1b1b1b;
    font-family:Arial, Helvetica, sans-serif;
    overflow:hidden;
}

.game{
    background:#2a2a2a;
    padding:20px;
    box-shadow:
    0 0 20px rgba(0,0,0,.5),
    inset 0 0 10px rgba(255,255,255,.05);
}

.topbar{
    display:flex;
    justify-content:space-between;
    align-items:center;
    margin-bottom:15px;
    background:#1f1f1f;
    padding:10px;
   
}

.info{
    color:white;
    font-size:22px;
    font-weight:bold;
    min-width:80px;
    text-align:center;
}

#restart{
    width:55px;
    height:55px;
    border:none;
    cursor:pointer;
    background:#3b3b3b;
    color:white;
    font-size:28px;
    transition:.2s;
}

#restart:hover{
    transform:scale(1.08);
    background:#4c4c4c;
}

#board{
    display:grid;
    grid-template-columns:repeat(10, 42px);
    gap:4px;
}

.cell{
    width:42px;
    height:42px;
    background:#3c3c3c;
    display:flex;
    justify-content:center;
    align-items:center;
    cursor:pointer;
    font-weight:bold;
    font-size:20px;
    color:white;
    transition:.15s;
    user-select:none;
}


.open{
    background:#bfbfbf;
    color:black;
}

.mine{
    background:#ff3d3d;
}

.flag{
    background:#ffb400;
}

.win{
    background:#35d07f;
}

.number1{ color:blue; }
.number2{ color:green; }
.number3{ color:red; }
.number4{ color:darkblue; }
.number5{ color:brown; }
.number6{ color:turquoise; }
.number7{ color:black; }
.number8{ color:gray; }

@media(max-width:600px){

    #board{
        grid-template-columns:repeat(10, 32px);
    }

    .cell{
        width:32px;
        height:32px;
        font-size:16px;
    }

}

</style>
</head>
<body>

<div class="game">

    <div class="topbar">

        <div class="info">
            💣 <span id="bombs">15</span>
        </div>

        <button id="restart">🙂</button>

        <div class="info">
            ⏱ <span id="timer">0</span>
        </div>

    </div>

    <div id="board"></div>

</div>

<script>

const rows = 10;
const cols = 10;
const minesCount = 15;

const board = document.getElementById("board");
const bombsText = document.getElementById("bombs");
const timerText = document.getElementById("timer");
const restartBtn = document.getElementById("restart");

let gameBoard = [];
let gameOver = false;
let timer = 0;
let timerInterval;

function startTimer(){

    clearInterval(timerInterval);

    timerInterval = setInterval(() => {
        timer++;
        timerText.textContent = timer;
    },1000);

}

function stopTimer(){
    clearInterval(timerInterval);
}

function createBoard(){

    gameBoard = [];
    board.innerHTML = "";
    gameOver = false;
    timer = 0;
    timerText.textContent = "0";
    bombsText.textContent = minesCount;
    restartBtn.textContent = "🙂";

    stopTimer();
    startTimer();

    for(let row = 0; row < rows; row++){

        gameBoard[row] = [];

        for(let col = 0; col < cols; col++){

            const cell = {
                mine:false,
                revealed:false,
                flagged:false,
                number:0,
                element:null
            };

            const div = document.createElement("div");
            div.classList.add("cell");

            div.addEventListener("click", () => revealCell(row,col));

            div.addEventListener("contextmenu", (e) => {
                e.preventDefault();
                toggleFlag(row,col);
            });

            board.appendChild(div);

            cell.element = div;

            gameBoard[row][col] = cell;

        }

    }

    placeMines();
    calculateNumbers();

}

function placeMines(){

    let placed = 0;

    while(placed < minesCount){

        const row = Math.floor(Math.random() * rows);
        const col = Math.floor(Math.random() * cols);

        if(!gameBoard[row][col].mine){

            gameBoard[row][col].mine = true;
            placed++;

        }

    }

}

function calculateNumbers(){

    for(let row = 0; row < rows; row++){

        for(let col = 0; col < cols; col++){

            if(gameBoard[row][col].mine) continue;

            let count = 0;

            for(let i = -1; i <= 1; i++){

                for(let j = -1; j <= 1; j++){

                    const newRow = row + i;
                    const newCol = col + j;

                    if(
                        newRow >= 0 &&
                        newRow < rows &&
                        newCol >= 0 &&
                        newCol < cols
                    ){

                        if(gameBoard[newRow][newCol].mine){
                            count++;
                        }

                    }

                }

            }

            gameBoard[row][col].number = count;

        }

    }

}

function revealCell(row,col){

    if(gameOver) return;

    const cell = gameBoard[row][col];

    if(cell.revealed || cell.flagged) return;

    cell.revealed = true;

    const div = cell.element;

    div.classList.add("open");

    if(cell.mine){

        div.classList.add("mine");
        div.textContent = "💣";

        gameLost();
        return;

    }

    if(cell.number > 0){

        div.textContent = cell.number;
        div.classList.add(`number${cell.number}`);

    }else{

        for(let i = -1; i <= 1; i++){

            for(let j = -1; j <= 1; j++){

                const newRow = row + i;
                const newCol = col + j;

                if(
                    newRow >= 0 &&
                    newRow < rows &&
                    newCol >= 0 &&
                    newCol < cols
                ){

                    revealCell(newRow,newCol);

                }

            }

        }

    }

    checkWin();

}

function toggleFlag(row,col){

    if(gameOver) return;

    const cell = gameBoard[row][col];

    if(cell.revealed) return;

    cell.flagged = !cell.flagged;

    if(cell.flagged){

        cell.element.textContent = "🚩";
        cell.element.classList.add("flag");

    }else{

        cell.element.textContent = "";
        cell.element.classList.remove("flag");

    }

}

function gameLost(){

    gameOver = true;

    stopTimer();

    restartBtn.textContent = "💀";

    for(let row = 0; row < rows; row++){

        for(let col = 0; col < cols; col++){

            const cell = gameBoard[row][col];

            if(cell.mine){

                cell.element.textContent = "💣";
                cell.element.classList.add("mine");

            }

        }

    }

}

function checkWin(){

    let revealedCells = 0;

    for(let row = 0; row < rows; row++){

        for(let col = 0; col < cols; col++){

            if(gameBoard[row][col].revealed){
                revealedCells++;
            }

        }

    }

    if(revealedCells === rows * cols - minesCount){

        gameOver = true;

        stopTimer();

        restartBtn.textContent = "😎";

        for(let row = 0; row < rows; row++){

            for(let col = 0; col < cols; col++){

                if(gameBoard[row][col].mine){

                    gameBoard[row][col].element.classList.add("win");
                    gameBoard[row][col].element.textContent = "💣";

                }

            }

        }

    }

}

restartBtn.addEventListener("click", createBoard);

createBoard();

</script>

</body>
</html>


//feito por ia, apenas para aprender mecher no github
