---
title: Indicator
hide_title: true 
---


```sql indicatorMeta
    select 
        "id",
        "Name",
        "Description",
        "Component",
        "Dataset",
        "Description - rules",
        "Var Type",
        "which is better",
        "label MIN",
        "label MAX",
        "Attribution",
        "Methodology Snapshot",
        "Source data",
        "Methodology" 
    FROM datahubGsheets.dh_Indicators  WHERE id = '${params.indicator}' 
```

```sql years
SELECT  DISTINCT TRY_CAST(Year AS FLOAT) AS Year  from datahubGsheets.dh_Data WHERE Indicator_ID = '${params.indicator}' order by Year ASC
```

```sql ziIndicators
  WITH CountryData AS (
    SELECT 
      d."ISO3 Country", 
      c.Flag,
      CONCAT(COALESCE(c.Name, d."ISO3 Country"), ' ', c.Flag) AS FlagName, 
      COALESCE(c.Name, d."ISO3 Country") AS "Country Name",
      d.Year, 
      TRY_CAST(d.Value AS FLOAT) AS Value,
      c."EUM Status" AS group_type
    FROM 
      datahubGsheets.dh_Data d
    LEFT JOIN
      datahubGsheets.dh_countries c ON d."ISO3 Country" = c.ISO3
    WHERE 
      d.Indicator_ID = '${params.indicator}' 
      AND d.Year = ${inputs.ziYears.value}
  ),
  GroupAverages AS (
    SELECT
      'AVG: ' || group_type AS "ISO3 Country",
      ' ' AS Flag,
       'AVG: ' || group_type AS FlagName,
      'AVG: ' || group_type AS "Country Name",
      Year,
      AVG(Value) AS Value,
      'average' AS row_type
    FROM
      CountryData
    WHERE
      group_type IS NOT NULL
    GROUP BY
      group_type, Year
  ),
  CombinedData AS (
    SELECT
      "ISO3 Country",
      Flag,
      FlagName,
      "Country Name",
      Year,
      Value,
      'country' AS row_type
    FROM
      CountryData
    UNION ALL
    SELECT
      "ISO3 Country",
      Flag,
      FlagName,
      "Country Name",
      Year,
      Value,
      row_type
    FROM
      GroupAverages
  )
  SELECT 
    *,
    CASE 
        WHEN row_type = 'average' THEN NULL
        ELSE CAST(DENSE_RANK() OVER (ORDER BY CASE WHEN row_type = 'country' THEN Value END DESC) AS VARCHAR) 
             || '/' || 
             CAST(COUNT(CASE WHEN row_type = 'country' THEN 1 END) OVER () AS VARCHAR)
    END AS ranking
FROM CombinedData 
  ORDER BY
    CASE WHEN row_type = 'average' THEN 0 ELSE 1 END,
    Value DESC
```

```sql IndicatorPath
  SELECT 
      dh_Indicators.id AS Indicator_ID,
      dh_Indicators.Name AS Indicator_Name,
      dh_Components.id AS Component_ID,
      dh_Components.Name AS Component_Name,
      dh_Themes.id AS Theme_ID,
      dh_Themes.Name AS Theme_Name,
      dh_Datasets.id AS Dataset_ID,
      dh_Datasets.Name AS Dataset_Name
  FROM 
      datahubGsheets.dh_Indicators
  JOIN 
      datahubGsheets.dh_Components ON dh_Indicators.Component = dh_Components.id
  JOIN 
      datahubGsheets.dh_Themes ON dh_Components.Theme = dh_Themes.id
  JOIN 
      datahubGsheets.dh_Datasets ON dh_Indicators.Dataset = dh_Datasets.id
  WHERE 
      dh_Indicators.id = '${params.indicator}';  
```

```sql siblingIndicators
  SELECT 
      dh_Indicators.id,
      dh_Indicators.Description,
      dh_Indicators.Name
  FROM 
      datahubGsheets.dh_Indicators
  JOIN 
      datahubGsheets.dh_Components ON dh_Indicators.Component = dh_Components.id
  WHERE 
      dh_Components.id = (
          SELECT Component 
          FROM datahubGsheets.dh_Indicators 
          WHERE id = '${params.indicator}'
      )
      AND dh_Indicators.id != '${params.indicator}'
  ORDER BY 
      dh_Indicators.Name;
```

<a class="markdown" href="/theme/{IndicatorPath[0].Theme_ID}">{IndicatorPath[0].Theme_Name}</a> / <a class="markdown" href="/component/{IndicatorPath[0].Component_ID}">{IndicatorPath[0].Component_Name}</a>

# **{indicatorMeta && indicatorMeta[0] ? indicatorMeta[0].Name : "Loading..."}** {inputs.ziYears.value} 
 
{indicatorMeta && indicatorMeta[0] ? indicatorMeta[0].Description : ""} <a href="#factsheet">  <Info description="See Indicator Factsheet below the chart" /></a>

