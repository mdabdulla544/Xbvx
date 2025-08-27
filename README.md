<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>🎮 Tic Tac Toe — Unbeatable AI</title>
<style>
:root {
  --bg1: #050a20; --bg2:#000;
  --x-color:#ff3d00; --o-color:#00e5ff; --accent:#ff00ff;
  --cell-size:90px;
}
*{box-sizing:border-box;margin:0;padding:0;font-family:Arial,sans-serif;}
body{height:100vh;display:flex;justify-content:center;align-items:center;overflow:hidden;background:var(--bg1);}
canvas#bgCanvas{position:absolute;top:0;left:0;width:100%;height:100%;z-index:-1;}
#startScreen, #gameWrap{display:flex;flex-direction:column;align-items:center;justify-content:center;gap:20px;}
#startScreen h1, #gameWrap h1{color:white;font-size:2rem;text-align:center;}
.btn{padding:12px 24px;border-radius:25px;border:2px solid white;background:transparent;color:white;font-size:1rem;cursor:pointer;transition:0.3s;outline:none;}
.btn:hover{background:white;color:#050a20;transform:scale(1.1);}
.board{display:grid;grid-template-columns:repeat(3,var(--cell-size));grid-template-rows:repeat(3,var(--cell-size));gap:12px;background:rgba(255,255,255,0.05);padding:12px;border-radius:20px;margin-top:20px;position:relative;}
.cell{background:rgba(255,255,255,0.02);border-radius:12px;display:flex;align-items:center;justify-content:center;font-size:2.5rem;color:white;cursor:pointer;transition:0.2s;position:relative;}
.cell:hover{transform:scale(1.1);}
.neon-x{color:var(--x-color); text-shadow:0 0 6px var(--x-color),0 0 12px var(--x-color);}
.neon-o{color:var(--o-color); text-shadow:0 0 6px var(--o-color),0 0 12px var(--o-color);}
#status{color:white;margin-top:15px;font-size:1.2rem;text-align:center;}
.win-line{position:absolute;height:8px;background:linear-gradient(90deg,#ff00ff,#00ffff);border-radius:999px;pointer-events:none;opacity:0;transition:opacity .3s;transform-origin:center center;}
</style>
</head>
<body>
<canvas id="bgCanvas"></canvas>
<div id="startScreen">
  <h1>🎮 Tic Tac Toe — Vin Start</h1>
  <button class="btn" id="startBtn">Start Game</button>
</div>

<div id="gameWrap">
  <h1>Tic Tac Toe</h1>
  <div class="board" id="board"></div>
  <h2 id="status">Player X's Turn</h2>
  <button class="btn" id="restartBtn">Restart</button>
</div>

<audio id="bgMusic" loop>
  <source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" type="audio/mpeg">
</audio>
<audio id="clickSound">
  <source src="https://www.soundjay.com/button/sounds/button-16.mp3" type="audio/mpeg">
</audio>
<audio id="winSound">
  <source src="https://www.soundjay.com/misc/sounds/bell-ringing-05.mp3" type="audio/mpeg">
</audio>

<script>
// Background particles
const canvas = document.getElementById('bgCanvas');
const ctx = canvas.getContext('2d');
let particles=[];
function resizeCanvas(){canvas.width=window.innerWidth; canvas.height=window.innerHeight;}
window.addEventListener('resize',resizeCanvas); resizeCanvas();
function initParticles(){particles=[]; for(let i=0;i<80;i++){particles.push({x:Math.random()*canvas.width,y:Math.random()*canvas.height,r:Math.random()*2+1,dx:(Math.random()-0.5)*0.5,dy:(Math.random()-0.5)*0.5});}}
function drawParticles(){ctx.clearRect(0,0,canvas.width,canvas.height); particles.forEach(p=>{ctx.beginPath();ctx.arc(p.x,p.y,p.r,0,Math.PI*2);ctx.fillStyle='rgba(255,255,255,0.05)';ctx.fill();p.x+=p.dx;p.y+=p.dy;if(p.x<0)p.x=canvas.width;if(p.x>canvas.width)p.x=0;if(p.y<0)p.y=canvas.height;if(p.y>canvas.height)p.y=0;}); requestAnimationFrame(drawParticles);}
initParticles(); drawParticles();

// Game logic
const startBtn=document.getElementById('startBtn');
const startScreen=document.getElementById('startScreen');
const gameWrap=document.getElementById('gameWrap');
const boardEl=document.getElementById('board');
const status=document.getElementById('status');
const restartBtn=document.getElementById('restartBtn');
const bgMusic=document.getElementById('bgMusic');
const clickSound=document.getElementById('clickSound');
const winSound=document.getElementById('winSound');

let board=[],currentPlayer='X',gameOver=false;
const wins=[[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]];

startBtn.onclick=()=>{
  startScreen.style.display='none';
  gameWrap.style.display='flex';
  bgMusic.volume=0.2; bgMusic.play();
  initGame();
}
restartBtn.onclick=initGame;

function initGame(){
  board=Array(9).fill('');
  currentPlayer='X'; gameOver=false; status.textContent=`Player ${currentPlayer}'s Turn`;
  boardEl.innerHTML='';
  for(let i=0;i<9;i++){
    const cell=document.createElement('div');
    cell.className='cell'; cell.dataset.index=i;
    cell.addEventListener('click',playerMove);
    boardEl.appendChild(cell);
  }
}

function playerMove(e){
  const i=Number(e.target.dataset.index);
  if(board[i]!==''||gameOver) return;
  makeMove(i,'X');
  if(!gameOver)setTimeout(computerMove,300);
}

function makeMove(i,player){
  board[i]=player;
  const cell=boardEl.children[i];
  cell.textContent=player;
  cell.classList.add(player==='X'?'neon-x':'neon-o');
  clickSound.currentTime=0; clickSound.play();
  const winCombo=wins.find(c=>c.every(idx=>board[idx]===player));
  if(winCombo){gameOver=true; status.textContent=`🎉 Player ${player} Wins!`; winSound.play(); drawWinLine(winCombo); return;}
  if(board.every(v=>v!=='')){gameOver=true; status.textContent="😲 It's a Draw!"; return;}
  currentPlayer=player==='X'?'O':'X'; status.textContent=`Player ${currentPlayer}'s Turn`;
}

// Unbeatable AI - Minimax
function computerMove(){
  if(gameOver) return;
  const i=minimax(board,'O').index;
  makeMove(i,'O');
}

function minimax(newBoard,player){
  const availSpots=newBoard.map((v,i)=>v===''?i:null).filter(v=>v!==null);
  if(wins.some(c=>c.every(i=>newBoard[i]==='X'))) return {score:-10};
  if(wins.some(c=>c.every(i=>newBoard[i]==='O'))) return {score:10};
  if(availSpots.length===0) return {score:0};
  const moves=[];
  for(let i of availSpots){
    const move={index:i};
    newBoard[i]=player;
    const result=minimax(newBoard,player==='O'?'X':'O');
    move.score=result.score;
    newBoard[i]='';
    moves.push(move);
  }
  let bestMove;
  if(player==='O'){
    let bestScore=-Infinity;
    for(let m of moves){if(m.score>bestScore){bestScore=m.score; bestMove=m;}}
  }else{
    let bestScore=Infinity;
    for(let m of moves){if(m.score<bestScore){bestScore=m.score; bestMove=m;}}
  }
  return bestMove;
}

// Win line
function drawWinLine(combo){
  const line=document.createElement('div'); line.className='win-line';
  const c0=boardEl.children[combo[0]].getBoundingClientRect();
  const c2=boardEl.children[combo[2]].getBoundingClientRect();
  const boardRect=boardEl.getBoundingClientRect();
  const x1=c0.left+c0.width/2-boardRect.left;
  const y1=c0.top+c0.height/2-boardRect.top;
  const x2=c2.left+c2.width/2-boardRect.left;
  const y2=c2.top+c2.height/2-boardRect.top;
  const dx=x2-x1, dy=y2-y1, len=Math.hypot(dx,dy), angle=Math.atan2(dy,dx)*180/Math.PI;
  line.style.width=len+'px'; line.style.left=((x1+x2)/2-len/2)+'px'; line.style.top=((y1+y2)/2-4)+'px';
  line.style.transform=`rotate(${angle}deg)`; boardEl.appendChild(line);
  requestAnimationFrame(()=>line.style.opacity=1);
}
</script>
</body>
</html>
