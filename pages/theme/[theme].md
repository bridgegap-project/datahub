---
title: Theme
hide_title: true 
---

 
```sql Components

SELECT * FROM datahubGsheets.dh_Components WHERE Theme = '${params.theme}'

```

```sql Indicators

SELECT * FROM datahubGsheets.dh_Indicators 

```


```sql CurrentTheme

SELECT * FROM datahubGsheets.dh_Themes WHERE id = '${params.theme}'

```

# Theme: **{CurrentTheme[0].Name}**

## Components

<div class="xlist">
 {#each Components as component}
 
    <h3 class="mt-5 text-lg"><a class="type-component markdown" href="/component/{component.id}">{component.Name}</a></h3>
    <p class="pb-5 border-b border-slate-500">
    {component.Description}
    <span class="block mt-2">
       <b>Indicators</b>:
       <span class="">
          <ChildIndicators ziIndicators = {Indicators} ziComponent = {component.id} />
       </span>
    </span>
    </p>     

    
 {/each}
 </div>

 <style>
 .xlist h3 {font-weight: bold;}
 .xlist p {line-height: 1.2;margin-bottom: 1ex; font-size: .9em;}
 </style>