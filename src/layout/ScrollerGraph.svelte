<script>
    import { onMount } from 'svelte';
    import Scroller from './Scroller.svelte';
    import Apexcharts from "./creer_apexchart.svelte";
    import Media from './Media.svelte';
    import InjectableHTMLTag from './injectableHtml.svelte';
    import Em from "../ui/Em.svelte";

    // Config
	const threshold = 0.65;
	// State
	let id = {}; // Object to hold visible section IDs of Scroller components
	let idPrev = {}; // Object to keep track of previous IDs, to compare for changes
    export let chartsConfig;
    export let idPrefix = '';
  
    let scrollerId = '';

    let currentConfig = chartsConfig[0];
    
 // Code to run Scroller actions when new caption IDs come into view
 function runActionsScroller(id_select) {
  console.log("runActionsScroller", id_select);
  if (typeof id_select !== 'string') {
  } else {

    var configObject = chartsConfig.find(function(item) {
      return item.id === id_select;
    });
    if (currentConfig !== configObject) {
      currentConfig = configObject;
    }
  }
};

  $: id && runActionsScroller(id); // Run above code when 'id' object changes
  onMount(() => {
      scrollerId = chartsConfig[0].id;
    });
    console.log("currentConfig :",currentConfig.title_graph)
    

    
  </script>
  
  <Scroller {threshold} bind:id={id} splitscreen={true}>
    <div slot="background">
        <figure>
          <div class="col-wide height-full">
            <div class="chart">
               
                {#if currentConfig}
                <Media caption={currentConfig.title_graph} col="wide">
              <Apexcharts
                    type_graph={currentConfig.type_graph}
                  data={currentConfig.data_chart}
                  name_series={currentConfig.name_series}
                  xkey= {currentConfig.xkey}
                  ykey={currentConfig.ykey}
                  unit={currentConfig.unit}
                  nb_decimal={currentConfig.nb_decimal}
                  colors={currentConfig.colors_chart}
                  animate={currentConfig.animate}
                  horizontal={currentConfig.horizontal}
                  distributed={currentConfig.distributed}
                /> 
              </Media> 
                {/if}
             


            </div>
          </div>
        </figure>
      </div>
  
    <div slot="foreground">
      {#each chartsConfig as config, i}
        <section data-id={`${idPrefix}chart${i.toString().padStart(2, '0')}`}>
          <div class="col-medium">
            <p>
              {@html config.text} 
            </p>
            
          </div>
        </section>
      {/each}
    </div>
  </Scroller>
