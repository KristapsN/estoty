<script lang="ts">
  import { onMount } from "svelte"
  import Chart, { type ChartItem } from "chart.js/auto"

  export let stats = [3, 1]

  let myChart: Chart<"bar", number[], string>
  let ctx: ChartItem
  let label = Array.from(Array(91), (_, i) => `D${i}`)

  $: if (myChart) {
    myChart.data.datasets[0].data = stats
    myChart.update()
  }

  onMount(async () => {
    myChart = new Chart(ctx, {
      type: "bar",
      data: {
        labels: label,
        datasets: [
          {
            label: "% of retention",
            data: stats,
            borderWidth: 1,
          },
        ],
      },
    })
  })
</script>

<canvas id="myChart" width="800" height="400" bind:this={ctx} />
