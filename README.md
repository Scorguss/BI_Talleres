# UNIVERSIDAD POLITÉCNICA NACIONAL

## Práctica de Data Warehouse y Consultas OLAP

**Asignatura:** Inteligencia de Negocios
**Paralelo:** [Paralelo]
**Docente:** [Nombre del docente]

**Integrantes:**

* [Integrante 1]
* [Integrante 2]
* [Integrante 3]

**Fecha:** 19 de junio de 2026

---

# Índice

1. [Introducción](#introducción)
2. [Objetivos](#objetivos)
3. [Descripción del Data Warehouse](#descripción-del-data-warehouse)
4. [Modelo Estrella Implementado](#modelo-estrella-implementado)
5. [Carga de Datos en PostgreSQL](#carga-de-datos-en-postgresql)
6. [Consultas OLAP Realizadas](#consultas-olap-realizadas)

   * [Consulta 1](#consulta-1-costo-total-de-atención-por-especialidad-ciudad-y-mes)
   * [Consulta 2](#consulta-2-emergencias-por-ciudad-mes-y-género)
   * [Consulta 3](#consulta-3-costo-promedio-por-diagnóstico-tipo-de-seguro-y-ciudad)
7. [Conclusiones](#conclusiones)
8. [Referencias Bibliográficas](#referencias-bibliográficas)
9. [Declaración de Uso de Inteligencia Artificial](#declaración-de-uso-de-inteligencia-artificial)

---

# Introducción

Los sistemas OLAP (Online Analytical Processing) permiten analizar grandes volúmenes de información desde múltiples perspectivas mediante el uso de modelos multidimensionales. En la presente práctica se implementó un esquema estrella en PostgreSQL utilizando información relacionada con atenciones médicas, pacientes, diagnósticos, seguros y especialidades médicas.

Posteriormente se realizaron consultas analíticas para obtener indicadores relevantes que permitan apoyar la toma de decisiones mediante técnicas de análisis multidimensional.

# Objetivos

## Objetivo General

Implementar un modelo dimensional tipo estrella en PostgreSQL y ejecutar consultas OLAP para el análisis de información médica.

## Objetivos Específicos

* Implementar las dimensiones y tabla de hechos en PostgreSQL.
* Importar datos desde archivos CSV generados a partir de un archivo Excel.
* Ejecutar consultas OLAP utilizando agregaciones multidimensionales.
* Analizar los resultados obtenidos mediante diferentes perspectivas de negocio.

# Descripción del Data Warehouse

El Data Warehouse fue construido utilizando un modelo estrella compuesto por una tabla de hechos denominada **Fact_Visita** y siete dimensiones:

* Dim_Fecha
* Dim_Paciente
* Dim_Ciudad
* Dim_Especialidad
* Dim_Diagnostico
* Dim_Seguro
* Dim_Resultado

La tabla de hechos almacena métricas relacionadas con costos, duración de estancia y tipo de atención médica.

# Modelo Estrella Implementado

## Figura 1. Modelo Estrella del Data Warehouse

**Descripción:** Esquema estrella utilizado para la implementación del almacén de datos en PostgreSQL.

> Insertar aquí la imagen del modelo estrella.

![Modelo Estrella](imagenes/modelo_estrella.png)

# Carga de Datos en PostgreSQL

Los datos fueron exportados desde archivos XLSX hacia archivos CSV independientes para cada dimensión y para la tabla de hechos.

Posteriormente se realizó la importación mediante las herramientas de PostgreSQL utilizando la funcionalidad de importación de archivos CSV.

## Tabla 1. Dimensiones cargadas

| Tabla            |
| ---------------- |
| dim_fecha        |
| dim_paciente     |
| dim_ciudad       |
| dim_especialidad |
| dim_diagnostico  |
| dim_seguro       |
| dim_resultado    |

**Descripción:** Dimensiones utilizadas dentro del modelo estrella.

## Tabla 2. Tabla de Hechos

| Tabla       |
| ----------- |
| fact_visita |

**Descripción:** Tabla que almacena las métricas utilizadas para el análisis multidimensional.

# Consultas OLAP Realizadas

## Consulta 1: Costo total de atención por especialidad, ciudad y mes

### Descripción

Esta consulta permite analizar el costo total de las atenciones médicas agrupadas por especialidad médica, ciudad y mes.

### Consulta SQL

```sql
SELECT
    e.especialidad,
    c.ciudad,
    f.mes,
    SUM(v.total_cost) AS costo_total
FROM fact_visita v
JOIN dim_especialidad e
    ON v.id_especialidad = e.id_especialidad
JOIN dim_ciudad c
    ON v.id_ciudad = c.id_ciudad
JOIN dim_fecha f
    ON v.id_fecha = f.id_fecha
GROUP BY
    e.especialidad,
    c.ciudad,
    f.mes
ORDER BY
    e.especialidad,
    c.ciudad,
    f.mes;
```

### Resultado

**Figura 2. Resultado de la Consulta 1**

> Insertar captura de pantalla del resultado.

![Consulta1](imagenes/consulta1.png)

**Descripción:** Costos totales agrupados por especialidad médica, ciudad y mes.

---

## Consulta 2: Emergencias por ciudad, mes y género

### Descripción

Esta consulta permite identificar la cantidad de emergencias registradas según la ciudad, el mes y el género de los pacientes.

### Consulta SQL

```sql
SELECT
    f.mes,
    p.genero,
    c.ciudad,
    COUNT(*) AS total_emergencias
FROM fact_visita v
JOIN dim_fecha f
    ON v.id_fecha = f.id_fecha
JOIN dim_paciente p
    ON v.sk_paciente = p.sk_paciente
JOIN dim_ciudad c
    ON v.id_ciudad = c.id_ciudad
WHERE v.is_emergency = TRUE
GROUP BY
    f.mes,
    p.genero,
    c.ciudad
ORDER BY
    f.mes,
    p.genero,
    total_emergencias DESC;
```

### Resultado

**Figura 3. Resultado de la Consulta 2**

> Insertar captura de pantalla del resultado.

![Consulta2](imagenes/consulta2.png)

**Descripción:** Total de emergencias agrupadas por ciudad, mes y género.

---

## Consulta 3: Costo promedio por diagnóstico, tipo de seguro y ciudad

### Descripción

Esta consulta permite identificar el costo promedio de las visitas médicas considerando el diagnóstico, el tipo de seguro y la ciudad donde se realizó la atención.

### Consulta SQL

```sql
SELECT
    d.diagnostico,
    s.seguro,
    c.ciudad,
    ROUND(AVG(v.total_cost),2) AS costo_promedio
FROM fact_visita v
JOIN dim_diagnostico d
    ON v.id_diagnostico = d.id_diagnostico
JOIN dim_seguro s
    ON v.id_seguro = s.id_seguro
JOIN dim_ciudad c
    ON v.id_ciudad = c.id_ciudad
GROUP BY
    d.diagnostico,
    s.seguro,
    c.ciudad
ORDER BY costo_promedio DESC;
```

### Resultado

**Figura 4. Resultado de la Consulta 3**

> Insertar captura de pantalla del resultado.

![Consulta3](imagenes/consulta3.png)

**Descripción:** Costo promedio por visita considerando diagnóstico, seguro y ciudad.

# Conclusiones

1. La implementación del esquema estrella permitió organizar eficientemente la información para análisis multidimensionales.

2. PostgreSQL proporciona funcionalidades suficientes para ejecutar consultas OLAP mediante operaciones de agregación y agrupamiento.

3. Las consultas realizadas permitieron identificar patrones relacionados con costos médicos, distribución de emergencias y comportamiento de los seguros médicos.

4. El modelo dimensional facilita la generación de reportes y el análisis de información para la toma de decisiones.

# Referencias Bibliográficas

Kimball, R., & Ross, M. (2013). *The Data Warehouse Toolkit: The Definitive Guide to Dimensional Modeling* (3rd ed.). Wiley.

Inmon, W. H. (2005). *Building the Data Warehouse* (4th ed.). Wiley.

PostgreSQL Global Development Group. (2026). *PostgreSQL Documentation*. https://www.postgresql.org/docs/

# Declaración de Uso de Inteligencia Artificial

Para el desarrollo de esta práctica se utilizaron herramientas de Inteligencia Artificial como apoyo en actividades relacionadas con la generación de consultas SQL, validación de sintaxis y asistencia en la redacción de la documentación.

El porcentaje estimado de apoyo proporcionado por herramientas de Inteligencia Artificial corresponde aproximadamente al **20%** del trabajo realizado. La implementación, ejecución, validación de resultados y análisis final fueron efectuados por los integrantes del grupo.
