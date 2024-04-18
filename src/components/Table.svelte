<script lang="ts">
  type RetentionDataProps = {
    app_id: string
    app_ver: string
    country: string
    days: number[]
  }

  export let filteredRetentionData: RetentionDataProps[]
</script>

<div class={"table_wrapper"}>
  <table>
    <thead>
      <tr>
        <th class="country_columns first-col">Version</th>
        <th class="country_columns second-col">Country</th>
        {#each { length: 91 } as _, index}
          <th class="days_columns">D{index}</th>
        {/each}
      </tr>
    </thead>
    <tbody>
      {#each filteredRetentionData as { app_ver, country, days }}
        <tr>
          <td class="country_columns first-col frozen_column_background"
            >{app_ver}</td
          >
          <td class="country_columns second-col frozen_column_background"
            >{country}</td
          >
          {#each days as day}
            <td class="days_columns">{Math.round((day / days[0]) * 100)}%</td>
          {/each}
        </tr>
      {/each}
    </tbody>
  </table>
</div>

<style>
  .table_wrapper {
    max-height: 750px;
    overflow-x: scroll;
    overflow-y: visible;
    width: 90vw;
  }

  .days_columns {
    min-width: 3rem;
    text-align: center;
  }

  .country_columns {
    min-width: 10rem;
  }

  .frozen_column_background {
    background-color: #e2e2e2;
  }

  .first-col {
    width: 10rem;
    min-width: 10rem;
    max-width: 10rem;
    left: 0px;
    position: sticky;
  }

  .second-col {
    width: 10rem;
    min-width: 10rem;
    max-width: 10rem;
    left: 12rem;
    position: sticky;
  }

  table {
    font-family: Arial, Helvetica, sans-serif;
    border-collapse: collapse;
    width: 100%;
  }

  table tr:nth-child(even) {
    background-color: #f2f2f2;
  }

  table tr:hover {
    background-color: #ddd;
  }

  table th {
    background-color: #04aa6d;
    text-align: left;
    color: #ffffff;
  }

  table th,
  table td {
    max-width: 300px;
    padding: 8px 16px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  table thead {
    position: sticky;
    inset-block-start: 0;
    background-color: #ddd;
    z-index: 10;
  }
</style>
