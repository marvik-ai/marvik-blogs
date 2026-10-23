# Material para el blog: filtros de decisión en Facebook Marketplace e Instagram (WBD)

**Qué es:** el material de base para que un agente (o una persona) redacte el blog sobre
modelos de decisión cuando la mayoría de las respuestas son NO, sumando los dos casos de
Warner Bros. Discovery (WBD) al caso de deduplicación de catálogo de `03-blog-draft.md`.
**No es el blog** ni lo modifica. Reúne el caso de uso, los números con su fuente, lo que se
puede y no se puede afirmar, y las decisiones que quedan abiertas.
**Relevado:** 2026-10-23, rama `WBD-200-jev-multimodal`.
**Leer antes:** `01-analisis-blogs-marvik.md` (formato y tono del blog),
`02-draft-contenido.md` (estructura y números del caso de deduplicación),
`03-blog-draft.md` (el post de ese caso).

> **Ojo con la palabra "marketplace".** En el caso de `03-blog-draft.md` el marketplace es el
> **cliente** (un marketplace de e-commerce con productos duplicados). En el caso de WBD,
> **Facebook Marketplace es una superficie donde se publican avisos piratas**, y el cliente es
> un grupo de medios. Son dos casos distintos. El redactor no debe mezclarlos. El scout de
> **MercadoLibre** de WBD existe, pero su filtro es un esqueleto que no está conectado (11 casos
> en la base al 9/10), así que no entra en el blog.

---

## 1. La tesis en una línea, y cómo encajan los tres casos

**Cuando la mayor parte de las decisiones son NO, no hace falta que el modelo caro las tome
todas. Un modelo de decisión barato, con preguntas cerradas que devuelven una probabilidad,
descarta los NO claros y deja que el paso caro (un LLM que razona, una captura de pantalla, un
analista humano) se ocupe solo de lo que sí necesita juicio.**

| | Deduplicación de catálogo (`03-blog-draft.md`) | WBD · Facebook Marketplace | WBD · Instagram |
|---|---|---|---|
| Qué se decide | ¿Estos dos productos son el mismo? | ¿Este aviso vende acceso pirata a TV o fútbol? | ¿Este post ofrece acceso pirata a TV o fútbol? |
| Cuántas respuestas son NO | 97 de cada 100 pares | 88 de cada 100 avisos abiertos en el run en vivo (20/10); 89% de los avisos que descartaba la regex (spike, 288 avisos) | 80% de los casos de Instagram que llegaron a la base en un mes (7.329, 10/9–8/10) los descartó después el analizador |
| Qué había antes | Un agente LLM que razona juzga cada par | Una regex sobre el título elige qué avisos abrir; otra regex sobre la descripción decide | Un LLM generativo con la captura (`validate_finding`) dice sí o no |
| Qué es "lo caro" que se protege | El agente que razona (~15 s y ~USD 0,01 por par) | Abrir el aviso, sacar la captura y mandarla al modelo; el analizador LLM; el tiempo de los analistas | El analizador LLM y los analistas, que recibían ruido |
| Rol del modelo de decisión | **Puerta delante** del juez caro; no lo reemplaza | **Reemplaza a la regex** y arma una **cascada texto → imagen** delante del analizador | **Reemplaza al gate LLM** con una sola pregunta y la imagen |
| Qué error duele más | Perder un duplicado | Perder un pirata (no hay caso, no hay denuncia) | Mandar ruido (la meta de negocio es precisión ≥ 80%) |

Los tres casos muestran **tres posiciones** del mismo componente: delante de un juez caro
(deduplicación), en lugar de reglas que no ven nada (Marketplace) y en lugar de un LLM más lento
y menos preciso (Instagram). En los tres, el umbral lo fija el negocio y la decisión final se
toma **en código**, no en el prompt.

---

## 2. El contexto de negocio de WBD (del documento de estrategia)

Fuente: `Warner Discovery - Anti Piracy Strategy - Three POCs.pdf`, junio 2026, v1.0. **Cada
página dice "Confidential information prepared for Internal use".** Cualquier número de esta
sección que vaya al blog necesita aprobación de WBD (ver §10).

### 2.1 El problema

| Dato | Valor | Página |
|---|---|---|
| Suscriptores de TNT Sports en Chile | de ~1 M a ~600 mil en cinco años (desde 2021); los ~400 mil perdidos, "most likely absorbed by piracy" | 3, 18 |
| Hogares con TV paga en Chile | de 3,5 M a 2,5 M, mientras la banda ancha creció a 4,8 M hogares | 3 |
| Velocidad de internet en Chile | 600–800 Mbps, "among the fastest in the world": los streams ilegales andan sin cortes | 3 |
| Tráfico de IPTV pirata | ~20% del tráfico de los ISP | 3, 5, 15 |
| Hogares chilenos con IPTV pirata | estimado en más de 1 M | 3, 15, 18 |
| Precio del IPTV pirata | ~1/3 de una suscripción legal, con el equivalente a 4 o 5 servicios y 1.000+ canales | 3, 15 |
| Web streams gratis | un solo dominio (futbollibretv.su) con hasta 8,8 M de visitas en abril 2026 | 3, 7 |
| Costo de montar un IPTV pirata | ~USD 2.000, lo que alimenta una cola larga de operadores chicos | 7 |
| Tiempo medio de un takedown DMCA | ~59 minutos: un tiempo entero de un partido de 90 | 3, 14 |
| Streams que siguen activos | ~60% siguen 60–120 minutos en vivo (proveedor Friend MTS) | 11 |
| Cumplimiento real de un proveedor | ~30% de las fuentes de video reales frente al ~81% que reporta (IP House) | 10 |
| Bloqueo dinámico de dominios en tiempo real | legalmente no disponible en Chile ni en Argentina | 3, 14 |

### 2.2 Por qué importa el IPTV (y por qué el filtro busca ofertas pagas)

- **Dos perfiles de usuario pirata (p. 15–16):** el de web streams gratis ("pirate by nature",
  difícil de convertir) y el de IPTV, que **ya paga** una suscripción y depende del servicio
  para todo (deportes, series, películas). WBD prioriza al segundo.
- **Evidencia de conversión (p. 7 y 19):** bloquear servicios IPTV en Argentina en febrero de
  2026 ("Operación Valoroso") generó **31.000 suscripciones incrementales a HBO Max** y un
  **aumento del 67% en altas brutas**.
