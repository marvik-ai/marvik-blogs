# Draft de contenido: post de Business Impact sobre el filtro antes de DPV

**Estado:** borrador para revisar (2026-10-22). El post en sí está en `03-blog-draft.md`.

## 1. Qué es el post

- **Categoría:** Business Impact, subtipo caso de cliente (formato Prestige / Zellblick / Nivii; ver `01-analisis-blogs-marvik.md`).
- **Idioma:** inglés, como todo el blog.
- **Audiencia:** líderes de producto, catálogo y data en marketplaces y retail; tech leads que pagan una factura de LLM por decisión.
- **Largo objetivo:** ~1.500 palabras.
- **Mensaje en una frase:** no reemplaces al juez caro, ponele una puerta adelante. Un modelo de decisión barato, con pocas preguntas tipadas y la foto, descarta 9 de cada 10 pares que no son duplicados, deja pasar el 95% de los duplicados y baja el costo y el tiempo de la detección un 86%.
- **Diferencial frente a los otros casos del blog:** validación ciega a prevalencia real, con umbrales fijados antes de mirar el test. Ninguno de los tres casos publicados mide su sistema así.

## 2. De dónde sale cada parte

| Parte del post | Fuente |
|---|---|
| Problema, objetivo y flujo CAFU → gate → DPV | `reports/poc-dpv-eval-10k-2026-10-22/reporte-interactivo-v3.html` (secciones Impacto y Qué hicimos) |
| Por qué no reemplazar a DPV | `reports/poc-jev-dpv-2026-09-30-v1.html` §0 (Jev 5 puntos de F1 debajo de E18; como prefiltro descarta 71–79% con recall 96–98%) y `reports/poc-dpv-decisions-2026-10-20/README.md` (E18 con `gpt-6-luna`: recall 40% golden / 57% CAFU) |
| Tipos de filtro y preguntas | `reports/poc-dpv-feature-discovery-2026-10-21/README.md` y el anexo de `reports/poc-dpv-eval-10k-2026-10-22/README.md` |
| Protocolo y resultados | `reports/poc-dpv-eval-10k-2026-10-22/README.md` §1–§8 |
| Ruido de etiquetas | `reports/poc-dpv-decisions-2026-10-20/README.md` (Sonnet le da la razón a Decisions en 40 de 63 duplicados rechazados, 8% de los positivos; pendiente de validar a mano) |
| Escala trimestral | `scripts/dpv_eval_10k/page/build_v2.py` (`QUARTER = 8_600_000`, el volumen de las PoC anteriores) |

## 3. Los objetivos de la PoC y cómo quedaron

| Objetivo (cuándo se fijó) | Resultado |
|---|---|
| ¿Puede un modelo barato reemplazar a DPV? (30/9) | No. Jev quedó 5 puntos de F1 debajo de E18 en test; E18 con un modelo barato se vuelve hiperconservador (recall 40–57%). |
| ¿Sirve como filtro antes de DPV? (30/9) | Sí en golden (descarta 71–79% con recall 96–98%), y ahora confirmado a escala real. |
| Recall ≥ 95% de duplicados que llegan a DPV, y la máxima especificidad posible (20/10) | En el 10k ciego, el banco de 4 preguntas con foto y logística llega a **95,0% de recall y 90,3% de especificidad**. |
| Superar al filtro anterior (pregunta única D4-ids): igual especificidad a menor costo, o más especificidad al mismo costo (plan del 20/10) | La pregunta única con foto no sostiene el 95% en el 10k (93,7%). El banco de 4 preguntas sí, con +0,9 puntos de especificidad y USD 0,03 más cada 1.000 pares. |
| Medir costo y tiempo de toda la cadena (21–22/10) | Medido con el prompt de DPV (E18) y `gpt-5`: USD 9,75 y 14,9 s por par. La cascada ahorra 86% en costo y en tiempo. |
| Pendientes | Validar etiquetas de CAFU, probar los bancos con Jev y foto, corrida en sombra, evaluación unificada con IC (plan del 21/10, diferido). |

