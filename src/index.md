---
toc: false
sql:
  directorio: ./data/Directorio_Oficial_EE_2024.parquet
  paes: ./data/ANALISIS_PAES20204.parquet
---

```sql id=data
WITH tabla as (SELECT *, puntaje as PROMEDIO_PAES,
FROM paes),


intervalosPorComuna AS (
    SELECT
        NOM_COM_RBD,
        dependencia,
        tipo,
        COUNT(*) AS N,
        AVG(PROMEDIO_PAES) AS puntajePromedio,
        STDDEV(PROMEDIO_PAES) AS stdev,
        -- Calcula el percentil 2.5% y 97.5% dentro de cada grupo
        PERCENTILE_CONT(0.025) WITHIN GROUP (ORDER BY PROMEDIO_PAES) AS p2_5,
        PERCENTILE_CONT(0.975) WITHIN GROUP (ORDER BY PROMEDIO_PAES) AS p97_5
    FROM tabla
    LEFT JOIN directorio ON tabla.RBD = directorio.RBD
    GROUP BY
        NOM_COM_RBD,
        dependencia,
        tipo
  ORDER BY NOM_COM_RBD, tipo,dependencia)
  
SELECT  
  tabla.MRUN, 
  directorio.RBD,
  directorio.NOM_COM_RBD,
  directorio.NOM_RBD,
  N,
  tabla.dependencia,
  tabla.tipo,
  tabla.PROMEDIO_PAES, 
  p2_5,
  p97_5,
  CASE WHEN tabla.PROMEDIO_PAES < p2_5 THEN 'bajo'
   WHEN tabla.PROMEDIO_PAES > p97_5 THEN 'alto'
  ELSE 'normal' END as ratingPercentil


  
FROM tabla
LEFT JOIN directorio on tabla.RBD = directorio.RBD
LEFT JOIN intervalosPorComuna on directorio.NOM_COM_RBD = intervalosPorComuna.NOM_COM_RBD 
  AND intervalosPorComuna.tipo = tabla.tipo
  AND intervalosPorComuna.dependencia = tabla.dependencia
```

```sql id=indicadoresPorComuna
WITH tabla as (SELECT *, puntaje as PROMEDIO_PAES,
FROM paes),


intervalosPorComuna AS (
    SELECT
        NOM_COM_RBD,
        dependencia,
        tipo,
        COUNT(*) AS N,
        AVG(PROMEDIO_PAES) AS puntajePromedio,
        STDDEV(PROMEDIO_PAES) AS stdev,
        -- Calcula el percentil 2.5% y 97.5% dentro de cada grupo
        PERCENTILE_CONT(0.025) WITHIN GROUP (ORDER BY PROMEDIO_PAES) AS p2_5,
        PERCENTILE_CONT(0.975) WITHIN GROUP (ORDER BY PROMEDIO_PAES) AS p97_5
    FROM tabla
    LEFT JOIN directorio ON tabla.RBD = directorio.RBD
    GROUP BY
        NOM_COM_RBD,
        dependencia,
        tipo
  ORDER BY NOM_COM_RBD, tipo,dependencia)
  
SELECT  *
FROM intervalosPorComuna
```

```js
const factor = 10
const aliasDependencia = {
  PP: "Particular Pagados",
  PS: "Particular Subvencionados",
  PUB: "Municipales / Servicios Locales"
}
const aliasTipo = {
  1: "Prioritario",
  0: "No prioritario"
}
```

```js
const comuna = selectComuna.comuna
```
```js
const tipo = selectTipo.tipo
```
```js
const dependencia = selectDependencia.dependencia
```

```js
const selectComuna =  (() => {
const optionsComunas = _.chain([...data])
  .groupBy((d) => d.NOM_COM_RBD)
  .map((items, key) => ({ comuna: key, estudiantes: items.length }))
  .sortBy((d) => d.estudiantes)
  .reverse()
  .value()

return view(Inputs.select(optionsComunas,
  {
    label: "Comuna",
    format: (d) => `${d.comuna} (${d.estudiantes})`
  }
))
})()

```

```js
const selectTipo =  (() => {
const options = _.chain([...data])
    .filter((d) => d.NOM_COM_RBD == comuna)
    .groupBy((d) => d.tipo)
    .map((items, key) => ({ tipo: key, estudiantes: items.length }))
    .sortBy((d) => d.estudiantes)
    .reverse()
    .value();

return view(Inputs.select(options,
  {
    label: "Comuna",
    format: (d) => `${aliasTipo[d.tipo]} (${d.estudiantes})`
  }
))
})()

```


