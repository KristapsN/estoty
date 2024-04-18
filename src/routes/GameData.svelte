<script lang="ts">
  import Select from "svelte-select";
  import GameSelect from "../components/GameSelect.svelte";
  import Table from "../components/Table.svelte";
  import Chart from "../components/Chart.svelte";
  import Switch from "../components/Switch.svelte";

  type RetentionDataProps = {
    app_id: string;
    app_ver: string;
    country: string;
    days: number[];
  };

  type GameDataProps = {
    app_id: string;
    name: string;
    icon: string;
  };

  type CountryProps = Pick<RetentionDataProps, "country" | "days">;
  type VersionProps = Pick<RetentionDataProps, "app_ver" | "days">;

  let game: GameDataProps = { app_id: "All", name: "All", icon: "" };
  let versions: RetentionDataProps[] = [];
  let countries: RetentionDataProps[] = [];
  let filteredRetentionData: RetentionDataProps[] = [];
  let version: VersionProps = {
    app_ver: "All",
    days: [],
  };
  let country: CountryProps = {
    country: "All",
    days: [],
  };
  let chartData: number[] = [];
  let switchValue: string;

  export const fetchStatistics = async () => {
    const gameResponse = await fetch(
      "https://storage.googleapis.com/estoty-temp/games.json",
    );
    const retentionResponse = await fetch(
      "https://storage.googleapis.com/estoty-temp/retention.json",
    );

    if (gameResponse.ok && retentionResponse.ok) {
      const gameData = await gameResponse.json();
      const retentionData = await retentionResponse.json();

      filteredRetentionData = retentionData
      return {
        gameData,
        retentionData,
      };
    } else {
      // 404, 500 and other common error handling should be done here
      return 'Something went wrong';
    }
  };

  const calculateVersionDevices = (versionData: RetentionDataProps[]) => {
    const uniqVersions: VersionProps[] = [];
    versionData.map(({ app_ver, days }) => {
      if (
        uniqVersions.length > 0 &&
        uniqVersions.some((uniqVersion) => uniqVersion.app_ver === app_ver)
      ) {
        const existingVersion = uniqVersions.find(
          (uniqVersion) => uniqVersion.app_ver === app_ver,
        );

        existingVersion.days = existingVersion.days.map(
          (day, index) => day + days[index],
        );
      } else {
        uniqVersions.push({ app_ver, days });
      }
    });

    return uniqVersions;
  };

  const calculateCountryDevices = (countryData: RetentionDataProps[]) => {
    const uniqCountries: CountryProps[] = [];
    countryData.map(({ country, days }) => {
      if (
        uniqCountries.length > 0 &&
        uniqCountries.some((uniqCountry) => uniqCountry.country === country)
      ) {
        const existingVersion = uniqCountries.find(
          (uniqCountry) => uniqCountry.country === country,
        );

        existingVersion.days = existingVersion.days.map(
          (day, index) => day + days[index],
        );
      } else {
        uniqCountries.push({ country, days });
      }
    });

    return uniqCountries;
  };

  const versionTransformer = (version: string) => {
    return parseInt(version.split(".").join(""));
  };

  const filterVersionsByGame = (
    retentionData: RetentionDataProps[],
    game: string,
  ) => {
    versions = retentionData;

    if (game !== "All") {
      versions = versions.filter(({ app_id }) => app_id === game);
    }

    versions = versions.sort(
      (a, b) => versionTransformer(a.app_ver) - versionTransformer(b.app_ver),
    );
    filteredRetentionData = versions;

    return versions;
  };

  const filterVersionByCountry = (
    retentionData: RetentionDataProps[],
    country: string | undefined,
  ) => {
    versions = filterVersionsByGame(retentionData, game.app_id);
    filteredRetentionData = versions;

    if (version.app_ver !== "All") {
      filteredRetentionData = filteredRetentionData.filter(
        ({ app_ver }) => app_ver === version.app_ver,
      );
    }

    if (country !== "All") {
      filteredRetentionData = filteredRetentionData.filter(
        (item) => item.country === country,
      );
      versions = versions.filter((item) => item.country === country);
    }

    return versions;
  };

  const filterCountriesByGame = (
    retentionData: RetentionDataProps[],
    game: string,
  ) => {
    countries = retentionData;

    if (game !== "All") {
      countries = countries.filter(({ app_id }) => app_id === game);
    }

    countries = countries.sort((a, b) =>
      a.country > b.country ? 1 : b.country > a.country ? -1 : 0,
    );
    filteredRetentionData = countries;

    return countries;
  };

  const filterCountriesByVersion = (
    retentionData: RetentionDataProps[],
    appVersion: string | undefined,
  ) => {
    countries = filterCountriesByGame(retentionData, game.app_id);
    filteredRetentionData = countries;

    if (country.country !== "All") {
      filteredRetentionData = filteredRetentionData.filter(
        (item) => item.country === country.country,
      );
    }

    if (appVersion !== "All") {
      filteredRetentionData = filteredRetentionData.filter(
        ({ app_ver }) => app_ver === appVersion,
      );
      countries = countries.filter(({ app_ver }) => app_ver === appVersion);
    }

    return countries;
  };

  const versionHandler = (
    retentionData: RetentionDataProps[],
    game: string,
  ) => {
    country = { country: "All", days: [] };
    version = { app_ver: "All", days: [] };

    filteredRetentionData = filterVersionsByGame(retentionData, game);
    filteredRetentionData = filterCountriesByGame(retentionData, game);

    versions = filterVersionsByGame(retentionData, game);
    countries = filterCountriesByGame(retentionData, game);
  };

  const updateChartData = (
    chartRawData: RetentionDataProps[],
    index: number,
  ) => {
    chartData = [];

    if (chartRawData.length > 0) {
      chartData = chartRawData[index].days.map((day) => {
        return Math.round((day / chartRawData[index].days[0]) * 100);
      });
    }
    chartData = chartData;
  };
