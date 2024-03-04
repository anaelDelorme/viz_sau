<script>
    import ApexCharts, { ApexOptions } from 'apexcharts';
    import { onMount, onDestroy } from 'svelte';
 
    export let options;
    export let animate = true;
  
    let prevOptions;
    let ref;
    let chart;
  
    onMount(async () => {
      const ApexCharts = (await import('apexcharts')).default;
      chart = new ApexCharts(ref, { ...options });
      chart.render();
      console.log("rendu")
    });
  
    onDestroy(() => {
      chart?.destroy();
      console.log("détruit")

    });
  
    $: {
      if (chart !== undefined) {
        if (options !== undefined && options !== prevOptions) {
          chart.updateOptions({ ...options }, true, animate);
          console.log("mise à jour")

        } 
      }
      prevOptions = options;
    }
  </script>
  
  <div class="container" bind:this={ref} />
  
  <style>
    .container {
      width: 100%;
      height: 100%;
    }
  </style>