- **El valor en juego (p. 18):** a ~USD 16 por suscripción por mes, recuperar el 10% de los
  hogares con IPTV pirata en Chile (100 mil) serían **USD 1,6 M de ingresos por mes**, solo en
  Chile.
- **Los puntos débiles del pirata (p. 14–15):** cobran por canales regulados (Mercado Pago,
  transferencias, tarjetas) y captan suscriptores por mensajería (WhatsApp, Telegram, DMs de
  Instagram) atendida por personas, con un techo de capacidad. **Las redes sociales y los
  marketplaces son la vidriera** donde se publican esas ofertas: por eso el Social Media
  Monitor busca ahí.

### 2.3 Cómo se trabajaba antes (el dolor operativo)

- El equipo de redes de TNT Sports Chile **monitoreaba a mano** los comentarios de las cuentas
  oficiales en días de partido, sacaba capturas a mano y las mandaba por **WhatsApp** al equipo
  de Anti-Piracy Management (p. 9, 13, 19).
- Ese equipo es de **dos personas** y consolida todo por mail a la casilla `protect@`, que no
  tiene tickets, números de caso ni estados (p. 9, 13).
- Juntar la evidencia de una fuente nueva (crear cuentas de prueba, navegar, capturar) se hace
  **a mano, caso por caso** (p. 13).
- Armar el panorama completo de una fuente pirata implica **hasta 4 mails** a distintas áreas y
  **un par de días** (p. 19).
- Caso testigo (p. 12): en mayo de 2026 el equipo de redes detectó revendedores que vendían por
  Instagram acceso a partidos exclusivos vía Discord. Las cuentas se dieron de baja en horas, y
  como pedían comprobante de transferencia, se obtuvieron nombre, DNI y datos fiscales para una
  denuncia penal. **Lo que se encontró a mano funcionó; el problema es la escala.**

### 2.4 Lo que se le pidió al Social Media Monitor (los KPIs comprometidos)

Del PoC "Social Media Monitor Agent" (p. 25–26):

| KPI | Meta |
|---|---|
| Aumento del volumen detectado | **≥ 5x** publicaciones piratas por semana frente al monitoreo manual (el resumen ejecutivo, p. 4, dice "≥ 3-5x") |
| **Precisión de detección** | **≥ 80%** de lo marcado confirmado como verdadero positivo, "to ensure enforcement resources are not wasted" |
| Completitud del paquete de evidencia | % de casos con evidencia lista para escalar sin enriquecerla a mano |
| Largo plazo | menos publicaciones piratas activas, menos tiempo hasta la baja, menos links piratas encontrables con una búsqueda simple |

**Este es el puente con el blog:** la meta de precisión ≥ 80% es justamente lo que obliga a
decir NO bien. Un scout que encuentra mucho pero manda ruido gasta el tiempo de un equipo de dos
personas y el de Legal.

---

## 3. El caso de uso: dónde están las decisiones

### 3.1 El sistema (lo mínimo que el redactor tiene que entender)

1. **Scouts** (uno por plataforma) manejan un navegador Chromium real, con una cuenta logueada,
   y recorren Facebook (Ad Library, grupos, Marketplace), X, Instagram y MercadoLibre buscando
   ofertas de IPTV pirata, con consultas como "tv digital", "pack futbol", "xuper", "flujo tv",
   en ciudades de Argentina y Chile (proxies LATAM).
2. Antes de que exista un caso, **el scout decide** si el candidato vale la pena. Si pasa, guarda
   la captura como evidencia y crea un caso `pending` en la API.
3. Un **analizador LLM** toma cada caso pendiente, lo lee con la captura y extrae los datos
   estructurados (tipo de oferta, precio, contacto, si es pirata: `right_detection`).
4. Los **analistas** revisan los casos en Studio (la app web) y pueden encolar la denuncia del
   aviso a Facebook, que hace otro worker por la propia interfaz de Facebook. Los casos
   confirmados pasan a la Data Layer (otro PoC).

### 3.2 La cadena como cascada

El caso de WBD es una **cascada de decisiones de costo creciente**, y el filtro de decisión es el
primer escalón con modelo:

```
avisos/posts que ve el scout
  → reglas de limpieza en código (link válido, duplicados, región AR/CL, antigüedad)   gratis
  → filtro de decisión, etapa 1: solo texto (Marketplace)                              ~0,4 s, ~1.000 tokens
  → filtro de decisión, etapa 2: texto + captura                                       ~1 s, ~3.000 tokens + una captura
  → caso pending → analizador LLM con la captura (extracción completa)                 un LLM generativo por caso
  → analista humano en Studio → denuncia a Facebook / escalamiento a Legal             minutos de una persona
```

Cada escalón decide NO sobre la mayoría de lo que recibe y deja pasar lo dudoso o lo positivo.
El filtro no reemplaza al analizador ni al analista: **decide qué les llega.**

### 3.3 Por qué decir NO bien es difícil acá

- **El vocabulario es local y cambia.** "TV digital" en Argentina es el nombre de calle del IPTV
  pirata, pero también es el nombre de la TV digital abierta legal (antenas, decodificadores
  ISDB-T). Las apps cambian de nombre (MagisTV → Súper / Flujo TV; Xuper; Stella TV).
- **La oferta muchas veces está solo en la imagen.** En Instagram, 23 de los 31 piratas que se le
  escaparon al modelo de solo texto tenían el caption vacío de contenido ("Compartan",
  hashtags, un alias de donación) y la oferta en el flyer.
- **Hay bordes que son de política, no de modelo.** ¿Un TV box vendido con Xuper instalado es
  pirata? ¿Un ISP identificado que vende su combo de internet + TV? ¿Una noticia sobre el bloqueo
  de MagisTV? ¿Una promoción gratis de una app pirata, sin precio? Las respuestas las fija el
  negocio y van escritas en la pregunta.
- **Los dos errores tienen costos distintos.** Un falso negativo es un pirata que nunca llega a
  denunciarse. Un falso positivo es tiempo de un equipo de dos personas, un analizador que
  procesa ruido y una meta de precisión que no se cumple.

---

## 4. Caso A: Facebook Marketplace (`facebook.marketplace`)

### 4.1 Cómo era antes

- El scout de Facebook (en modo agéntico, que es lo que corre en staging) tiene un agente que
  elige ciudad y consulta y llama a la herramienta `search_marketplace`.
