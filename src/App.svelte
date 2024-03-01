<script>
	// CORE IMPORTS
	import { setContext, onMount } from "svelte";
	
	import { getMotion } from "./utils.js";
	import { themes } from "./config.js";
	import ONSHeader from "./layout/ONSHeader.svelte";
	import ONSFooter from "./layout/ONSFooter.svelte";
	import Header from "./layout/Header.svelte";
	import Section from "./layout/Section.svelte";
	import Media from "./layout/Media.svelte";
	import Scroller from "./layout/Scroller.svelte";
	import Filler from "./layout/Filler.svelte";
	import Divider from "./layout/Divider.svelte";
	import Toggle from "./ui/Toggle.svelte";
	import Arrow from "./ui/Arrow.svelte";
	import Em from "./ui/Em.svelte";

	
	import Treemap from "./plot/treemap.svelte";
	import treemap_data from "./plot/treemap.json"

	import Bubblechart from "./plot/bubblechart.svelte";
	import bubble_data_init from './plot/bubble_data.json';
	import bubble_data_froment from './plot/bubble_data_froment.json';
	import bubble_data_avoine from './plot/bubble_data_avoine.json';
	import bubble_data_mais from './plot/bubble_data_maïs.json';
	import bubble_data_orge_meteil from './plot/bubble_data_orge_meteil.json';
	import bubble_data_seigle from './plot/bubble_data_seigle.json';
	import bubble_data_sarrasin from './plot/bubble_data_sarrasin.json';
	function getRightWord(id) {
		if (typeof id !== 'string') {
        return ''; // ou retournez une valeur par défaut appropriée
    }

    const words = id.split('.');
    return words[1]; // Le mot de droite
	}
	const bubble_data_init_barchart = bubble_data_init.map(item => ({
		...item,
		id: getRightWord(item.id),
		}));

	// DEMO-SPECIFIC IMPORTS
	
	import bbox from "@turf/bbox";
	import { getData, setColors, getTopo, getBreaks, getColor } from "./utils.js";
	import { colors, units } from "./config.js";
	import { ScatterChart, LineChart, BarChart } from "@onsvisual/svelte-charts";
	import { Map, MapSource, MapLayer, MapTooltip } from "@onsvisual/svelte-maps";

	// CORE CONFIG (COLOUR THEMES)
	// Set theme globally (options are 'light', 'dark' or 'lightblue')
	let theme = "light";
	setContext("theme", theme);
	setColors(themes, theme);

	// CONFIG FOR SCROLLER COMPONENTS
	// Config
	const threshold = 0.65;
	// State
	let animation = getMotion(); // Set animation preference depending on browser preference
	let id = {}; // Object to hold visible section IDs of Scroller components
	let idPrev = {}; // Object to keep track of previous IDs, to compare for changes
	onMount(() => {
		idPrev = {...id};
	});

	// DEMO-SPECIFIC CONFIG
	// Constants
	const datasets = ["region", "departement"];
	//const topojson = "./data/geo_lad2021.json";
	//const topojsonFR = "./data/dep2021.json";
	import topojsonFR from "./plot/dep2021.json";

	const mapstyle = "https://bothness.github.io/ons-basemaps/data/style-omt.json";
	const mapbounds = {
		uk: [
			[-9, 49 ],
			[ 2, 61 ]
		],
		fr: [
			[-5.1, 51.1 ], 
			[9.6, 41.3 ]  
		] 
	};

	// Data
	let data = {departement: {}, region: {}};
	let metadata = {departement: {}, region: {}};
	//let geojson;
	//let geojsonFR;

	// Element bindings
	let map = null; // Bound to mapbox 'map' instance once initialised

	// State
	let hovered; // Hovered departement (chart or map)
	let selected; // Selected departement (chart or map)
	$: region = selected && metadata.departement.lookup ? metadata.departement.lookup[selected].parent : null; // Gets region code for 'selected'
	let mapHighlighted = []; // Highlighted departement (map only)
	let choix = 'tous'; // rKey (radius) for scatter chart
	let mapKey = "nb_exp"; // Key for data to be displayed on map
	let explore = false; // Allows chart/map interactivity to be toggled on/off

	// FUNCTIONS (INCL. SCROLLER ACTIONS)
	
	// Functions for chart and map on:select and on:hover events
	function doSelect(e) {
		console.log(e);
		selected = e.detail.id;
		if (e.detail.feature) fitById(selected); // Fit map if select event comes from map
	}
	function doHover(e) {
		hovered = e.detail.id;
	}

	// Functions for map component
	function fitBounds(bounds) {
		if (map) {
			map.fitBounds(bounds, {animate: animation, padding: 30});
		}
	}
	function fitById(id) {
		if (geojsonFR && id) {
			let feature = geojsonFR.features.find(d => d.properties.code == id);
			let bounds = bbox(feature.geometry);
			fitBounds(bounds);
		}
	}

	// Actions for Scroller components
	const actions = {
		map: { // Actions for <Scroller/> with id="map"
			map01: () => { // Action for <section/> with data-id="map01"
				//fitBounds(mapbounds.uk);
				fitBounds(mapbounds.fr);
				mapKey = "nb_exp";
				mapHighlighted = [];
				explore = false;
			},
			map02: () => {
				//fitBounds(mapbounds.uk);
				fitBounds(mapbounds.fr);
				mapKey = "surf_2022";
				mapHighlighted = [];
				explore = false;
			},
			map03: () => {
				let hl = [...data.departement.indicators].sort((a, b) => b.surf_2022 - a.surf_2022)[0];
				fitById(hl.code);
				mapKey = "surf_2022";
				mapHighlighted = [hl.code];
				explore = false;
			},
			map04: () => {
				//fitBounds(mapbounds.uk);
				fitBounds(mapbounds.fr);
				mapKey = "surf_2022";
				mapHighlighted = [];
				explore = true;
			}
		},
		chart: {
			chart00: () => {
				choix = "tous";
			},
			chart01: () => {
				choix = "Froment";
			},
			chart02: () => {
				choix = "Avoine";
			},
			chart03: () => {
				choix = "Seigle";
			},
			chart04: () => {
				choix = "Orge, Méteil";
			},
			chart05: () => {
				choix = "Sarrasin";
			},
			chart06: () => {
				choix = "Maïs";
			},
			chart07:() =>{
				choix = "Barchart"
			}
		}
	};

	// Code to run Scroller actions when new caption IDs come into view
	function runActions(codes = []) {
		
		codes.forEach(code => {
			if (id[code] != idPrev[code]) {
				if (actions[code][id[code]]) {
					actions[code][id[code]]();
					
				}
				idPrev[code] = id[code];
			}
		});
	}
	$: id && runActions(Object.keys(actions)); // Run above code when 'id' object changes

	// INITIALISATION CODE
	datasets.forEach(geo => {
		getData(`./data/data_${geo}.csv`)
		.then(arr => {
			let meta = arr.map(d => ({
				code: d.code,
				//name: d.name,
				parent: d.parent ? d.parent : null
			}));
			let lookup = {};
			meta.forEach(d => {
				lookup[d.code] = d;
			});
			metadata[geo].array = meta;
			metadata[geo].lookup = lookup;

			let indicators = arr.map((d, i) => ({
				...meta[i],
				//area: d.area,
				surf_2022: d['2022'],
				surf_2021: d['2021'],
				density: d.density,
				nb_exp: d.nb_exp
			}));

			if (geo == "departement") {
				['nb_exp', 'surf_2022'].forEach(key => {
					let values = indicators.map(d => d[key]).sort((a, b) => a - b);
					let breaks = getBreaks(values);
					indicators.forEach((d, i) => indicators[i][key + '_color'] = getColor(d[key], breaks, colors.seq));
				});
			}
			data[geo].indicators = indicators;

			let years = [
				2007, 2008, 2009, 2010,
				2011, 2012, 2013, 2014, 2015,
				2016, 2017, 2018, 2019, 2020, 2021, 2022
			];

			let timeseries = [];
			arr.forEach(d => {
				years.forEach(year => {
					timeseries.push({
						code: d.code,
						//name: d.name,
						value: d[year],
						year
					});
				});
			});
			data[geo].timeseries = timeseries;
		});
	});

	/*getTopo(topojsonFR, 'geog')
	.then(geo => {
		geo.features.sort((a, b) => a.properties.code.localeCompare(b.properties.code));
		geojsonFR = geo;
	});*/

	// Fonction pour convertir le JSON en GeoJSON
	function convertToGeoJSON(jsonData) {
		console.log(jsonData)
		if (!jsonData || !jsonData.features || !Array.isArray(jsonData.features)) {
			console.error('Invalid JSON data. Unable to convert to GeoJSON.');
			return null; // ou une valeur par défaut appropriée selon votre cas
		}

		const features = jsonData.features.map(feature => ({
			type: 'Feature',
			geometry: feature.geometry,
			properties: feature.properties // Si des propriétés sont présentes dans le JSON, vous pouvez les ajouter ici
		}));

		return {
			type: 'FeatureCollection',
			features: features
		};
	}


	// Convertir le JSON en GeoJSON
	const geojsonFR = convertToGeoJSON(topojsonFR);
