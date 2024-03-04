<script>
	// CORE IMPORTS
	import { setContext, onMount } from "svelte";
	import { tooltip } from "@svelte-plugins/tooltips";
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
	import treemap_data from "./plot/treemap.json";
	import Apexcharts from "./plot/creer_apexchart.svelte";
	import sau from "./plot/sau.json";
	import nb_exploit from "./plot/nb_exploitation.json";
	import otex_sau from "./plot/otex_sau.json";
	import Bubblechart from "./plot/bubblechart.svelte";
	import bubble_data_init from "./plot/bubble_data.json";
	import bubble_data_froment from "./plot/bubble_data_froment.json";
	import bubble_data_avoine from "./plot/bubble_data_avoine.json";
	import bubble_data_mais from "./plot/bubble_data_maïs.json";
	import bubble_data_orge_meteil from "./plot/bubble_data_orge_meteil.json";
	import bubble_data_seigle from "./plot/bubble_data_seigle.json";
	import bubble_data_sarrasin from "./plot/bubble_data_sarrasin.json";
	function getRightWord(id) {
		if (typeof id !== "string") {
			return ""; // ou retournez une valeur par défaut appropriée
		}

		const words = id.split(".");
		return words[1]; // Le mot de droite
	}
	const bubble_data_init_barchart = bubble_data_init.map((item) => ({
		...item,
		id: getRightWord(item.id),
	}));

	// DEMO-SPECIFIC IMPORTS

	import bbox from "@turf/bbox";
	import {
		getData,
		setColors,
		getTopo,
		getBreaks,
		getColor,
	} from "./utils.js";
	import { colors, units } from "./config.js";
	import {
		ScatterChart,
		LineChart,
		BarChart,
	} from "@onsvisual/svelte-charts";
	import {
		Map,
		MapSource,
		MapLayer,
		MapTooltip,
	} from "@onsvisual/svelte-maps";

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
		idPrev = { ...id };
	});

	// DEMO-SPECIFIC CONFIG
	// Constants
	const datasets = ["region", "departement"];
	//const topojson = "./data/geo_lad2021.json";
	//const topojsonFR = "./data/dep2021.json";
	import topojsonFR from "./plot/dep2021.json";

	const mapstyle =
		"https://bothness.github.io/ons-basemaps/data/style-omt.json";
	const mapbounds = {
		uk: [
			[-9, 49],
			[2, 61],
		],
		fr: [
			[-5.1, 51.1],
			[9.6, 41.3],
		],
	};

	// Data
	let data = { departement: {}, region: {} };
	let metadata = { departement: {}, region: {} };
	//let geojson;
	//let geojsonFR;

	// Element bindings
	let map = null; // Bound to mapbox 'map' instance once initialised

	// State
	let hovered; // Hovered departement (chart or map)
	let selected; // Selected departement (chart or map)
	$: region =
		selected && metadata.departement.lookup
			? metadata.departement.lookup[selected].parent
			: null; // Gets region code for 'selected'
	let mapHighlighted = []; // Highlighted departement (map only)
	let graph_select = bubble_data_sarrasin; // rKey (radius) for scatter chart
	let mapKey = "nb_exp"; // Key for data to be displayed on map
	let explore = false; // Allows chart/map interactivity to be toggled on/off

	// scroller 1
	let type_graph = "bar";
	let data_chart = nb_exploit;
	let title_graph = "Nombre d'exploitations par classe de SAU";
	let name_series = ["2020", "2010"];
	let ykey = ["nb_exploit_2020", "nb_exploit_2010"];
	let xkey = "taille";
	let unit = "";
	let nb_decimal = 0;
	let colors_chart = ["#5f86af", "#999999"];
	let animate = true;
	let horizontal = true;
	let distributed = false;

	// scroller 2
	let type_graph2 = "bar";
	let data_chart2 = nb_exploit;
	let title_graph2 = "Nombre d'exploitations par classe de SAU";
	let name_series2 = ["2020", "2010"];
	let ykey2 = ["nb_exploit_2020", "nb_exploit_2010"];
	let xkey2 = "taille";
	let unit2 = "";
	let nb_decimal2 = 0;
	let colors_chart2 = ["#5f86af", "#999999"];
	let animate2 = true;
	let horizontal2 = true;
	let distributed2 = false;
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
			map.fitBounds(bounds, { animate: animation, padding: 30 });
		}
	}
	function fitById(id) {
		if (geojsonFR && id) {
			let feature = geojsonFR.features.find(
				(d) => d.properties.code == id,
			);
			let bounds = bbox(feature.geometry);
			fitBounds(bounds);
		}
	}

	// Actions for Scroller components
	const actions = {
		map: {
			// Actions for <Scroller/> with id="map"
			map01: () => {
				// Action for <section/> with data-id="map01"
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
				let hl = [...data.departement.indicators].sort(
					(a, b) => b.surf_2022 - a.surf_2022,
				)[0];
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
			},
		},
		chart: {
			chart00: () => {
				type_graph = "bar";
				data_chart = nb_exploit;
				title_graph = "Nombre d'exploitations par classe de SAU";
				name_series = ["2020", "2010"];
				ykey = ["nb_exploit_2020", "nb_exploit_2010"];
				xkey = "taille";
				unit = "";
				nb_decimal = 0;
				colors_chart = ["#5f86af", "#999999"];
				horizontal = true;
				distributed = false;
			},
			chart01: () => {
				type_graph = "bar";
				data_chart = nb_exploit;
				title_graph = "Nombre d'exploitations par classe de SAU";
				name_series = ["2020", "2010"];
				ykey = ["nb_exploit_2020", "nb_exploit_2010"];
				xkey = "taille";
				unit = "";
				nb_decimal = 0;
				colors_chart = [
					"#34cb6a",
					"#999999",
					"#999999",
					"#999999",
					"#999999",
				];
				horizontal = true;
				distributed = true;
			},
			chart02: () => {
				type_graph = "area";
				data_chart = sau;
				title_graph =
					"Évolution du nombre d’exploitations (en millions)";
				name_series = ["< 20 ha", "total"];
				ykey = ["nbexpl_moins_20_ha", "nbexpl_tot"];
				xkey = "annee";
				unit = "";
				nb_decimal = 0;
				colors_chart = ["#ffe552", "#999999"];
				horizontal = false;
				distributed = true;
			},
			chart03: () => {
				type_graph = "bar";
				data_chart = nb_exploit;
				title_graph = "Nombre d'exploitations par classe de SAU";
				name_series = ["2020", "2010"];
				ykey = ["nb_exploit_2020", "nb_exploit_2010"];
				xkey = "taille";
				unit = "";
				nb_decimal = 0;
				colors_chart = [
					"#999999",
					"#999999",
					"#999999",
					"#999999",
					"#34cb6a",
				];
				horizontal = true;
				distributed = true;
			},
			chart04: () => {
				type_graph = "area";
				data_chart = sau;
				title_graph =
					"Évolution du nombre d’exploitations (en millions)";
				name_series = [">= 100 ha", "total"];
				ykey = ["nbexpl_sup_100_ha", "nbexpl_tot"];
				xkey = "annee";
				unit = "";
				nb_decimal = 0;
				colors_chart = ["#ffe552", "#999999"];
				horizontal = false;
				distributed = true;
			},
			chart05: () => {
				type_graph = "area";
				data_chart = sau;
				title_graph =
					"Évolution du nombre d’exploitations (en millions)";
				name_series = [">= 100 ha", "total"];
				ykey = ["nbexpl_sup_100_ha", "nbexpl_tot"];
				xkey = "annee";
				unit = "";
				nb_decimal = 0;
				colors_chart = ["#ffe552", "#999999"];
				horizontal = false;
				distributed = true;
				animate = false;
			},
		},
		chart2: {
			chart200: () => {
				type_graph2 = "bar";
				data_chart2 = nb_exploit;
				title_graph2 = "Nombre d'exploitations par classe de SAU";
				name_series2 = ["2020", "2010"];
				ykey2 = ["nb_exploit_2020", "nb_exploit_2010"];
				xkey2 = "taille";
				unit2 = "";
				nb_decimal2 = 0;
				colors_chart2 = ["#5f86af", "#999999"];
				horizontal2 = true;
				distributed2 = false;
			},
			chart201: () => {
				type_graph2 = "bar";
				data_chart2 = nb_exploit;
				title_graph2 = "Nombre d'exploitations par classe de SAU";
				name_series2 = ["2020", "2010"];
				ykey2 = ["nb_exploit_2020", "nb_exploit_2010"];
				xkey2 = "taille";
				unit2 = "";
				nb_decimal2 = 0;
				colors_chart2 = [
					"#34cb6a",
					"#999999",
					"#999999",
					"#999999",
					"#999999",
				];
				horizontal2 = true;
				distributed2 = true;
			},
			chart202: () => {
				type_graph2 = "area";
				data_chart2 = sau;
				title_graph2 =
					"Évolution du nombre d’exploitations (en millions)";
				name_series2 = ["< 20 ha", "total"];
				ykey2 = ["nbexpl_moins_20_ha", "nbexpl_tot"];
				xkey2 = "annee";
				unit2 = "";
				nb_decimal2 = 0;
				colors_chart2 = ["#ffe552", "#999999"];
				horizontal2 = false;
				distributed2 = true;
			},
			chart203: () => {
				type_graph2 = "bar";
				data_chart2 = nb_exploit;
				title_graph2 = "Nombre d'exploitations par classe de SAU";
				name_series2 = ["2020", "2010"];
				ykey2 = ["nb_exploit_2020", "nb_exploit_2010"];
				xkey2 = "taille";
				unit2 = "";
				nb_decimal2 = 0;
				colors_chart2 = [
					"#999999",
					"#999999",
					"#999999",
					"#999999",
					"#34cb6a",
				];
				horizontal2 = true;
				distributed2 = true;
			},
			chart204: () => {
				type_graph2 = "area";
				data_chart2 = sau;
				title_graph2 =
					"Évolution du nombre d’exploitations (en millions)";
				name_series2 = [">= 100 ha", "total"];
				ykey2 = ["nbexpl_sup_100_ha", "nbexpl_tot"];
				xkey2 = "annee";
				unit2 = "";
				nb_decimal2 = 0;
				colors_chart2 = ["#ffe552", "#999999"];
				horizontal2 = false;
				distributed2 = true;
			},
			chart205: () => {
				type_graph2 = "area";
				data_chart2 = sau;
				title_graph2 =
					"Évolution du nombre d’exploitations (en millions)";
				name_series2 = [">= 100 ha", "total"];
				ykey2 = ["nbexpl_sup_100_ha", "nbexpl_tot"];
				xkey2 = "annee";
				unit2 = "";
				nb_decimal2 = 0;
				colors_chart2 = ["#ffe552", "#999999"];
				horizontal2 = false;
				distributed2 = true;
				animate2 = false;
			}
		}
	};
	// Code to run Scroller actions when new caption IDs come into view
	function runActions(codes = []) {
		codes.forEach((code) => {
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
	datasets.forEach((geo) => {
		getData(`./data/data_${geo}.csv`).then((arr) => {
			let meta = arr.map((d) => ({
				code: d.code,
				//name: d.name,
				parent: d.parent ? d.parent : null,
			}));
			let lookup = {};
			meta.forEach((d) => {
				lookup[d.code] = d;
			});
			metadata[geo].array = meta;
			metadata[geo].lookup = lookup;

			let indicators = arr.map((d, i) => ({
				...meta[i],
				//area: d.area,
				surf_2022: d["2022"],
				surf_2021: d["2021"],
				density: d.density,
				nb_exp: d.nb_exp,
			}));

			if (geo == "departement") {
				["nb_exp", "surf_2022"].forEach((key) => {
					let values = indicators
						.map((d) => d[key])
						.sort((a, b) => a - b);
					let breaks = getBreaks(values);
					indicators.forEach(
						(d, i) =>
							(indicators[i][key + "_color"] = getColor(
								d[key],
								breaks,
								colors.seq,
							)),
					);
				});
			}
			data[geo].indicators = indicators;

			let years = [
				2007, 2008, 2009, 2010, 2011, 2012, 2013, 2014, 2015, 2016,
				2017, 2018, 2019, 2020, 2021, 2022,
			];

			let timeseries = [];
			arr.forEach((d) => {
				years.forEach((year) => {
					timeseries.push({
						code: d.code,
						//name: d.name,
						value: d[year],
						year,
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
		console.log(jsonData);
		if (
			!jsonData ||
			!jsonData.features ||
			!Array.isArray(jsonData.features)
		) {
			console.error("Invalid JSON data. Unable to convert to GeoJSON.");
			return null; // ou une valeur par défaut appropriée selon votre cas
		}

		const features = jsonData.features.map((feature) => ({
			type: "Feature",
			geometry: feature.geometry,
			properties: feature.properties, // Si des propriétés sont présentes dans le JSON, vous pouvez les ajouter ici
		}));

		return {
			type: "FeatureCollection",
			features: features,
		};
	}

	// Convertir le JSON en GeoJSON
	const geojsonFR = convertToGeoJSON(topojsonFR);

	let horizontal_graph1 = false;
	let distributed_graph1 = false;
	let colors_sau_moyenne = ["#34cb6a", "#FFF"];
</script>

<ONSHeader filled={true} center={false} />

<Header
	bgcolor="#206095"
	bgfixed={true}
	theme="dark"
	center={false}
	short={true}
>
	<h1>La superficie agricole utilisée</h1>
	<p class="text-big" style="margin-top: 5px">
		En 2020, l’agriculture occupe 269 000 km2 de l’espace français. En <u
			title="En Guyane, dont la superficie est le sixième de la métropole, l'agriculture n'utilise que 0,4 % du territoire."
			use:tooltip={{ theme: "custom-tooltip" }}>métropole</u
		>, cela représente <Em color={"#ffe552"}>49 % du territoire</Em>. Alors
		qu'elle <b>diminue régulièrement depuis 50 ans</b>, la
		<u
			title="La SAU comprend les terres arables (y compris pâturages temporaires, jachères, cultures sous abri, jardins familiaux...), les surfaces toujours en herbe et les cultures permanentes (vignes, vergers...)."
			use:tooltip={{ theme: "custom-tooltip" }}
			>superficie agricole utilisée</u
		> varie peu entre 2010 et 2020 (-1 %).
	</p>
	<p style="margin-top: 20px">20 février 2024</p>
	<p>
		<Toggle
			label="Animation {animation ? 'activée' : 'désactivée'}"
			mono={true}
			bind:checked={animation}
		/>
	</p>
	<div style="margin-top: 90px;">
		<Arrow color="white" {animation}>Faites défiler pour lire la viz.</Arrow
		>
	</div>
</Header>

<Section>
	<h2>
		L'agriculture occupe près de la moitié de l'espace en métropole, part
		qui se stabilise.
	</h2>
</Section>

<Media caption="Part des terres agricoles en métropole, en %" col="wide">
	<Apexcharts
		type_graph="bar"
		data={sau}
		name_series="Part de la SAU en métropole !"
		xkey="annee"
		ykey="part_sau"
		unit=" %"
		nb_decimal="1"
		horizontal={horizontal_graph1}
		distributed={distributed_graph1}
	/>
</Media>

<Divider />

<Section theme="lightblue">
	<p>
		Les exploitations agricoles sont moins nombreuses à utiliser cet espace
		: <b>100 000 en moins depuis 2010.</b> Leur SAU moyenne continue d’augmenter.
		En 2020, elle s’élève en métropole à 69 ha (contre 55 ha en 2010).
	</p>

	<h2>
		La SAU moyenne augmente toujours au même rythme
		<h2></h2>
	</h2></Section
>
<Media caption="SAU moyenne en métropole, en ha" col="wide" theme="lightblue">
	<Apexcharts
		type_graph="line"
		data={sau}
		name_series="SAU moyenne en métropole"
		xkey="annee"
		ykey="sau_m_ha"
		unit=" ha"
		nb_decimal="0"
		colors={colors_sau_moyenne}
	/>
</Media>

<Scroller {threshold} bind:id={id["chart"]} splitscreen={true}>
	<div slot="background">
		<figure>
			<div class="col-wide height-full">
				<div class="chart">
					<Media caption={title_graph} col="wide">
						<Apexcharts
							{type_graph}
							data={data_chart}
							{name_series}
							{xkey}
							{ykey}
							{unit}
							{nb_decimal}
							colors={colors_chart}
							{animate}
							{horizontal}
							{distributed}
						/>
					</Media>
				</div>
			</div>
		</figure>
	</div>

	<div slot="foreground">
		<section data-id="chart00">
			<div class="col-medium">
				<p>
					Le nombre d'exploitations de <Em color={"#5f86af"}
						>moins de 100 ha</Em
					> recule tandis que celui des plus grandes augmente. Pour autant,
					<b
						>les exploitations de moins de 50 ha sont toujours
						majoritaires</b
					> (54 % en 2020).
				</p>
			</div>
		</section>
		<section data-id="chart01">
			<div class="col-medium">
				<p>
					Les <Em color={"#34cb6a"}
						>exploitations de moins de 20 ha</Em
					> concentrent les 2/3 de la diminution totale.
				</p>
			</div>
		</section>
		<section data-id="chart02">
			<div class="col-medium">
				<p>
					Ces petites exploitations restent les plus présentes, mais
					en 50 ans leur part dans l'ensemble est passée de <b
						>58 % à 38 %</b
					>.
				</p>
			</div>
		</section>
		<section data-id="chart03">
			<div class="col-medium">
				<p>
					A l'opposé, le nombre d'exploitations de <Em
						color={"#34cb6a"}>plus de 200 ha</Em
					> augmente d'un tiers entre 2010 et 2020.
				</p>
			</div>
		</section>
		<section data-id="chart04">
			<div class="col-medium">
				<p>
					Sur plus long terme, les exploitations de <b
						>100 ha et plus</b
					> sont de plus en plus nombreuses.
				</p>
			</div>
		</section>
		<section data-id="chart05">
			<div class="col-medium">
				<p>
					Plus de 100 000 en 2020, elles représentent un quart du
					total, <b>contre 2 % 50 ans auparavant.</b>
				</p>
			</div>
		</section>
	</div>
</Scroller>

<Section theme="lightblue">
	<p>
		Les exploitations spécialisées en <Em color={"#34cb6a"}
			>grandes cultures</Em
		> occupent plus d’un tiers de la SAU en métropole, celles spécialisées dans
		les <Em color={"#16456c"}>élevages bovins</Em> presqu'autant.
	</p>

	<h2>
		La SAU moyenne augmente toujours au même rythme
	</h2>
			<Media
				caption="SAU moyenne en métropole, en ha"
				col="wide"
				theme="lightblue"
			>
				<Apexcharts
					type_graph="donut"
					data={otex_sau}
					name_series="SAU par OTEX"
					xkey="otex"
					ykey="sau"
					unit=" %"
					nb_decimal="0"
				/>
			</Media>
			<p>La surface des exploitations varie avec leur spécialisation.

			</p>
	</Section>

<Section>
	<Scroller {threshold} bind:id={id["chart2"]} splitscreen={true}>
		<div slot="background">
			<figure>
				<div class="col-wide height-full">
					<div class="chart">
						<Media caption={title_graph2} col="wide">
							
						</Media>
					</div>
				</div>
			</figure>
		</div>
	
		<div slot="foreground">
			<section data-id="chart200">
				<div class="col-medium">
					<p>
						Le nombre d'exploitations de <Em color={"#5f86af"}
							>moins de 100 ha</Em
						> recule tandis que celui des plus grandes augmente. Pour autant,
						<b
							>les exploitations de moins de 50 ha sont toujours
							majoritaires</b
						> (54 % en 2020).
					</p>
				</div>
			</section>
			<section data-id="chart201">
				<div class="col-medium">
					<p>
						Les <Em color={"#34cb6a"}
							>exploitations de moins de 20 ha</Em
						> concentrent les 2/3 de la diminution totale.
					</p>
				</div>
			</section>
			<section data-id="chart202">
				<div class="col-medium">
					<p>
						Ces petites exploitations restent les plus présentes, mais
						en 50 ans leur part dans l'ensemble est passée de <b
							>58 % à 38 %</b
						>.
					</p>
				</div>
			</section>
			<section data-id="chart203">
				<div class="col-medium">
					<p>
						A l'opposé, le nombre d'exploitations de <Em
							color={"#34cb6a"}>plus de 200 ha</Em
						> augmente d'un tiers entre 2010 et 2020.
					</p>
				</div>
			</section>
			<section data-id="chart204">
				<div class="col-medium">
					<p>
						Sur plus long terme, les exploitations de <b
							>100 ha et plus</b
						> sont de plus en plus nombreuses.
					</p>
				</div>
			</section>
			<section data-id="chart205">
				<div class="col-medium">
					<p>
						Plus de 100 000 en 2020, elles représentent un quart du
						total, <b>contre 2 % 50 ans auparavant.</b>
					</p>
				</div>
			</section>
		</div>
	</Scroller>
</Section>

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

	:global(.tooltip.custom-tooltip) {
		--tooltip-background-color: #ffe552;
		--tooltip-box-shadow: 0 1px 8px white;
		--tooltip-color: black;
		--tooltip-font-size: 16px;
	}
</style>