- La búsqueda devuelve **hasta 120 tarjetas** (título, precio, ubicación, link, antigüedad).
- Una **regex sobre el título** (`signals.filter_market`) decidía qué tarjetas abrir. Solo se
  abrían unas pocas. Una tarjeta cuyo título pasaba la regex **se archivaba como caso sin que
  nadie leyera el aviso**. Las que la regex rechazaba pero parecían TV pasaban por una segunda
  regex sobre la descripción.

### 4.2 Qué estaba mal (los números del spike, 7–10/10/2026)

Fuente: `~/Documents/Warner-data/spike-report-2026-10-12.html` §3 y §6, y
`docs/spikes/spec-20261016-marketplace-gate-jev-provider.md` §2.

| Hallazgo | Número |
|---|---|
| Piratas que la regex atrapaba entre los avisos que ella misma descartaba | **1 de 33** (recall 3%, F1 0,06) sobre 288 candidatos descartados de 10 búsquedas (5 consultas × Buenos Aires y Santiago) |
| Proporción de piratas entre lo descartado | 33 de 288 = 11% (el resto, 89%, eran NO de verdad) |
| Acuerdo de una etiqueta hecha solo con el título contra la etiqueta humana | **49%** (47 de 95): el título solo no alcanza ni para una persona |
| Casos de Marketplace que el analizador confirmaba como piratas | **54–82% por semana** (1.114 casos en la base al 9/10): entre 2 y 5 de cada 10 casos eran ruido |
| Modelo de decisión (Jev) con una pregunta, solo título | F1 0,59 (AUC 0,92) |
| OpenAI Decisions, título + miniatura | F1 0,66 (recall 0,85) sobre 170 filas |
| Jev con 18 preguntas + regresión logística, solo título | F1 0,65 contra 0,41 de la pregunta única en el test (10 positivos); 0,76 contra 0,74 de OpenAI + miniatura en las mismas 53 filas, a 1/5 del costo |

Conflicto de política descubierto en el spike: la persona que etiquetó dejó "TV box con apps o
desbloqueo" como dudoso o no pirata, y eso explicaba **11 de los 17 falsos positivos** de Jev.
Pero el analizador confirmaba como pirata **219 de 219** casos que tipificó como `tv_box`.
**Decisión D3:** un TV box, stick o decodificador vendido con apps piratas, canales, desbloqueo o
pack de fútbol cargados **es pirata**, y eso se escribió en la pregunta.

### 4.3 La solución (`FacebookMarketplaceFilter`, versión `v1.2026-10-19.2`)

Código: `workers/scout/facebook/filter_gate.py`, `filter_questions_marketplace.yaml`,
`tools_marketplace.py`.

1. **Se elimina la regex de título.** Quedan solo las reglas de limpieza (link válido, sin
   duplicados, sin basura de la página, región AR/CL, no más viejo que 30 días).
2. **Se abre cada tarjeta que pasa la limpieza**, hasta `SCOUT_FB_MARKET_OPEN_MAX` (30) por
   búsqueda. Se saltean los avisos que ya son caso y los que el modelo rechazó en un scan anterior
   (memoria `marketplace.not_pirate`).
3. **Prefiltro en código:** si el texto nombra una región fuera de AR/CL, se rechaza sin llamar
   al modelo.
4. **Etapa 1, solo texto** (título, precio, ubicación, descripción; sin vendedor, URL, consulta
   ni ciudad): si `P(es_oferta_pirata) ≤ 0,2` → **NO**. Esta etapa **nunca acepta**: lo que no es
   un NO claro pasa a la etapa 2.
5. **Etapa 2, texto + captura del aviso:** la captura se saca recién ahora (de forma diferida) y
   es el mismo archivo que se sube como evidencia. `P ≥ 0,5` → caso; si no, NO, se borra la
   captura y el aviso se recuerda.
6. Lo aceptado se archiva como caso `pending` y **el analizador lo vuelve a revisar**, como antes.

**La pregunta que decide** (en español porque los avisos lo están; cualquier cambio sube la
versión y requiere replay):

> ¿El anuncio de Facebook Marketplace ofrece o vende acceso no autorizado a TV paga, canales en
> vivo, fútbol o deportes, apps de streaming o cuentas premium?
> **Sí:** el título, la descripción o la captura venden un servicio, lista de canales, app,
> activación o cuenta para ver TV o deportes, o un TV box, stick o decodificador que se entrega
> con esas apps, canales, "desbloqueo" o un pack de fútbol cargados.
> **No:** hardware solo, sin apps ni canales prometidos; un ISP o cableoperador identificado que
> vende su propio servicio; juegos, consolas, merchandising o repuestos; servicio técnico sin
> contenido.

Una segunda pregunta, `es_espanol`, se hace y se registra pero **no decide** en Marketplace.

**Por qué dos etapas:** la captura es lo caro (cargar la página, sacarla, ~3x tokens). La
cascada texto → imagen fue lo que mejor anduvo en Instagram (F1 0,92 contra 0,89). Y la etapa 1
solo puede descartar NO claros, porque si no tira los avisos cuya oferta está solo en la imagen.

**Proveedor:** OpenAI Decisions API (`gpt-6-luna`) por defecto; Jev de TypeSafe (`jev-1.14.0`)
seleccionable por superficie con `SCOUT_FILTER_PROVIDER_FACEBOOK_MARKETPLACE=jev`. Un solo
proveedor para las dos etapas, sin mezclar.

### 4.4 Resultados medidos

**a) Run en vivo en `enforce`, 2026-10-20** (fuente: `docs/spikes/marketplace-filter-audit-20261020.html`,
datos embebidos; dos scans, 16:04–16:24 UTC, Buenos Aires 73 y Santiago 27 avisos):

| Métrica | Valor |
|---|---|
| Avisos juzgados | 100 (consultas: "tv digital" 56, "fire stick" 19, "pack futbol canales" 8, "cuentas premium" 8, "demo tv digital" 5, "flujo tv" 3, otras 1) |
| Rechazados por el prefiltro de región | 1 |
| **Descartados en la etapa 1, sin sacar captura** | **86 de 99 (87%)** |
| Pasaron a la etapa 2 (con captura) | 13 |
| **Aceptados como caso** | **12 (12%)**; 1 rechazado en la etapa 2 |
| Errores | 0 |
| Latencia por aviso | p50 425 ms, p95 2,2 s |
| Tokens de entrada | etapa 1 ~1.000; etapa 2 ~3.000 |
| Avisos con probabilidad final entre 0,2 y 0,8 | 1 de 100: el modelo casi nunca duda |

