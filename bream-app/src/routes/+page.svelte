<script lang="ts">
  import { onMount } from 'svelte';

  type Card = {
    id: number;
    value: string;
    flipped: boolean;
    matched: boolean;
  };

  const values = [
    '🐟', '🐠', '🐡', '🦑', '🦀', '🦐', '🐬', '🦈'
  ];
  let cards: Card[] = [];
  let firstCard: Card | null = null;
  let secondCard: Card | null = null;
  let lockBoard = false;
  let currentPlayer = 0; // 0 or 1
  let scores = [0, 0];
  let gameOver = false;

  function shuffle<T>(array: T[]): T[] {
    let arr = array.slice();
    for (let i = arr.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
    return arr;
  }

  function resetGame() {
    const doubled = [...values, ...values];
    const shuffled = shuffle(doubled);
    cards = shuffled.map((value, i) => ({
      id: i,
      value,
      flipped: false,
      matched: false
    }));
    firstCard = null;
    secondCard = null;
    lockBoard = false;
    currentPlayer = 0;
    scores = [0, 0];
    gameOver = false;
  }

  function flipCard(card: Card) {
    if (lockBoard || card.flipped || card.matched) return;
    card.flipped = true;
    if (!firstCard) {
      firstCard = card;
    } else if (!secondCard) {
      secondCard = card;
      checkForMatch();
    }
  }

  function checkForMatch() {
    if (!firstCard || !secondCard) return;
    lockBoard = true;
    if (firstCard.value === secondCard.value) {
      firstCard.matched = true;
      secondCard.matched = true;
      scores[currentPlayer]++;
      setTimeout(() => {
        resetTurn();
        if (cards.every((c) => c.matched)) {
          gameOver = true;
        }
      }, 800);
    } else {
      setTimeout(() => {
        firstCard!.flipped = false;
        secondCard!.flipped = false;
        resetTurn();
        currentPlayer = 1 - currentPlayer;
      }, 1200);
    }
  }

  function resetTurn() {
    [firstCard, secondCard] = [null, null];
    lockBoard = false;
  }

  onMount(() => {
    resetGame();
  });
</script>

<style>
.game-container {
  max-width: 400px;
  margin: 2rem auto;
  padding: 1rem;
  background: #f8fafc;
  border-radius: 1rem;
  box-shadow: 0 2px 8px #0001;
}
.grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 0.5rem;
}
.card {
  aspect-ratio: 1/1;
  background: #38bdf8;
  color: #fff;
  font-size: 2rem;
  border-radius: 0.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  user-select: none;
  transition: background 0.2s, transform 0.2s;
  box-shadow: 0 1px 4px #0002;
}
.card.flipped, .card.matched {
  background: #fbbf24;
  color: #222;
  transform: scale(1.05);
}
.card.matched {
  background: #22c55e;
  color: #fff;
}
.status {
  display: flex;
  justify-content: space-between;
  margin-bottom: 1rem;
  font-size: 1.1rem;
}
@media (max-width: 600px) {
  .game-container {
    max-width: 98vw;
    padding: 0.5rem;
  }
  .card {
    font-size: 1.5rem;
  }
}
.card-back {
  color: #fff;
  font-size: 2rem;
  opacity: 0.6;
  pointer-events: none;
}
</style>

<div class="game-container">
  <div class="status">
    <div>Player 1: {scores[0]}</div>
    <div>Player 2: {scores[1]}</div>
  </div>
  <div class="status">
    {#if !gameOver}
      <span>Turn: Player {currentPlayer + 1}</span>
    {:else}
      <span>Game Over! {scores[0] === scores[1] ? 'Draw!' : `Winner: Player ${scores[0] > scores[1] ? 1 : 2}`}</span>
    {/if}
    <button on:click={resetGame} style="margin-left:1rem;">Restart</button>
  </div>
  <div class="grid">
    {#each cards as card (card.id)}
      <div
        class="card {card.flipped || card.matched ? 'flipped' : ''} {card.matched ? 'matched' : ''}"
        on:click={() => flipCard(card)}
        tabindex="0"
        aria-label={card.flipped || card.matched ? card.value : 'Hidden card'}
        role="button"
      >
        {#if card.flipped || card.matched}
          {card.value}
        {:else}
          <span class="card-back">?</span>
        {/if}
      </div>
    {/each}
  </div>
</div>
