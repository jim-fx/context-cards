<script lang="ts">
  import Select from "svelte-select";
  import Deck from "./Deck.svelte";

  export let activeDeck;

  const items = [
    { value: "ws2021_be", label: "WS20/21 Betriebswissenschaften" },
  ];

  if (items.length === 1) {
    activeDeck = items[0];
  }

  $: data =
    activeDeck &&
    fetch("decks/" + activeDeck.value + ".json").then((res) => res.json());
</script>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>

<main>
  {#if data}
    {#await data}
      <h2>Loading {activeDeck.label}</h2>
    {:then _data}
      <Deck cards={_data} title={activeDeck.label} />
    {/await}
  {:else}
    <Select {items} bind:selectedValue={activeDeck} />
  {/if}
</main>
