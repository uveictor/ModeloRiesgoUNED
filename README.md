# Riesgo climático y crédito hipotecario

Repositorio asociado al trabajo **“Riesgo climático y crédito hipotecario: evidencia empírica sobre la exposición bancaria en zonas inundables en España”**.

El proyecto construye una base de datos reproducible para analizar si la exposición provincial al riesgo físico de inundación se relaciona con variables relevantes del riesgo de crédito hipotecario en España. Para ello se combinan datos geoespaciales de zonas inundables con información pública sobre hipotecas, ejecuciones hipotecarias, precios de vivienda, población, renta y mercado laboral.

A partir de esta base se construyen proxies agregadas de **PD, LGD, EAD y pérdida esperada**, se realiza un análisis descriptivo, se estiman modelos econométricos y se desarrollan ejercicios de *stress test* climático.

---

## 1. Estructura del proyecto

La información se organiza dentro de la carpeta `data`, separando los datos originales de los datos procesados.

```text
data/
│
├── raw/
│   ├── cnig_limites/
│   ├── ine/
│   │   ├── epa_provincial.csv
│   │   ├── ine_ejecuciones_hipotecarias.csv
│   │   ├── ine_hipotecas.csv
│   │   ├── poblacion_por_provincia.xlsx
│   │   └── renta_media_provincial.csv
│   ├── mivau/
│   │   └── precio_vivienda_libre.XLS
│   └── snczi/
│       └── T100/
│           ├── canarias/
│           └── peninsula_baleares/
│
└── processed/
    ├── exposicion_inundacion_provincial_T100.csv
    ├── hipotecas_provincia_anio.csv
    ├── ejecuciones_hipotecarias_provincia_anio.csv
    ├── precio_vivienda_provincia_anio.csv
    ├── epa_provincia_anio.csv
    ├── poblacion_provincia_anio.csv
    ├── renta_provincia_anio.csv
    ├── panel_final_provincia_anio.csv
    ├── panel_modelo_principal.csv
    ├── panel_actualizado.csv
    ├── provincias_cnig_3035.gpkg
    ├── provincias_exposicion_inundacion_T100.gpkg
    └── zonas_inundables_T100.gpkg
```

La carpeta `raw` contiene las fuentes originales descargadas de organismos oficiales. La carpeta `processed` contiene los ficheros ya depurados y preparados para el análisis.

---

## 2. Fuentes de datos

El proyecto utiliza fuentes públicas oficiales.

| Bloque | Fuente | Uso |
|---|---|---|
| Zonas inundables | SNCZI / MITECO | Exposición provincial a inundaciones T100 |
| Límites provinciales | CNIG / IGN | Cálculo de superficies e intersección espacial |
| Hipotecas | INE | Número, importe total e importe medio hipotecario |
| Ejecuciones hipotecarias | INE | Proxy de deterioro hipotecario |
| Precio de vivienda | MIVAU / MITMA | Aproximación al valor del colateral |
| Población, renta y empleo | INE / EPA | Variables de control |

---

## 3. Nota sobre `zonas_inundables_T100.gpkg`

El fichero `data/processed/zonas_inundables_T100.gpkg` no se incluye en el repositorio porque tiene un tamaño elevado y puede superar los límites de subida de plataformas como GitHub.

Este archivo se genera localmente mediante la celda de código correspondiente del proceso geoespacial. La celda descarga o lee las capas originales del SNCZI, las unifica, limpia las geometrías y guarda el resultado en formato `.gpkg`.

El objetivo de este fichero es mejorar el rendimiento. En lugar de leer y procesar repetidamente todos los shapefiles originales, el análisis carga directamente `zonas_inundables_T100.gpkg` una vez generado.

El flujo es:

```text
Capas originales SNCZI T100
        ↓
celda de descarga / lectura geoespacial
        ↓
unificación y limpieza
        ↓
zonas_inundables_T100.gpkg
        ↓
intersección con provincias
        ↓
exposicion_inundacion_provincial_T100.csv
```

Si `zonas_inundables_T100.gpkg` no existe en `data/processed`, debe ejecutarse primero la celda de preparación geoespacial.

---

## 4. Preparación de los datos

El proceso general sigue estas fases:

```text
data/raw
   ↓
limpieza de fuentes originales
   ↓
homogeneización territorial y temporal
   ↓
data/processed
   ↓
panel provincia-año
   ↓
variables de riesgo de crédito
   ↓
análisis descriptivo, modelos y stress test
```

Primero se calcula la exposición provincial a inundaciones a partir de las capas geoespaciales. Después se procesan las fuentes estadísticas del INE y MIVAU, transformándolas a una estructura común:

```text
provincia | año | variable
```

