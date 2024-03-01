<script>
    import { InternSet, hierarchy, pack, range, scaleOrdinal, schemeTableau10 } from 'd3';
    export let bubble_data;
    const width = 780; //the margin top, bottom, left, right margin offset relative to the radius
    const padding = 13; // the all padding all around each circle, in pixels
    const margin = 1; // the all margin all around, in pixels
    const textColor = 'black'; //the color of the text
    const fill = '#ccc'; // a static fill color, if no group channel is specified
    const fillOpacity = 0.9; // the fill opacity of the bubbles
    const strokeColor = 'none'; // a static stroke around the bubbles
    const strokeWidth = 3; // the stroke width around the bubbles, if any
    const strokeOpacity = 1; // the stroke opacity around the bubbles, if any
    const height = width; // the outer height of the chart, in pixels
    const marginLeft = margin; // the left margin, in pixels
    const marginRight = margin; // the right margin, in pixels
    const marginTop = margin; // the top margin, in pixels
    const marginBottom = margin; // the bottom margin, in pixels
  
    //update link to the desired address path or remove.
    const link = (d) => ``;
    // Compute the values.
    const dVals = bubble_data.map((el) => el);
    const vVals = bubble_data.map((el) => el.value);
    const gVals = bubble_data.map((el) => el.id.split('.')[0]);
    const iVals = range(vVals.length).filter(i => vVals[i] > 0);
  
    let groups = iVals.map(i => gVals[i]);
    groups = new InternSet(groups);
    const colorScale = scaleOrdinal(groups, schemeTableau10);  

  
    // // Compute labels.
    const lVals = bubble_data.map((el) => [...el.id.split('.').pop().split(/(?=[A-Z][a-z])/g), el.value.toLocaleString('fr')].join('\n'));
    const tVals = bubble_data.map((el) => `${el.id}\n${el.value.toLocaleString('fr')}`);
  
    const uid = `O-${Math.random().toString(16).slice(2)}`;
  
    const root = pack()
    .size([width - marginLeft - marginRight, height - marginTop - marginBottom])
    .padding(padding)
    (hierarchy({children: iVals})
        .sum(i => vVals[i]));
  </script>
    
  <svg {width} {height} viewBox="{-marginLeft} {-marginTop} {width} {height}" fill={textColor}>
      {#each root.leaves() as leaf, i}
        <a href={link === null ? null : link(dVals[leaf.data])} target="_blank" rel="noopener noreferrer">
          <g class='node' transform="translate({(leaf.x)},{(leaf.y)})">
            <circle id="node-{i}"
              stroke={strokeColor} stroke-width={strokeWidth} stroke-opacity={strokeOpacity}
              fill={gVals ? colorScale(gVals[leaf.data]) : fill == null ? 'none' : fill}
              fill-opacity={fillOpacity}
              r={leaf.r}
            >
              <title>{tVals[i].split('.').pop()}</title>
            </circle>
            <clipPath id={`${uid}-clip-${leaf.data}`}>
              <circle r={leaf.r}></circle>
            </clipPath>
            <text clip-path={`url(${new URL(`#${uid}-clip-${leaf.data}`, location)})`}>
              {#each `${lVals[leaf.data]}`.split(/\n/g) as subtext, j}
                <tspan 
                  x='0'  
                  y={`${j - `${lVals[leaf.data]}`.split(/\n/g).length / 2 + 0.85}em`}
                  fill-opacity={j === `${lVals[leaf.data]}`.split(/\n/g).length - 1 ? 0.7 : null}
                  font-size={leaf.r * 0.3}
                >
                  {subtext}
                </tspan>
              {/each}
            </text>
          </g>
        </a>
      {/each}
  </svg>
  
  <style>
    svg {
      max-width: 100%;
      height: auto;
      height: intrinsic;
      font-size: 10;
      font-family: sans-serif;
      text-anchor: middle;
    }
  
    .node {
      cursor: pointer;
    }
  
    .node:hover {
      font-weight: 700;
    }
  </style>