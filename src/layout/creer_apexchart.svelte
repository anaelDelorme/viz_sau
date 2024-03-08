<script>
  import ApexCharts from 'apexcharts';
  import { onMount, onDestroy } from 'svelte';
 

  export let animate = true;
  
  export let data;
  //console.log("Les données sont :", data);

  export let name_series;
  export let title_graph="";
  export let type_graph;
  export let xkey;
  export let unit = "";
  export let nb_decimal = 1;
  export let colors = ["#206095", "#ffe552", "#34cb6a"];
  export let ykey;
  export let horizontal = false;
  export let distributed = false;
  export let unit_total_donut = "kk";
  export let nb_decimal_total_donut = 0;
  let el; // Chart DOM element

  //console.log(ykey);


  function setOptions(type_graph, name_series, xkey, ykey, nb_decimal, colors, animate, horizontal, distributed){
    
    if (typeof ykey === 'string') {
        ykey = [ykey];
    }
    if (typeof name_series === 'string') {
        name_series = [name_series];
    }
    const xkey_values = data.map((item) => item[xkey]);
    let series_data;
    let data_labels;
    if (type_graph === "pie" || type_graph==="donut"){
       series_data = data.map(item => item[ykey]);
       data_labels = data.map(item => item[xkey]);
       console.log("series_data"+series_data); 
    }else{
      series_data = ykey.map((ykey_detail, index) => {
        return {
            name: name_series[index],
            data: data.map((item) => item[ykey_detail])
        };
    });
    }
    console.log("series_data : "+ series_data)

    let options_return = {
    chart: {
      type: type_graph,
      dropShadow: {
        enabled: true,
        color: "#000",
        top: 18,
        left: 7,
        blur: 5,
        opacity: 0.2,
      },
      toolbar: {
        show: false,
      },
    },
    series: series_data,
    plotOptions: {
          bar: {
            horizontal: horizontal,
            distributed: distributed,
          },
          pie:{
            donut: {
            size: '50%',
            background: 'transparent',
            labels: {
              show: true,
              name: {
                show: true,
                fontSize: '22px',
                fontFamily: 'Helvetica, Arial, sans-serif',
                fontWeight: 600,
                color: undefined,
                offsetY: -10,
                formatter: function (val) {
                  return val
                }
              },
              value: {
                show: true,
                fontSize: '18px',
                fontFamily: 'Helvetica, Arial, sans-serif',
                fontWeight: 400,
                color: undefined,
                offsetY: 16,
                formatter: function (val) {
                  return val.toFixed(nb_decimal_total_donut) + unit_total_donut;
                }
              },
              total: {
                show: true,
                showAlways: true,
                label: 'Total',
                fontSize: '22px',
                fontFamily: 'Helvetica, Arial, sans-serif',
                fontWeight: 600,
                color: '#373d3f',
                formatter: function (w) {
                  return w.globals.seriesTotals.reduce((a, b) => {
                    return a + b
                  }, 0)
                }
              }
            }
            }
          }     
        },

    markers: {
      size: 3,
    },
    xaxis: {
      categories: xkey_values
    },
    yaxis: {
      labels: {
        formatter: function (value) {
          if (typeof value === 'number') {
            return value.toFixed(nb_decimal) + unit;
          } else {
           // console.error('La valeur passée en argument n\'est pas un nombre :', value);
            return value + unit;
          }
        },
      },
    },
    dataLabels: {
      formatter: function (value) {
          if (typeof value === 'number') {
            return value.toFixed(nb_decimal) + unit;
          } else {
           // console.error('La valeur passée en argument n\'est pas un nombre :', value);
            return value + unit;
          }
        },
      enabled: true,
      positions: top,
    },
    fill: {
      colors: colors,
    },
    colors: colors,
    animations: {
        enabled: animate
      },
    legend:{
      show: true,
    }
  };
  if (type_graph === "pie" || type_graph === "donut") {
      options_return.labels = data_labels;
    }
   // console.log("OPTIONS RETURN colors",options_return.colors, type_graph)
   return options_return;
  }

  let options = setOptions(type_graph, name_series, xkey, ykey, nb_decimal, colors,animate, horizontal, distributed)

  let prevOptions = options;
  let ref;
  let chart;
  
    onMount(async () => {
      const ApexCharts = (await import('apexcharts')).default;
      chart = new ApexCharts(ref, { ...options });
      chart.render();
    });
  
    onDestroy(() => {
      chart?.destroy();

    });
  
    function updateOptions(type_graph, name_series, xkey, ykey, nb_decimal, colors, animate, horizontal, distributed) {
      options = setOptions(type_graph, name_series, xkey, ykey, nb_decimal, colors, animate, horizontal, distributed)
      if (chart !== undefined) {
        if (options !== undefined && options !== prevOptions) {
          //console.log("Options changent")
          chart.destroy();
          chart = new ApexCharts(ref, { ...options });
          chart.render(options)
        } 
      }
      prevOptions = options;
    }

    $: updateOptions(type_graph, name_series, xkey, ykey, nb_decimal, colors, animate, horizontal, distributed)


</script>

<div bind:this={el}>
  <h2> </h2>
  <div class="container" bind:this={ref} />
</div>

