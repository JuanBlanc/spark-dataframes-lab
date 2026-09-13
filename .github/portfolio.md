---
title: Spark DataFrames Lab
description: Clientes y pedidos sobre un clúster Spark real en Docker, hasta escribir Parquet particionado por segmento.
slug: spark-dataframes-lab
tags: [Apache Spark, PySpark, Jupyter, Parquet, Docker]
cover: docs/img/07_agregaciones.png
order: 6
---

Práctica de **Apache Spark con DataFrames** sobre dos ficheros de clientes y
pedidos, resuelta de punta a punta: lectura con esquema, limpieza, transformación,
join, agregaciones, clasificación por segmento, la misma consulta en Spark SQL y
escritura del resultado en Parquet.

## El entorno

No es un `local[*]` disfrazado. El `docker compose` levanta un clúster de verdad:

```
Spark master   :8080
Worker 1       :8081
Worker 2       :8082
History server :18080
JupyterLab     :8888
```

Con dos workers separados del driver, el shuffle de un join deja de ser un
detalle invisible: se ve el plan en la interfaz del master y la ejecución
terminada en el history server. Eso es lo que hace que la práctica enseñe algo
sobre Spark y no solo sobre la API de DataFrames.

## Decisiones

- **Esquema explícito en la lectura**, no `inferSchema`. Inferir obliga a Spark a
  recorrer el fichero una vez de más solo para adivinar tipos, y adivina: un
  código postal con ceros delante acaba siendo un entero y se pierden los ceros.
- **Parquet particionado por segmento** en la salida. El resultado queda como
  `segmento=Estandar/` y `segmento=Premium/`, así que una consulta por segmento
  lee solo su carpeta en vez de todo el conjunto. Columnar y comprimido, además,
  frente a un CSV que hay que volver a parsear entero.
- **La misma agregación resuelta dos veces**, con la API de DataFrames y con
  Spark SQL, para ver que acaban en el mismo plan físico. Elegir una u otra es una
  cuestión de a quién le toque leer el código, no de rendimiento.

## Cómo se levanta

```bash
cd spark_jupyter
docker compose -f docker-compose-jupyter.yml up -d --build
```

JupyterLab en `:8888` (token `spark`); el cuaderno es
`notebooks/practica_clientes_pedidos.ipynb` y los datos están montados en
`/opt/spark-apps/datos/`. El enunciado y las evidencias de cada paso están en
`docs/`.