<Dropdown 
    data={years} 
    name=ziYears 
    value=Year 
    order=Year 
    title="Select Year" 
    class="font-semibold p-2"
/> 

<div class="bg-gray-100 border inline-block rounded-md p-1">
  <DownloadData data={ziIndicators} text="Download Data" queryID=bridgegap-{params.indicator}-{inputs.ziYears.value} class="text-gray-950 font-semibold " />
</div>
 
<Tabs>
  <Tab label="Map">
        
        <!-- <h2 class=""><span class="font-semibold">{indicatorMeta[0].Name}</span> ({inputs.ziYears.value})</h2> -->

         {#if indicatorMeta[0]["which is better"] || (indicatorMeta[0]["label MIN"] && indicatorMeta[0]["label MAX"])}
        <div class="text-sm text-gray-600">
          {#if indicatorMeta[0]["which is better"]}
            <u>Note</u>: For this indicator, {indicatorMeta[0]["which is better"].toLowerCase()} values are better.  {indicatorMeta[0]["label MIN"]} to {indicatorMeta[0]["label MAX"]} 
          {/if}
        </div>
        {/if}
        <AreaMap 
            data={ziIndicators} 
            areaCol="ISO3 Country"
            nameCol="Country Name"
            geoJsonUrl="/ne_110m_admin_0_countries.geojson"
            geoId=iso_a3
            height=1280
            value=Value
            legendType=scalar
            legendPosition=topLeft
            borderWidth=2
            borderColor=#FFE       
            tooltip={[
                {id: 'Flag',  showColumnName: false, valueClass: 'text-3xl inline'},
                {id: 'Country Name',  showColumnName: false, valueClass: 'inline text-base'},
                {id: 'Value', fmt: 'num2', showColumnName: false, valueClass: 'text-xl font-semibold'},                
                {id: 'ranking', fmt: 'num0', showColumnName: false, valueClass: ''}                
            ]}                                   
        />
   
    </Tab>
    <Tab label="Bar">
         {#if indicatorMeta[0]["which is better"] || (indicatorMeta[0]["label MIN"] && indicatorMeta[0]["label MAX"])}
        <div class="text-sm text-gray-600">
          {#if indicatorMeta[0]["which is better"]}
            <u>Note</u>: For this indicator, {indicatorMeta[0]["which is better"].toLowerCase()} values are better.  {indicatorMeta[0]["label MIN"]} to {indicatorMeta[0]["label MAX"]} 
          {/if}
        </div>
        {/if}
        <div class="mb-3">
          <div class="flex gap-4 justify-end mb-2">
            <div class="flex items-center gap-1">
              <span class="inline-block w-3 h-3 bg-[#2ecc71]"></span>
              <span class="text-sm">EUMS Average</span>
            </div>
            <div class="flex items-center gap-1">
              <span class="inline-block w-3 h-3 bg-[#e74c3c]"></span>
              <span class="text-sm">EUCC Average</span>
            </div>
            <div class="flex items-center gap-1">
              <span class="inline-block w-3 h-3 bg-[#f39c12]"></span>
              <span class="text-sm">Non-EUMS Average</span>
            </div>
          </div>
          <BarChart 
              data={ziIndicators}
              x="FlagName"
              y=Value
              swapXY=true
              yFmt= num2
              labels=true
              title="{indicatorMeta[0].Name} for {inputs.ziYears.value}"
              seriesField="row_type"
              echartsOptions={{ 
                autoColor: false,
                colorBy: "data",
                yAxis: { 
                  type: "category",
                  axisLabel: {
                    formatter: function(value) {
                      // Truncate long country names
                      return value.length > 20 ? value.substring(0, 17) + '...' : value;
                    }
                  }
                },
                series: [
                  {
                    name: "Values",
                    encode: {
                      x: "Value",
                      y: "Country Name" 
                    },
                    itemStyle: {
                      color: function(params) {
                        // Get country name from params.data[1]
                        const countryName = params.data[1] ? params.data[1].toString() : "";
                        
                        // Different colors for different average groups
                        if (countryName.includes("AVG: EUMS")) {
                          return '#2ecc71'; // Green for EU Member States
                        } else if (countryName.includes("AVG: EUCC")) {
                          return '#e74c3c'; // Red for EU Candidate Countries
                        } else if (countryName.includes("AVG: Non-EUMS")) {
                          return '#f39c12'; // Orange for Non-EU Member States
                        } else {
                          // Regular country color
                          return '#5470c6'; 
                        }
                      }
                    },
                    barWidth: function(params) {
                      const countryName = params.data[1] ? params.data[1].toString() : "";
                      return countryName.startsWith("AVG:") ? '80%' : '60%';
                    }
                  }
                ],
        
                grid: {
                  right: '5%',
                  containLabel: true
                }
              }}
          />
        </div>
    </Tab>
</Tabs>

<table class="align-top table-cells-padding text-sm border rounded-sm" id="factsheet">
<thead>
<tr class="border-b border-gray-200">
 <th colspan="2" class="text-left text-lg p-1 pl-2 bg-gray-200 border-b">Indicator Factsheet</th>
</tr>
</thead>
<tbody>

<tr class="border-b border-gray-200">
<td class="align-top bg-gray-100 p-1 pl-2">Theme</td>
<td class="align-top p-1 pl-2"><a class="markdown" href="/theme/{IndicatorPath[0].Theme_ID}">{IndicatorPath[0].Theme_Name}</a></td>
</tr>
<tr class="border-b border-gray-200">
<td class="align-top bg-gray-100 p-1 pl-2">Component</td>
<td class="align-top p-1 pl-2"><a class="markdown" href="/component/{IndicatorPath[0].Component_ID}">{IndicatorPath[0].Component_Name}</a>
</td>
</tr>
<tr class="border-b border-gray-200">
<td class="align-top bg-gray-100 p-1 pl-2">Dataset</td>
<td class="align-top p-1 pl-2"><a class="markdown"  href="/dataset/{IndicatorPath[0].Dataset_ID}">{IndicatorPath[0].Dataset_Name}</a>
</td>
</tr>

<tr class="border-b border-gray-200">
  <td class="align-top bg-gray-100 p-1 pl-2">Name</td><td class="align-top p-1 pl-2"> {indicatorMeta[0].Name} </td>
</tr>
<tr class="border-b border-gray-200">
  <td class="align-top bg-gray-100 p-1 pl-2">Description</td><td class="align-top p-1 pl-2">{indicatorMeta[0].Description}</td>
</tr>

 
 {#if indicatorMeta[0]["Methodology Snapshot"]}
  
  <tr class="border-b border-gray-200">
    <td class="align-top bg-gray-100 p-1 pl-2">Methodology <br/>snapshot</td>  <td class="whitespace-pre-line p-1 pl-2">{indicatorMeta[0]["Methodology Snapshot"]}</td>
  </tr>
    
  {/if}

  {#if indicatorMeta[0]["label MIN"] && indicatorMeta[0]["label MAX"]}
    <tr class="border-b border-gray-200">
      <td class="align-top bg-gray-100 p-1 pl-2">Scale</td><td class="whitespace-pre-line p-1 pl-2"> {indicatorMeta[0]["label MIN"]} to {indicatorMeta[0]["label MAX"]} 
      {#if indicatorMeta[0]["which is better"]}  <br/>Note: For this indicator, {indicatorMeta[0]["which is better"].toLowerCase()} values are better.  {/if}
      </td>
    </tr>
  {/if}

 {#if indicatorMeta[0]["Source"] && indicatorMeta[0]["Source"].trim() && indicatorMeta[0]["Source"].trim().startsWith('http')}
  <tr class="border-b border-gray-200">
    <td class="align-top bg-gray-100 p-1 pl-2">Source</td><td class="align-top p-1 pl-2"><a href="{indicatorMeta[0]['Source'].trim()}" target="_blank" rel="noopener noreferrer" class="text-blue-600 hover:underline">{indicatorMeta[0]['Source'].trim()}</a></td>
  </tr>      
{/if}

 {#if indicatorMeta[0]["Methodology"] && indicatorMeta[0]["Methodology"].trim() && indicatorMeta[0]["Methodology"].trim().startsWith('http')}
  <tr class="border-b border-gray-200">
    <td class="align-top bg-gray-100 p-1 pl-2">Methodology</td>  <td class="align-top p-1 pl-2"><a href="{indicatorMeta[0]['Methodology'].trim()}" target="_blank" rel="noopener noreferrer" class="text-blue-600 hover:underline">{indicatorMeta[0]['Methodology'].trim()}</a></td>
  </tr>      
{/if}


{#if indicatorMeta[0].Attribution}
<tr class="border-b border-gray-200">
  <td class="align-top bg-gray-100 p-1 pl-2">Attribution </td>
  <td class="align-top p-1 pl-2">{indicatorMeta[0].Attribution}</td>
</tr>
{/if}
</tbody>
</table>

---

{#if siblingIndicators.length !== 0}

### Other Indicators of the **{IndicatorPath[0].Component_Name}** component

<div class="grid xl:grid-cols-2 2xl:grid-cols-3 gap-5">
 {#each siblingIndicators as indicator}
 
    <div class=" my-1">
        <h3><a class="type-indicator markdown" href="/indicator/{indicator.id}"><b>{indicator.Name}</b></a></h3>
        <p>{indicator.Description}</p>
    </div>

    
 {/each}
 </div>
{/if}
 

<!-- {@partial "indicator-list.md"} -->





<style>
  
 
  </style>