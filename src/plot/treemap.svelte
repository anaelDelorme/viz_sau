<!-- Treemap.svelte -->
<script>
    import * as d3 from "d3";
    import Swatches from "./swatches.svelte";
  
    // Receive plot treemap_data as prop.
    export let treemap_data;
    // Specify the chart’s dimensions.
    const width = 1500;
    const height = 1154;
  
    // Specify the color scale.
    const color = d3.scaleOrdinal(
      treemap_data.children.map((d) => d.name),
      d3.schemeTableau10,
    );
  
    // Specify formatting function.
    const format = d3.format(",d");
    
  
    // Specify initial tiling method.
    let tile = d3.treemapBinary;
  
    // Compute the layout.
    $: root = d3
      .treemap()
      .tile(tile) // e.g., d3.treemapSquarify
      .size([width, height])
      .padding(1)
      .round(true)(
      d3
        .hierarchy(treemap_data)
        .sum((d) => d.value)
        .sort((a, b) => b.value - a.value),
    );
  
    // Helper function to recursively find the parent of a given leaf at depth 1.
    function getFirstDepthParent(leaf) {
      return leaf.depth > 1 ? getFirstDepthParent(leaf.parent) : leaf;
    }
  </script>
  

  
  <!-- Legend -->
  <Swatches {color} />
  
  <svg
    {width}
    {height}
    viewBox="0 0 {width} {height}"
    style="max-width: 100%; height: auto;"
  >
    {#each root.leaves() as leaf, leafIndex}

      <!-- Get nodes of current leaf -->
      {@const nodes = leaf.data.name
        .split(/(?=[A-Z][a-z])|\s+/g)
        .concat(leaf.value.toLocaleString('fr'))}
  
      <!-- Add a cell for each leaf of the hierarchy -->
      <g transform="translate({leaf.x0},{leaf.y0})">
        <!-- Add a tooltip -->
        <title>
          {`${leaf
            .ancestors()
            .reverse()
            .map((leaf) => leaf.data.name)
            .join(".")
            .split(".")
            .pop()}\n${leaf.value.toLocaleString('fr')}`}
        </title>
  
        <!-- Add a color rectangle -->
        <rect
          id="rect-{leafIndex}"
          fill={color(getFirstDepthParent(leaf).data.name)}
          fill-opacity={0.8}
          width={leaf.x1 - leaf.x0}
          height={leaf.y1 - leaf.y0}
        />
  
        <!-- Add a clipPath to ensure text does not overflow -->
        <clipPath id="clip-{leafIndex}">
          <use href="#rect-{leafIndex}" />
        </clipPath>
  
        <!-- Add multiline text. The last line shows the value and has a specific formatting. -->
        <text clip-path="url(#clip-{leafIndex})">
          {#each nodes as node, nodeIndex}
            <tspan
              x="3"
              y="{(nodeIndex === nodes.length - 1) * 0.3 +
                1.1 +
                nodeIndex * 0.9}em"
              fill-opacity={nodeIndex === nodes.length - 1 ? 0.7 : null}
            >
              {node}
            </tspan>
          {/each}
        </text>
      </g>
    {/each}
  </svg>