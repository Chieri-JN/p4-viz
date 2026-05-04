<script lang=ts>
	import favicon from '$lib/assets/favicon.svg';
	import * as d3 from "d3";
	import "../app.css";


	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<!-- neue haas grotesk -->
	<link rel="stylesheet" href="https://use.typekit.net/cfe6wef.css">
</svelte:head>

{@render children()}