</script>

<ONSHeader filled={true} center={false} />

<Header bgcolor="#206095" bgfixed={true} theme="dark" center={false} short={true}>
	<h1>La superficie agricole utilisée</h1>
	<p class="text-big" style="margin-top: 5px">
		En 2020, l’agriculture occupe 269 000 km2 de l’espace français. En métropole⁠?, cela représente <Em color={"#ffe552"}> 49 % du territoire</Em>.

		Alors qu'elle diminue régulièrement depuis 50 ans, la superficie agricole utilisée⁠? varie peu entre 2010 et 2020 (-1 %).			

	</p>
	<p style="margin-top: 20px">
		20 février 2024
	</p>
	<p>
		<Toggle label="Animation {animation ? 'on' : 'off'}" mono={true} bind:checked={animation}/>
	</p>
	<div style="margin-top: 90px;">
		<Arrow color="white" {animation}>Scroll to begin</Arrow>
	</div>
</Header>


<Media col="full" caption="Rosa Bonheur - Labourage nivernais">
	<div class="media" style="width: 100%;"><img src="img/Rosa_Bonheur_-_Labourage_nivernais.jpg" alt="Rosa Bonheur - Labourage nivernais"></div>
</Media>

<Section>
	<h2>Répartition des surfaces agricoles par espèce cultivée en 1852</h2>
	<p>
		Les surfaces agricoles étaient majoritairement des fourrages et céréales. 
	</p>
	<p>
		Parmi les céréales, c’est le froment qui est majoritaire avec deux fois plus de surface que le froment et trois fois plus que le seigle. 
	</p>
	<p>	
	Les cultures arborescentes sont dominées par la vigne, et les châtaignerais. 
	</p>
	<p>Parmi les cultures diverses, la culture de pommes de terre domine, suivie des légumes secs.
	</p>

