---
toc: false
sql:
  directorio: ./data/Directorio_Oficial_EE_2024.parquet
  paes: ./data/ANALISIS_PAES20204.parquet
---



# Mirada alternativa a los puntajes PAES: Comparando peras con peras
## Autor: [@elaval.bsky.social](https://bsky.app/profile/elaval.bsky.social)

Los resultados de las pruebas de selección universitaria nos informan cómo se distribuyen los puntajes según el desempeño en dichas pruebas.

Sin embargo, existe evidencia de que los puntajes están asociados a diferentes variables, entre ellas:

### Comuna

Existen diferencias estadísticamente significativas en los puntajes que obtienen los estudiantes dependiendo de la comuna asociada al establecimiento escolar. Por ejemplo, los estudiantes de Vitacura obtienen mejores resultados que los de Santiago, y estos últimos mejores que los de Angol.

### Dependencia del establecimiento

También hay diferencias estadísticamente significativas en los puntajes según la dependencia del establecimiento educacional. Los estudiantes de establecimientos **particulares pagados** obtienen mejores resultados que los de establecimientos **particulares subvencionados**, y estos a su vez mejores que los de establecimientos **municipales**.

### Nivel Socioeconómico: Prioritario vs. No Prioritario

Para incluir el nivel socioeconómico en el análisis, una variable útil es si el estudiante es **prioritario** o **no prioritario**. El Ministerio de Educación (Mineduc) clasifica a los estudiantes como prioritarios en base a antecedentes socioeconómicos que estarían asociados a una mayor dificultad en los procesos de aprendizaje.

Existen diferencias estadísticamente significativas en los puntajes obtenidos por estudiantes prioritarios y no prioritarios, siendo los estudiantes prioritarios quienes obtienen puntajes más bajos.

Además, estas variables (comuna, dependencia y ser prioritario) interactúan entre sí. Por ejemplo, los estudiantes **no prioritarios** de establecimientos **municipales** obtienen mejores resultados que los estudiantes **prioritarios** en establecimientos **particulares subvencionados**.

### Grupo de Referencia

En esta página, presento resultados de análisis considerando un "grupo de referencia" que incluye a los estudiantes que provienen de la misma comuna, misma dependencia y que pertenecen al grupo de estudiantes prioritarios o no prioritarios.

Todos los estudiantes de un grupo (por ejemplo, **no prioritarios** de colegios **privados** en **Vitacura**) pueden obtener puntajes relativamente altos. Obtener 900 puntos puede no ser excepcionalmente alto y no está necesariamente asociado al establecimiento, sino a características personales del estudiante dentro de ese grupo de referencia.

Por otro lado, un estudiante **prioritario** en un establecimiento **municipal** de **Temuco** que obtiene 900 puntos puede considerarse excepcionalmente alto, ya que una proporción muy pequeña de los estudiantes en ese grupo lo alcanza.

### Proporción de Puntajes Excepcionales

Todos los establecimientos educacionales pueden tener una proporción pequeña de estudiantes con puntajes excepcionalmente altos. Sin embargo, **aquellos establecimientos con una mayor proporción de estudiantes en el segmento superior de su grupo son indicadores de una situación que merece atención**. Esto puede deberse a una mejor calidad de enseñanza o a una concentración de mejores estudiantes por diversas razones.

### Cómo Utilizar la Página

A continuación, puedes seleccionar una combinación de **Comuna**, **Dependencia** y **Tipo de estudiante** (Prioritario / No prioritario) para visualizar la distribución de los puntajes de todos los jóvenes en ese grupo que egresaron de educación media en 2023 y dieron la PAES en el proceso regular de 2024.

En esta distribución, identificamos al **2,5%** de los estudiantes con los puntajes más bajos, al **2,5%** con los puntajes más altos y al **95%** restante de los estudiantes entre estos dos extremos.

Consideramos a los estudiantes en el **2,5% superior** (por encima del percentil 97,5) como un foco de análisis debido a sus puntajes considerablemente altos.


