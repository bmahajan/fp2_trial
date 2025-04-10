<script>
    export let data = [];
  
    export let selectedAgeGroup;
    export let selectedRace;
    export let selectedHousehold;
    export let selectedTenure;
    export let selectedInvestor;
  
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
  
  <div class="p-4 bg-gray-100 rounded shadow max-w-md">
    <h2 class="text-xl font-bold mb-1">Filter Both Histograms</h2>
    <p class="text-sm text-gray-600 mb-4">Apply filters to both the 2010 and 2020 data views</p>
  
    <div class="mb-4">
      <label class="block font-medium mb-1">Age Group</label>
      <select bind:value={selectedAgeGroup}>
        {#each ageGroups as a}
          <option value={a}>{a}</option>
        {/each}
      </select>
    </div>
  
    <div class="mb-4">
      <label class="block font-medium mb-1">Race / Ethnicity</label>
      <select bind:value={selectedRace}>
        {#each raceOptions as r}
          <option value={r}>{r}</option>
        {/each}
      </select>
    </div>
  
    <div class="mb-4">
      <label class="block font-medium mb-1">Household Type</label>
      <select bind:value={selectedHousehold}>
        {#each householdTypes as h}
          <option value={h}>{h}</option>
        {/each}
      </select>
    </div>
  
    <div class="mb-4">
      <label class="block font-medium mb-1">Tenure</label>
      <select bind:value={selectedTenure}>
        {#each tenureOptions as t}
          <option value={t}>{t}</option>
        {/each}
      </select>
    </div>
  
    <div class="mb-4">
      <label class="block font-medium mb-1">Investor Type</label>
      <select bind:value={selectedInvestor}>
        {#each investorTypes as i}
          <option value={i}>{i}</option>
        {/each}
      </select>
    </div>
  </div>
  