Los 12 aceptados, por título, son ofertas piratas claras salvo uno que solo se entiende con la
imagen: "TV Stick con Xuper TV + Canales Argentinos + Pack Fútbol", "Pack futbol, canales,
peliculas y series", "Basta de Pagar Cable", "App Lumix Tv", "DISFRUTÁ STELLATV", "APP
D.IG.I.TA.L", tres "TV Digital", "Deco" (CLP 18.000), entre otros. **Ninguno de los 100 tiene
etiqueta humana:** la página de auditoría permite etiquetar, pero las etiquetas quedan en el
navegador y no están en el archivo. No se puede calcular precisión ni recall de este run.

**b) Replay de la etapa 1 sobre 1.094 casos reales** (2026-10-19; fuente:
`docs/spikes/marketplace-filter-live-20261019.html`). Casos de Marketplace del 20/8 al 8/10,
etiquetados por el analizador (732 piratas, 362 no). Solo texto, porque el dump no tiene
capturas.

| `reject_lte` | OpenAI: piratas perdidos | OpenAI: no piratas filtrados | Jev: piratas perdidos | Jev: no piratas filtrados |
|---|---|---|---|---|
| 0,02 | 6,8% | 42,3% | 1,0% | 4,1% |
| 0,05 | 15,2% | 61,6% | 4,5% | 19,9% |
| 0,10 | 24,3% | 78,2% | 15,4% | 32,6% |
| **0,20 (el valor actual)** | **28,1%** | **85,6%** | **19,9%** | **46,7%** |

- AUC del puntaje de solo texto: OpenAI 0,85, Jev 0,79. Latencia texto p50: OpenAI 342 ms, Jev
  271 ms. 0 errores en ~2.200 llamadas.
- **La causa de las pérdidas es "TV digital":** aparece en 531 de 1.094 títulos. El modelo lo lee
  como TV digital legal y descarta el **43,2% (OpenAI) / 33,1% (Jev)** de esos piratas. En el
  resto de los títulos pierde solo 9,5% / 3,7%. Tampoco conoce nombres de apps como Xuper. **No
  es un problema de calibración: a la pregunta le falta vocabulario local.**
- Un bug de datos: el texto del aviso arrastraba la tira "Today's picks" de Facebook con avisos de
  George Town (Malasia), aparentemente por la ubicación de la cuenta del scout. El prefiltro de
  región lo leía como la región del aviso y descartaba piratas argentinos. La regex vieja tenía
  el mismo problema.
- Estas pérdidas son una **cota superior**: el analizador etiquetó viendo la captura, y algunos
  avisos no se pueden decidir solo con el texto. Además, el set es lo que abría la regex vieja
  (casos difíciles); en enforce también aparecen NO obvios (consolas, camisetas, televisores),
  donde la etapa 1 ahorra más.

**c) 14 avisos sintéticos, las dos etapas** (prueba de integración, no de exactitud): los dos
proveedores acertaron 13 de 14, sin errores. Todo pirata claro pasó las dos etapas con ≥ 0,98 y
todo no pirata claro se descartó en la etapa 1 sin captura. El que fallaron tenía texto vago
("TV Box 4K oferta / Consultar por privado") y la oferta de Magis TV solo en la imagen; con la
etapa 1 en 0,05, la etapa 2 lo aceptó.

### 4.5 Costo estimado (cálculo propio, a confirmar)

Con el precio de la Decisions API que usan los specs (US$ 0,10 por millón de tokens de entrada;
la salida no se cobra) y los tokens del run del 20/10:

| | Por cada 1.000 avisos abiertos |
|---|---|
| Etapa 1 a todos (~1.000 tokens) | ~US$ 0,10 |
| Etapa 2 al 13% (~3.000 tokens) | ~US$ 0,04 |
| **Cascada** | **~US$ 0,14** y ~130 capturas |
| Todo con imagen (sin etapa 1) | ~US$ 0,30 y 1.000 capturas |

El costo del modelo es despreciable en cualquier caso. **Lo que ahorra la etapa 1 son capturas
(87% menos) y la exposición del scout a los controles anti-bot de Facebook**, no dólares de
modelo. Esto es distinto del caso de deduplicación, donde lo caro es el LLM que razona.

### 4.6 Estado (verificar con el equipo antes de publicar)

- `docker-compose.ecr.yml` (el overlay de staging en EC2) pone
  `SCOUT_FILTER_MODE_FACEBOOK_MARKETPLACE` en **`enforce` por defecto**, con la muestra de
  auditoría en 0.
- El spec del 19/10 decía "Not ready for enforce" por el problema de "TV digital", y la pregunta
  del YAML **todavía no tiene el vocabulario local** (sigue en `v1.2026-10-19.2`). Hay que
  confirmar si staging corre así, si se pasó a shadow o si hay una versión nueva en camino.
- Pendientes que el propio equipo listó: sumar "TV digital" como servicio, nombres de apps (Xuper,
  Magis, Flujo) y "demo / prueba gratis" a los criterios; cortar el texto antes de "Today's
  picks"; repetir el replay; bajar las capturas reales del bucket y medir la etapa 2; corrida en
  shadow con etiquetas humanas.

---

## 5. Caso B: Instagram (`instagram.post`)

### 5.1 Cómo era antes

- El scout busca por palabra clave, abre cada post (y después los otros posts del vendedor) y
  saca una captura de página completa.
- Un gate LLM generativo (`validate_finding`, `gpt-6-luna`) leía caption, usuario, URL y captura
  y respondía sí o no: **en español Y una oferta pirata de streaming/IPTV**. Noticias, avisos
  sobre piratería y posts en otro idioma, fuera.
- **Ante un error, el gate fallaba abierto** (dejaba pasar el post).

### 5.2 Qué estaba mal

| Hallazgo | Número | Fuente |
|---|---|---|
| Casos de Instagram en la base, 10/9–8/10 | **7.329**; el analizador confirmó solo el **20%** | spike-report §6 |
| Peor semana (14/9) | **4.388 casos, 11% confirmados**: 9 de cada 10 eran ruido | spike-report §6 |
| Causa probable | el modelo rechazaba un parámetro (`reasoning_effort="minimal"`) con un 400, el gate fallaba abierto **y aprobaba todo** | spike-report §4 y §6 (inferido de los números, no verificado contra `scout.log`) |
| Calidad del gate viejo en el spike | F1 0,89, latencia p50 3,2 s / p95 5,5 s | spike-report §4 |