*Seleccione un grupo de referencia para observar como se distribuyen los puntajes en dicho grupo:*
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
  directorio.COD_DEPE2,
  directorio.NOM_RBD,
  directorio.NOM_COM_RBD,
  directorio.COD_REG_RBD,
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
const region = selectRegion.region
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
const selectRegion =  (() => {
const options = _.chain([...data])
  .groupBy((d) => d.COD_REG_RBD)
  .map((items, key) => ({ region: key, estudiantes: items.length }))
  .filter(d => d.region !== "null")
  .sortBy((d) => ordenRegiones[d.region])
  .value()

return view(Inputs.select(options,
  {
    label: "Región",
    format: (d) => `${aliasRegiones[d.region]}`
  }
))
})()

```


```js
const selectComuna =  (() => {
const optionsComunas = _.chain([...data])
  .filter((d) => d.COD_REG_RBD == region)
  .groupBy((d) => d.NOM_COM_RBD)
  .map((items, key) => ({ comuna: key, estudiantes: items.length }))
  .sortBy((d) => d.estudiantes)
  .reverse()
  .value()

return view(Inputs.select(optionsComunas,
  {
    label: "Comuna",
    format: (d) => `${d.comuna} (${d.estudiantes} estudiantes)`
  }
))
})()

```


```js
const selectDependencia =  (() => {
const options = _.chain([...data])
    .filter((d) => d.NOM_COM_RBD == comuna)
    .groupBy((d) => d.dependencia)
    .map((items, key) => ({ dependencia: key, COD_DEPE2: ![1,5].includes(items[0].COD_DEPE2) ? items[0].COD_DEPE2: slepComuna.esSLEP ? 5 : 1, estudiantes: items.length }))
    .sortBy((d) => d.estudiantes)
    .reverse()
    .value();

return view(Inputs.select(options,
  {
    label: "Dependencia",
    format: (d) => `${ d.dependencia !== 'PUB' 
    ? aliasDependencia[d.dependencia]
    : slepComuna.esSLEP
    ? aliasDependencia2[5]
    : aliasDependencia2[1]
    } (${d.estudiantes} estudiantes)`
  }
))
})()