</Section>


<Divider/>

<Media caption="Répartition des surfaces agricoles par espèce cultivée en 1852 !" col="wide">
		<Treemap treemap_data={treemap_data} />
</Media>


<Divider/>


<Section>
	<h2>Une production de froment qui domine</h2>
</Section>


<Scroller {threshold} bind:id={id['chart']} splitscreen={true}>
	<div slot="background">
		<figure>
			<div class="col-wide height-full">
					<div class="chart">
						
						{#if choix === 'tous'}
                        	<Bubblechart bubble_data={bubble_data_init} />
						{:else if choix === 'Froment'}
                        	<Bubblechart bubble_data={bubble_data_froment} />
						{:else if choix === 'Avoine'}
                        	<Bubblechart bubble_data={bubble_data_avoine} />
						{:else if choix === 'Orge, Méteil'}
                        	<Bubblechart bubble_data={bubble_data_orge_meteil} />
						{:else if choix === 'Sarrasin'}
                        	<Bubblechart bubble_data={bubble_data_sarrasin} />
						{:else if choix === 'Seigle'}
                        	<Bubblechart bubble_data={bubble_data_seigle} />
						{:else if choix === 'Maïs'}
                        	<Bubblechart bubble_data={bubble_data_mais} />
						{:else if choix === 'Barchart'}
							<BarChart
								data={bubble_data_init_barchart}
								xKey="value" 
								yKey="id"
								barHeight=80
								title="Production de céréales en 1852, en quintaux métriques" />


						{:else}
							<Bubblechart bubble_data={bubble_data_init} />
                    	{/if}
					</div>
			</div>
		</figure>
	</div>

	<div slot="foreground">
		<section data-id="chart00">
			<div class="col-medium">
				<p>
					Voici les principales céréales cultivées en 1852, en quintaux métriques.
				</p>
			</div>
		</section>
		<section data-id="chart01">
			<div class="col-medium">
				<p>
					Le <Em color={"#5f86af"}>froment</Em> est la céréale la plus produite. Avec du froment, on produit de la farine pour faire du pain, des pâtes et divers produits de boulangerie. En 1852 la production s’élève à 92,2 millions de quintaux métriques.
				</p>
			</div>
		</section>
		<section data-id="chart02">
			<div class="col-medium">
				<p>
					L’<Em color={"#f39941"}>avoine</Em> est la deuxième céréale produite avec 39.5 millions de quintaux. L'avoine était utilisée principalement comme aliment pour le bétail, en particulier pour les chevaux, mais elle était également consommée par les humains sous forme de gruau ou de bouillie.
				</p>
			</div>
		</section>
		<section data-id="chart03">
			<div class="col-medium">
				<p>
					Juste derrière arrive le <Em color={"#f39941"}>seigle</Em>, utilisée essentiellement pour le pain.
				</p>
			</div>
		</section>
		<section data-id="chart04">
			<div class="col-medium">
				<p>
					L’<Em color={"#f39941"}>orge</Em>  et le <Em color={"#f39941"}>méteil</Em> ont une production similaire. L'orge était utilisée pour la production de bière et pour l'alimentation animale. Le méteil était utilisé pour l’alimentation animale.
				</p>
			</div>
		</section>
		<section data-id="chart05">
			<div class="col-medium">
				<p>
				  Le <Em color={"#f39941"}>sarrasin</Em> était utilisé pour la fabrication de galettes et de crêpes, ainsi que pour la nourriture des animaux.

				</p>
			</div>
		</section>
		<section data-id="chart06">
			<div class="col-medium">
				<p>
					A noter, la très faible production de <Em color={"#f39941"}>maïs</Em>.			
				</p>
			</div>
		</section>
		<section data-id="chart07">
			<div class="col-medium">
				<p>
					Et voilà ce que cela donne quand on change de type de graphique.	
				</p>
			</div>
		</section>
	</div>
</Scroller>

<Divider />


<Section>
	<h2>Le Bio en France</h2>
	<p>C'était un peu galère de refaire les voronoi du recensement de 1952 alors j'ai choisi de prendre les données du Bio en open data. Voilà ce que cela donne.</p>
</Section>
<Divider />
{#if geojsonFR && data.departement.indicators}
<Scroller {threshold} bind:id={id['map']}>
	<div slot="background">
		<figure>
			<div class="col-full height-full">
				<!--<Map style={mapstyle} bind:map interactive={false} location={{bounds: mapbounds.uk}}>-->
					<Map style={mapstyle} bind:map interactive={false} location={{bounds: mapbounds.fr}}>

					<MapSource
					  id="lad"
					  type="geojson"
					  data={geojsonFR}
					  promoteId="code"
					  maxzoom={13}>
					  <MapLayer
					  	id="lad-fill"
							idKey="code"
							colorKey={mapKey + "_color"}
					  	data={data.departement.indicators}
					  	type="fill"
							select {selected} on:select={doSelect} clickIgnore={!explore}
							hover {hovered} on:hover={doHover}
							highlight highlighted={mapHighlighted}
					  	paint={{
					  		'fill-color': ['case',
					  			['!=', ['feature-state', 'color'], null], ['feature-state', 'color'],
					  			'rgba(255, 255, 255, 0)'
					  		],
					  		'fill-opacity': 0.7
					  	}}>
								<MapTooltip content={
									hovered ? `${metadata.departement.lookup[hovered].code}<br/><strong>${data.departement.indicators.find(d => d.code == hovered)[mapKey].toLocaleString()} ${units[mapKey]}</strong>` : ''
								}/>
							</MapLayer>
						<MapLayer
					  	id="lad-line"
					  	type="line"
					  	paint={{
					  		'line-color': ['case',
					  			['==', ['feature-state', 'hovered'], true], 'orange',
					  			['==', ['feature-state', 'selected'], true], 'black',
					  			['==', ['feature-state', 'highlighted'], true], 'black',
					  			'rgba(255,255,255,0)'
					  		],
					  		'line-width': 2
					  	}}
				    />
				  </MapSource>
				</Map>
			</div>
		</figure>
	</div>

	<div slot="foreground">
		<section data-id="map01">
			<div class="col-medium">
				<p>
					Le <Em color={colors.seq[0]}>froment</Em> est produit sur tout le territoire française, mais en moins grande quantité dans le Massif central, les Landes et la Bretagne.
				</p>
			</div>
		</section>
		<section data-id="map02">
			<div class="col-medium">
				<p>
					Les régions qui ont une faible production de froment compense par une importante production de <Em color={colors.seq[0]}>seigle</Em>.</p>
			</div>
		</section>
		<section data-id="map03">
			<div class="col-medium">
				L’<Em color={colors.seq[0]}>avoine</Em> est une spécialité de la moitié Nord de la France.
			</div>
		</section>
		<section data-id="map04">
			<div class="col-medium">
				La <strong>Bretagne</strong> est la principale zone de production de <Em color={colors.seq[0]}>sarrasin</Em>, probablement pour les traditionnelles crèpes.
			</div>
		</section>
		<section data-id="map05">
			<div class="col-medium">
				Le <Em color={colors.seq[0]}>maïs</Em> est déjà une spécialité du <strong>quart Sud-Ouest de la France Bretagne.</strong>
			</div>
		</section>
	</div>
</Scroller>
{/if}

<Divider />


<ONSFooter />

<style>
	/* Styles specific to elements within the demo */
	:global(svelte-scroller-foreground) {
		pointer-events: none !important;
	}
	:global(svelte-scroller-foreground section div) {
		pointer-events: all !important;
	}
	select {
		max-width: 350px;
	}
	.chart {
		margin-top: 45px;
		width: calc(100% - 5px);
	}
	.chart-full {
		margin: 0 20px;
	}
	.chart-sml {
		font-size: 0.85em;
	}
	/* The properties below make the media DIVs grey, for visual purposes in demo */
	.media {
		background-color: #fff;
		display: -webkit-box;
		display: -ms-flexbox;
		display: flex;
		-webkit-box-orient: vertical;
		-webkit-box-direction: normal;
		-ms-flex-flow: column;
		flex-flow: column;
		-webkit-box-pack: center;
		-ms-flex-pack: center;
		justify-content: center;
		text-align: center;
		color: #aaa;
	}
</style>