**La historia para el blog:** un gate que ante la duda dice SÍ convierte una falla del proveedor
en 4.388 casos basura en una semana. El filtro nuevo **falla cerrado**: un error con un post lo
descarta, y una caída del proveedor **aborta el scan** en lugar de dejar pasar todo.

### 5.3 La solución (`InstagramPostFilter`, versión `2026-10-19.1`)

Código: `workers/scout/instagram/filter_gate.py`, `filter_questions.yaml`. Doc:
`docs/instagram-filter-pipeline.md`.

- **Una sola etapa con imagen**, en la OpenAI Decisions API. El modelo ve **solo el caption
  enmascarado** (teléfonos → `<PHONE>`, URLs → `<URL>`, usuarios → `<HANDLE>`, recortado a 1.500
  caracteres) y la captura. **No** ve el usuario ni la URL.
- Dos preguntas en la misma llamada:
  - `es_oferta_pirata` decide: `P ≥ 0,5` → caso.
  - `es_espanol` **es requisito**: si `P < 0,8`, NO, sea cual sea el puntaje pirata (decisión de
    producto, ver §5.4).
- La pregunta pirata (verbatim del spike; la medida es esta redacción exacta):

> ¿El post de Instagram ofrece o vende acceso no autorizado a TV paga, listas de canales, fútbol
> o deportes, apps de streaming, cuentas premium o películas y series 'digitales'?
> **Sí:** el propio post promociona un servicio, app, pack, lista de canales, activación o
> reventa de cuentas/contenido para ver TV, deportes o streaming, aunque no muestre precio ni
> WhatsApp, o anuncia el lanzamiento de un servicio nuevo que nombra sus canales o packs.
> **No:** noticias o advertencias sobre una plataforma existente (bloqueo, cierre, canales
> eliminados); ISP o cableoperador identificado (empresa, dirección, agente oficial, Mbps,
> instalación); hardware suelto sin servicio; rankings o notas de entretenimiento;
> merchandising de fútbol, memes, denuncias de estafa o comunicados institucionales.

Los **NO** de la pregunta son la política del negocio: noticias, ISP legales, hardware suelto,
memes. Es la parte más larga del criterio porque es la más frecuente.

### 5.4 Resultados medidos

Dataset: 229 posts reales que visitó el scout (dos scans del 8/10), cada uno con su captura;
etiquetas "silver" del analizador (183 piratas, 46 no). Fuente: `docs/TESTING.md`
§"Instagram filter pipeline replay" y `docs/spikes/spec-20261013-gate-pipeline.md` §4.4 y §10.

| Configuración | Precisión | Recall | F1 | Latencia p50 |
|---|---|---|---|---|
| Gate viejo (LLM generativo, caption + captura) | 0,91 | 0,88 | 0,89 | 3,2 s |
| **Decisions API, 1 pregunta + captura, umbral 0,5** (replay en vivo 15/10) | **0,93** | **0,94** | **0,935** | 0,85 s en el spike; 1,4 s en un scan real |
| Ídem, umbral 0,3 / 0,7 | 0,91 / 0,93 | 0,95 / 0,93 | 0,928 / 0,932 | |
| **Con español requerido** (lo que corre hoy, replay cacheado) | **0,93** | 0,82 | 0,872 | |

- **El F1 se mantiene en 0,93 para umbrales de 0,3 a 0,7**: el modelo separa bien (AUC 0,94–0,95)
  y el umbral se puede mover según cuánto ruido se tolere.
- **Precisión 0,91–0,94 en todas las variantes**, por encima de la meta de negocio de ≥ 80%
  (medida contra etiquetas del analizador, no humanas).
- **Exigir español cuesta recall** (0,93 → 0,82): 21 posts etiquetados como piratas tienen el
  caption en otro idioma o solo hashtags. Es una decisión de producto que el negocio tomó a
  sabiendas: la precisión no cambia, se pierden ofertas fuera del mercado objetivo.
- Costo: ~2.850 tokens de entrada por post, **≈ US$ 0,29 cada 1.000 posts**. Al volumen de un mes
  como el de septiembre (7.329 posts que se volvieron caso) serían ~US$ 2.
- **Scan real en enforce (15/10, prueba de humo):** 14 posts llegaron al filtro, 10 aceptados y 4
  rechazados, 0 errores. Revisando las capturas a mano: 9 de los 10 aceptados eran piratas; el
  décimo era un plan legal de DGO vendido por un ISP identificado, con puntaje 0,52, justo arriba
  del umbral. De los 4 rechazados, 3 bien (promos oficiales de ESPN Premium / TNT Sports y una
  noticia) y 1 discutible (un post que apunta a un sitio de streams gratis: no ofrece acceso pago,
  que es lo que la pregunta pregunta). ~US$ 0,004 por el scan. Muestra chica: no es una medición
  de precisión.

### 5.5 Lo que se probó y no ganó (útil para la sección de lecciones)

Del spike de Instagram (spike-report §4):

| Variante | Resultado |
|---|---|
| Jev, solo caption, 1 pregunta | F1 0,90 (P 0,94, R 0,86), **p95 0,45 s**, ~852 tokens (US$ 3,58 cada 100k posts): iguala al gate viejo a 1/10 de la latencia |
| La misma pregunta en inglés | F1 0,884: el idioma de la pregunta casi no importa |
| 10 preguntas chicas + regresión logística | F1 0,87–0,88 contra 0,89 de la pregunta única: **partir la pregunta no aportó** (en Marketplace con solo título sí aportó: 0,41 → 0,65) |
| Cascada Jev texto (acepta ≥ 0,6, rechaza ≤ 0,2) → gate con imagen | F1 0,92, **Jev decide solo el 79% de los posts** (optimista: umbrales elegidos sobre las mismas filas) |
| Calibración de Jev (386 posts) | la banda 0,9–1,0 acertó 104 de 104; la banda 0,0–0,1 escondía 10 piratas, casi todos con la oferta solo en la imagen |

### 5.6 Estado

