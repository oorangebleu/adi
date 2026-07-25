const COLS = 10;
const ROWS = 20;
const EMPTY = null;
const SHAPES = {
  I: { matrix: [[0, 0, 0, 0], [1, 1, 1, 1], [0, 0, 0, 0], [0, 0, 0, 0]], color: '#38bdf8' },
  O: { matrix: [[1, 1], [1, 1]], color: '#facc15' },
  T: { matrix: [[0, 1, 0], [1, 1, 1], [0, 0, 0]], color: '#a78bfa' },
  S: { matrix: [[0, 1, 1], [1, 1, 0], [0, 0, 0]], color: '#4ade80' },
  Z: { matrix: [[1, 1, 0], [0, 1, 1], [0, 0, 0]], color: '#fb7185' },
  J: { matrix: [[1, 0, 0], [1, 1, 1], [0, 0, 0]], color: '#60a5fa' },
  L: { matrix: [[0, 0, 1], [1, 1, 1], [0, 0, 0]], color: '#fb923c' },
};

const boardEl = document.getElementById('board');
const nextEl = document.getElementById('next');
const scoreEl = document.getElementById('score');
const overlayEl = document.getElementById('overlay');
const restartBtn = document.getElementById('restartBtn');

let board = createBoard();
let currentPiece = null;
let nextPiece = null;
let score = 0;
let isRunning = false;
let isPaused = false;
let isGameOver = false;
let dropTimer = null;

function createBoard() {
  return Array.from({ length: ROWS }, () => Array(COLS).fill(EMPTY));
}

function getRandomShape() {
  const types = Object.keys(SHAPES);
  const type = types[Math.floor(Math.random() * types.length)];
  return { type, matrix: SHAPES[type].matrix.map((row) => [...row]), color: SHAPES[type].color };
}

function createPiece(type) {
  const shape = SHAPES[type];
  return {
    type,
    matrix: shape.matrix.map((row) => [...row]),
    color: shape.color,
    row: 0,
    col: Math.floor(COLS / 2) - 1,
  };
}

function rotateMatrix(matrix) {
  return matrix[0].map((_, colIndex) => matrix.map((row) => row[colIndex]).reverse());
}

function spawnPiece() {
  if (!nextPiece) {
    nextPiece = getRandomShape();
  }

  currentPiece = createPiece(nextPiece.type);
  currentPiece.matrix = nextPiece.matrix.map((row) => [...row]);
  currentPiece.color = nextPiece.color;
  nextPiece = getRandomShape();

  if (!isValidPosition(currentPiece)) {
    endGame();
    return false;
  }

  render();
  return true;
}

function isValidPosition(piece, offsetRow = 0, offsetCol = 0, matrix = piece.matrix) {
  return matrix.every((row, rowIndex) =>
    row.every((value, colIndex) => {
      if (!value) return true;
      const newRow = piece.row + rowIndex + offsetRow;
      const newCol = piece.col + colIndex + offsetCol;
      if (newCol < 0 || newCol >= COLS || newRow >= ROWS) return false;
      if (newRow < 0) return true;
      return board[newRow][newCol] === EMPTY;
    })
  );
}

function mergePiece() {
  currentPiece.matrix.forEach((row, rowIndex) => {
    row.forEach((value, colIndex) => {
      if (value) {
        const boardRow = currentPiece.row + rowIndex;
        const boardCol = currentPiece.col + colIndex;
        if (boardRow >= 0 && boardRow < ROWS && boardCol >= 0 && boardCol < COLS) {
          board[boardRow][boardCol] = currentPiece.color;
        }
      }
    });
  });
}

function clearLines() {
  const remainingRows = board.filter((row) => row.some((cell) => cell === EMPTY));
  const cleared = ROWS - remainingRows.length;
  if (cleared > 0) {
    score += [0, 100, 300, 500, 800][cleared];
    while (remainingRows.length < ROWS) {
      remainingRows.unshift(Array(COLS).fill(EMPTY));
    }
    board = remainingRows;
    updateScore();
  }
}

function movePiece(offsetRow, offsetCol) {
  if (!currentPiece || isPaused || isGameOver) return;
  if (isValidPosition(currentPiece, offsetRow, offsetCol)) {
    currentPiece.row += offsetRow;
    currentPiece.col += offsetCol;
    render();
  } else if (offsetRow > 0) {
    mergePiece();
    clearLines();
    if (!spawnPiece()) {
      return;
    }
    render();
  }
}

