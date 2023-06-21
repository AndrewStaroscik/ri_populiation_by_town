<script>

  import TownsMap from './lib/TownsMap.svelte';
  import ChartDiv from './lib/barChart.svelte';

  

  let dat = 'State Total';

  let onChange = function(dat) {
    // console.log(`from main: ${dat}`);
  }

  $: onChange(dat)

</script>


<main>


  <div id="mainwrapper">
    <div id="chartwrap">
      <ChartDiv townName = {dat}/>
    </div>
    <div id="mapwrap">
      <TownsMap  bind:townFromMap={dat}/>
    </div>
  </div>
  <div id='srcinfo'>
    Data Source: <a href='https://planning.ri.gov/planning-areas/data-center/ri-data-center-census-data/census-and-acs-data'>Rhode Island Division of statewide Planning</a>
  </div>
  


</main>

<style>
  #mainwrapper {
    position: relative;
    width: 1020px;
    height: 750px;
  }

  #mapwrap {
    position: absolute;
    left: 0;
    bottom: 0;
    z-index: 1;

  }
  #chartwrap {
    position: absolute;
    top: 0;
    right: 0;
    border: #232323 solid 2px;
    border-radius: 10px;
    background-color: #fff;
    z-index: 10;
  }

  #srcinfo {
    text-align: right;
    font-size: 0.75em;
    color: #676767;
  }

</style>