## 4. Estructura del post (sección por sección)

**Título (opciones):**
1. *Put a gate before the judge: cutting LLM duplicate-detection costs by 86% in a marketplace catalog*
2. *Ask cheaper questions first: how a decision filter cut duplicate-detection costs by 86% without losing duplicates*
3. *Fewer calls, same duplicates: a cheap decision gate in front of an LLM catalog agent*

**Dek:** "How Marvik helped [cliente]'s catalog team put a cheap decision filter in front of its duplicate-detection agent, keeping 95% of true duplicates while sending only 1 in 8 candidate pairs to the expensive model."

1. **H2 contexto: "Catalog quality is a pairs problem"**. Por qué importan los duplicados en un marketplace (reseñas, historial de ventas, búsqueda partidos). El flujo: un generador de candidatos propone pares, un agente LLM juzga cada par con título, atributos y fotos.
2. **H2 "The challenge: an accurate judge that spends most of its time saying no"**.
   - 97 de cada 100 pares candidatos no son duplicados.
   - El agente razona: ~15 s y ~USD 0,01 por par; el 77% del costo son tokens de salida, casi todos de razonamiento.
   - A ~8,6 M de pares por trimestre: ~USD 84k y ~36k horas de cómputo.
   - La restricción: perder un duplicado es peor que dejar pasar un falso positivo; el agente no puede mergear lo que no ve. Meta: ≥ 95%.
   - Lo que no funcionó: reemplazar al agente por un modelo barato.
3. **H2 "The approach: typed questions, probabilities and a threshold"**.
   - Modelos de decisión (OpenAI Decisions API; Jev): opciones cerradas, una probabilidad por opción, la salida no se cobra. ~USD 0,0002 y 0,25 s por par.
   - La probabilidad es un dial: el umbral se ajusta al recall que el negocio exige.
   - Tres familias de filtros: pregunta única; banco escrito a mano + decisor aprendido (logística o CatBoost); autoresearch (un LLM propone, prueba y poda preguntas sobre datos de desarrollo). Cada una con y sin foto.
   - La política del negocio va en las preguntas: GTIN distinto es ruido, GTIN compartido apoya pero se copia, marcas placeholder son neutras, "Preto = Negro = Black".
4. **H2 "How we validated it"**.
   - Dev: 2.000 pares etiquetados. Umbral fijado para recall 95% con validación cruzada agrupada por producto.
   - Test: 10.000 pares nuevos de una semana de tráfico real (13–20/10/2026), 300 duplicados (3%, la prevalencia real), 901 dominios, ningún producto en común con dev. Se corrió una vez, sin retocar nada.
   - El costo de DPV se midió en vez de estimarlo: el doble de la fórmula previa, por el razonamiento.
   - 8 de los 33 filtros que llegaban a 95% en dev quedaron debajo en el test.
5. **H2 "Results"** (viñetas con métricas en negrita + figura 2).
6. **H2 "What this project shows"**: 5 lecciones numeradas (ver post).
7. **H2 "Where it stands and what's next"**: PoC; corrida en sombra, revisión de los duplicados perdidos, Jev con foto, contraste con la factura real.
8. **Cierre en negrita.**

## 5. Figuras propuestas

1. **Cascada** (en "The approach"): pares candidatos → gate (0,25 s, USD 0,0002) → DPV (15 s, USD 0,01) → merge. Con los volúmenes: de 1.000 pares candidatos (30 duplicados), 123 pasan al agente, y entre ellos el 95% de los duplicados. Se puede sacar del diagrama del reporte v3.
2. **Recall vs especificidad de los 33 filtros** (en "Results"): el scatter "Duplicados que llegan contra no duplicados descartados" del reporte v3, con la banda de 95% marcada.

Los casos de cliente del blog no usan figuras en el cuerpo; dos es el máximo razonable.

## 6. Números que usa el post (todos del 10k, umbral de dev para recall 95%, DPV con `gpt-5`)

