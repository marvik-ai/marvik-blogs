# Análisis del blog de Marvik (junio 2025 – septiembre 2026)

Fuente: https://www.marvik.ai/blog (páginas 1 a 4 del listado). Relevado el 2026-10-22.

## 1. Corpus

40 posts entre el 12/06/2025 y el 23/09/2026 (no hay posts de octubre 2026 todavía).

| Categoría | Posts | Qué son |
|---|---|---|
| Technical | 22 | Exploraciones de herramientas o modelos (Claude Managed Agents, Genie vs Mosaic, S3 Tables, MCP, TTS, VLA, BioNeMo), lecciones de proyectos (Simplicity vs Control, Spec-Driven Development) y algún experimento propio (Prompt Hacking with Agents) |
| Business Impact | 9 | **Dos subtipos**: 3 casos de cliente (2026) y 6 piezas de thought leadership (2025 – ene 2026) |
| News | 5 | Partnerships, eventos (OpenAI Select Partner, GTC 2026, SF Tech Week) |
| People & Culture | 4 | Trayectorias de carrera |

**Business Impact, completo:**

| Fecha | Título | Subtipo |
|---|---|---|
| 23/09/2026 | Making healthcare billing automation more reliable with Claude (Prestige) | Caso de cliente |
| 28/08/2026 | AI-powered code review in a digital pathology stack (Zellblick) | Caso de cliente |
| 23/06/2026 | From Dashboards to Dialogue: Nivii's Text-to-SQL layer with Anthropic | Caso de cliente |
| 30/01/2026 | CAIO: The Executive Role That Turns AI Into Strategic Advantage | Thought leadership |
| 26/12/2025 | AI Consulting Services: How Businesses Are Turning Data Into Decisions | Thought leadership (no leído en detalle) |
| 26/12/2025 | The AI Stack Is Shifting and Vertical Integration Is Becoming a Real Advantage | Thought leadership |
| 26/11/2025 | AI in Biotechnology: Accelerating Drug Discovery and Genomics | Thought leadership (no leído en detalle) |
| 13/11/2025 | ChatGPT vs. Custom AI Models | Thought leadership |
| 19/09/2025 | AI Shopping Agents: The New Front Door to Commerce | Thought leadership |

Leídos en detalle: los 3 casos de cliente, 4 de thought leadership y 5 Technical (Genie vs Mosaic, Claude Managed Agents, Simplicity vs Control, Prompt Hacking with Agents, Spec-Driven Development).

**Tendencia:** desde junio 2026 Business Impact son solo casos de cliente, y los tres van atados a un partner (Claude/Anthropic, AWS Bedrock). En agosto 2026 Marvik pasó a ser **OpenAI Select Partner**: un caso hecho con OpenAI Decisions API calza con esa línea.

## 2. Business Impact: el caso de cliente (el formato que nos sirve)

### Estructura (casi idéntica en los tres)

1. **Dek de una línea** debajo del título: "How Marvik helped X move from A to B".
2. **H2 de contexto de industria**, sin cliente todavía ("Healthcare billing needs more than fragile automation", "When AI accelerates development, review has to keep up", "Business Intelligence Is Moving Beyond Static Dashboards").
3. **"The challenge: …"**: el problema del cliente, con 3–4 puntos en negrita y los requisitos (confiable, monitoreable, seguro).
4. **"Why <vendor> / Why X was a good fit"**: justificación de la tecnología. Siempre presentan el modelo como **un componente acotado dentro de una arquitectura controlada**, nunca como la solución entera.
5. **"How it works" / "The solution"**: 3 caminos o 3 capacidades, en lista numerada.
6. **"How it was built" / "From setup to production in three weeks"**: fases (discovery, build, calibración).
7. **"Results" / "Where the system stands" + "Production Impact"**: viñetas con las métricas en **negrita**.
8. **Cierre que generaliza**: "What this project shows about…", "Looking ahead". Termina con una frase en negrita tipo lema ("Automation needs to move work faster, but it also needs to be reliable, explainable…").

### Rasgos

- **Largo:** 1.000 – 1.500 palabras.
- **Firma:** "Marvik", sin autor individual. Tercera persona ("Marvik", "Prestige") mezclada con "we" en el de Zellblick.
- **Profundidad:** media. Nombran modelos y features (Haiku/Sonnet, reasoning effort, computer use, Bedrock) pero sin código, prompts ni arquitectura detallada.
- **Visuales:** ninguno en el cuerpo. Sin tablas, sin código, sin diagramas. Solo la imagen de cabecera.
- **Métricas:** pocas y titulares ("reduced human review effort by 20%", "96.8% (1,227 of 1,268)", "failure rate of just 1.85%"). Concretas, con el n entre paréntesis cuando lo tienen.
- **Validación:** el punto débil de los tres. Ninguno reporta precisión, recall, línea base ni test ciego; Nivii dice que la exactitud es crítica y nunca la mide. **Acá podemos diferenciarnos.**
- **Testimonio:** a veces un blockquote del cliente (Prestige, Zellblick). Nivii no tiene.
- **CTA:** ninguno en el cuerpo; el "Let's Talk" es del footer.
- **Tono:** medido, sin hype. Palabras recurrentes: reliable, control, oversight, production, operations.

## 3. Business Impact: thought leadership (para contraste)

- 550 – 950 palabras, con autor (CEO, VP, Tech Manager, Data Scientist).
- Tesis + estadísticas de terceros con link (McKinsey, HBR, Salesforce, Adobe) + listas + cierre con pitch de Marvik ("If your company is looking to unlock the potential of AI, Marvik is ready to help").
- Sin datos propios. No es el formato para un resultado de PoC.

## 4. Technical (por si se hace un post complementario)

- 1.300 – 2.200 palabras, **con autores y rol** (2 a 4 personas, ML Engineers / Tech Leads).
- "We" en primera plural (Spec-Driven Development usa "I").
- Más visuales: 2 a 7 screenshots o diagramas, a veces un bloque de código; casi nunca tablas.
- Secciones de "lessons" numeradas (Genie: "6 Lessons…"), "Trade-offs and limitations", "Future work".
- El más cercano a nuestro contenido es **Prompt Hacking with Agents**: experimento propio con n=1.000, % de resultados, costo por intento, mediana y media de latencia, limitaciones explícitas. También **Simplicity vs Control**: casos con fechas, costos en USD y "Our Take".
- Cierres con frase corta y memorable ("Both tools worked. Neither one did the hard part for us." / "Design for the solution you need now. … It will.").

## 5. Guía de estilo para nuestro post

- Formato **caso de cliente de Business Impact**: dek → contexto de industria → challenge → por qué este enfoque → cómo funciona → cómo se validó → resultados → qué muestra → próximos pasos.
- 1.300 – 1.600 palabras, en inglés.
- Métricas titulares en negrita, siempre con el n y la condición ("at 95% recall, on 10,000 unseen pairs").
- El modelo como componente acotado: el filtro no decide merges, solo decide qué ve el agente caro.
- Una sección de validación honesta, corta y legible: es lo que los otros casos no tienen.
- Máximo 2 figuras (cascada y recall vs especificidad) y, como mucho, una tabla chica. Sin código.
- Cerrar con una frase-lema en negrita.