- `docker-compose.ecr.yml` pone `SCOUT_FILTER_MODE_INSTAGRAM_POST` en **`enforce` por defecto**
  en staging (EC2). Fuera de ese overlay el default del código es `off`.
- Pendientes: etiquetas humanas (todas las métricas son contra el analizador), revisar el borde de
  los ISP (el falso positivo de 0,52), definir si los sitios de streams gratis entran en el
  alcance, y recuperar con la imagen los posts sin texto útil.

---

## 6. El patrón común: cómo está construido el filtro

Código: `workers/scout/filter_gate/`. Doc: `docs/ARCHITECTURE.md#filter-pipeline` y
`workers/scout/README.md#filter-pipeline`. Es un solo algoritmo para todas las superficies; cada
superficie solo declara qué pregunta, qué ve el modelo y con qué umbrales decide.

| Principio | Cómo se implementó | Por qué le importa al lector del blog |
|---|---|---|
| **Preguntas cerradas, no prompts abiertos** | Cada pregunta es un sí/no (o una elección) con criterios de "sí" y de "no" escritos | La política del negocio queda escrita y versionada en un YAML, no enterrada en un prompt |
| **Probabilidades, no veredictos** | El modelo de decisión ("System One": OpenAI Decisions API o Jev) devuelve una probabilidad por opción; la salida no se cobra | El umbral es un dial que fija el negocio; el F1 de Instagram no se mueve entre 0,3 y 0,7 |
| **La decisión se toma en código** | `ScoutFilterPipeline.decide()` es fijo: prefiltro → estado enmascarado → etapas con bandas (`accept_gte`, `reject_lte`) → requisitos (`es_espanol ≥ 0,8`) → política de error | Las reglas duras (región, idioma) no dependen del humor del modelo |
| **Cascadas por etapas** | Una etapa puede ser "solo descarta" (`accept_gte = NEVER`): la etapa 1 de Marketplace | Lo barato filtra los NO claros; lo caro (imagen) solo para lo dudoso |
| **El modelo ve solo lo necesario** | Sin usuario, URL, consulta ni ciudad; teléfonos, URLs y handles enmascarados | Menos sesgo (no aprende "este usuario siempre es pirata") y menos datos personales enviados |
| **Falla cerrado, y una caída frena** | Error con un ítem → NO (`fail_closed`). Caída del proveedor en `enforce` → se aborta el scan, sin volver a la regex | Contraste directo con los 4.388 casos basura del gate que fallaba abierto |
| **Se mide antes de confiar** | Tres modos por superficie: `off`, `shadow` (decide el gate viejo, el nuevo solo registra y se compara), `enforce` | Pasar a producción es cambiar una variable, después de ver los números |
| **Todo queda registrado** | Un JSONL por decisión (probabilidades, banda, tokens, latencia, versión) + una muestra de rechazos sin enmascarar para etiquetar a mano | El sistema produce sus propios datos de evaluación |
| **Cambiar una pregunta es un cambio de versión** | Editar preguntas, umbrales o lo que ve el modelo sube `VERSION` y exige un replay contra datos etiquetados (`filter_gate/replay.py`) | Las mejoras se comparan contra la misma vara |
| **Un proveedor es una clase** | `OpenAIDecisionsProvider` y `JevProvider` (urllib, sin SDK); se elige por superficie con una variable | Se puede cambiar de proveedor sin tocar el pipeline |

---

## 7. Comparación con el caso de deduplicación (para unir los casos en un solo post)

| | Deduplicación | Marketplace (WBD) | Instagram (WBD) |
|---|---|---|---|
| Ítem que se juzga | Un par de productos | Un aviso | Un post |
| Tasa de positivos | 3% | 11–12% (spike y run en vivo); hasta 120 tarjetas por búsqueda | 20% de lo que llegaba a la base (con el gate roto) |
| Preguntas | Banco de 4 + regresión logística | 1 que decide (+ español, registrada) | 1 que decide + español como requisito |
| Imagen | Primera foto a 512 px | Captura del aviso, solo en la etapa 2 | Captura de página completa |
| Validación | Ciega, 10.000 pares, prevalencia real, umbrales fijados antes | Replay de 1.094 casos (solo texto) + run en vivo de 100 sin etiquetar | Replay de 229 posts (etiquetas del analizador) + scan de humo de 14 |
| Número titular | −86% costo y tiempo con 95% de recall | 87% de los avisos descartados sin captura; regex vieja: 1 de 33 piratas | F1 0,89 → 0,935; latencia 3,2 s → 0,85 s; precisión ≥ 0,91 |
| Qué ahorra | Llamadas a un LLM que razona | Capturas, cargas de página, ruido al analizador | Ruido al analizador y a los analistas; latencia |
| Madurez de la evidencia | Alta | **Baja**: falta medir la etapa 2 en datos reales y etiquetas humanas | Media: buen replay, pero 229 posts y etiquetas silver |

**Lo que une a los tres:** el paso caro tiene que gastar su tiempo donde hace falta juicio, y la
mayor parte del flujo son NO que se pueden reconocer barato.

**Lo que cambia, y conviene decirlo:** en la deduplicación el modelo barato **no** pudo reemplazar
al juez y sirvió como puerta. En WBD reemplazó a un gate LLM (Instagram) y a una regex
(Marketplace), y el juez de verdad sigue estando más abajo (el analizador y el analista). La
diferencia es el tipo de tarea: clasificar un ítem con una pregunta bien escrita es más fácil que
comparar dos productos casi iguales.

---

## 8. Lecciones, con la evidencia de cada una

1. **Reglas que dicen NO a todo no son un filtro.** La regex de Marketplace atrapaba 1 de 33
   piratas entre lo que descartaba. Un modelo con una pregunta recupera lo que la regla tira.
2. **Un gate que ante la duda dice SÍ es peor que no tener gate.** 4.388 casos de Instagram en
   una semana con 11% confirmados. Fallar cerrado y frenar ante una caída es una decisión de
   diseño, no un detalle.
3. **El vocabulario local es parte de la pregunta.** "TV digital" hizo que el modelo descartara
   el 43% de los piratas con ese título. No se arregla bajando el umbral (a 0,05 todavía se
   pierde el 15%); se arregla escribiendo en el criterio qué significa "TV digital" en Argentina.
4. **La imagen cuesta, pero a veces es todo el caso.** 23 de 31 piratas que se le escaparon al
   modelo de texto tenían la oferta solo en el flyer. Por eso la etapa de texto solo descarta NO
   claros y la imagen decide.
