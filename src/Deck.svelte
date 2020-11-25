<script lang="ts">
  import hash from "short-hash";
  import { onMount } from "svelte";

  interface Card {
    back: string;
    front: string;
    id: number;
  }

  export let cards: Card[];
  export let title: string = "Undefined";
  let modus = "test";
  $: deckId = "coco-cards-" + hash(title);

  let votes: {
    [id: string]: number;
  } = {};

  let activeCard;
  let showBack = false;
  let maxVotes = 5;
  let amountVotes = 0;
  let currentPercent = 0;

  const saveVotes = () => {
    localStorage.setItem(deckId, JSON.stringify(votes));
  };

  function shuffle(array) {
    var currentIndex = array.length,
      temporaryValue,
      randomIndex;

    // While there remain elements to shuffle...
    while (0 !== currentIndex) {
      // Pick a remaining element...
      randomIndex = Math.floor(Math.random() * currentIndex);
      currentIndex -= 1;

      // And swap it with the current element.
      temporaryValue = array[currentIndex];
      array[currentIndex] = array[randomIndex];
      array[randomIndex] = temporaryValue;
    }

    return array;
  }

  const loadVotes = () => {
    if (deckId in localStorage) {
      const s = localStorage.getItem(deckId);
      try {
        votes = JSON.parse(s);
      } catch (error) {
        console.error(error);
        alert("Could not parse localStorage");
      }
    }

    let needsSave = false;
    cards.forEach((card) => {
      if (!(card.id in votes)) {
        needsSave = true;
        votes[card.id.toString()] = 0;
      }
    });

    needsSave && saveVotes();
  };

  const addVote = (id, value) => {
    amountVotes += 1;
    votes[id] = value;

    saveVotes();
    pickNewCard();
  };

  const pickNewCard = () => {
    showBack = false;
    const sortedVotes = Object.entries(votes)
      .map(([id, vote]) => {
        return {
          id: parseInt(id),
          value: vote,
        };
      })
      .sort((a, b) => (a.value > b.value ? 1 : -1));

    activeCard = cards.find((c) => c.id === sortedVotes[0].id);

    currentPercent =
      (sortedVotes.reduce((a, b) => a + b.value, 0) / (cards.length * 3)) * 100;

    console.log(activeCard);
  };

  onMount(() => {
    cards = shuffle(cards);
    loadVotes();
    pickNewCard();
  });
</script>

<style>
  :global(body) {
    display: grid;
    place-content: center;
  }

  #progress-bar {
    position: absolute;
    top: 0;
    left: 0;
    width: 100vw;
    height: 40px;
  }

  #progress-bar > p {
    height: 100%;
    line-height: 100%;
  }

  #bar {
    z-index: -1;
    position: absolute;
    background-color: yellow;
    height: 100%;
    transition: width 0.3s ease;
  }

  h1 {
    position: fixed;
    font-size: 3em;
    opacity: 0.3;
    left: 0;
    top: 0;
    width: 100vh;
    text-align: center;
    transform: rotate(90deg) translateY(-100%);
    transform-origin: 0 0;
  }

  .content {
    max-width: 600px;
  }

  .overview-wrapper {
    position: relative;
    border-radius: 10px;
    overflow: hidden;
    background-color: white;
    box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.4);
  }

  .overview-wrapper > .cards {
    max-width: 600px;
    max-height: 70vh;
    overflow-y: auto;

    display: flex;
    justify-content: center;
    flex-wrap: wrap;
  }

  .overview-wrapper > .cards > .card {
    width: 100px;
    border-radius: 10px;
    outline: solid thin;
    padding: 5px;
    margin: 5px;
  }
  .overview-wrapper > .cards > .card > .card-title {
    max-height: 90px;
    max-width: 100px;
    text-overflow: ellipsis;
    overflow: hidden;
    word-break: break-word;
  }

  .overview-wrapper > .icons > button {
    position: absolute;
    right: 0;
    top: 0;
  }

  #show-overview {
    position: fixed;
    top: 0;
    right: 0;
  }
</style>

<div id="progress-bar">
  <div style={`width: ${currentPercent}%`} id="bar" />
  <b>Confidence: {Math.floor(currentPercent)}%</b>
  {amountVotes % cards.length}/{cards.length}
</div>

{#if modus === 'test'}
  <button
    id="show-overview"
    on:click={() => {
      modus = 'overview';
    }}>Overview</button>

  <div class="content">
    {#if activeCard}
      <h2>
        {@html activeCard.flds.split('\u001f')[0]}
      </h2>

      {#if showBack}
        <p>
          {@html activeCard.flds
            .split('\u001f')[1]
            .replace(activeCard.front, '')}
        </p>
        <button on:click={() => addVote(activeCard.id, 0)}>Keine Ahnung</button>
        <button on:click={() => addVote(activeCard.id, 2)}>Okay...</button>
        <button on:click={() => addVote(activeCard.id, 3)}>Easy</button>
      {:else}<button on:click={() => (showBack = true)}>show</button>{/if}
    {/if}
  </div>
{:else if modus === 'overview'}
  <div class="overview-wrapper">
    <div class="icons">
      <h3>Overview</h3>
      <button on:click={() => (modus = 'test')}>x</button>
    </div>

    <div class="cards">
      {#each cards as card}
        <div class="card">
          <p class="card-title">{card.front}</p>

          {#if card.id in votes}
            <p>{Math.floor((votes[card.id] / 3) * 100)}%</p>
          {/if}
        </div>
      {/each}
    </div>
  </div>
{/if}

<h1>{title}</h1>
