<script>
  export let data = [];

  export let selectedAgeGroup;
  export let selectedRace;
  export let selectedHousehold;
  export let selectedTenure;
  export let selectedInvestor;
  export let BUCKETS;
  export let color;

  const ageGroups = ["All", "Under 18", "18-64", "65 and over"];

  const raceOptions = [
    "All",
    "Non-Hispanic White",
    "Non-Hispanic Black or African American",
    "Non-Hispanic American Indian",
    "Non-Hispanic Asian",
    "Non-Hispanic Pacific Islander",
    "Non-Hispanic Multi-Race",
    "Non-Hispanic Other",
    "Hispanic or Latino"
  ];

  const householdTypes = ["All", "Family Household", "Nonfamily Household"];
  const tenureOptions = ["All", "Owner-Occupied", "Renter-Occupied"];
  const investorTypes = [
    "All",
    "Institutional",
    "Large Investor",
    "Medium Investor",
    "Small Investor",
    "Non-Investor"
  ];
</script>

<div class="container">
  <div class="header-container">
    <h2>Count of census tracts based on short term flips, 2010 vs 2020</h2>
  </div>
  <div class="filter-legend-container">
    <div>
      <h3>Filters</h3>
      <p style="margin: 0px 0px 16px 0px;">Use filters to compare and explore both data sets.</p>
      <div class="filter-container">
        <div class="filter-item">
          <label>Age Group</label>
          <select bind:value={selectedAgeGroup}>
            {#each ageGroups as a}
              <option value={a}>{a}</option>
            {/each}
          </select>
        </div>

        <div class="filter-item">
          <label>Household Type</label>
          <select bind:value={selectedHousehold}>
            {#each householdTypes as h}
              <option value={h}>{h}</option>
            {/each}
          </select>
        </div>

        <div class="filter-item">
          <label>Tenure</label>
          <select bind:value={selectedTenure}>
            {#each tenureOptions as t}
              <option value={t}>{t}</option>
            {/each}
          </select>
        </div>

        <div class="filter-item">
          <label>Investor Type</label>
          <select bind:value={selectedInvestor}>
            {#each investorTypes as i}
              <option value={i}>{i}</option>
            {/each}
          </select>
        </div>

        <div class="filter-item">
          <label>Race / Ethnicity</label>
          <select bind:value={selectedRace}>
            {#each raceOptions as r}
              <option value={r}>{r}</option>
            {/each}
          </select>
        </div>
      </div>
    </div>

    <div class="legend-outer-container">
      <h3>Legend</h3>
      <div class="legend-container">
        <div class="legend-header">
          <h3>Income brackets</h3>
        </div>
        <div class="legend-content">
          <div class="legend-item">
            <span class="legend-color" style="background-color: oklch(0.64 0.175 141.55)"></span>
            <span class="legend-label">High income (&gt;$100k)</span>
          </div>
          <div class="legend-item">
            <span class="legend-color" style="background-color: oklch(0.89 0.1809 95.96)"></span>
            <span class="legend-label">Medium income ($60k-$100k)</span>
          </div>
          <div class="legend-item">
            <span class="legend-color" style="background-color: oklch(0.58 0.0619 271.13)"></span>
            <span class="legend-label">Low income (&lt;$60k)</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
