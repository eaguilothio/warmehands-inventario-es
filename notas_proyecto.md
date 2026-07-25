# Análisis de Inventario — WarmeHands Logistics Inc.

## Contexto del proyecto
WarmeHands Logistics contrató un equipo de analistas de datos para hacer un análisis de inventario que ayude a optimizar el control de stock y reducir gastos por mala gestión. El equipo de datos se organiza por país: cada analista es responsable de dar seguimiento a un mercado específico, aplicando sobre él las reglas de negocio definidas de forma centralizada (cálculo de Ingresos, COGS, Beneficio).

## Objetivo del proyecto
En este proyecto se asume el rol de analista responsable del mercado de **España** durante el año **2021**, con el objetivo de evaluar sus **ingresos, costes (COGS) y beneficio**, desglosados por categoría de producto, y aplicar las reglas de negocio establecidas por la organización para determinar si el mercado está cumpliendo los objetivos esperados. Se elige 2021 como periodo de referencia por ser el primer año completo con datos consistentes tras la limpieza.

## Preguntas de negocio a responder
- ¿Cuáles son los ingresos, el COGS y el beneficio generados por el mercado de España en 2021?
- ¿Cómo se distribuyen esos ingresos y costes entre las distintas categorías de producto?

---

## 1. Preparación y limpieza de datos

### 1.1 Reglas generales aplicadas a todas las tablas
- Se cargó el archivo Excel completo (todas sus tablas) en **Power Query Editor**.
- Regla aplicada a **todas las tablas**: `Inicio → Transformar datos → Transformar → Recortar (Trim) → Limpiar (Clean)`.
  - Método rápido para varias columnas a la vez: `Ctrl + clic` en los encabezados → pestaña **Transformar** → **Recortar** → **Limpiar**.
  - Justificación: sin este paso pueden quedar espacios invisibles al final de un valor (ej. `"toys & edibles "` frente a `"toys & edibles"`), que Power BI trata como dos categorías distintas aunque visualmente sean iguales.
  - Después de recortar + limpiar, cambiar el tipo de dato a **numérico** donde corresponda.
- **Reglas para columnas ID**: ninguna fila debe tener espacios en blanco o duplicados.

### 1.2 Columna `retail_price` (tabla `Stock`)
- Contenía el símbolo `$`. Se limpió con **Reemplazar valores**:
  - Click derecho en el encabezado → *Reemplazar los valores* → buscar `$` → reemplazar con **nada (en blanco)**.

### 1.3 Archivo `categories.csv`
- Importado desde Power Query (carpeta *Datasets* del escritorio), usando la primera fila como encabezado.
- Se aplicó recortar + limpiar por el mismo motivo explicado en 1.1.
- Columna `category`: se detectaron valores incorrectos. Los únicos valores válidos son:
  - `decoration`
  - `office & school`
  - `jewelry`
  - `home accessories`
  - `toys & edibles`
  - Corrección: filtrar por el valor incorrecto → reemplazar por el valor correcto → quitar el filtro para que vuelvan a aparecer todas las filas.
- Columna `ID`: tenía el prefijo `SKU-`. Se eliminó ese prefijo (click derecho en el encabezado → Reemplazar valores) para que coincidiera con el formato de ID de las demás tablas. Justificación: si una tabla tiene valores como `"SKU-12345"` y otra tiene solo `"12345"` para referirse exactamente al mismo producto, las relaciones no funcionan, porque Power BI exige que los textos/códigos sean 100% idénticos para enlazarlos (las tablas pueden llamarse distinto, pero los valores de la clave tienen que coincidir exactamente).

### 1.4 Columna `Country` (tabla `Orders`)
- Formato inconsistente respecto a la tabla `Country`: aparecían valores con prefijo y guion bajo, como `NA_United Kingdom`, en vez de `United Kingdom`.
- Se corrigió en Power Query con **Reemplazar valores**, sustituyendo el prefijo `NA_` por **nada (vacío)**, para que el nombre del país quedara idéntico en ambas tablas.
- Se limpió para mostrar solo el nombre del país en toda la columna y se aseguró que no hubiera duplicados al filtrar.