Las series mensuales se agregan a frecuencia anual y las series trimestrales se convierten en medias anuales. Finalmente, todas las fuentes se unen mediante provincia y año.

---

## 5. Ficheros principales

| Fichero | Descripción |
|---|---|
| `exposicion_inundacion_provincial_T100.csv` | Exposición provincial a inundaciones T100 |
| `panel_final_provincia_anio.csv` | Panel completo provincia-año |
| `panel_modelo_principal.csv` | Panel utilizado en los modelos econométricos |
| `panel_actualizado.csv` | Panel con información reciente disponible |
| `provincias_exposicion_inundacion_T100.gpkg` | Capa provincial con exposición a inundación |
| `zonas_inundables_T100.gpkg` | Capa procesada de zonas inundables, generada localmente |

---

## 6. Variables construidas

A partir del panel se construyen proxies agregadas de riesgo de crédito.

| Variable | Descripción |
|---|---|
| PD proxy | Aproximación al deterioro hipotecario mediante ejecuciones hipotecarias relativas |
| EAD proxy | Aproximación a la exposición mediante el importe medio hipotecario |
| LTV proxy | Relación entre importe medio hipotecario y valor estimado de la vivienda |
| LGD proxy | Pérdida potencial no cubierta por el valor recuperable del colateral |
| LGD climática | LGD proxy ajustada por exposición a inundación |
| PE proxy | Pérdida esperada aproximada mediante PD × LGD × EAD |
| PE climática | Pérdida esperada ajustada por el canal climático del colateral |

Estas variables son aproximaciones empíricas construidas con datos públicos agregados y no deben interpretarse como parámetros internos regulatorios de una entidad financiera.

---

## 7. Análisis realizado

El análisis se divide en tres bloques.

### 7.1. Análisis descriptivo

Se estudia la distribución territorial de la exposición a inundaciones y su relación con variables hipotecarias, inmobiliarias y de riesgo de crédito.

### 7.2. Modelos econométricos

Se estiman tres modelos sobre el panel provincia-año:

1. Modelo de deterioro hipotecario, usando la PD proxy.
2. Modelo de severidad, usando la LGD proxy climática.
3. Modelo de pérdida esperada, usando la PE proxy climática.

Los modelos incorporan efectos fijos temporales para controlar shocks comunes por año. No se incluyen efectos fijos provinciales porque la exposición a inundación es una característica estructural de cada provincia.

### 7.3. Stress test climático

Se plantean tres escenarios de deterioro del valor del colateral:

| Escenario | Shock sobre el colateral |
|---|---:|
| Base | 0% |
| Moderado | Haircut máximo del 5% |
| Severo | Haircut máximo del 10% |

El objetivo es evaluar cómo cambia la pérdida esperada cuando se introduce un shock climático sobre el valor recuperable de la vivienda.

---

## 8. Cómo ejecutar el proyecto

El orden recomendado de ejecución es:

```text
1. Colocar las fuentes originales en data/raw.
2. Ejecutar la celda de descarga o lectura de las capas geoespaciales del SNCZI.
3. Generar zonas_inundables_T100.gpkg en data/processed.
4. Cargar zonas_inundables_T100.gpkg en las siguientes ejecuciones.
5. Calcular exposicion_inundacion_provincial_T100.csv.
6. Procesar hipotecas, ejecuciones, precios de vivienda, población, renta y EPA.
7. Homogeneizar nombres y códigos provinciales.
8. Construir panel_final_provincia_anio.csv.
9. Construir panel_modelo_principal.csv.
10. Ejecutar análisis descriptivo, modelos econométricos y stress test climático.
```

---

## 9. Replicabilidad

El proyecto está diseñado para que el fichero final pueda reconstruirse desde las fuentes originales. Los datos originales se conservan en `data/raw`, los datos transformados se guardan en `data/processed` y las transformaciones se realizan mediante scripts o celdas documentadas.

No se modifican manualmente los ficheros originales. El fichero pesado `zonas_inundables_T100.gpkg` puede regenerarse localmente a partir de las capas originales del SNCZI.

---

## 10. Limitaciones

El análisis utiliza datos públicos agregados, por lo que las variables de PD, LGD, EAD y pérdida esperada son proxies. Además, la unidad de análisis es la provincia, por lo que no se observa la localización exacta de cada vivienda hipotecada ni las características individuales del prestatario.

A pesar de estas limitaciones, el enfoque permite construir una aproximación empírica replicable para estudiar la relación entre riesgo físico de inundación y riesgo de crédito hipotecario en España.

---

## Autor

Víctor González Ballesteros  
Máster de Investigación en Economía  
Asignatura: Modelos de riesgo de crédito y supervisión bancaria
