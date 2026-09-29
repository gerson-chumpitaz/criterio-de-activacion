# Criterio de Activación

Un principio de gestión de decisiones para escalar entre una representación operativa de bajo costo y una representación de verificación de alta fidelidad, activado únicamente por señales de riesgo explícitas y auditables.

## Resumen

Cuando existen dos representaciones posibles de una misma fuente de información (una barata y rápida, otra costosa y de mayor fidelidad), la pregunta operativa no es cuál usar, sino cuándo pasar de una a la otra. El Criterio de Activación resuelve esa pregunta con una regla de tránsito explícita: usar siempre la representación operativa por defecto, y consultar la representación de verificación solo cuando se detecta una señal de riesgo concreta y predefinida.

Sin esta regla, la alternativa habitual es una de dos posturas igual de costosas: verificar todo de forma sistemática, lo que anula la ganancia de eficiencia de tener una representación operativa, o confiar ciegamente sin verificar nunca, lo que expone a errores silenciosos que pueden alterar una conclusión. El Criterio de Activación es la vía intermedia entre ambas.

## Los tres componentes

**1. Representación operativa**

La fuente de bajo costo y uso por defecto. Se optimiza para velocidad de procesamiento y eficiencia de recursos (en el caso de un modelo de lenguaje, consumo de tokens). Se asume que es suficiente en la mayoría de los casos, pero no infalible.

**2. Señales de riesgo**

El componente central del framework. Son condiciones observables y verificables en la representación operativa que indican una posible pérdida de fidelidad respecto a la fuente original. Deben ser explícitas y predefinidas, nunca intuitivas, para que el criterio sea auditable y reproducible por terceros.

**3. Verificación selectiva**

La consulta puntual a la representación de alta fidelidad, activada exclusivamente cuando se dispara una señal de riesgo, nunca de forma sistemática ni exhaustiva. Cada activación debe quedar registrada: qué señal la disparó, qué elemento se verificó y cuál fue el resultado (confirmación o corrección de la representación operativa). Este registro es lo que distingue un criterio trazable de una práctica ad hoc.

## La tensión que resuelve

El framework opera sobre tres variables que compiten entre sí: fidelidad de la información, costo de procesamiento y tiempo de ejecución.

- Usar solo la representación operativa maximiza ahorro de costo y tiempo, pero expone a pérdida de fidelidad en los casos donde la representación de bajo costo falla.
- Usar solo la representación de verificación maximiza fidelidad, pero elimina la ganancia de eficiencia que motivó tener una representación operativa en primer lugar.
- El Criterio de Activación concentra el costo de verificación únicamente en los casos de riesgo real, preservando la mayor parte del ahorro sin sacrificar fidelidad en los puntos críticos.

## Caso de aplicación

Este repositorio documenta un caso de aplicación desarrollado y usado de forma repetida: extracción de información desde documentos PDF académicos, usando Markdown como representación operativa y el PDF original como representación de verificación.

Ver [`case-study-pdf-markdown.md`](./case-study-pdf-markdown.md) para la aplicación completa, incluyendo las señales de riesgo específicas y el formato de registro de activación.

## Alcance y generalización

La evidencia de aplicación documentada en este repositorio es específica al caso de documentos PDF y Markdown. La estructura de tres componentes (representación operativa, señales de riesgo explícitas, verificación selectiva registrada) es conceptualmente análoga a criterios de validación de delegación en sistemas de agentes, donde una acción de bajo riesgo se ejecuta de forma autónoma y una acción de alto riesgo requiere confirmación explícita.

Esta analogía se presenta como ilustración conceptual del alcance potencial del principio, no como evidencia de aplicación verificada en ese dominio. Extender el framework a otros contextos de decisión con umbral de riesgo es un área abierta, no una afirmación de este repositorio.

## Autoría

Formalizado por Gerson Chumpitaz, estudiante de Administración y Marketing en Universidad ESAN (Lima, Perú), a partir de una práctica operativa desarrollada y aplicada de forma reiterada en flujos de investigación académica con documentos PDF.

Fecha de formalización: septiembre de 2026.

## Licencia

MIT. Ver [`LICENSE`](./LICENSE).