5. **La política se escribe antes de medir.** El desacuerdo sobre los TV box con apps explicaba
   11 de 17 falsos positivos. Hasta que el negocio decidió (D3), no se podía saber si el modelo
   se equivocaba.
6. **El umbral es del negocio.** Exigir español baja el recall de 0,93 a 0,82 sin tocar la
   precisión; WBD lo eligió porque su mercado es AR/CL. El F1 estable entre 0,3 y 0,7 muestra
   que hay margen para mover el dial.
7. **Menos preguntas pueden alcanzar.** En Instagram una pregunta bien escrita (F1 0,89) le ganó a
   diez chicas (0,87–0,88). En Marketplace con solo título, partirla ayudó (0,41 → 0,65). No hay
   receta: se mide.
8. **Los datos sucios se cuelan por donde no se mira.** La tira "Today's picks" con avisos de
   Malasia hacía que un prefiltro de región correcto descartara piratas argentinos. Revisar lo que
   el modelo realmente lee es parte de la evaluación.
9. **Las etiquetas también son parte del sistema.** Todas las métricas de WBD son contra el
   analizador LLM, no contra personas; el spike pedía ≥ 100 positivos revisados por plataforma
   antes de un GO.

---

## 9. Tabla maestra de números (para copiar con su fuente)

Confianza: **A** medido y reproducible; **B** medido en muestra chica o con etiquetas del
analizador; **C** estimación o cálculo propio; **D** dato del cliente, confidencial.

| Número | Valor | Fuente | Conf. |
|---|---|---|---|
| Suscriptores TNT Sports Chile | ~1 M → ~600 mil en 5 años | PDF p. 3 | D |
| IPTV pirata, tráfico de ISP | ~20% | PDF p. 3 | D |
| Hogares chilenos con IPTV pirata | > 1 M | PDF p. 3 | D |
| Suscripciones HBO Max tras bloquear IPTV en Argentina (feb 2026) | +31.000; +67% en altas brutas | PDF p. 7 | D |
| Valor de recuperar el 10% (Chile) | USD 1,6 M por mes (100 mil × USD 16) | PDF p. 18 | D |
| Takedown DMCA promedio | ~59 min | PDF p. 3, 14 | D |
| Meta de precisión del Social Media Monitor | ≥ 80% | PDF p. 26 | D |
| Meta de volumen detectado | ≥ 5x (≥ 3-5x en el resumen) | PDF p. 4, 25 | D |
| Equipo de Anti-Piracy Management | 2 personas | PDF p. 9 | D |
| Regex de Marketplace: piratas atrapados | 1 de 33 (288 descartados) | spike-report §3 | B |
| Acuerdo título solo vs humano | 49% (47 de 95) | spike-report §3 | B |
| Casos de Marketplace confirmados por el analizador | 54–82% por semana (1.114 casos) | spike-report §6 | B |
| Casos de Instagram, 10/9–8/10 | 7.329, 20% confirmados | spike-report §6 | B |
| Instagram, semana del 14/9 | 4.388 casos, 11% confirmados | spike-report §6 | B |
| Marketplace enforce 20/10: descartados sin captura | 86 de 99 (87%) | audit 20/10 (cálculo sobre los datos embebidos) | B |
| Marketplace enforce 20/10: aceptados | 12 de 100 | ídem | B |
| Marketplace enforce 20/10: latencia por aviso | p50 425 ms, p95 2,2 s | ídem | B |
| Marketplace etapa 1, pérdida de piratas a 0,2 | 28,1% OpenAI / 19,9% Jev (1.094 casos) | live 19/10 | B |
| Marketplace "TV digital", pérdida a 0,2 | 43,2% OpenAI / 33,1% Jev de 405 | live 19/10 | B |
| Marketplace etapa 1, no piratas filtrados a 0,2 | 85,6% OpenAI / 46,7% Jev | live 19/10 | B |
| Marketplace, costo del filtro | ~US$ 0,14 cada 1.000 avisos (cascada) vs ~US$ 0,30 (todo con imagen) | cálculo propio con US$ 0,10/M tokens | C |
| Instagram, F1 del gate viejo | 0,89 (p50 3,2 s) | spike-report §4 | B |
| Instagram, F1 Decisions + imagen | 0,929 cacheado / 0,935 en vivo | TESTING.md; spec 13/10 §10 | A (sobre etiquetas silver) |
| Instagram, precisión | 0,91–0,94 en todas las variantes | ídem | A (silver) |
| Instagram, con español requerido | P 0,93, R 0,82, F1 0,872 | TESTING.md | A (silver) |
| Instagram, AUC | 0,941 cacheado / 0,954 en vivo | ídem | A (silver) |
| Instagram, costo | ≈ US$ 0,29 cada 1.000 posts (~2.850 tokens) | spec 13/10 §6 | C |
| Instagram, scan de humo en enforce | 9 de 10 aceptados piratas; 3 de 4 rechazos correctos | spec 13/10 §10 | B |
| Jev, solo caption | F1 0,90, p95 0,45 s | spike-report §4 | B |
| Cascada Jev → gate con imagen | F1 0,92, Jev decide 79% | spike-report §4 | B (optimista) |
| Offer only in image | 23 de 31 FN de Jev | spike-report §5 | B |
| Política TV box | 11 de 17 FP; analizador 219 de 219 | spike-report §3 | B |

---

## 10. Lo que el blog no puede decir (o solo con aprobación)

- **Nada del PDF sin aprobación de WBD.** El documento es confidencial. Si no se aprueba,
  hablar de "a global media company with live football rights in Latin America" y usar solo
  fuentes públicas para el contexto de piratería.
- **Que el filtro de Marketplace está validado.** No hay etiquetas humanas, la etapa 2 no se midió
  en avisos reales, y el problema de "TV digital" sigue abierto en la versión actual.
- **Que Marketplace "reduce el ruido del analizador del 18–46% a X".** No hay medición de cuánto
  confirma el analizador desde que corre el filtro.
- **Precisión "humana".** Todas las métricas de WBD son contra el analizador LLM (etiquetas silver).
  Decir "against labels from our analysis model" o similar.
- **Que el filtro de Instagram arregló los 4.388 casos.** El ruido de septiembre se debió a un bug
  del gate viejo (inferido, no verificado contra los logs). Se puede decir que el diseño nuevo
  falla cerrado; no que se midió la mejora en producción.
