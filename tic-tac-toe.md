---
layout: index
title: "Tic Tac Toe"
heading: "Tic Tac Toe"
subheading: "Classic Board Game"
description: "A Tic Tac Toe game built with JavaScript"
user-story: "As a player, I want to take turns marking X and O on a grid so that I can see who wins or if the game ends in a draw."
---

<style>
  .game-wrap{padding:40px 20px;background:#66ccff;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:16px;min-height:520px}
  #ttt-status{font-size:22px;font-weight:700}
  .ttt-board{display:grid;grid-template-columns:repeat(3, minmax(70px, 110px));grid-template-rows:repeat(3, minmax(70px, 110px));gap:8px}
  .ttt-cell{background:#fff;border:3px solid #000;font-size:2.5rem;font-weight:700;display:flex;align-items:center;justify-content:center;cursor:pointer;user-select:none}
  .ttt-cell:hover{background:#eee}
  .ttt-cell.taken{cursor:default}
</style>

<div class="game-wrap">
  <div id="ttt-status">Player X's turn</div>
  <div id="ttt-board" class="ttt-board"></div>
  <button id="ttt-reset" class="btn btn-warning btn-lg" type="button">Reset Game</button>
</div>

<script>
  (function () {
    const boardEl = document.getElementById('ttt-board');
    const statusEl = document.getElementById('ttt-status');
    const resetBtn = document.getElementById('ttt-reset');

    const WIN_LINES = [
      [0, 1, 2], [3, 4, 5], [6, 7, 8],
      [0, 3, 6], [1, 4, 7], [2, 5, 8],
      [0, 4, 8], [2, 4, 6]
    ];

    let cells, current, over;

    function render() {
      boardEl.innerHTML = '';
      cells.forEach((value, i) => {
        const cell = document.createElement('div');
        cell.className = 'ttt-cell' + (value ? ' taken' : '');
        cell.textContent = value || '';
        cell.addEventListener('click', () => handleMove(i));
        boardEl.appendChild(cell);
      });
    }

    function checkWinner() {
      for (const [a, b, c] of WIN_LINES) {
        if (cells[a] && cells[a] === cells[b] && cells[a] === cells[c]) return cells[a];
      }
      return cells.every((v) => v) ? 'draw' : null;
    }

    function handleMove(i) {
      if (over || cells[i]) return;
      cells[i] = current;
      const winner = checkWinner();
      if (winner === 'draw') {
        statusEl.textContent = "It's a draw!";
        over = true;
      } else if (winner) {
        statusEl.textContent = 'Player ' + winner + ' wins!';
        over = true;
      } else {
        current = current === 'X' ? 'O' : 'X';
        statusEl.textContent = "Player " + current + "'s turn";
      }
      render();
    }

    function resetGame() {
      cells = Array(9).fill(null);
      current = 'X';
      over = false;
      statusEl.textContent = "Player X's turn";
      render();
    }

    resetBtn.addEventListener('click', resetGame);
    resetGame();
  })();
</script>