</script>

{#await fetchStatistics()}
  <p>...waiting</p>
{:then data}
  <div class="filter_wrapper">
    <div class="filter_inputs">
      <GameSelect
        items={[
          { app_id: "All", name: "All" },
          ...data.gameData.sort((a, b) =>
            a.name > b.name ? 1 : b.name > a.name ? -1 : 0,
          ),
        ]}
        onChange={(event) => {
          versionHandler(data.retentionData, event.detail.app_id);
          chartData = [];
        }}
        bind:game
      />
      <div>
        <span class="select_label">Version:</span>
        <Select
          --width="15rem"
          itemId={"app_ver"}
          label={"app_ver"}
          clearable={false}
          items={[
            { app_ver: "All", days: [] },
            ...calculateVersionDevices(versions),
          ]}
          bind:value={version}
          on:change={(event) => {
            countries = filterCountriesByVersion(
              data.retentionData,
              event.detail.app_ver,
            );

            if (version.app_ver !== "All") {
              const versionWithAllDevices = calculateVersionDevices(versions);
              const index = versionWithAllDevices
                .map(({ app_ver }) => app_ver)
                .indexOf(version.app_ver);

              updateChartData(versionWithAllDevices, index);
            } else {
              const countryWithAllDevices = calculateCountryDevices(countries);
              const index = countryWithAllDevices
                .map(({ country }) => country)
                .indexOf(country.country);

              updateChartData(countryWithAllDevices, index);
            }
          }}
        >
          <div slot="item" let:item>
            {item.app_ver}
            {#if item.days[0]}
              ({item.days[0]})
            {/if}
          </div>
          <div slot="selection" let:selection>
            {selection.app_ver}
            {#if selection.days[0]}
              ({selection.days[0]})
            {/if}
          </div>
        </Select>
      </div>
      <div>
        <span class="select_label">Country:</span>
        <Select
          --width="15rem"
          itemId={"country"}
          label={"country"}
          clearable={false}
          items={[
            { country: "All", days: [] },
            ...calculateCountryDevices(countries),
          ]}
          bind:value={country}
          on:change={(event) => {
            versions = filterVersionByCountry(
              data.retentionData,
              event.detail.country,
            );
            const countryWithAllDevices = calculateCountryDevices(countries);

            if (country.country !== "All") {
              const index = countryWithAllDevices
                .map(({ country }) => country)
                .indexOf(country.country);
              updateChartData(countryWithAllDevices, index);
            } else {
              const versionWithAllDevices = calculateVersionDevices(versions);
              const index = versionWithAllDevices
                .map(({ app_ver }) => app_ver)
                .indexOf(version.app_ver);

              updateChartData(versionWithAllDevices, index);
            }
          }}
        >
          <div slot="item" let:item>
            {item.country}
            {#if item.days[0]}
              ({item.days[0]})
            {/if}
          </div>
          <div slot="selection" let:selection>
            {selection.country}
            {#if selection.days[0]}
              ({selection.days[0]})
            {/if}
          </div>
        </Select>
      </div>
    </div>
    <div>
      <Switch bind:value={switchValue} options={["Table", "Chart"]} />
    </div>
  </div>
  {#if switchValue === "Table"}
    <Table {filteredRetentionData} />
  {/if}
  {#if switchValue === "Chart"}
    <Chart stats={chartData} />
  {/if}
{:catch error}
  <p>An error occurred! {error}</p>
{/await}

<style>
  .filter_wrapper {
    position: relative;
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin: 2rem 0;
    z-index: 20;
    margin: 2rem 4rem;
  }

  .filter_inputs {
    display: flex;
  }

  .counter {
    display: flex;
    border-top: 1px solid rgba(0, 0, 0, 0.1);
    border-bottom: 1px solid rgba(0, 0, 0, 0.1);
    margin: 1rem 0;
  }

  .select_label {
    font-size: 0.8rem;
  }
</style>
