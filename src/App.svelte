<script>
  import * as d3 from 'd3';
  import {legendColor} from 'd3-svg-legend';
	import {onMount} from 'svelte';

  let width = 500;
	let height = 500;
	
	let data = [];
	onMount(async function() {
    // load data from csv
    let table = d3.csv('chi-health-data.csv', (d) => ({
          ...d,
          Name: d.Name.substring(6),
        }));

    let geocoord = d3.json('census-tracts.geojson')
      .then((d) => d.features);
    
    await Promise.all([table, geocoord]).then((values) => {
      console.log(values);
      let table = values[0];
      let geocoord = values[1];
      // join the variables we want to show on the map
      for (let i = 0; i < geocoord.length; i++) {
        let tract = geocoord[i].properties.name10;
        let found = false;
        let j = 0;
        while (!found && table.length > j) {
          if (table[j].Name == tract) {
            found = true;
            data.push(geocoord[i]);
            data[data.length - 1].properties['pop'] = table[j]['Population']
            // data[data.length - 1].properties['popChange'] = table[j]['CIP_2010-2020']
            // data[data.length - 1].properties['rentRatio'] = table[j]['RBU_2018-2022'] / table[j]['RBS_2018-2022']
            data[data.length - 1].properties['walk'] = table[j]['EKW_2022']
            // grab other variables as needed
          } else {
            j++;
          }
        }
      }
      console.log(data);
    });
	});

  let proj = d3.geoMercator()
    .scale(40000)
    .center([-87.39, 41.52])
    .translate([width, height]);
  let path = d3.geoPath().projection(proj);

  // $: scale = d3.scaleOrdinal(d3.schemeDark2)
  //   .domain(d3.extent(data.map((d) => +d.properties.name10)));
  // $: scale = d3.scaleSequential(d3.interpolateGreens)
  //   .domain(d3.extent(data.map((d) => +d.properties.walk)));
  $: scale = d3.scaleSequential(d3.interpolatePiYG)
    .domain([d3.min(data.map((d) => +d.properties.pop)), d3.median(data.map((d) => +d.properties.pop)), d3.max(data.map((d) => +d.properties.pop))]);

  let legend;
  $: {	
		const colorLegend = legendColor()
			.shape('square')
      .cells(9)
			.scale(scale);
		
		d3.select(legend)
			.call(colorLegend);
	}

</script>

<main>
  <h1>Chicago Census Data</h1>
  
  <svg {width} {height}>
    {#each data as d}
      <path class = "geo"
        style = "fill: {scale(+d.properties.pop)};"
        d={path(d)}/>
    {/each}

    <g transform="translate({width - 100}, 50)"
		  bind:this={legend} />
  </svg>

</main>

<style>
  .geo {
    stroke: grey;
    stroke-width: 1px;
  }
</style>
