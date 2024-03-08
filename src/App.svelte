<script>
	// CORE IMPORTS
	import { setContext } from "svelte";
	import { tooltip } from "@svelte-plugins/tooltips";
	import { themes } from "./config.js";
	import ONSHeader from "./layout/ONSHeader.svelte";
	import ONSFooter from "./layout/ONSFooter.svelte";
	import Header from "./layout/Header.svelte";
	import Section from "./layout/Section.svelte";
	import Media from "./layout/Media.svelte";
	import Divider from "./layout/Divider.svelte";
	import Toggle from "./ui/Toggle.svelte";
	import Arrow from "./ui/Arrow.svelte";
	import Em from "./ui/Em.svelte";
	import ScrollerGraph from "./layout/ScrollerGraph.svelte";


	import Apexcharts from "./layout/creer_apexchart.svelte";
	import sau from "./plot/sau.json";
	import nb_exploit from "./plot/nb_exploitation.json";
	import otex_sau from "./plot/otex_sau.json";
	import otex_sau_moyen from "./plot/otex_sau_moyen.json";
	import otex_sau_nul from "./plot/otex_sau_nul.json";
	import evol_sau_2010_2020 from "./plot/evol_sau_2010_2020.json";
	import surface_espece from "./plot/surface_espece.json";
	import chartsConfig_01_json from './plot/chartsConfig_01.json';
	import chartsConfig_02_json from './plot/chartsConfig_02.json';
	import chartsConfig_03_json from './plot/chartsConfig_03.json';

	const dataSets = {
		nb_exploit,
		sau,
		otex_sau,
		otex_sau_moyen,
		surface_espece
	};
	const chartsConfig_01 = chartsConfig_01_json.map(config => ({
		...config,
		data_chart: dataSets[config.data_chart],
	}));
	const chartsConfig_02 = chartsConfig_02_json.map(config => ({
		...config,
		data_chart: dataSets[config.data_chart],
	}));
	const chartsConfig_03 = chartsConfig_03_json.map(config => ({
		...config,
		data_chart: dataSets[config.data_chart],
	}));
	
	import {
		setColors,
	} from "./utils.js";
	
	// Set theme globally (options are 'light', 'dark' or 'lightblue')
	let theme = "light";
	setContext("theme", theme);
	setColors(themes, theme);

	let animation = true;
	let horizontal_graph1 = false;
	let distributed_graph1 = false;
	let colors_sau_moyenne = ["#34cb6a", "#FFF"];

	let color_chart_evolution_sau = ["#D41501","#FF5443","#FF9E93", "#86E8A7", "#58D983", "#35CB68", "#13C04E", "#009B35" ];
	let horizontal_chart_evolution_sau = true;
	let distributed_chart_evolution_sau = true;
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


<ScrollerGraph chartsConfig={chartsConfig_01} />

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
					unit_total_donut = "km2"
					nb_decimal_total_donut=0
				/>
			</Media>
			<p>La surface des exploitations varie avec leur spécialisation.

			</p>
	</Section>

<ScrollerGraph chartsConfig={chartsConfig_02} />

<Section theme="lightblue">
<p>
La SAU peut s’avérer nulle. C’est le cas pour plus de <u
title="Hors apiculture pour qui la notion de SAU est difficile à évaluer, les surfaces butinées étant soit des espaces publics ou des surfaces mises à disposition par des exploitants ou des particuliers."
use:tooltip={{ theme: "custom-tooltip" }}
>5 000 exploitations</u>, où seuls les bâtiments servent à la production agricole.
</p>
<h2>Exploitations avec une SAU nulle</h2>

<Media
				caption="Part des sxploitations avec une SAU nulle"
				col="wide"
				theme="lightblue"
			>
				<Apexcharts
					type_graph="donut"
					data={otex_sau_nul}
					name_series="Part des sxploitations avec une SAU nulle par OTEX"
					xkey="otex"
					ykey="nb_exploitation_sau_nul"
					unit=" %"
					nb_decimal="0"
				/>
			</Media>
<p>
	<b>Près de la moitié</b> des exploitations (46 %) avec une SAU nulle sont celles de <Em color="#35cb6a">l’élevage de volailles</Em> ; elles représentent un quart des exploitations avicoles. <Em color="#226095">L’élevage et l’engraissement de porcins</Em> viennent ensuite (18 %).
	<br/>
</p>

</Section>

<Section>
	<h2>Évolutions par type de culture en métropole</h2>
	<p>
		Entre 2010 et 2020, l’exploitation de la SAU a évolué de façon contrastée selon les cultures : celle des céréales et des oléagineux recule tandis que celle des plantes à fibres et des prairies grandit.
	</p>

<Media
				caption="Evolution de la SAU selon la culture"
				col="wide"
				theme="light"
			>
				<Apexcharts
					type_graph="bar"
					data={evol_sau_2010_2020}
					name_series="Evolutions de la SAU par type de cultures"
					xkey="cultures"
					ykey="evol_sau_ha_2010_2020"
					unit=" ha"
					nb_decimal="0"
					colors={color_chart_evolution_sau}
					colors_chart= {color_chart_evolution_sau}
					horizontal= {horizontal_chart_evolution_sau}
					distributed= {distributed_chart_evolution_sau}
				/>
			</Media>
			<p>
				La surface cultivée des « plantes à fibres » grandit nettement ; elle est <b>multipliée par 2,5</b> entre 2010 et 2020.

			</p>
		</Section>

			<ScrollerGraph chartsConfig={chartsConfig_03} />




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