### 1.5 División de columna (prefijo - país) en la tabla `Orders`
Los nombres de los países aparecían mezclados con prefijos numéricos (por ejemplo, `14083_Australia`). Se utilizó la herramienta **Dividir columna por delimitador** en Power Query, usando el guion bajo, lo que permitió separar los datos en dos columnas distintas.

Posteriormente se generó un pequeño conflicto de tipo de dato, porque Power Query interpretaba el prefijo como una nueva columna numérica en lugar de texto. Se eliminó el paso automático y se reconfiguró el campo como texto de forma manual, evitando así errores y logrando una separación limpia y correcta de los países para su óptima visualización en Power BI. Las columnas resultantes se renombraron a `country_prefix` y `country_name`.

Al revisar el gráfico se detectó que un 5% de los datos quedaban nulos, lo que indica que no se cargaron bien en el modelo. Dado que este porcentaje es bajo y no afecta significativamente al análisis global, se decidió ocultar los valores nulos directamente desde los filtros de Power BI para asegurar una visualización limpia y correcta del gráfico.

### 1.6 Notas sobre nomenclatura
- Se intentó aplicar `snake_case` de forma uniforme, pero los IDs, fechas y las columnas resultantes del merge daban problemas → se decidió **no forzar el cambio de nombre** en esos casos para evitar romper referencias.

### 1.7 Incidencia detectada: producto RED DRAGONFLY HELICOPTER
Durante la revisión de la tabla `Stock`, se detectó que el producto *RED DRAGONFLY HELICOPTER* aparecía con `retail_price = 0`. 

Al tratarse de un producto comercial, un precio de 0 no tenía sentido de negocio (ningún artículo del catálogo se vende gratis), por lo que se descartó dejarlo así o eliminarlo directamente. Se optó por corregirlo con criterio: se confirmó con el departamento el valor real del `retail_price` de ese producto, **11.91**, restableciéndolo en la tabla.

Este caso sirvió como comprobación de que un valor "raro" (0, nulo, atípico) debe investigarse antes de decidir si se corrige, se sustituye o se elimina — no se toma como válido ni se descarta a la ligera sin entender primero de dónde viene.

---

## 2. Combinación de tablas (Merge) — Stock + Price

**Objetivo:** añadir el precio a cada artículo de `Stock`, asegurando que haya coincidencia para cada uno. Solo había una columna en la tabla `Price`, así que se procede a eliminarla del modelo tras el merge.

**Pasos:**
1. Seleccionar la tabla `Stock` en el panel de consultas.
2. `Inicio → Combinar consultas → Combinar consultas`.
3. En la ventana de combinación: tabla superior `Stock`, tabla inferior `Price`.
4. Marcar la columna clave en común (SKU/ID) en ambas tablas.
5. Tipo de combinación: **Externo izquierdo** (todos los registros de `Stock`, coincidencias de `Price`).
6. Expandir la columna nueva (`Price`), desmarcar la columna ID/SKU duplicada, marcar solo la columna de precio, y desmarcar "usar el nombre original como prefijo".
7. `Cerrar y aplicar` para guardar los cambios.

**Criterio usado para decidir merge vs. relación:**

| Situación | Decisión |
|---|---|
| El valor es único o casi único por fila (precio, SKU, DNI) | ✅ Merge — tabla plana, más cómoda |
| El valor se repite mucho (país, proveedor, categoría, tipo de cliente) | ❌ No merge — mantener tabla separada y conectar por relación en el modelo |

**Sobre la tabla `Price` tras el merge:**
- No se puede eliminar directamente en Power Query porque `Stock` depende de ella.
- Se **deshabilitó su carga** (click derecho en la tabla → desmarcar *Habilitar carga*) y se eliminó del modelo, ya que toda la información relevante (el precio) ya vive en `Stock`.

---

## 3. Modelo de datos y relaciones

Power BI detecta automáticamente relaciones entre tablas que comparten nombres de columna idénticos o claves obvias (SKU, InvoiceNo). Por eso `Orders`–`Stock` y `Stock`–`Costs` quedaron conectadas sin intervención manual.

`Country` y `Customer`, en cambio, no se autodetectaron y hubo que decidir manualmente cómo tratarlas:

| Tabla | Relación con | Campo usado | Estado | Cardinalidad |
|---|---|---|---|---|
| `Country` | `Orders` | `country_name` | ✅ Conectada | 1 a * |
| `Customer` | `Orders` | `InvoiceDate` | ❌ No conectada | Varios a varios (descartada) |

**`Country` — sí se conecta:**
Tras limpiar la tabla `Country` (quitando duplicados de país y eliminando una columna `InvoiceNo` que no le correspondía, residuo de un merge anterior), quedó con un país por fila. Esto permite una relación normal de dimensión-hecho: 1 país → muchos pedidos en `Orders`, usando `country_name` como clave.

**`Customer` — no se conecta:**
La tabla `Customer` no cuenta con una columna `CustomerID` presente también en `Orders`, por lo que no existe un identificador único compartido entre ambas tablas para establecer la relación. El único campo en común es `InvoiceDate`, pero esta fecha no es única ni en `Customer` ni en `Orders`: varios clientes pueden realizar compras el mismo día, y una misma fecha se repite en múltiples filas dentro de `Orders`. Si se forzara la relación a través de este campo, se generaría una cardinalidad de varios a varios, lo que produciría resultados duplicados, inflados o ambiguos en cualquier medida que cruzara datos de clientes con pedidos. Por este motivo, se decide no establecer la relación entre `Customer` y `Orders`.

**Regla general del modelo:** las relaciones y uniones entre tablas se construyen únicamente a partir de identificadores únicos (IDs), evitando el uso de fechas u otros campos no unívocos como clave de conexión. Además, se quitan duplicados de la primera columna (identificador) en todas las tablas dimensión, excepto en la tabla de hechos (`Orders`), donde los duplicados son normales y esperables.

---

## 4. Dashboards

### 4.1 Dashboard: Ingresos por País y Categorías
Se han incorporado dos tarjetas informativas superiores que muestran de forma directa y resumida los indicadores globales filtrados para el país seleccionado —una con la Suma de `retail_price` (57,52 mil) y otra con la Suma de `quantity` (3 mil)—, permitiendo visualizar de un vistazo las métricas principales de valor y volumen. Las acompañan:
- Una **Tabla** detallada con los campos `country_name` y `category`, junto con los porcentajes del total (%TG Suma de `retail_price` y %TG Suma de `quantity`), para consultar los valores exactos de participación.
- Un **gráfico de barras apiladas (100%)** configurado con `country_name` en el Eje Y, el porcentaje del precio de venta (%TG Suma de `retail_price`) en el Eje X y `category` como Leyenda, que visualiza cómo se compone porcentualmente cada categoría dentro del total de ventas de cada país.

**Definiciones y justificación de métricas:**
- **`retail_price`** (precio de venta al público): precio final por el que se vende cada artículo al cliente. La Suma de `retail_price` representa el valor total de los ingresos generados por las ventas (cuánto dinero ha entrado por la venta de esos productos).
- **`quantity`** (cantidad): número de unidades físicas de producto vendidas. La Suma de `quantity` indica el volumen total de artículos entregados o despachados.
- **Por qué suma y no promedio:** usar promedios en métricas de costes o ingresos genera una pérdida de magnitud total. Oculta el volumen real de dinero que se mueve en el negocio y distorsiona por completo la realidad financiera. Para cualquier análisis de gestión o control, lo que manda es el acumulado total (suma).

### 4.2 Dashboard: Coste de los Bienes Vendidos (COGS)
Se copia el Dashboard 1 y, en lugar de `retail_price`, se usan `raw_material`, `factory_labor` y `factory_equipment_rent`. Se excluyen los costes de:
- **Distribution** (entrega o distribución de mercancías)
- **Advertisement** (marketing o publicidad)

Se representa como el % del Total.

**Definición:** COGS (*Cost of Goods Sold*) representa la suma de los costes de fabricación o adquisición de los artículos vendidos. No incluye los costes de distribución ni de marketing, ya que esos no forman parte del coste de producir/adquirir el producto en sí.

### 4.3 Dashboard: Beneficios
Como ya se tienen los ingresos (Revenue) y el coste de los bienes vendidos (COGS), se crea una nueva medida con una fórmula sencilla:

