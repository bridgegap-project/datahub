---
title: Dataset
hide_title: true
---


```sql datasetMeta
SELECT * FROM datahubGsheets.dh_Datasets WHERE id = '${params.dataset}'
```

```sql Indicators
SELECT * FROM datahubGsheets.dh_Indicators WHERE Dataset = '${params.dataset}'

```

# {datasetMeta[0].Name} 

{datasetMeta[0].Description}


## Indicators
<div class="xlist">
 {#each Indicators as indicator}
 
    <h3><a class="type-indicator markdown" href="/indicator/{indicator.id}"><small><code>{indicator.id}</code></small> <b>{indicator.Name}</b></a></h3>
    <p>{indicator.Description}</p> 
    
 {/each}
 </div>

 <style>
 .xlist h3 {font-weight: bold;}
 .xlist p {line-height: 1.2;margin-bottom: 1ex; font-size: .9em;}
 </style>