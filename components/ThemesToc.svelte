<script>
    import { buildQuery } from "@evidence-dev/component-utilities/buildQuery";
    // import ChevronCircleUp from 'svelte-icons/fa/FaChevronCircleUp.svelte';

    // export let ziTheme; // if to show a single theme?
    const qThemes = buildQuery("select * from datahubGsheets.dh_Themes", "nonssr1");   
	const qComponents = buildQuery("select * from datahubGsheets.dh_Components", "nonssr");   
	const qIndicators = buildQuery("select * from datahubGsheets.dh_Indicators", "nonssr");   
</script>

  
  <!-- <div class="icon">
    <ChevronCircleUp />
  </div> -->
 
<div class="fullToc">

{#each qThemes as theme}
<div class="level-0-wrapper mb-8 pb-8">
    <h2 class="bg-slate-100 px-3 pt-2 pb-0 text-xl rounded-t inline-block font-bold">{theme.Name}</h2>
    <div class="xdescription text-lg bg-slate-100 p-3  pb-4 rounded-bl rounded-br rounded-tr leading-tight text-base"  >  {theme.Description}
    <div class="level-0   xl:columns-2 2xl:columns-3 pt-2 mt-2">
    {#each qComponents as component}
    {#if component.Theme == theme.id}
        <div class="level-1-wrapper p-0 rounded  mb-4 border border-slate-300 leading-tight group cursor-pointer">
            <h3 class="border-b border-slate-400 bg-slate-200 px-2 py-2 font-bold"> {component.Name}</h3>     
            <div class="description bg-slate-50 p-3 ">{component.Description}</div>
            <div class="level-1 px-3 py-2 bg-slate-100 border shadow border-slate-300 hidden group-hover:block absolute">
            {#each qIndicators as indicator}        
                {#if indicator.Component == component.id}
                <div class="level-2 mt-1 mb-2">
                    <h4> <b><a href="/indicator/{indicator.id}" class="markdown">{indicator.Name}</a></b></h4>                    
                    <div class="description text-sm">{indicator.Description}</div>
                </div>                
                {/if}
            {/each}
            </div>
        </div>
    
        {/if}   
    {/each}
    </div></div>
    </div>
  
{/each}
 </div>
<style>
    div:empty {display: none;}  
    .level-1-wrapper {break-inside: avoid;  }
    .level-1-wrapper:has(.description + .level-1:empty) {display: none}
    .level-1 {max-width: 32em !important;}
</style>