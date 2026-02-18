<script>
	export let ziComponent;
    export let ziIndicators;   
</script>

<ul class="">
{#each ziIndicators as indicator}

{#if indicator.Component === ziComponent}
<li><a class="markdown" href="/indicator/{indicator.id}"><b>{indicator.Name}</b></a> {indicator.Description} &nbsp;</li>
 

{/if}

{/each}
</ul>

