<script lang="ts">
  import Select from "svelte-select"

  type GameDataProps = {
    app_id: string
    name: string
    icon: string
  }

  export let items: GameDataProps[] = []
  export let onChange: (event: CustomEvent<GameDataProps>) => void
  export let game: GameDataProps

  $: { game?.name }
</script>

<div>
  <span class="select_label">Game:</span>
  <Select
    --width="15rem"
    itemId={"app_id"}
    label={"name"}
    clearable={false}
    {items}
    on:change={onChange}
    bind:value={game}
  >
    <div class="option_wrapper" slot="item" let:item>
      {#if item.icon}
        <img
          class="option_image"
          src={item.icon}
          alt={item.name}
          width="20px"
        />
      {/if}
      {item.name}
    </div>
    <div class="option_wrapper" slot="selection" let:selection>
      {#if selection.icon}
        <img
          class="option_image"
          src={selection.icon}
          alt={selection.name}
          width="20px"
        />
      {/if}
      {selection.name}
    </div>
  </Select>
</div>

<style>
  .option_wrapper {
    display: flex;
    align-items: center;
  }
  .option_image {
    margin-right: 5px;
  }
</style>
