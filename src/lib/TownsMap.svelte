<script>
  import * as d3 from 'd3';
  import TownLines from '../data/mygeodata/mygeodata_merged_ri.json';
  import MaLines from '../data/mygeodata/mygeodata_merged_ma.json';
  import CtLines from '../data/mygeodata/mygeodata_merged_ct.json';

  export let townFromMap = '';

  import {onMount} from 'svelte';

  let htxt = '_';
  let townData;

  const colors = {
    RI: '#cbe7be',
    CT: '#e8e1c9',
    MA: '#e8d6c9',
    water: '#c3f4fe'
  }

  let raceData;
  let deomograpicData;
  let svg = null;
  const width = 750;
  const height = 700;

  let projection = d3.geoAlbers()
    .scale(43500)
    .center([41, 71])
    .translate([475, -21320]);

  let geoGenerator = d3.geoPath()
    .projection(projection)

  geoGenerator(TownLines)

  const onChange = function(raceData) {
    chart();
  }

  const chart = () => {
    
    svg = d3.select('#vis_container')
      .append('svg')
      .attr('id', 'mapsv')
      .attr('width', width)
      .attr('height', height);

    let bac = d3.select('#mapsv')
      .append('rect')
      .attr('x', 0)
      .attr('y', 0)
      .attr('rx', 15)
      .attr('width', width)
      .attr('height', height)
      .style('fill', colors.water);


    let ma = d3.select('#mapsv')
      .selectAll('.ma')
      .data(MaLines.features)
      .join('path')
      .attr('d', geoGenerator)
      .attr('class', 'ma')
      .attr('nm', d =>  `${d.properties.NAME}, MA`)
      .style('stroke', d3.rgb(colors.MA).darker(2))
      .style('fill', d3.rgb(colors.MA))
      .style('stroke-width', 1)
      .style('stroke-opacity', 0.4)
      .on('mouseover', function (d, i) {
        d3.select(this).transition()
          .duration('100')
          .style('fill', d3.rgb(colors.MA).darker());

          htxt = d3.select(this).attr('nm');
      })
      .on('mouseout', function (d, i) {
        d3.select(this).transition()
              .duration('200')
              .style('fill', d3.rgb(colors.MA));
        htxt = '_';
      });  

      let ct = d3.select('#mapsv')
      .selectAll('.ct')
      .data(CtLines.features)
      .join('path')
      .attr('d', geoGenerator)
      .attr('class', 'ct')
      .attr('nm', d => `${d.properties.NAME}, CT`)
      .style('stroke', d3.rgb(colors.CT).darker(2))
      .style('fill', d3.rgb(colors.CT))
      .style('stroke-width', 1)
      .style('stroke-opacity', 0.4)
      .on('mouseover', function (d, i) {
        d3.select(this).transition()
          .duration('100')
          .style('fill', d3.rgb(colors.CT).darker());

          htxt = d3.select(this).attr('nm');
      })
      .on('mouseout', function (d, i) {
        d3.select(this).transition()
              .duration('200')
              .style('fill', d3.rgb(colors.CT));
        htxt = '_';
      }); 
      
      

    let t = d3.select('#mapsv')
      .selectAll('.townshp')
      .data(TownLines.features)
      .join('path')
      .attr('d', geoGenerator)
      .attr('class', 'townshp')
      .attr('nm', d => d.properties.NAME)
      .style('stroke', d3.rgb(colors.RI).darker(2))
      .style('fill', d3.rgb(colors.RI))
      .style('stroke-width', 2)
      .on('mouseover', function (d, i) {
        d3.select(this).transition()
          .duration('100')
          .style('fill', d3.rgb(colors.RI).darker())
          .style('stroke-width', 3);

          htxt = d3.select(this).attr('nm');
          getTownData(htxt);
      })
      .on('mouseout', function (d, i) {
        d3.select(this).transition()
              .duration('200')
              .style('fill', d3.rgb(colors.RI))
              .style('stroke-width', 2);
        htxt = '_';
        getTownData('State Total');
      });

 

    let bor = d3.select('#mapsv')
      .append('rect')
      .attr('x', 1.5)
      .attr('y', 1.5)
      //.attr('rx', 15)
      .attr('width', width-3)
      .attr('height', height-3)
      .style('stroke', d3.rgb(colors.RI).darker(2))
      .style('fill', 'none')
      .style('stroke-width', 3);
  }

  let getTownData = (name) => {
    //console.log(name)
    townFromMap = name;
    
  }

  onMount(
    async () => {
      raceData = await d3.csv('./src/data/ri2020byRace.csv')
    } 
  )
  $: onChange(raceData);

</script>

<div id='vis_parent'>
  <div id='vis_container'></div>
  {htxt}
</div>

<style>
  #vis_container {
  }

  #vis_parent {
  }

  #mapsv {
  }
  

</style>