```
Beneficio Total = [Total Ingresos] - [Total COGS]
```

Para organizar las medidas, se crea la carpeta `_Calculations` (vista modelo, nueva tabla):
```
_Calculations = ROW("Column", BLANK())
```

Dentro de ella se crean las siguientes medidas:

```
Total Ingresos = SUM(Stock[retail_price])

Total COGS = SUM(Costs[factory_equipment_rent]) + SUM(Costs[factory_labor]) + SUM(Costs[raw_material])

Beneficio Total = [Total Ingresos] - [Total COGS]
```

### 4.4 Dashboard: Ventas por año
La información de ventas de todos los años está junta, pero se necesita comparar los años por separado. Para poder calcular el stock actual, primero hay que calcular la cantidad vendida en 2021 y 2022 para cada artículo.

En vista de modelo, se crea una tabla `Calendar`:

```
Calendar = 
CALENDAR(
    MIN(Orders[InvoiceDate]),
    MAX(Orders[InvoiceDate])
)
```

Con columnas asociadas al tiempo:

```
Year = YEAR(Calendar[Date])
Month = MONTH(Calendar[Date])
Day = DAY(Calendar[Date])
```

Después, se administra la relación: `Calendar (1) → Orders (muchos)`.

Finalmente, se filtran los dashboards anteriores por el año 2021 para comparar el rendimiento año a año.

---

## 5. Conclusiones

Para el mercado de **España** durante **2021**, los dashboards muestran lo siguiente:

- **Ingresos totales:** 57,52 mil
- **COGS total:** 6,73 mil
- **Beneficio total:** 50,79 mil

* Ver los números globales (Ingresos de 57,52 mil, COGS de 6,73 mil y un Beneficio de 50,79 mil) solo te dice que el negocio está operando con números positivos, pero es una foto demasiado general que no te dice dónde está el problema ni dónde se debe actuar. 

El verdadero valor analítico está en el desglose por categorías:

Categorías Altamente Rentables (Eficientes)

Aquí el coste porcentual es inferior a su aportación al beneficio. Tienen márgenes muy sanos.

- Decoration (37,21% beneficio): No solo es la que más volumen mueve, sino que sus costes están controlados. Recomendación: Asegurar siempre el stock aquí (evitar quiebres a toda costa), ya que es el motor económico de la región.

- Jewelry (16,80% beneficio): Tiene un margen muy sólido. Aporta una buena parte del beneficio global sin disparar los gastos operativos o de fabricación.

- Office & school (5,47% beneficio): Es la "joya oculta". Aunque su volumen de ventas es el más pequeño, su coste es mínimo. Recomendación: Se podría sugerir al equipo de marketing lanzar promociones para esta categoría; al tener tan buen margen, cualquier aumento en ventas impactará positivamente en la caja.

Categorías Críticas o Ineficientes (Márgenes Ajustados)
Aquí el peso porcentual de los costes supera a lo que la categoría aporta al beneficio total.

- Home accessories (29,77% beneficio): Genera mucho dinero, pero cuesta demasiado. Su peso en la estructura de costes de la empresa es desproporcionado.

- Toys & edibles (10,75% beneficio): Presenta el mismo problema de ineficiencia. Su contribución a los costes totales supera a su contribución al beneficio.

🎯 Conclusiones para WarmeHands Logistics:

El problema de altos gastos y posible mala gestión no es generalizado, sino que está focalizado en las categorías "Home accessories" y "Toys & edibles".

Próximos pasos recomendados para la empresa: 

Reducción de costes de producción: Optimizar de forma directa los componentes que integran el COGS (Coste de los Bienes Vendidos), enfocándose específicamente en:

raw_material (Materia prima): Renegociar contratos con los proveedores actuales de estas dos categorías o buscar alternativas en el mercado que ofrezcan materias primas más económicas sin sacrificar la calidad.

factory_labor (Mano de obra de fábrica): Analizar la eficiencia operativa y los tiempos de producción para reducir costes de personal asociados a la fabricación de estos artículos.

factory_equipment_rent (Alquiler de equipos de fábrica): Revisar la optimización y el rendimiento del uso de la maquinaria destinada a estas líneas de producto para evitar subutilizaciones que encarecen el proceso.