# Caso de aplicación: extracción de evidencia desde PDF académicos

Este documento describe la aplicación del Criterio de Activación (ver [`README.md`](./README.md)) a un flujo de trabajo real y usado de forma repetida: lectura y extracción de información desde artículos académicos en PDF para revisión de literatura e investigación de mercados.

## Contexto del flujo

El insumo son PDFs de artículos académicos, con dos columnas, tablas, figuras, ecuaciones y anexos, algunos escaneados. El objetivo es extraer título, autores, diseño metodológico, constructos, resultados numéricos e interpretación de figuras o modelos conceptuales, sin perder fidelidad respecto al documento original y sin incurrir en el costo de procesar cada página como imagen.

## Componente 1: representación operativa

El PDF se convierte a Markdown mediante un pipeline propio que combina tres herramientas según el tipo de contenido de cada documento:

- **pymupdf4llm**, para extracción de texto estructurado y jerarquía de encabezados.
- **pdfplumber**, para extracción de tablas con preservación de estructura de filas y columnas.
- **pytesseract**, para reconocimiento óptico de caracteres en páginas o secciones escaneadas donde no hay capa de texto extraíble.

El Markdown resultante es la fuente principal de lectura para el modelo. Reduce el consumo de tokens frente a enviar el PDF completo como imagen en cada página, y permite trabajar con la jerarquía del documento (títulos, secciones, tablas) en lugar de texto plano sin estructura.

## Componente 2: señales de riesgo

Estas son las condiciones observables en el Markdown que activan la verificación contra el PDF original. Cada señal está definida de forma explícita, no como criterio subjetivo:

| Señal | Descripción | Ejemplo de disparo |
|---|---|---|
| Referencia no traducida | El Markdown menciona un elemento visual que no tiene contenido correspondiente extraído | Aparece "Figure 3" o "Table 2" sin datos, texto o valores asociados |
| Marcador de truncamiento u OCR | Presencia de marcadores de error de conversión | `[image]`, `[illegible]`, elipsis de corte, caracteres no reconocidos |
| Estructura tabular dañada | La tabla extraída presenta inconsistencias estructurales | Celdas vacías sospechosas, número de columnas distinto entre filas, separadores rotos |
| Fórmula o símbolo parcial | Notación estadística o matemática reconocida de forma incompleta | Símbolos griegos faltantes, superíndices perdidos, subíndices desalineados |
| Mezcla de orden de lectura | El texto extraído combina contenido de columnas distintas de forma incoherente | Una oración interrumpida por contenido que pertenece a la columna adyacente |
| Discrepancia narrativa-tabular | El texto describe un resultado que no coincide con el valor correspondiente en la tabla extraída | El cuerpo del artículo menciona un coeficiente que no aparece igual en la tabla de resultados |

## Componente 3: verificación selectiva y registro

Cuando una señal se dispara, se consulta la página correspondiente del PDF original, no el documento completo, y se corrige o confirma el dato en el Markdown. Cada activación se registra con la siguiente estructura:

```json
{
  "documento_id": "paper-ieee-anonimizado-01",
  "pagina": 4,
  "señal_disparada": "estructura_tabular_dañada",
  "elemento_verificado": "TABLE I, tabla comparativa con bordes en layout de doble columna",
  "resultado": "markdown_corregido_parcial"
}
```

Este caso es real: en un paper con formato IEEE de doble columna, la tabla principal del documento (identificada aquí como "TABLE I") es, en el PDF original, una figura compuesta con texto superpuesto sobre un diagrama, no una tabla con estructura subyacente reconocible por el extractor. Al verificar contra el PDF, se confirmó que el fragmento en el Markdown correspondía efectivamente a esa tabla, pero que su reconstrucción perfecta no era posible sin visión por LLM, dado que no existe una estructura de celdas real que extraer, solo coordenadas de texto en el espacio. El resultado de esa verificación se registró como corrección parcial, no como corrección completa, lo cual es en sí mismo información útil: indica que ese tipo de tabla requiere una ruta de procesamiento distinta (visión por LLM sobre la página específica) en lugar de una corrección manual del Markdown.

Un segundo caso, relacionado pero inverso, ocurrió en un documento técnico donde un bloque de código con bordes visuales fue identificado por el extractor de tablas como una tabla real, un falso positivo. La verificación contra el PDF confirmó que no era una estructura tabular sino un bloque de código, y la corrección aplicada fue tratarlo como texto plano en lugar de forzar una conversión a tabla Markdown.

Este registro cumple dos funciones. Primero, deja trazabilidad de qué partes del documento requirieron verificación manual y por qué, útil si otra persona necesita auditar el proceso de extracción. Segundo, permite identificar con el tiempo qué tipo de documento o qué señal se dispara con más frecuencia, lo que en este caso llevó a una decisión concreta de arquitectura: añadir una heurística de densidad de celdas vacías para filtrar falsos positivos de tabla antes de que lleguen al Markdown final.

## Resultado observado

Con este flujo, la mayoría de las páginas de un documento se procesan exclusivamente en Markdown, sin consulta al PDF, y solo una fracción menor de páginas (las que contienen las señales listadas arriba) requieren verificación visual. Esto reduce el volumen de contenido que necesita procesarse como imagen frente a un enfoque que trata cada página del PDF como imagen de forma sistemática, sin perder la fidelidad en los puntos donde la conversión a Markdown es más propensa a fallar: tablas complejas, figuras y notación estadística.

## Limitaciones reconocidas

Este caso de aplicación no cuenta con una medición formal de precisión, recall o tasa de alucinación comparada contra una verdad de referencia construida por revisores ciegos. Las señales de riesgo se definieron y refinaron de forma iterativa a partir de uso repetido, no de un diseño experimental controlado. Este documento describe una práctica operativa validada por uso reiterado, no un estudio empírico con métricas formales. Una evaluación rigurosa del framework, con corpus de prueba y ground truth manual, queda como trabajo futuro.