| Número | Valor | Fuente |
|---|---|---|
| Filtro elegido | `seed-img · logistic`: 4 preguntas, foto 512 px, regresión logística | README 10k §5 fila 3; es el "más barato que sostiene 95%" del reporte v3 |
| Recall / especificidad | 95,0% (285 de 300) / 90,3% | README 10k §5 |
| Pasa a DPV | 12,3% | README 10k §5 |
| AUC | 0,978 | README 10k §5 |
| Costo filtro | USD 0,181 cada 1.000 | README 10k §5 |
| Cascada vs DPV solo | USD 1,38 vs 9,76 cada 1.000 (−85,9%) | README 10k §8 |
| Tiempo | 35 vs 249 min cada 1.000 (−86%) | README 10k §8 |
| Trimestre (8,6 M pares) | ~USD 11,8k vs ~83,9k; ~5,0k vs ~35,6k h; ~7,5 M llamadas menos | cálculo propio con los dos anteriores |
| Más margen | `qlog-img · logistic` con umbral de 97%: 97,3% recall, 87,9% espec., 14,7% pasa | README 10k §6 |
| Foto | ~8 puntos más de especificidad que D3 al mismo recall; 512 px mejor que completa con la mitad de tokens | README 10k §1 y §5 |
| Bancos chicos vs grandes | 4 preguntas: 90,3% espec.; 12–15 chequeos CQI: 81,8–86,5% | README 10k §5 |
| DPV con modelo sin razonamiento | USD 0,46 cada 1.000; el ahorro baja a 45–57% | README 10k §8 |
| Costo de la corrida | ~USD 37 y ~55 min para 33 filtros × 10.000 pares | README 10k §1 |

## 7. Decisiones que necesito de vos

1. **¿Se nombra al cliente?** Los casos del blog nombran al cliente (con su aprobación). El draft dice "Mercado Libre" entre corchetes. Publicar costos, volúmenes y nombres internos de MELI hace falta aprobarlo con ellos. Alternativa: "one of Latin America's largest marketplaces".
2. **Nombres internos (CAFU, DPV, Jev, E18).** El draft usa nombres genéricos: *candidate generator*, *duplicate-detection agent*. Jev aparece una vez, como alternativa más barata.
3. **Jev vs Decisions API.** Todos los resultados del 10k son con OpenAI Decisions API (`gpt-6-luna`), no con Jev. El draft lo dice así. Si se quiere contar como "filtros Jev", hace falta la corrida con Jev y foto. Como Marvik es OpenAI Select Partner desde agosto, contarlo con Decisions API calza con la línea editorial.
4. **Nombres de modelos y precios.** `gpt-5` y `gpt-6-luna` con precios por millón: ¿los dejamos o lo decimos como "a reasoning model" / "a small decision model"?
5. **Firma.** Los casos de cliente van firmados "Marvik". ¿Lo dejamos así o con autores, como los Technical?
6. **Proyección trimestral.** Los 8,6 M pares por trimestre son el volumen de las PoC anteriores, no un dato de este export. Confirmar antes de publicar los USD 84k.
7. **¿Testimonio?** Los casos de Prestige y Zellblick tienen una cita del cliente. Si alguien del equipo de catálogo quiere dar una, va en "What this project shows".
8. **¿Post Technical aparte?** Hay material para un segundo post Technical (el loop de autoresearch de preguntas, CatBoost vs logística, el formato CQI, por qué los bancos chicos ganan). Lo dejaría para después de publicar este.

## 8. Lo que el post no puede decir

- Que está en producción: es una PoC sobre una muestra de 10.000 pares.
- Que pierde "solo" 15 duplicados sin aclarar que las etiquetas de CAFU no se revisaron (en rondas anteriores parte de los duplicados que todos los filtros rechazaban eran errores de etiqueta).
- Un costo real de DPV: se midió con su prompt (E18) sobre 20 pares, no con la factura del agente de producción.
- Intervalos de confianza: no se calcularon en el 10k (por pedido). Con 300 duplicados, cada uno vale 0,33 puntos de recall; el IC es de alrededor de ±2,5 puntos.
