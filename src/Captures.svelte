<script>
  export let attachments;

  $: images = attachments.filter((x) => x.extension === "jpg");
</script>

{#if attachments.length > 0}
  <div class="captures">
    {#each images as image}
      <a href={image.url}>
        <img src={image.url} alt={image.name} />
      </a>
    {/each}
  </div>
{/if}

<style>
  .captures {
    height: 100px;
    width: 100%;
    display: inline-block;
    overflow-x: scroll;
    margin-top: 8px;
  }

  img {
    height: 100%;
  }
</style>
