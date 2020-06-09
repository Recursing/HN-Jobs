<script>
  import { slide } from "svelte/transition";

  export let title = "title";
  export let content = "content";
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();
  let expanded = false;
</script>

<style>
  .box {
    max-width: 900px;
    margin: auto;
  }

  .columns {
    margin-bottom: -0.75rem !important;
  }
  .description {
    text-align: left;
    padding-top: 1em;
    padding-left: 1em;
    padding-right: 0.75em;
  }
  .button {
    width: 100%;
  }
</style>

<div class="box">
  <div class="columns">
    <div class="column is-narrow">
      <button class="button is-large" on:click={() => dispatch('delete')}>
        Ignore
      </button>
    </div>
    <div class="column" on:click={() => (expanded = !expanded)}>
      <p class="title is-4">
        {@html title}
      </p>
    </div>
    <div class="column is-narrow">
      <button class="button is-large" on:click={() => dispatch('save')}>
        Save
      </button>
    </div>
  </div>
  {#if expanded}
    <p class="description" transition:slide|local>
      {@html content}
    </p>
  {/if}
</div>
