<script lang="ts">

  export let options: string[] = []
  export let value = "Table"

  const uniqueID = Math.floor(Math.random() * 100)
</script>

<div class="s s--multi">
  <div
    role="radiogroup"
    class="group-container"
    aria-labelledby={`label-${uniqueID}`}
    id={`group-${uniqueID}`}
  >
    {#each options as option}
      <input
        type="radio"
        id={`${option}-${uniqueID}`}
        value={option}
        bind:group={value}
      />
      <label for={`${option}-${uniqueID}`}>
        {option}
      </label>
    {/each}
  </div>
</div>

<style>
  :root {
    --accent-color: CornflowerBlue;
    --gray: #ccc;
  }

  .group-container {
    border: none;
    padding: 0;
    white-space: nowrap;
  }

  label {
    display: inline-block;
    line-height: 1.6;
    position: relative;
    z-index: 2;
  }

  input {
    opacity: 0;
    position: absolute;
  }

  label:first-of-type {
    padding-right: 5em;
  }

  label:last-child {
    margin-left: -5em;
    padding-left: 5em;
  }

  label:first-of-type:before,
  label:first-of-type:after {
    content: "";
    height: 1.25em;
    overflow: hidden;
    pointer-events: none;
    position: absolute;
    vertical-align: middle;
  }

  label:first-of-type:before {
    border-radius: 100%;
    z-index: 2;
    position: absolute;
    width: 1.2em;
    height: 1.2em;
    background: #fff;
    top: 0.2em;
    right: 1.2em;
    transition: transform 0.3s;
  }

  label:first-of-type:after {
    background: var(--accent-color);
    border-radius: 1em;
    margin: 0 1em;
    width: 3em;
    height: 1.6em;
  }

  input:first-of-type:checked ~ label:first-of-type:after {
    background: var(--gray);
  }

  input:first-of-type:checked ~ label:first-of-type:before {
    transform: translateX(-1.4em);
  }

  input:last-of-type:checked ~ label:last-of-type {
    z-index: 1;
  }

  input:focus {
    box-shadow: 0 0px 8px var(--accent-color);
    border-radius: 1.5em;
  }

</style>
