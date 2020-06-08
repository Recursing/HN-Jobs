<script>
  import Button from "svelma/src/components/Button.svelte";
  import Collapse from "svelma/src/components/Collapse.svelte";
  export let title = "title";
  export let content = "content";
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();
</script>

<style>
  .box {
    max-width: 900px;
    margin: auto;
  }
  .description {
    text-align: left;
    padding-top: 1em;
  }
</style>

<div class="box">
  <div class="columns">
    <div class="column is-narrow">
      <Button type="is-large" on:click={() => dispatch('delete')}>
        Ignore
      </Button>
    </div>
    <div class="column">
      <Collapse open={false}>
        <p class="title is-4" slot="trigger">
          {@html title}
        </p>
        <p class="description">
          {@html content}
        </p>
      </Collapse>
    </div>
    <div class="column is-narrow">
      <Button type="is-large" on:click={() => dispatch('save')}>Save</Button>
    </div>
  </div>
</div>
