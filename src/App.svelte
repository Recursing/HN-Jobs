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
  let post_title = "Who is hiring on HN?";

  function addListing(item) {
    const listing = {
      id: item.id,
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

  function fetchListings(post) {
    post.kids.sort((a, b) => b - a);
    Promise.all(
      post.kids.map((kid) =>
        fetch(`https://hacker-news.firebaseio.com/v0/item/${kid}.json`)
          .then((r) => r.json())
          .then((json) => {
            addListing(json);
          })
      )
    );
  }

  onMount(async () => {
    let r = await fetch(
      "https://hacker-news.firebaseio.com/v0/user/whoishiring.json"
    );
    let user = await r.json();
    let post_id = "";
    for (post_id of user["submitted"]) {
      r = await fetch(
        `https://hacker-news.firebaseio.com/v0/item/${post_id}.json`
      );
      let post = await r.json();
      if (post.title.includes("hiring")) {
        post_title = post.title;
        fetchListings(post);
        break;
      }
    }
    if (window.localStorage.threadid != post_id) {
      window.localStorage.threadid = post_id;
      window.localStorage.saved = "[]";
      window.localStorage.deleted = "[]";
    }
    if (window.localStorage.saved)
      saved = new Set(JSON.parse(window.localStorage.saved));
    if (window.localStorage.deleted)
      deleted = new Set(
        JSON.parse(window.localStorage.deleted).filter(id => !saved.has(id))
      );
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
  <h1 class="title">{post_title}</h1>
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
      url={`https://news.ycombinator.com/item?id=${listing.id}`}
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
