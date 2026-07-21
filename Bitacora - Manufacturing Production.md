# Bitácora - Manufacturing Production Dashboard

## Preguntas de negocio
¿Cuál es la producción total de la planta?
¿Estamos cumpliendo la meta de producción?
¿Qué máquinas producen más y cuáles menos?
¿Qué máquinas son las más eficientes?
¿Qué materiales representan la mayor producción?
¿Cómo ha evolucionado la producción a lo largo del tiempo?
¿Qué máquinas consumen más energía?
¿Cuántas órdenes se han procesado?
¿Cuál es el tiempo promedio de producción por orden?
¿Dónde existen oportunidades para mejorar la eficiencia o reducir costos?

¿Cuántas órdenes se han procesado en total?
¿Qué % de órdenes cumple la meta (Completed vs total)?
¿Qué máquinas procesan más y cuáles menos órdenes?
¿Qué máquinas son más eficientes (Optimization_Category)?
¿Qué tipos de operación (Operation_Type) representan mayor volumen de trabajo?
¿Cómo ha evolucionado el número de órdenes a lo largo del tiempo?
¿Qué máquinas consumen más energía?
¿Cuál es el tiempo promedio de producción por orden?
¿Dónde hay oportunidades de mejora (cruce de Failed/Delayed por máquina u operación)?

## Limpieza de datos
- **Nulos en Actual_Start/Actual_End (129 filas)**: se dejan en blanco, 
  corresponden 1:1 a Job_Status = "Failed". No se imputan.
- **Tipos de datos**: fechas convertidas a Fecha/Hora, IDs como texto.
- Revision de cada tipo de dato en general
Job_ID/Machine_ID/Operation_Type/Job_Status estén como **Texto**
Processing_Time/Machine_Availability como **Entero**
Energy_Consumption/Material_Used **Decimal**

las 4 fechas como **Fecha/Hora**.

## Decisiones de modelado
-**Creacion de tabla calendario**
-**Sincronizado**
## DAX
## DAX

### Columna calculada (tabla `hybrid_manufacturing_categorical`)
```dax
Eficiencia_Score = 
SWITCH(
    hybrid_manufacturing_categorical[Optimization_Category],
    "Low Efficiency", 25,
    "Moderate Efficiency", 50,
    "High Efficiency", 75,
    "Optimal Efficiency", 100
)
```

### Medidas (tabla `Medidas`)
```dax
Ordenes_Totales = COUNTROWS(hybrid_manufacturing_categorical)

Produccion_Total = 
CALCULATE(COUNTROWS(hybrid_manufacturing_categorical), hybrid_manufacturing_categorical[Job_Status] = "Completed")

%_Cumplimiento = 
DIVIDE(
    CALCULATE(COUNTROWS(hybrid_manufacturing_categorical), hybrid_manufacturing_categorical[Job_Status] = "Completed"),
    COUNTROWS(hybrid_manufacturing_categorical)
)

Tiempo_Promedio = AVERAGE(hybrid_manufacturing_categorical[Processing_Time])

Energia_Total = SUM(hybrid_manufacturing_categorical[Energy_Consumption])

Eficiencia_Promedio = AVERAGE(hybrid_manufacturing_categorical[Eficiencia_Score])
```
## Descripción del Reporte

Este dashboard analiza el desempeño de una planta de manufactura con datos de 
1000 órdenes de producción distribuidas en 5 máquinas (M01-M05) y 5 tipos de 
operación (Lathe, Grinding, Milling, Additive, Drilling), en un rango de 8 días 
(18-25 de marzo 2023).

### Objetivo
Responder preguntas clave de negocio sobre producción, eficiencia y consumo 
energético, permitiendo identificar qué máquinas rinden más, cuáles consumen 
más energía, y dónde existen oportunidades de mejora (órdenes fallidas o 
retrasadas).

### Estructura
- **6 KPIs**: Producción Total, Órdenes Totales, Eficiencia Promedio, Tiempo 
  Promedio, Energía Total, % Cumplimiento.
- **6 visuales**: evolución temporal, producción por máquina, eficiencia por 
  máquina, distribución por tipo de operación, consumo energético por máquina, 
  y detalle tabular por orden.
- **4 segmentadores**: Fecha, Máquina, Tipo de Operación, Estado de la Orden.

### Hallazgos clave
- M01 es la máquina más productiva y también la de mayor consumo energético — 
  correlación esperada por volumen, no necesariamente ineficiencia.
- La eficiencia general es baja (~38%), con poca variación entre máquinas: la 
  mayoría de las órdenes caen en categoría "Low Efficiency".
- Solo el 67.3% de las órdenes se completan exitosamente.

### Limitaciones del dataset
- No incluye una columna real de "producción" (unidades) ni "material" 
  categórico ni "meta" definida — se usaron proxies (conteo de órdenes 
  completadas, Operation_Type, y meta = % Completed).
- El rango de fechas es muy corto (8 días), limitando el análisis de tendencia 
  temporal.