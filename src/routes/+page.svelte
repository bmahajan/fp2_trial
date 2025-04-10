<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import { writable } from 'svelte/store';
    import FilterPanel from './components/FilterPanel.svelte'; // you'll create this file

    let chartContainer2010;
    let chartContainer2020;

    const BUCKETS = ["Low", "Medium", "High"];

    // Define graph colors
    const color = d3.scaleOrdinal()
      .domain(BUCKETS)
      .range([
        'oklch(0.58 0.0619 271.13)',  // muted blue
        'oklch(0.89 0.1809 95.96)', // yellow
        'oklch(0.64 0.175 141.55)'  // green
      ]);
    const selectedBin = writable(null);

  let data2010 = [];
  let data2020 = [];
  let filteredData2010 = [];
  let filteredData2020 = [];
  let selectedAgeGroup = 'All';
  let selectedRace = 'All';
  let selectedHousehold = 'All';
  let selectedTenure = 'All';
  let selectedInvestor = 'All';

  // Precompute data, bin, and stack
  function processAndBinData(data) {
    const flipVals = data.map(d => d.flip_ind_sum);
    const xMin = d3.min(flipVals);
    const xMax = d3.max(flipVals);
    const binGen = d3.bin()
      .value(d => d.flip_ind_sum)
      .domain([xMin, xMax])
      .thresholds(d3.range(0, xMax + 5, 5));
    const bins = binGen(data);

    const binData = bins.map(bin => {
        const roll = d3.rollups(bin, v => v.length, d => d.income_bucket);
        const mapB = new Map(roll);
        return {
          x0: bin.x0,
          x1: bin.x1,
          Low: mapB.get("Low") || 0,
          Medium: mapB.get("Medium") || 0,
          High: mapB.get("High") || 0
        };
      })

    const stack = d3.stack().keys(BUCKETS);
    const series = stack(binData);
    const yMax = d3.max(series[series.length - 1], d => d[1]);

    return {
      series,
      binData,
      xMin,
      xMax,
      yMax
    };
  }

  function applyFilters() {
    // Store the precomputed binned data
    const processed2010 = processAndBinData(filteredData2010);
    const processed2020 = processAndBinData(filteredData2020);
    const maxYMax = Math.max(processed2010.yMax, processed2020.yMax);

    // Apply filters
    console.log("Filter state:", {
      selectedAgeGroup,
      selectedRace,
      selectedHousehold,
      selectedTenure,
      selectedInvestor
    });

    const raceColumnMap = {
      'Non-Hispanic White': 'nhwhi',
      'Non-Hispanic Black or African American': 'nhaa',
      'Non-Hispanic American Indian': 'nhna',
      'Non-Hispanic Asian': 'nhas',
      'Non-Hispanic Pacific Islander': 'nhpi',
      'Non-Hispanic Multi-Race': 'nhmlt',
      'Non-Hispanic Other': 'nhoth',
      'Hispanic or Latino': 'lat'
    };

    const investorColumnMap = {
      'Institutional': 'restype_R1F_sum',
      'Large Investor': 'sum_large_investor',
      'Medium Investor': 'sum_medium_investor',
      'Small Investor': 'sum_small_investor',
      'Non-Investor': 'sum_non_investor'
    };

    function filterFn(d) {
      const agePass =
        selectedAgeGroup === 'All' ||
        (selectedAgeGroup === 'Under 18' && +d.pop_u18 > 0) ||
        (selectedAgeGroup === '18-64' && +d.pop_18_64 > 0) ||
        (selectedAgeGroup === '65 and over' && +d.pop_65o > 0);

      const racePass =
        selectedRace === 'All' ||
        (+d[raceColumnMap[selectedRace]] > 0);

      const householdPass =
        selectedHousehold === 'All' ||
        (selectedHousehold === 'Family Household' && +d.fhh > 0) ||
        (selectedHousehold === 'Nonfamily Household' && +d.nfhh > 0);

      const tenurePass =
        selectedTenure === 'All' ||
        (selectedTenure === 'Owner-Occupied' && +d.o_mhi > 0) ||
        (selectedTenure === 'Renter-Occupied' && +d.r_mhi > 0);

      const investorPass =
        selectedInvestor === 'All' ||
        (+d[investorColumnMap[selectedInvestor]] > 0);

      return agePass && racePass && householdPass && tenurePass && investorPass;
    }
    
    console.log("Before filtering:", data2020.length);
    console.log("After filtering:", filteredData2020.length);

    filteredData2010 = data2010.filter(filterFn);
    filteredData2020 = data2020.filter(filterFn);

    createStackedHistogram({
      container: chartContainer2010,
      processedData: {...processed2010, yMax: maxYMax}, // Sets the same yMax for both histograms
      compareData: processed2020.binData,
      title: "2010 Data"
    });

    createStackedHistogram({
      container: chartContainer2020,
      processedData: {...processed2020, yMax: maxYMax}, // Sets the same yMax for both histograms
      compareData: processed2010.binData,
      title: "2020 Data"
    });
  }

  onMount(async () => {
    const raw2010 = await d3.csv("aggregated2010.csv");
    raw2010.forEach(d => {
      d.flip_ind_sum = +d.flip_ind_sum;
      const mhiVal = +d.mhi;
      d.income_bucket = mhiVal < 60000 ? "Low" : (mhiVal <= 100000 ? "Medium" : "High");
    });

    const raw2020 = await d3.csv("aggregated2020.csv");
    raw2020.forEach(d => {
      d.flip_ind_sum = +d.flip_ind_sum;
      const mhiVal = +d.mhi;
      d.income_bucket = mhiVal < 60000 ? "Low" : (mhiVal <= 100000 ? "Medium" : "High");
    });

    data2010 = raw2010;
    data2020 = raw2020;
    applyFilters();
  });

    function createStackedHistogram({ container, processedData, compareData, title }) {
      if (container) container.innerHTML = "";
      const margin = { top: 50, right: 30, bottom: 40, left: 60 };
      const width = 600;
      const height = 400;

      const svg = d3.select(container).append("svg").attr("width", width).attr("height", height);

      const { series, xMin, xMax, yMax } = processedData;

      const y = d3.scaleLinear().domain([0, yMax]).range([height - margin.bottom, margin.top]).nice();
      const x = d3.scaleLinear().domain([xMin, xMax]).range([margin.left, width - margin.right]);

      svg.append("g").attr("transform", `translate(0, ${height - margin.bottom})`).call(d3.axisBottom(x));
      svg.append("g").attr("transform", `translate(${margin.left}, 0)`).call(d3.axisLeft(y));

      const tooltip = d3.select(container)
        .append("div")
        .style("position", "absolute")
        .style("visibility", "hidden")
        .attr("class", "tooltip");

      const rects = svg.selectAll("g.layer")
        .data(series).join("g")
        .attr("class", "layer")
        .attr("fill", d => color(d.key))
        .selectAll("rect")
        .data(d => d).join("rect")
        .attr("x", d => x(d.data.x0))
        .attr("width", d => x(d.data.x1) - x(d.data.x0))
        .attr("y", d => y(d[1]))
        .attr("height", d => y(d[0]) - y(d[1]))
        .on("mouseover", function (event, d) {
          const binRange = `[${d.data.x0}, ${d.data.x1})`;
          tooltip.html(`<strong>Range:</strong> ${binRange}<br/>Low: ${d.data.Low}<br/>Medium: ${d.data.Medium}<br/>High: ${d.data.High}<br/>Total: ${d.data.Low + d.data.Medium + d.data.High}`)
            .style("visibility", "visible");
          //console.log("Hovered bin:", d.data.x0, d.data.x1);
          selectedBin.set({ x0: d.data.x0, x1: d.data.x1 });
        })
        .on("mousemove", function (event) {
          tooltip.style("top", event.pageY + 10 + "px").style("left", event.pageX + 10 + "px");
        })
        .on("mouseout", function () {
          tooltip.style("visibility", "hidden");
          selectedBin.set(null);
        });
      // Title
      svg
        .append("text")
        .attr("x", width / 2)
        .attr("y", margin.top / 2)
        .attr("class", "chart-title")
        .text(title || "Stacked Histogram (Bin = 5)");
      // X-axis label
      svg
        .append("text")
        .attr("x", width / 2)
        .attr("y", height - 5)
        .attr("text-anchor", "middle")
        .attr("class", "chart-axis")
        .text("Flips per tract (binned)");
      // Y-axis label
      svg
        .append("text")
        .attr("transform", "rotate(-90)")
        .attr("y", margin.left - 40)
        .attr("x", -(height / 2))
        .attr("text-anchor", "middle")
        .attr("class", "chart-axis")
        .text("Count of tracts");
      
      // TOOLTIP
      selectedBin.subscribe(bin => {
        if (bin == null) {
          // If store is null, remove highlight from all rects
          rects.attr("opacity", 1);
          tooltip.style("visibility", "hidden");
        } else {
          // We have a bin range => highlight matching bin
          rects.each(function (d) {
            if (d.data.x0 === bin.x0 && d.data.x1 === bin.x1) {

              d3.select(this).attr("opacity", 1);

              const binRange = `[${d.data.x0}, ${d.data.x1})`;
              const otherBin = compareData.find(b => b.x0 === d.data.x0 && b.x1 === d.data.x1) || { Low: 0, Medium: 0, High: 0 };

              const thisYear = d.data;
              const lastYear = otherBin;

              const totalThis = thisYear.Low + thisYear.Medium + thisYear.High;
              const totalLast = lastYear.Low + lastYear.Medium + lastYear.High;

              const pct = (now, prev) =>
                prev === 0 ? 'N/A' : `${((now - prev) / prev * 100).toFixed(1)}%`;

              // Check if this is the 2020 chart
              const is2020 = title.includes("2020");

              // Only show percentage change for 2020
              const html = is2020
              ? `
              <strong>Number of flips:</strong> ${binRange}<br/>
              <hr style="border: none; border-top: 1px solid #ddd;" />
              High: ${thisYear.High} (${pct(thisYear.High, lastYear.High)})<br/>
              Medium: ${thisYear.Medium} (${pct(thisYear.Medium, lastYear.Medium)})<br/>
              Low: ${thisYear.Low} (${pct(thisYear.Low, lastYear.Low)})<br/>
              Total: ${totalThis} (${pct(totalThis, totalLast)})<br/>
              `
              : `
              <strong>Number of flips:</strong> ${binRange}<br/>
              <hr style="border: none; border-top: 1px solid #ddd;" />
              High: ${thisYear.High}<br/>
              Medium: ${thisYear.Medium}<br/>
              Low: ${thisYear.Low}<br/>
              Total: ${totalThis}<br/>
              `;
              tooltip.html(html).style("visibility", "visible");
            } else {
              d3.select(this).attr("opacity", 0.6);
            }
          });
        }
      });
    }
  $: [selectedAgeGroup, selectedRace, selectedHousehold, selectedTenure, selectedInvestor, data2020] && applyFilters();
  </script>

  <div style="display: flex; flex-direction: column; align-items: center; gap: 2rem;">
    <!-- Filter Panel centered on top -->
    <FilterPanel
      {data2020}
      bind:selectedAgeGroup
      bind:selectedRace
      bind:selectedHousehold
      bind:selectedTenure
      bind:selectedInvestor
      {BUCKETS}
      {color}
    />

    <!-- Histograms side by side -->
    <div style="display: flex; gap: 2rem;">
      <div bind:this={chartContainer2010}></div>
      <div bind:this={chartContainer2020}></div>
    </div>
  </div>

  <style>
    select, input[type="range"] {
      display: block;
      margin-bottom: 1rem;
    }
</style>
