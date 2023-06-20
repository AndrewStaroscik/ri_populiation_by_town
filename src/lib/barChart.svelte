<script>

  import {onMount} from 'svelte';
  import * as d3 from 'd3';

  import wrap from './wrap.js';

  let deomographicData;
  let townKeyVal = {}; // the number in the data object leading to the data
  let demogroups = [];
  let activeTown = 'State Total';
  let activeData;
  let totoalCount;

  const races = {
    'One': 'One Race', 
    'White': 'White',
    'AfrAmer': 'African American',
    'AIAN': 'American Indian & Alaskan Native',
    'Asian': 'Asian',
    'NH_PI': 'Native Hawaiian & Pacific Islander',
    'Other': 'Other',
    'Two_or_more': 'Two or More Races' 
  }


  let svg = null,
      g = null,
      xAxis,
      yAxis;
  const margin = {top: 20, right: 30, bottom: 50, left: 70},
        width = 500 - margin.left - margin.right,
        height = 250 - margin.top - margin.bottom; 

  let formatNumb = d3.format(',');
  
  export let townName = 'State Total';

  const getTownObjectNames = function(deomogrDat) {
    let done = 0;
    for (let key in deomogrDat) {
      townKeyVal[deomogrDat[key]['cit_twn']] = key;
      if (done == 0) {
        for (let twnkey in deomogrDat[key]) {
          if (twnkey != 'cit_twn') {
            demogroups.push(twnkey)
            done = 1;
          }
        }
      }
    }
  }

  
  const onChange = function(nm) {
    if (deomographicData) {
      let data = formatData(deomographicData[townKeyVal[nm]])
      update(data);
    }
  }

  const chart = (data) => {    
    if (svg == null) {
      svg = d3.select('#bar_container')
        .append('svg')
        .attr('id', 'barsv')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom);

    svg.append('rect')
      .attr('x', 0)
      .attr('y', 0)
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom)
      .style('fill', '#fff');

      g = svg.append('g')
        .attr('transform', `translate(${margin.left}, ${margin.top})`);

      

      xAxis = d3.scaleBand()
        .range([0,width])
        .domain(data.map((d) => d.race))
        .padding(0.2)

      yAxis = d3.scaleLinear()
        .range([height, 0])
        .domain([0,d3.max(data, d => d.value)]);

          
      g.append('g')
        .attr('transform', `translate(0, ${height})`)
        .attr('class', 'axis')
        .call(d3.axisBottom(xAxis))
        .selectAll('text')
          .style('text-anchor', 'center')
          .style('fill', '#565656')
          .call(wrap, xAxis.bandwidth());

      g.append('g')
        .attr('id', 'y_axis')
        .call(d3.axisLeft(yAxis)
          .ticks(5)
        )
        .selectAll('text')
          .style('fill', '#565656');

      svg.append("text")
        .attr("text-anchor", "end")
        .attr("transform", "rotate(-90)")
        .attr("y", margin.left/4)
        .attr("x", -height/2)
        .style('fill', '#676767')
        .text("Count")

      // Bars
      g.selectAll('.bar')
        .data(data)
        .enter()
        .append('rect')
        .attr('class', 'bar')
        .attr('x', function(d) { return xAxis(d.race); })
        .attr('y', function(d) { return yAxis(d.value); })
        .attr('width', xAxis.bandwidth())
        .attr('height', d => height - yAxis(d.value))
        .style('fill', d3.rgb('#cbe7be').darker())
        .style('stroke', '#000')
        .style('stroke-width', .5);   
        
      g.selectAll('.bar_txt')
        .data(data)
        .enter()
        .append('text')
        .attr('class', 'bar_txt')
        .attr('x', d => xAxis(d.race) + xAxis.bandwidth()/2)
        .attr('y', d => yAxis(d.value) - 7)
        .attr('text-anchor', 'middle')
        .style('fill', d3.rgb('#cbe7be').darker())
        .style('font-size', '14px')
        .text(d => d3.format(',')(+d.value));

      

    
    }
  }

  const update = (data) => {
    
    // update yAxis
    yAxis
      .domain([0,d3.max(data, d => d.value)]);
    
    g.select("#y_axis")
	    .transition().duration(1000)
	    .call(d3.axisLeft(yAxis)
        .ticks(5)
      )
      .selectAll('text')
      .style('fill', '#565656');

    // update data
    g.selectAll('.bar')
      .data(data)
      .transition().duration(1000)
      .attr('y', function(d) { return yAxis(d.value); })
      .attr('height', function(d) { return height - yAxis(d.value); });


      g.selectAll('.bar_txt')
        .data(data)
        .transition().duration(1000)
        .attr('y', d => yAxis(d.value) - 7)
        .tween("text", function(d) {
          var i = d3.interpolate(
            parseFloat(this.textContent.replace(/,/g, '')) , 
            d.value),
            prec = (d + "").split("."),
            round = (prec.length > 1) ? Math.pow(10, prec[1].length) : 1;

          return function(t) {
            this.textContent = d3.format(',')(Math.round(i(t) * round) / round);
          };
    });



  }

  let formatData = (dataObj) => {
    let data = [];
    for (let key in dataObj) {
      if (key != 'cit_twn' && key != 'One') {
        let tempObj = {}
        tempObj.race = races[key];
        tempObj.value = parseFloat(dataObj[key].replace(/,/g, ''))
        data.push(tempObj)
      }
    }
    let tot = 0
    for (let el in data) {
      tot += data[el].value
    }
    totoalCount = tot;
    return data;
  }

  onMount(
    async () => {
      deomographicData = await d3.csv('./src/data/ri2020byRace.csv');
      getTownObjectNames(deomographicData);
      let data = formatData(deomographicData[townKeyVal['State Total']])
      chart(data)

      //onChange('State Total')
      
    } 
  )
  $: onChange(townName);

  

	function formatMobile(tick) {
		return "'" + tick.toString().slice(-2);
	}





</script>
<p class='cntr'>{townName}, Population: {d3.format(',')(totoalCount)}</p>
<div id="bar_container">

</div>


  




<style>
	h2, .cntr{
		text-align: center;
	}

  .bar_container {
    z-index: 5;
  }


  /* .tick {
		font-family: Helvetica, Arial;
		font-size: .725em;
		font-weight: 200;
		fill: #f00;
		text-anchor: start;
	} */

  /* .x-axis .tick text {
		text-anchor: middle;
    fill: #f00;
	}  */

	/* .chart {

		width: 100%;
		max-width: 500px;
		margin: 0 auto;
	} */

	/* #barsv {
		position: relative;
		width: 100%;
		height: 200px;
	} */

	/* 
	} */

	/* .tick line {
		stroke: #e2e2e2;
		stroke-dasharray: 2;
	}



	.tick.tick-0 line {
		stroke-dasharray: 0;
	}

	*/


</style>