```

```js
const selectTipo =  (() => {
const options = _.chain([...data])
    .filter((d) => d.NOM_COM_RBD == comuna && d.dependencia == dependencia)
    .groupBy((d) => d.tipo)
    .map((items, key) => ({ tipo: key, estudiantes: items.length }))
    .sortBy((d) => d.estudiantes)
    .reverse()
    .value();

return view(Inputs.select(options,
  {
    label: "Prioritario",
    format: (d) => `${aliasTipo[d.tipo]} (${d.estudiantes} estudiantes)`
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

  display(dataPlot.length >= 100 ?
  (html`Comuna <b>${comuna}</b>, estudiantes <b>${
    tipo == 1 ? "Prioritarios" : "No prioritarios"
  }</b> en establecimientos <b>${
    aliasDependencia[dependencia]
  }</b> <br><br>El <b>95%</b> de los estudiantes obtiene un puntaje entre <b>${d3.format(
    ".0f"
  )(registroFoco.p2_5)}</b> y <b>${d3.format(".0f")(
    registroFoco.p97_5
  )}</b><br><br>
  <b>${totalEstudiantesDestacados} estudiantes que están por sobre ese rango</b> provienen de ${establecimientosTop.length} establecimientos`)
   : html`<span></span>`)
```



<div class="card">
${dataPlot.length >= 100 ? dodgeChart : 'El número de estudiantes en este grupo es reducido (< 100) y el detalle se omite para evitar generalizaciones incorrectas.'}
</div>

```js
const factor = 10
const heightFactor = 0.5
```
```js

const dodgeChart = (() => {
  //return dataPlot;

  const media = _.chain(dataPlot)
    .map((d) => d.PROMEDIO_PAES)
    .mean()
    .value();

  const height = width * heightFactor;

  const r = Math.sqrt(((height-80) * (width-20)) / (dataPlot.length * Math.PI * factor));
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
          title: d => `Puntaje: ${d.PROMEDIO_PAES}\n${d.NOM_RBD}\n${aliasDependencia2[d.COD_DEPE2]}`,
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
    .filter(d => d.estudiantesSobreLoEsperado > 0)
    .sortBy((d) => d.porcentaje)
    .reverse()
    .value();
})()
```

```js
display(dataPlot.length >= 100 
? html`<h3> Establecimientos con mayor proporción de estudiantes en el segmento superior (sobre percentil 97,5%) </h3>
${comuna} | ${ dependencia !== 'PUB' 
    ? aliasDependencia[dependencia]
    : slepComuna.esSLEP
    ? aliasDependencia2[5]
    : aliasDependencia2[1]
    } | ${aliasTipo[tipo]}
<ul>
${establecimientosTop
    .filter((d) => d.porcentaje > 0.025 && d.totalEstudiantes > 10)
    .map(
      (d) =>
        html`<li>${d.NOM_RBD} ${d.estudiantesSobreLoEsperado} de ${
          d.totalEstudiantes
        } (${d3.format(".1%")(d.porcentaje)})`
    )} 
  </ul>
  <div class="muted">Nota: se omiten establecimientos con menos de 10 estudiantes en el grupo seleccionado</div>
`
: html`<span></span>`)
```

```sql id=tablaEstablecimientos
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
  ORDER BY NOM_COM_RBD, tipo,dependencia),

detalleEstudiantes as (
SELECT  
  tabla.MRUN, 
  directorio.RBD,
  directorio.NOM_RBD,
  directorio.NOM_COM_RBD,
  directorio.COD_REG_RBD,
  directorio.COD_DEPE2,
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
  AND intervalosPorComuna.dependencia = tabla.dependencia),

resumenPorRBD as (
SELECT RBD,NOM_RBD,
 NOM_COM_RBD,
 COD_DEPE2,
 dependencia, count(*) as N_total, 
SUM(CASE WHEN ratingPercentil == 'alto' THEN 1 ELSE 0 END)::Int as alto_total,
SUM(CASE WHEN tipo == 1 THEN 1 ELSE 0 END)::Int as N_prioritario,
SUM(CASE WHEN tipo == 1 AND ratingPercentil == 'alto' THEN 1 ELSE 0 END)::Int as alto_prioritario,

SUM(CASE WHEN tipo == 0 THEN 1 ELSE 0 END)::Int as N_no_prioritario,
SUM(CASE WHEN tipo == 0 AND ratingPercentil == 'alto' THEN 1 ELSE 0 END)::Int as alto_no_prioritario,

FROM  detalleEstudiantes
GROUP BY RBD,NOM_RBD, NOM_COM_RBD,COD_DEPE2,dependencia)

SELECT *, 
  alto_total/N_total as tasa_total,
  alto_prioritario/N_prioritario as tasa_prioritario,
  alto_no_prioritario/N_no_prioritario as tasa_no_prioritario
FROM resumenPorRBD
ORDER BY tasa_total DESC
```

## Establecimientos del país con mayor proporción de estudiantes sobre el percentil 97,5% en su respectivo grupo de referencia

```js
const selectDependeciaRanking =  (() => {
const options = [
  1,5,2,3
];

return view(Inputs.select(options,
  {
    label: "Dependencia",
    format: (d) => `${aliasDependencia2[d]}`
  }
))
})()

```

```js
// Generate the table for the number of students by type of institution
html`
<div class="table-responsive">
<table class="table tablaEscuela2">
<thead>
<tr>
<th colspan="3"></th>
<th colspan="2">Total estudiantes</th>
<th colspan="2">Estudiantes Prioritarios</th>
</tr>
<tr>
<th>Establecimiento</th>
<th>Comuna</th>
<th>Dependencia</th>
<th>Número</th>
<th>Sobre 97,5%</th>
<th>Número</th>
<th>Sobre 97,5%</th>
</tr>
</thead>
<tbody>
${_.chain([...tablaEstablecimientos])
.filter(d => d.N_total > 20 && d.tasa_total > 0.05 && d.COD_DEPE2 == selectDependeciaRanking)

.map(d => html`<tr>
<td>${d.NOM_RBD}</td>
<td>${d.NOM_COM_RBD}</td>
<td>${aliasDependencia2[d.COD_DEPE2]}</td>
<td>${d.N_total}</td>
<td>${d3.format(".1%")(d.tasa_total)}</td>
<td>${d.N_prioritario >= 10 ? d.N_prioritario : d.N_prioritario}</td>
<td>${d.N_prioritario >= 10 ? d3.format(".1%")(d.tasa_prioritario) : "*"}</td>


</tr>`).value()}
<tbody>
<caption>
Notas: 
<ul>
<li>Se incluyen establecimientos con 20 o más estudiantes egresados en 2023 con puntaje válido en PAES 2024
<li>Se incluyen establecimientos con al menos 5% del total de estudiantes sobre el 97.5% superior de puntajes de acuerdo a la respectiva comuna, dependencia y tipo de estudiante (prioritario / no priorotario) 
<li> Los establecimientos con Administraciín Delegada se consideran junto a los Particulares Subvencionados en el grupo de referencia
<li> (*) Cuando el establecimiento tiene menos de 10 estudiantes prioritarios, se omite la respectva proporción de estudiantes prioritarios sobre el percentil 97,5%.
</ul>
</caption>
</table>
</div>`
```

## Fuente de datos
Los datos utilizados en este análisis provienen de Datos Abiertos del MINEDUC (https://datosabiertos.mineduc.cl/).

* Datos de jóvenes egresados de Educación Media: https://datosabiertos.mineduc.cl/notas-de-ensenanza-media-y-percentil-jovenes/
* Alumnos preferentes, prioritarios y beneficiarios SEP: https://datosabiertos.mineduc.cl/alumnos-preferentes-prioritarios-y-beneficiarios-sep/
* Pruebas de admisión a la educación superior: https://datosabiertos.mineduc.cl/pruebas-de-admision-a-la-educacion-superior/
* Directorio de Establecimientos Educacionales: https://datosabiertos.mineduc.cl/directorio-de-establecimientos-educacionales/

```js
const aliasDependencia2 = ({
  1: "Municipal",
  2: "Particular Subvencionado",
  3: "Particular Pagado",
  4: "Adm. Delegada",
  5: "Servicio Local",

})
```

```js
const aliasRegiones = ({
  15: "De Arica y Parinacota",
  1: "De Tarapacá",
  2: "De Antofagasta",
  3: "De Atacama",
  4: "De Coquimbo",
  5: "De Valparaíso",
  13: "Metropolitana de Santiago",
  6: "Del Libertador B. O'Higgins",
  7: "Del Maule",
  16: "De Ñuble",
  8: "Del Bíobío",
  9: "De La Araucanía",
  14: "De Los Ríos",
  10: "De Los Lagos",
  11: "De Aisén del Gral. C. Ibáñez del Campo",
  12: "De Magallanes y de La Antártica Chilena"
})

const ordenRegiones = ({
  15: 0,
  1: 1,
  2: 2,
  3: 3,
  4: 4,
  5: 5,
  13: 6,
  6: 7,
  7: 8,
  16:9,
  8: 10,
  9: 11,
  14: 12,
  10: 13,
  11: 14,
  12: 15
})
```

```sql id=[slepComuna]
WITH tabla as (SELECT NOM_COM_RBD, COD_DEPE2,count(*) numEstablecimientos
FROM directorio
WHERE COD_DEPE2 = 1 OR COD_DEPE2 = 5
GROUP BY NOM_COM_RBD, COD_DEPE2)

SELECT NOM_COM_RBD, COD_DEPE2 = 5 as esSLEP
FROM tabla
WHERE NOM_COM_RBD  = ${comuna}
```