- **Que está en producción.** Lo que hay es staging (EC2) con `enforce` por defecto en las dos
  superficies. Confirmar con el equipo cómo describirlo.
- **Nombres de vendedores, IDs de avisos o capturas.** Tienen teléfonos y caras que el enmascarado
  de texto no cubre. Los ejemplos de títulos de §4.4 son de avisos públicos, pero conviene usarlos
  genéricos ("TV Digital, ARS 10.000").
- **Que mandar datos a OpenAI o TypeSafe está aprobado.** El spec lo deja como pendiente de
  aprobación; TypeSafe llama a su soporte de imágenes "dreamer-ready", no productivo.

---

## 11. Decisiones que tiene que tomar quien firme el post

1. **¿Un post con tres casos o un post de WBD aparte?** El formato de caso de cliente del blog es
   de un cliente por post (`01-analisis-blogs-marvik.md` §2). Opciones: (a) sumar WBD como
   segundo ejemplo en la sección "What this project shows" del post de deduplicación; (b) un post
   Business Impact propio de WBD; (c) un post Technical sobre el patrón ("decision gates") con los
   tres casos.
2. **¿Se nombra a WBD, TNT Sports, HBO Max?** Depende de la aprobación del cliente.
3. **¿Se cuenta como "OpenAI Decisions API" o como "Jev"?** Lo que corre por defecto y lo que se
   midió mejor es la Decisions API (calza con Marvik como OpenAI Select Partner). Jev aparece como
   alternativa más barata y rápida en texto.
4. **¿Qué número va de titular para WBD?** Candidatos: "1 de 33" (la regex), "87% descartados sin
   captura", "F1 0,89 → 0,935 a un cuarto de la latencia", "precisión ≥ 0,91 frente a una meta de
   80%". Recomendación: Instagram para el número de calidad y Marketplace para el de "decir NO
   barato", aclarando que Marketplace es un primer run.
5. **¿Esperar a tener etiquetas humanas de Marketplace?** Con 100–200 avisos etiquetados del modo
   enforce se podría publicar precisión y una estimación de recall. Hoy no se puede.
6. **¿Se cuenta el error de "TV digital"?** Es la mejor lección del caso y es honesta, pero muestra
   un sistema que todavía no está terminado.

---

## 12. Figuras propuestas (máximo dos, como en el blog)

1. **La cascada de WBD:** avisos que ve el scout → limpieza en código → etapa 1 texto (descarta
   el 87%) → etapa 2 con captura → analizador → analista. Con el ancho de cada paso proporcional
   al volumen del run del 20/10 (100 → 99 → 13 → 12).
2. **El dial del umbral en Marketplace:** piratas perdidos contra no piratas filtrados para
   `reject_lte` 0,02–0,30, con las curvas de OpenAI y Jev (tabla de §4.4 b). Muestra que el
   problema no se resuelve moviendo el umbral.

Alternativa para Instagram: barra de F1 / latencia del gate viejo contra el filtro (0,89 / 3,2 s
contra 0,935 / 0,85 s).

---

## 13. Fuentes

| Archivo | Qué tiene |
|---|---|
| `docs/blog-marvik/Warner Discovery - Anti Piracy Strategy - Three POCs.pdf` | Contexto de negocio, KPIs del Social Media Monitor (confidencial) |
| `~/Documents/Warner-data/spike-report-2026-10-12.html` | Spikes de Marketplace e Instagram (7–10/10), ruido en producción, preguntas probadas. Fuera del repo; tiene URLs y captions |
| `docs/spikes/spec-20261013-gate-pipeline.md` | Diseño del pipeline compartido (WBD-199), spike y replays de Instagram |
| `docs/spikes/spec-20261016-marketplace-gate-jev-provider.md` | Diseño del filtro de Marketplace y el proveedor Jev (WBD-200), decisiones D1–D11 |
| `docs/spikes/marketplace-filter-20261019.html` | Explicación para el equipo del filtro de Marketplace (antes y después) |
| `docs/spikes/marketplace-filter-live-20261019.html` | Replay de la etapa 1 sobre 1.094 casos, 14 sintéticos, el problema de "TV digital" |
| `docs/spikes/marketplace-filter-audit-20261020.html` | Run en vivo en enforce de 100 avisos, con capturas (datos embebidos en `const DATA`) |
| `docs/instagram-filter-pipeline.md` | Flujo de Instagram, los tres modos, diferencias con el gate viejo |
| `docs/TESTING.md` §"Instagram filter pipeline replay" | Replay con español requerido (versión actual) |
| `docs/ARCHITECTURE.md#filter-pipeline` | El modelo y el diagrama del pipeline |
| `workers/scout/filter_gate/`, `workers/scout/facebook/filter_gate.py`, `workers/scout/instagram/filter_gate.py` y los YAML de preguntas | El código y las preguntas tal como están |
| `docker-compose.ecr.yml` | Modos por defecto en staging (`enforce` en las dos superficies) |

---

## 14. Glosario mínimo para el redactor

| Término | Qué es |
|---|---|
| **Scout** | Worker que navega una plataforma con un navegador real y propone candidatos |
| **Superficie** | Dónde se encontró un candidato: `marketplace`, `post`, `ad-library`… El filtro se nombra `<plataforma>.<superficie>` |
| **Gate / filtro** | La decisión antes de crear un caso |
| **Modelo de decisión / System One** | Un modelo que responde preguntas cerradas con una probabilidad por opción, sin generar texto (OpenAI Decisions API, Jev de TypeSafe) |
| **Analizador** | El LLM que procesa cada caso pendiente con la captura y extrae los datos; sus etiquetas (`right_detection`) son las "silver labels" de las métricas |
| **Shadow / enforce** | Shadow: el filtro nuevo juzga y registra pero decide el viejo. Enforce: decide el nuevo |
| **Etapa (stage)** | Un paso del filtro con su propio umbral; la etapa 1 de Marketplace solo descarta |
| **Fail closed / fail open** | Ante un error, rechazar (cerrado) o dejar pasar (abierto) |
| **IPTV** | TV por internet; acá, servicios piratas pagos tipo MagisTV, Xuper, Flujo TV |
| **"TV digital"** | En Argentina, nombre de calle del IPTV pirata; también el de la TV digital abierta legal |
