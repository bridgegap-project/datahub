<script>
// import "@evidence-dev/tailwind/fonts.css";
import "../app.css";
import { EvidenceDefaultLayout } from "@evidence-dev/core-components";
import ThemesNav from "../components/ThemesNav.svelte";
// import ThemesNavStatic from "../components/ThemesNavStatic.svelte";
export let data;
</script>

<svelte:head>
 	
</svelte:head>
 
<header class="fixed w-full top-0 z-40 flex h-12 shrink-0 justify-start items-center gap-x-4 border-b border-gray-200  backdrop-blur print:hidden bg-slate-900 text-white  ">
	<div class=" max-w-full mx-auto px-6 sm:px-8 md:px-12 flex flex-1 items-center justify-between" style="max-width: 1800px;">
		<div class="flex gap-x-4 items-center">
		<a href="/" class="text-sm leading-none  block" style="width: 8em;"><img src="/assets/icon strong negative mono.svg" style="height: 2.75em;" alt="BridgeGap" class="float-start mr-2" /><div class="" style="font-size: 1.15em;"><small class="text-gray-300">BridgeGap</small> <br/> <b>Datahub</b></div></a>
	</div>    
 
  <ThemesNav />   

	<div class="flex gap-2 text-sm items-center ">  
		&rarr; <a href="https://corruptiondata.eu/" title="back to BridgeGap">
			<img src="/assets/icon strong negative.png" style="height: 2.5em;" alt="BridgeGap" /></a>
		<!-- <div class="leading-none">
		<a href="https://corruptiondata.eu/"><small>back to &rarr;</small><br/>BridgeGap</a> </div> -->
	</div></div>
</header>

<div class="my-5">&nbsp;</div>

<EvidenceDefaultLayout 
	{data} title="BridgeGap Datahub" 
	maxWidth=1480
	fullWidth=true
	hideSidebar
	hideHeader
	hideTOC
	hideBreadcrumbs=false
	xProfile="https://twitter.com/bridgegap_eu"
>

	<slot slot="content" />
	
</EvidenceDefaultLayout>

 