function rotatePiece() {
  if (!currentPiece || isPaused || isGameOver) return;
  const rotated = rotateMatrix(currentPiece.matrix);
  const kicks = [0, -1, 1, -2, 2];
  for (const kick of kicks) {
    if (isValidPosition(currentPiece, 0, kick, rotated)) {
      currentPiece.matrix = rotated;
      currentPiece.col += kick;
      render();
      return;
    }
  }
}

function hardDrop() {
  if (!currentPiece || isPaused || isGameOver) return;
  while (isValidPosition(currentPiece, 1, 0)) {
    currentPiece.row += 1;
  }
  mergePiece();
  clearLines();
  spawnPiece();
  render();
}

function tick() {
  if (!isRunning || isPaused || isGameOver) return;
  movePiece(1, 0);
}

function startGame() {
  board = createBoard();
  score = 0;
  isPaused = false;
  isGameOver = false;
  isRunning = true;
  updateScore();
  if (dropTimer) clearInterval(dropTimer);
  dropTimer = setInterval(tick, 650);
  nextPiece = getRandomShape();
  spawnPiece();
  overlayEl.classList.add('hidden');
  render();
}

function pauseGame() {
  if (!isRunning || isGameOver) return;
  isPaused = !isPaused;
  overlayEl.textContent = isPaused ? 'Paused' : '';
  overlayEl.classList.toggle('hidden', !isPaused);
}

function endGame() {
  isRunning = false;
  isGameOver = true;
  overlayEl.textContent = 'Game Over';
  overlayEl.classList.remove('hidden');
  clearInterval(dropTimer);
}

function updateScore() {
  scoreEl.textContent = score;
}

function renderBoard() {
  boardEl.innerHTML = '';
  const cells = [];
  for (let row = 0; row < ROWS; row += 1) {
    for (let col = 0; col < COLS; col += 1) {
      const cell = document.createElement('div');
      cell.className = 'cell';
      const value = board[row][col];
      if (value) {
        cell.classList.add('filled');
        cell.style.setProperty('--cell-color', value);
      }
      cells.push(cell);
    }
  }
  boardEl.append(...cells);
}

function renderPiece() {
  if (!currentPiece) return;
  currentPiece.matrix.forEach((row, rowIndex) => {
    row.forEach((value, colIndex) => {
      if (!value) return;
      const boardRow = currentPiece.row + rowIndex;
      const boardCol = currentPiece.col + colIndex;
      if (boardRow >= 0 && boardRow < ROWS && boardCol >= 0 && boardCol < COLS) {
        const cell = boardEl.children[boardRow * COLS + boardCol];
        if (cell) {
          cell.classList.add('filled');
          cell.style.setProperty('--cell-color', currentPiece.color);
        }
      }
    });
  });
}

function renderNextPiece() {
  nextEl.innerHTML = '';
  const preview = nextPiece?.matrix || SHAPES.I.matrix;
  const cells = [];
  for (let row = 0; row < 4; row += 1) {
    for (let col = 0; col < 4; col += 1) {
      const cell = document.createElement('div');
      cell.className = 'cell';
      const value = preview[row]?.[col] || 0;
      if (value) {
        cell.classList.add('filled');
        cell.style.setProperty('--cell-color', nextPiece?.color || SHAPES.I.color);
      }
      cells.push(cell);
    }
  }
  nextEl.append(...cells);
}

function render() {
  renderBoard();
  renderPiece();
  renderNextPiece();
}

window.addEventListener('keydown', (event) => {
  if (event.key === 'Enter') {
    event.preventDefault();
    if (!isRunning || isGameOver) {
      startGame();
    }
    return;
  }

  if (!currentPiece || !isRunning) return;

  if (event.key === 'p' || event.key === 'P') {
    event.preventDefault();
    pauseGame();
    return;
  }

  if (isPaused || isGameOver) return;

  switch (event.key) {
    case 'ArrowLeft':
      event.preventDefault();
      movePiece(0, -1);
      break;
    case 'ArrowRight':
      event.preventDefault();
      movePiece(0, 1);
      break;
    case 'ArrowDown':
      event.preventDefault();
      movePiece(1, 0);
      break;
    case 'ArrowUp':
      event.preventDefault();
      rotatePiece();
      break;
    case ' ':
    case 'Spacebar':
      event.preventDefault();
      hardDrop();
      break;
    case 'r':
    case 'R':
      event.preventDefault();
      startGame();
      break;
    default:
      break;
  }
});

restartBtn.addEventListener('click', startGame);

render();
updateScore();
