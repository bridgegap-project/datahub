---
title: Component
hide_title: true 
---




```sql Indicators

SELECT * FROM datahubGsheets.dh_Indicators WHERE Component = '${params.component}'

```

```sql CurrentComponent

SELECT * FROM datahubGsheets.dh_Components WHERE id = '${params.component}'

```


# Component: **{CurrentComponent[0].Name}**

## Indicators

<div class="xlist">
 {#each Indicators as indicator}
 
    <h3><a class="type-indicator markdown" href="/indicator/{indicator.id}">{indicator.Name}</a></h3>
    <p>{indicator.Description}</p> 
 

 {/each}
 </div>

 <style>
 .xlist h3 {font-weight: bold;}
 .xlist p {line-height: 1.2;margin-bottom: 1ex; font-size: .9em;}
 </style>