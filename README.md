# Análisis de Inventario — Mercado España 2021

**WarmeHands Logistics Inc.** — Informe en Power BI (`inventario.pbix`)

## Contexto del proyecto
WarmeHands Logistics contrató un equipo de analistas de datos para hacer un análisis de inventario que ayude a optimizar el control de stock y reducir gastos por mala gestión. El equipo de datos se organiza por país: cada analista es responsable de dar seguimiento a un mercado específico, aplicando sobre él las reglas de negocio definidas de forma centralizada (cálculo de Ingresos, COGS, Beneficio).

## Objetivo del proyecto
En este proyecto se asume el rol de analista responsable del mercado de **España** durante el año **2021**, con el objetivo de evaluar sus **ingresos, costes (COGS) y beneficio**, desglosados por categoría de producto, y aplicar las reglas de negocio establecidas por la organización para determinar si el mercado está cumpliendo los objetivos esperados. Se elige 2021 como periodo de referencia por ser el primer año completo con datos consistentes tras la limpieza.

## Preguntas de negocio a responder
- ¿Cuáles son los ingresos, el COGS y el beneficio generados por el mercado de España en 2021?
- ¿Cómo se distribuyen esos ingresos y costes entre las distintas categorías de producto?

---

## Rentabilidad por categoría

### 🟢 Categorías eficientes (coste % inferior a su aportación al beneficio)

- **Decoration — 37,21% del beneficio.** Mayor volumen de ventas y costes controlados. Es el motor económico del mercado; prioridad máxima evitar quiebres de stock.
- **Jewelry — 16,80% del beneficio.** Margen sólido, aporta beneficio sin disparar gastos operativos o de fabricación.
- **Office & school — 5,47% del beneficio.** Menor volumen pero coste mínimo ("joya oculta"): cualquier impulso en ventas se traduce directamente en beneficio.

### 🔴 Categorías ineficientes (peso de costes superior a su aportación al beneficio)

- **Home accessories — 29,77% del beneficio.** Genera mucho ingreso, pero su estructura de costes es desproporcionada.
- **Toys & edibles — 10,75% del beneficio.** Mismo patrón: su contribución a los costes totales supera su contribución al beneficio.

## Conclusión

El problema de altos gastos no afecta a todo el catálogo, sino que se concentra en **Home accessories** y **Toys & edibles**.

## Recomendaciones

**Sobre stock:**
- Asegurar la disponibilidad de *Decoration* en todo momento; es la categoría que más beneficio aporta.
- Evaluar promociones en *Office & school*: dado sus bajos costes y altos beneficios, un aumento de ventas impactaría de forma muy positiva.

**Sobre reducción de costes (enfocado en Home accessories y Toys & edibles):**
- **Materia prima:** renegociar con proveedores actuales o buscar alternativas más económicas sin sacrificar calidad.
- **Mano de obra de fábrica:** revisar eficiencia operativa y tiempos de producción.
- **Alquiler de equipos:** auditar el uso de maquinaria para detectar subutilización que encarezca el proceso.
