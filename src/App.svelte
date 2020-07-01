<script>
  import { onMount } from "svelte";

  import "bulma/css/bulma.css";
  import Listing from "./Listing.svelte";
  let listings = [];
  let deleted = new Set();
  let saved = new Set();
  let showDeleted = false;
  let showSaved = false;
  let showOthers = true;
  let searchText = "";

  function addListing(item) {
    const listing = {
      id: item.id,
      kids: item.kids,
      text: item.text,
      time: item.time
    };
    if (!listing.text) {
      console.log(
        `Failed to get https://news.ycombinator.com/item?id=${item.id}`
      );
      console.log(item);
    } else {
      listings.push(listing);
      listings = listings;
    }
  }

  onMount(async () => {
    if (window.localStorage.saved)
      saved = new Set(JSON.parse(window.localStorage.saved));
    if (window.localStorage.deleted)
      deleted = new Set(
        JSON.parse(window.localStorage.deleted).filter(id => !saved.has(id))
      );
    fetch("https://hacker-news.firebaseio.com/v0/item/23702122.json")
      .then(r => r.json())
      .then(item => item.kids)
      .then(kids => {
        kids.sort((a, b) => b - a);
        Promise.all(
          kids.map(kid =>
            fetch(`https://hacker-news.firebaseio.com/v0/item/${kid}.json`)
              .then(r => r.json())
              .then(json => {
                addListing(json);
              })
          )
        );
      });
  });

  $: lowerSearchText = searchText.toLowerCase();

  $: filteredListings = listings.filter(
    l =>
      l.text.toLowerCase().includes(lowerSearchText) &&
      ((deleted.has(l.id) && showDeleted) ||
        (saved.has(l.id) && showSaved) ||
        (!(deleted.has(l.id) || saved.has(l.id)) && showOthers))
  );
</script>

<style>
  main {
    text-align: center;
    padding: 1em;
    margin: 0 auto;
  }
</style>

<main>
  <div class="field has-addons has-addons-centered">
    <input type="text" bind:value={searchText} />
    <div class="buttons has-addons">
      <button
        class="button is-light"
        class:is-inverted={showOthers}
        on:click={() => (showOthers = !showOthers)}>
        Others {listings.length - deleted.size - saved.size}
      </button>
      <button
        class="button is-light"
        class:is-inverted={showDeleted}
        on:click={() => (showDeleted = !showDeleted)}>
        Deleted {deleted.size}
      </button>
      <button
        class="button is-light"
        class:is-inverted={showSaved}
        on:click={() => (showSaved = !showSaved)}>
        Saved {saved.size}
      </button>
    </div>
  </div>
  {#each filteredListings as listing (listing.id)}
    <Listing
      title={listing.text.split('<p>', 1)[0]}
      content={listing.text}
      on:delete={() => {
        deleted.add(listing.id);
        saved.delete(listing.id);
        saved = saved;
        deleted = deleted;
        window.localStorage.deleted = JSON.stringify(Array.from(deleted));
      }}
      on:save={() => {
        saved.add(listing.id);
        deleted.delete(listing.id);
        saved = saved;
        deleted = deleted;
        window.localStorage.saved = JSON.stringify(Array.from(saved));
      }} />
  {/each}
</main>