```js
const selectDependencia =  (() => {
const options = _.chain([...data])
    .filter((d) => d.NOM_COM_RBD == comuna && d.tipo == tipo)
    .groupBy((d) => d.dependencia)
    .map((items, key) => ({ dependencia: key, estudiantes: items.length }))
    .sortBy((d) => d.estudiantes)
    .reverse()
    .value();

return view(Inputs.select(options,
  {
    label: "Comuna",
    format: (d) => `${aliasDependencia[d.dependencia]} (${d.estudiantes})`
  }
))
})()

```

```js
const dataPlot = [...data].filter(
  (d) =>
    d.NOM_COM_RBD == comuna && d.tipo == tipo && d.dependencia == dependencia
)
```

```js
  const registroFoco = [...indicadoresPorComuna].filter(
    (d) =>
      d.NOM_COM_RBD == comuna && d.dependencia == dependencia && d.tipo == tipo
  )[0];

  const totalEstudiantes = dataPlot.length;
  const totalEstudiantesNormales = dataPlot.filter(
    (d) => d.ratingPercentil == "normal"
  ).length;
  const totalEstudiantesDestacados = dataPlot.filter(
    (d) => d.ratingPercentil == "alto"
  ).length;

  display(html`En <b>${comuna}</b> para estudiantes <b>${
    tipo == 1 ? "Prioritarios" : "No prioritarios"
  }</b> en establecimientos <b>${
    aliasDependencia[dependencia]
  }</b>, <br>el <b>95%</b> de los estudiantes obtiene un puntaje entre <b>${d3.format(
    ".0f"
  )(registroFoco.p2_5)}</b> y <b>${d3.format(".0f")(
    registroFoco.p97_5
  )}</b><br><br>
  Un ${d3.format(".1%")(
    totalEstudiantesDestacados / totalEstudiantes
  )} (${totalEstudiantesDestacados} de ${totalEstudiantes}) está por sobre este rango. `)
```

<div class="card">
${dodgeChart}
</div>

```js
const dodgeChart = (() => {
  //return dataPlot;

  const media = _.chain(dataPlot)
    .map((d) => d.PROMEDIO_PAES)
    .mean()
    .value();

  const height = width * 0.5;

  const r = Math.sqrt((height * width) / dataPlot.length / Math.PI / factor);
  return Plot.plot({
    height: height,
    width: width,
    x: { domain: [0, 1000] },
    y: { domain: [0, 1], axis: "right", tickSize: 0, tickFormat: (d) => "" },
    marks: [
      Plot.ruleX([dataPlot[0].p2_5, dataPlot[0].p97_5]),
      Plot.text([
        {x:dataPlot[0].p2_5 / 2, label:"2.5%"},
        {x:dataPlot[0].p2_5 / 2, label:"2.5%"},
        {x:(dataPlot[0].p2_5+dataPlot[0].p97_5 )/ 2, label:"95%"},
        {x:(1000+dataPlot[0].p97_5 )/ 2, label:"2.5%"},
        ], {
        x: (d) => d.x,
        y: 1,
        text: (d) => d.label,
        textAnchor: "middle",
        fontSize: height/20,
        fill: "grey"
      }),
      Plot.dot(
        dataPlot,
        Plot.dodgeY({
          x: "PROMEDIO_PAES",
          r: r,
          fill: (d) =>
            d.ratingPercentil == "alto"
              ? "blue"
              : d.ratingPercentil == "bajo"
              ? "red"
              : "lightgrey",
          tip: true,
          channels: {
            RBD: "RBD",
            Establecimiento: "NOM_RBD",
            dependencia: "dependencia",
            prioritario: "tipo"
          }
        })
      )
    ]
  });
})()
```

```js
const establecimientosTop = (() => {
  
  return  _.chain(dataPlot)
    .groupBy((d) => d.RBD)
    .map((items, key) => ({
      RBD: key,
      NOM_RBD: items[0].NOM_RBD,
      totalEstudiantes: items.length,
      estudiantesSobreLoEsperado: items.filter(
        (d) => d.ratingPercentil == "alto"
      ).length,
      porcentaje:
        items.filter((d) => d.ratingPercentil == "alto").length / items.length
    }))
    .filter((d) => d.totalEstudiantes > 10 && d.porcentaje >= 0.05)
    .sortBy((d) => d.porcentaje)
    .reverse()
    .value();

 

})()

```

### Establecimientos con sobre un 5% de estudiantes en el 2.5% superior del grupo 
(${comuna}/${aliasDependencia[dependencia]}/${aliasTipo[tipo]})
${establecimientosTop
    .filter((d) => d.porcentaje > 0.025)
    .map(
      (d) =>
        html`<li>${d.NOM_RBD} ${d.estudiantesSobreLoEsperado} de ${
          d.totalEstudiantes
        } (${d3.format(".1%")(d.porcentaje)})`
    )}