# LM Studio — Apunte completo para desarrolladores .NET

> Material de estudio autocontenido para un desarrollador que sabe programar (stack .NET/C#) pero parte de cero en LLMs locales.
> **Objetivo final del lector:** armar chatbots con modelos locales, darles una base de conocimiento (RAG), conectarlos a herramientas (MCP) y **consumirlos por API desde .NET**, con la opción de dockerizar.
>
> **Datos verificados en la web el 2026-06-14.** LM Studio cambia rápido; donde un dato es volátil, lo aclaro y dejo la fuente. Versión de referencia usada en este apunte: **LM Studio 0.4.16** (8 de junio de 2026).

---

## Tabla de contenidos

1. [Introducción y conceptos base](#1-introducción-y-conceptos-base)
2. [Instalación y primer modelo](#2-instalación-y-primer-modelo)
3. [Qué se puede hacer con LM Studio (panorama)](#3-qué-se-puede-hacer-con-lm-studio-panorama)
4. [Cómo elegir el modelo según el hardware](#4-cómo-elegir-el-modelo-según-el-hardware)
5. [Construir un chatbot](#5-construir-un-chatbot)
6. [El servidor de API local](#6-el-servidor-de-api-local)
7. [Agregar una base de conocimiento (RAG)](#7-agregar-una-base-de-conocimiento-rag)
8. [Integración con MCP](#8-integración-con-mcp-model-context-protocol)
9. [Consumir el modelo local desde .NET](#9-consumir-el-modelo-local-desde-net)
10. [Dockerizar / despliegue headless](#10-dockerizar--despliegue-headless)
11. [Escenario de producción: modelo dockerizado con base de conocimiento y API para .NET](#11-escenario-de-producción-modelo-dockerizado-con-base-de-conocimiento-y-api-para-net)
12. [Resumen y ruta de aprendizaje recomendada](#12-resumen-y-ruta-de-aprendizaje-recomendada)
- [Anexo A — Tabla maestra de modelos de referencia](#anexo-a--tabla-maestra-de-modelos-de-referencia)
- [Anexo B — Referencia de comandos `lms`](#anexo-b--referencia-de-comandos-lms)
- [Anexo C — Referencia de endpoints de la API](#anexo-c--referencia-de-endpoints-de-la-api)
- [Anexo D — Glosario completo](#anexo-d--glosario-completo)

---

## 1. Introducción y conceptos base

### Qué es LM Studio

**LM Studio** es una **aplicación de escritorio** para descargar y ejecutar modelos de lenguaje (LLMs) **localmente** en tu propia máquina. Combina tres cosas en un solo paquete:

- una **interfaz gráfica (GUI)** para chatear con modelos y descargarlos,
- un **servidor de API local** compatible con OpenAI (y también con Anthropic),
- una **CLI** (`lms`) y un modo **headless** para automatización y servidores.

> ⚠️ **No es un IDE.** No reemplaza a Visual Studio ni a VS Code. Es un *runtime* + GUI + servidor para modelos. Lo que vos programás (tu app .NET) vive afuera y le habla por HTTP.

### Qué problema resuelve

| Problema | Cómo lo resuelve LM Studio |
|---|---|
| **Privacidad** | Los datos nunca salen de tu máquina. Ideal para información sensible. |
| **Costo** | Costo cero por token. Pagás el hardware una vez, después inferís gratis. |
| **Offline** | Una vez descargado el modelo, funciona sin internet. |
| **Control total** | Elegís el modelo, la cuantización, el contexto y los parámetros. |

### Glosario inicial mínimo

> **Modelo:** archivo con los pesos (parámetros) de una red neuronal entrenada que predice texto.
>
> **Inferencia:** el acto de *ejecutar* el modelo para generar una respuesta a partir de una entrada (prompt). Es lo que consume CPU/GPU/RAM.
>
> **Parámetros (7B / 8B / 70B):** cantidad de pesos del modelo. "B" = *billion* (mil millones). Más parámetros ≈ más capacidad, pero más memoria y más lento.
>
> **Cuantización:** técnica para reducir la precisión numérica de los pesos (de 16 bits a 4, por ejemplo) para que el modelo ocupe menos memoria y corra más rápido, a cambio de algo de calidad.
>
> **GGUF:** formato de archivo de modelo cuantizado usado por el motor llama.cpp. Multiplataforma (Windows/Linux/Mac).
>
> **MLX:** formato/motor de inferencia de Apple, **solo para Apple Silicon** (chips M1/M2/M3/M4).
>
> **Ventana de contexto (context window):** cantidad máxima de tokens (entrada + salida) que el modelo puede "tener en cuenta" a la vez. Si la conversación la supera, se trunca.
>
> **Token:** unidad mínima de texto que procesa el modelo (≈ ¾ de palabra en inglés). Todo se mide en tokens: contexto, costo, velocidad.

### Motores de ejecución

LM Studio trae **dos motores de inferencia** y podés elegir por modelo:

| Motor | Formato | Plataformas | Notas |
|---|---|---|---|
| **llama.cpp** | **GGUF** | Windows, Linux, macOS | Motor por defecto. Soporta K-quants (control fino de cuantización). Es el que vas a usar en Windows. |
| **Apple MLX** | **MLX** | **Solo Apple Silicon** (M1–M4) | Más rápido sobre Metal; mejor para visión. No corre en Mac Intel ni en Windows/Linux. |

*Fuentes: [lmstudio.ai/docs/app](https://lmstudio.ai/docs/app), [blog v0.3.4](https://lmstudio.ai/blog/lmstudio-v0.3.4).*

---

## 2. Instalación y primer modelo

### Instalación

Descargá el instalador desde **[lmstudio.ai](https://lmstudio.ai)** según tu sistema:

| SO | Requisitos clave (verificados) |
|---|---|
| **Windows** | x64 (requiere **AVX2**) o ARM (Snapdragon X). 16 GB+ RAM recomendado; GPU con ≥4 GB VRAM recomendado. |
| **macOS** | **Solo Apple Silicon** (M1/M2/M3/M4), macOS 14.0+. 16 GB+ recomendado. Mac Intel no soportado. |
| **Linux** | x64 y ARM64, formato **AppImage**, Ubuntu 20.04+. AVX2 incluido en x64 por defecto. |

*Fuente: [system-requirements](https://lmstudio.ai/docs/app/system-requirements).*

En Windows: ejecutás el `.exe`, instalación estándar, y abrís la app.

### Buscar y descargar un modelo desde la GUI

1. Abrí LM Studio y andá a la pestaña **🔍 Discover** (o "Search").
2. Buscá un modelo (LM Studio indexa **Hugging Face**). Para empezar liviano: `Llama 3.2 3B Instruct` o `Qwen3 4B`.
3. LM Studio te muestra las **variantes de cuantización** disponibles (Q4_K_M, Q8_0, etc.) y te indica con un color/etiqueta si entra cómodo en tu RAM.
4. Elegí **Q4_K_M** (el "sweet spot") y dale **Download**.

> 💡 Regla mental: en **Q4_K_M**, un modelo ocupa aproximadamente **la mitad de su número de parámetros, en GB**. Un 7B ≈ 4 GB; un 14B ≈ 8 GB. (Ver [sección 4](#4-cómo-elegir-el-modelo-según-el-hardware).)

### Primer chat en la GUI

1. Andá a la pestaña **💬 Chat**.
2. Arriba, en el selector, **cargá el modelo** (botón "Select a model to load"). LM Studio lo sube a RAM/VRAM.
3. Escribí un mensaje y enviá. Listo: tenés inferencia local funcionando, sin internet y sin costo por token.

---

## 3. Qué se puede hacer con LM Studio (panorama)

Antes de profundizar, este es el mapa completo de capacidades. Las secciones siguientes desarrollan cada una.

| Capacidad | Qué es | Sección |
|---|---|---|
| **Chat local en la GUI** | Conversar con el modelo desde la app. | [§5](#5-construir-un-chatbot) |
| **Servidor de API local** | Expone el modelo por HTTP. Compatible con **OpenAI**, **Anthropic** y una **REST nativa**. Esto es lo que consume tu app .NET. | [§6](#6-el-servidor-de-api-local) |
| **Document chat (RAG)** | Arrastrás un PDF/Word/TXT y el modelo responde sobre su contenido. | [§7](#7-agregar-una-base-de-conocimiento-rag) |
| **Integración MCP** | LM Studio conecta servidores MCP para darle **herramientas** al modelo (acceso a archivos, web, APIs). | [§8](#8-integración-con-mcp-model-context-protocol) |
| **CLI `lms` + headless `llmster`** | Control por línea de comandos y ejecución sin GUI en servidores/CI. | [§10](#10-dockerizar--despliegue-headless), [Anexo B](#anexo-b--referencia-de-comandos-lms) |
| **SDKs oficiales** | Librerías para integrar desde código. | abajo 👇 |

### SDKs oficiales

| Lenguaje | Paquete | Instalación |
|---|---|---|
| **Python** | `lmstudio` (repo *lmstudio-python*) | `pip install lmstudio` |
| **TypeScript / JS** | `@lmstudio/sdk` (repo *lmstudio-js*) | `npm install @lmstudio/sdk` |

> **¿Y .NET?** No hay SDK oficial de LM Studio para .NET. **No lo necesitás:** como la API es **compatible con OpenAI**, usás cualquier cliente OpenAI de .NET (o `HttpClient` pelado) apuntando a `localhost`. Eso es exactamente lo que vemos en la [sección 9](#9-consumir-el-modelo-local-desde-net).

*Fuentes: [SDK blog](https://lmstudio.ai/blog/introducing-lmstudio-sdk), [pypi.org/project/lmstudio](https://pypi.org/project/lmstudio/), [@lmstudio/sdk](https://www.npmjs.com/package/@lmstudio/sdk).*

---

## 4. Cómo elegir el modelo según el hardware

La pregunta central es: **¿qué entra y corre bien en mi máquina?** Depende de tres variables ligadas:

```
tamaño del modelo (parámetros)  ×  nivel de cuantización  =  memoria necesaria
```

- En **GPU**, lo que importa es la **VRAM** (memoria de la placa). Si el modelo entra en VRAM, vuela.
- En **CPU-only**, lo que importa es la **RAM** del sistema. Funciona, pero más lento.
- Lo ideal es que **modelo + contexto** entren completos en VRAM; si no, parte va a RAM (más lento) o no carga.

### Regla práctica para estimar memoria

```
RAM/VRAM (GB) ≈ Parámetros(B) × bytes_por_parámetro × 1.2
```
El `×1.2` cubre el *KV-cache* y overhead. Bytes por parámetro según precisión:

- **FP16** = 2 bytes → ~2 GB por cada 1B de parámetros
- **Q8** ≈ 1 byte → ~1 GB / 1B
- **Q4_K_M** ≈ 0.55–0.6 bytes → **~0.5–0.6 GB / 1B**

> **Atajo:** en **Q4_K_M**, memoria ≈ (parámetros ÷ 2) GB, **más** headroom para el contexto. El KV-cache crece con la ventana de contexto y puede sumar varios GB en contextos largos — es "el monstruo oculto" de la memoria.

### TABLA 1 — Qué corre realista en cada rango de memoria

| Memoria | CPU-only (RAM) | GPU (VRAM) | Cuantización típica |
|---|---|---|---|
| **8 GB** | 1B–4B (lento pero usable) | 3B–4B holgado | Q4_K_M |
| **16 GB** | 7B–8B | 7B–14B | Q4_K_M / Q5_K_M |
| **24 GB** | 13B–14B | 14B–32B | Q4_K_M |
| **32 GB** | 14B–32B (lento) | 32B cómodo | Q4_K_M / Q5_K_M |
| **64 GB+** | 70B (lento en CPU) | 70B+ y **MoE grandes** | Q4_K_M (70B ≈ 39–42 GB) |

> **MoE (Mixture of Experts):** modelos que tienen muchos parámetros totales pero solo **activan una fracción** por token (p. ej. *Qwen3 30B-A3B* tiene 30B totales / 3B activos). Rinden como uno chico en velocidad pero con conocimiento de uno grande. Muy convenientes para hardware limitado.

### TABLA 2 — Niveles de cuantización GGUF (trade-off calidad/tamaño)

| Nivel | Bits aprox. | Calidad | Cuándo usarlo |
|---|---|---|---|
| **Q2_K** | ~2.6 | Degradación notable | Último recurso, memoria extrema |
| **Q3_K_M** | ~3.4 | Aceptable, pérdida visible | Hardware muy ajustado |
| **Q4_K_M** | ~4.5 | **~92–95% de la calidad full** a ~25% del tamaño | ⭐ **Sweet spot general** |
| **Q5_K_M** | ~5.5 | Casi indistinguible, mejor en código | Si sobra memoria (12–16 GB) |
| **Q6_K** | ~6.6 | Prácticamente = FP16 | Tareas sensibles |
| **Q8_0** | ~8.5 | Casi idéntico al original | Memoria de sobra (32 GB+) |
| **IQ2/IQ3/IQ4…** | sub-4 | Mejor calidad que Q clásico a igual bitrate | Maximizar calidad a bits muy bajos (requiere *imatrix*) |

> Las variantes `_K_M` usan **K-quants**: cuantizan distintas capas a distinta precisión según su importancia, dando mejor calidad que una cuantización uniforme del mismo bitrate.

### Tamaño en disco/RAM por nivel (referencia)

| Modelo | FP16 | Q8_0 | Q5_K_M | **Q4_K_M** | Q3_K_M |
|---|---|---|---|---|---|
| **7B** | 13.0 GB | 6.9 GB | 4.6 GB | **3.9 GB** | 3.2 GB |
| **13B** | 24.2 GB | 12.9 GB | 8.6 GB | **7.3 GB** | 5.9 GB |
| **34B** | 63.3 GB | 33.6 GB | 22.6 GB | **19.0 GB** | 15.4 GB |
| **70B** | 130.4 GB | 69.3 GB | 46.4 GB | **39.1 GB** | 31.8 GB |

*Valores aproximados (varían según cuantizador). El archivo en disco ≈ lo que ocupa en RAM al cargar, más el contexto.*

### Modelos de referencia por gama (vigentes a junio 2026)

| Gama | Modelos recomendados | Para qué |
|---|---|---|
| **8 GB** | Llama 3.2 3B · Qwen3 4B · Gemma 3 4B · Phi-4-mini (3.8B) | Chat básico, pruebas, tool calling liviano |
| **16 GB** | Qwen3 8B · Llama 3.1/3.3 8B · Gemma 3 12B · Phi-4 (14B) | Chatbot serio, RAG, agentes |
| **24–32 GB** | Qwen3 32B · Qwen3 30B-A3B (MoE) · Gemma 3 27B · Devstral 24B (código) · **gpt-oss-20b** | Asistente potente, código, tool use fuerte |
| **64 GB+** | **gpt-oss-120b** · Llama 3.3 70B · Qwen3 235B-A22B (MoE) | Razonamiento de frontera, producción local |

*El catálogo cambia seguido; verificá nombres exactos en [lmstudio.ai/models](https://lmstudio.ai/models) dentro de la app. Ver detalle ampliado en el [Anexo A](#anexo-a--tabla-maestra-de-modelos-de-referencia).*

> 🔧 **Nota especial — chatbots con herramientas/MCP:** priorizá modelos **"instruct"** con buen **function/tool calling**. A junio 2026 los más sólidos en tool use son **gpt-oss (20b/120b)**, **Qwen3** (soporta MCP nativamente), **Llama 3.x** y **Granite 4.1**. Evitá modelos de "razonamiento puro" destilados (p. ej. DeepSeek-R1-Distill) si tu prioridad es llamar herramientas: razonan bien pero llaman tools peor.

---

## 5. Construir un chatbot

### Qué es un "chatbot" sobre un LLM

Un chatbot no es más que una **secuencia de mensajes** que le mandás al modelo en cada turno. Tres piezas:

> **System prompt:** instrucción inicial que define el rol/comportamiento del bot ("Sos un asistente de soporte técnico, respondé corto y en español").
>
> **Historial de mensajes:** la lista de turnos previos (`user` / `assistant`) que le das al modelo para que tenga memoria de la conversación. El LLM en sí **no recuerda nada**: vos le reenviás el historial completo en cada request.
>
> **Parámetros:** ajustes de generación. Los dos más importantes:
> - `temperature` (0–2): creatividad/aleatoriedad. **0.2–0.4** para respuestas precisas/deterministas; **0.7–1.0** para creatividad.
> - `context length`: cuántos tokens de historial el modelo tiene en cuenta. Si la conversación crece, hay que truncar o resumir.

El formato estándar de mensajes (igual que OpenAI):

```json
{
  "messages": [
    { "role": "system",    "content": "Sos un asistente conciso que responde en español." },
    { "role": "user",      "content": "¿Qué es la cuantización?" },
    { "role": "assistant", "content": "Es reducir la precisión de los pesos del modelo..." },
    { "role": "user",      "content": "¿Y eso afecta la calidad?" }
  ]
}
```

### Ejemplo vía la GUI

1. En **Chat**, cargá un modelo *instruct*.
2. En el panel derecho de **System Prompt**, escribí el rol del bot.
3. Ajustá `temperature` y `context length` en la barra de configuración.
4. Conversá. La GUI ya gestiona el historial por vos.

### Ejemplo vía API (curl)

Lo mismo, pero programático (esto es la base de la [sección 9](#9-consumir-el-modelo-local-desde-net)):

```bash
curl http://localhost:1234/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3-8b",
    "messages": [
      { "role": "system", "content": "Sos un asistente conciso en español." },
      { "role": "user",   "content": "Explicame qué es un token en una frase." }
    ],
    "temperature": 0.3
  }'
```

---

## 6. El servidor de API local

Esta es la pieza que conecta LM Studio con tu app .NET.

### Cómo levantar el servidor

**Desde la GUI:** pestaña **Developer** → activá el switch **"Start server"**.

**Desde la CLI:**

```bash
lms server start --port 1234
```

Flags útiles confirmados:

| Flag | Qué hace |
|---|---|
| `--port <n>` | Puerto a escuchar (si se omite, usa el último). |
| `--cors` | Habilita CORS (necesario si lo llamás desde un navegador/SPA). |
| `--bind <addr>` | Dirección de bind. **`127.0.0.1`** por defecto (solo local); **`0.0.0.0`** para exponer en la red. |

Para verificar / detener:

```bash
lms server status
lms server stop
```

### Qué expone

Por defecto, una API **compatible con OpenAI** en **`http://localhost:1234`**. Esto significa que cualquier herramienta o SDK pensado para OpenAI funciona apuntándolo a esa URL.

> **Endpoint OpenAI-compatible:** una API que replica las rutas y el formato de request/response de OpenAI (`/v1/chat/completions`, etc.). Cambiás la `BaseAddress` y el resto del código sigue igual.

Además de OpenAI, LM Studio (serie 0.4.x) expone:

- **REST nativa `/api/v1/*`** (recomendada para proyectos nuevos desde 0.4.0; soporta chats con estado, gestión de modelos y autenticación). La vieja **`/api/v0/*`** sigue existiendo como *legacy* (requería 0.3.6+).
- **Anthropic-compatible `/v1/messages`** (desde 0.4.1): para usar herramientas de la API de Claude (incluido **Claude Code**) contra modelos locales.

### TABLA de endpoints principales (OpenAI-compatible)

| Método | Ruta | Para qué sirve |
|---|---|---|
| `GET` | `/v1/models` | Lista los modelos cargados/disponibles. |
| `POST` | `/v1/chat/completions` | **El principal.** Chat con roles (system/user/assistant). Soporta streaming. |
| `POST` | `/v1/completions` | Completado de texto plano (sin estructura de chat). Legacy. |
| `POST` | `/v1/embeddings` | Genera **embeddings** (vectores) para RAG/búsqueda semántica. |
| `POST` | `/v1/responses` | Responses API de OpenAI (soporta Codex). |
| `POST` | `/v1/messages` | **Anthropic-compatible** (formato Claude). |

### Ejemplo de request (curl)

```bash
curl http://localhost:1234/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3-8b",
    "messages": [{ "role": "user", "content": "Say this is a test!" }],
    "temperature": 0.7
  }'
```

Para **streaming** (respuesta token a token vía SSE) agregás `"stream": true` al body.

### Limitaciones (importante)

- **Sin autenticación por defecto:** la API local no pide API key. Desde 0.4.0 hay un sistema **opcional** de *API tokens* (se habilita en *Developer Page → Server Settings*).
- **Pensado como servidor personal/local:** escucha en `127.0.0.1` por defecto. No está documentado como servidor multiusuario concurrente; tratalo como **single-user / un proceso a la vez** salvo que sepas lo que hacés.
- Si exponés con `--bind 0.0.0.0`, **activá los API tokens**: sin auth, cualquiera en la red puede usar tu modelo.

*Fuentes: [core/server](https://lmstudio.ai/docs/developer/core/server), [openai-compat](https://lmstudio.ai/docs/developer/openai-compat), [api-changelog](https://lmstudio.ai/docs/developer/api-changelog), [authentication](https://lmstudio.ai/docs/developer/core/authentication).*

---

## 7. Agregar una base de conocimiento (RAG)

### Qué es RAG

> **RAG (Retrieval-Augmented Generation):** técnica para que el modelo responda usando **tus documentos**, sin reentrenarlo. En lugar de meterle todo el conocimiento en los pesos, le **buscás los fragmentos relevantes** y se los pegás en el prompt al momento de preguntar.

Sirve para que el bot conteste sobre manuales, contratos, documentación interna, etc., con datos que el modelo nunca vio en su entrenamiento.

### La cadena RAG

```
documentos → chunks → embeddings → vector store → recuperación → contexto en el prompt → respuesta
```

> **Chunk:** fragmento de un documento (p. ej. 500 tokens). Se parte porque no entra todo y porque la búsqueda es más precisa por trozos.
>
> **Embedding:** vector de números que representa el *significado* de un texto. Textos parecidos → vectores cercanos. Se generan con un **modelo de embeddings** (distinto del LLM de chat).
>
> **Vector store:** base de datos especializada en guardar embeddings y encontrar los más parecidos a una consulta (búsqueda por similitud).
>
> **Recuperación (retrieval):** dada la pregunta del usuario, se calcula su embedding y se traen los chunks más cercanos del vector store.

Flujo en dos fases:

1. **Indexación (una vez):** partís los docs en chunks → generás un embedding por chunk → los guardás en el vector store.
2. **Consulta (cada pregunta):** embedding de la pregunta → traés los top-K chunks similares → los inyectás en el prompt → el LLM responde citando ese contexto.

### Opción A — RAG integrado de LM Studio (sin programar)

LM Studio tiene **"Chat with Documents"**: arrastrás un archivo a la conversación y listo.

- **Formatos soportados:** `.pdf`, `.docx`, `.txt`.
- **Cómo decide:** si el documento entra en la ventana de contexto del modelo, **lo inserta entero**; si es muy largo, aplica **RAG** (recupera solo los pasajes relevantes).
- **Límite:** la doc no fija un tamaño máximo de archivo; el límite real es el **contexto del modelo**.
- **Tip oficial:** al preguntar, mencioná términos que esperás encontrar en el material; mejora la recuperación.

Esto es perfecto para uso interactivo, pero **la lógica la maneja la app** — no controlás el chunking ni el vector store.

> 💡 Para volúmenes grandes existe el plugin de comunidad **"Big RAG"** (más formatos, chunk size/overlap configurables). No es lo mismo que el document chat integrado.

### Opción B — RAG propio usando el endpoint de embeddings (para tu app .NET)

Si querés control total (tu propio vector store, tu lógica), LM Studio te da solo la pieza de **embeddings** vía `/v1/embeddings`; **el vector store y la orquestación los ponés vos**.

> **Modelo de embeddings recomendado:** **`nomic-embed-text-v1.5`** (GGUF: `nomic-ai/nomic-embed-text-v1.5-GGUF`). Liviano, CPU-friendly, es el ejemplo oficial. Para multilingüe/mejor calidad: `nomic-embed-text-v2-moe` o `bge-m3`. Cargalo en LM Studio como cualquier otro modelo.

Pedido de embeddings (curl):

```bash
curl http://localhost:1234/v1/embeddings \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nomic-embed-text-v1.5",
    "input": "El motor llama.cpp usa el formato GGUF."
  }'
```

Respuesta (recortada): te devuelve el vector en `data[0].embedding`.

```json
{
  "object": "list",
  "data": [{ "object": "embedding", "index": 0, "embedding": [0.013, -0.041, 0.077, "..."] }],
  "model": "nomic-embed-text-v1.5"
}
```

### Ejemplo conceptual del flujo (pseudocódigo)

```text
# Fase de indexación (una vez)
para cada documento:
    chunks = partir(documento, tamaño=500, overlap=100)
    para cada chunk:
        vec = POST /v1/embeddings (chunk)      # modelo de embeddings
        vectorStore.guardar(vec, chunk)        # tu DB (SQLite+sqlite-vec, Postgres+pgvector, Qdrant...)

# Fase de consulta (cada pregunta del usuario)
qVec   = POST /v1/embeddings (pregunta)
top    = vectorStore.buscarSimilares(qVec, k=4)
prompt = "Contexto:\n" + unir(top) + "\n\nPregunta: " + pregunta
resp   = POST /v1/chat/completions (system + prompt)   # modelo de chat
```

> En .NET tenés librerías que ya implementan esta cadena (p. ej. **Microsoft.Extensions.AI** / **Semantic Kernel** con conectores de vector store). Como LM Studio es OpenAI-compatible, las configurás apuntando a `localhost:1234`. Ver [sección 9](#9-consumir-el-modelo-local-desde-net).

*Fuentes: [docs/app/basics/rag](https://lmstudio.ai/docs/app/basics/rag), [text-embeddings](https://lmstudio.ai/docs/text-embeddings).*

---

## 8. Integración con MCP (Model Context Protocol)

> **MCP (Model Context Protocol):** estándar abierto de **Anthropic** para darle a los modelos acceso a **herramientas** y **recursos** externos (archivos, web, APIs, bases de datos) de forma uniforme. Es como un "USB-C" para conectar capacidades a un LLM.

### Dos roles que no hay que confundir

| Rol | Qué significa en LM Studio |
|---|---|
| **LM Studio como MCP host/client** | LM Studio **se conecta a servidores MCP** para darle herramientas al modelo. El modelo puede, por ejemplo, leer un archivo o consultar la web a través de esos servidores. **Este es el caso típico.** |
| **LM Studio como proveedor de API** | LM Studio **expone su propia API** (OpenAI/Anthropic) para que *otros* clientes (tu app .NET, Claude Code) la consuman. Esto **no es MCP**: es el servidor de la [§6](#6-el-servidor-de-api-local). |

> En criollo: con MCP, **LM Studio es el que llama herramientas**. Con la API REST, **LM Studio es el llamado**.

### Desde qué versión

Soporte MCP disponible **desde LM Studio 0.3.17**. En **0.4.0** se agregó además poder usar MCP **vía la API REST**.

### Cómo agregar un servidor MCP

Dos caminos:

1. **Editando `mcp.json`:** barra lateral derecha → pestaña **Program** → **Install → Edit `mcp.json`**.
2. **Botón "Add to LM Studio":** *deeplink* de un clic que ofrecen algunos autores de servidores MCP.

LM Studio usa la **misma notación de `mcp.json` que Cursor**.

### Ejemplo mínimo de `mcp.json`

**Servidor remoto** (ejemplo oficial — el MCP de Hugging Face):

```json
{
  "mcpServers": {
    "hf-mcp-server": {
      "url": "https://huggingface.co/mcp",
      "headers": {
        "Authorization": "Bearer <TU_HF_TOKEN>"
      }
    }
  }
}
```

**Servidor local por stdio** (notación estándar Cursor con `command`/`args`/`env` — verificá los nombres en tu instalación):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "C:/datos/permitidos"],
      "env": {
        "API_KEY": "valor-si-hace-falta"
      }
    }
  }
}
```

### ⚠️ Advertencia de seguridad (oficial)

> *"Algunos servidores MCP pueden ejecutar código arbitrario, acceder a tus archivos locales y usar tu conexión de red. Tené siempre cuidado al instalar y usar servidores MCP."*
> **Nunca instales MCP de fuentes no confiables.**

Como mitigación, **cuando el modelo llama a una herramienta, LM Studio muestra un diálogo de confirmación** donde podés revisar (y editar) los argumentos antes de ejecutar.

*Fuentes: [blog 0.3.17](https://lmstudio.ai/blog/lmstudio-v0.3.17), [docs/app/mcp](https://lmstudio.ai/docs/app/mcp), [docs/app/plugins/mcp](https://lmstudio.ai/docs/app/plugins/mcp).*

---

## 9. Consumir el modelo local desde .NET

Esta es la sección clave para tu objetivo. La idea central:

> **LM Studio expone una API OpenAI-compatible en `http://localhost:1234`. Desde .NET la consumís igual que consumirías OpenAI, pero cambiando la `BaseAddress` a localhost.** La API key no se valida (poné cualquier string).

**Pre-requisito:** tener el servidor levantado (`lms server start --port 1234`) y un modelo *instruct* cargado.

### 9.1 — Con `HttpClient` pelado (sin dependencias)

Ideal para entender qué pasa por debajo. POST a `/v1/chat/completions` y parseo del JSON.

```csharp
using System.Net.Http.Json;
using System.Text.Json.Serialization;

// --- Modelos de request/response (los campos que nos importan) ---
record ChatMessage(
    [property: JsonPropertyName("role")] string Role,
    [property: JsonPropertyName("content")] string Content);

record ChatRequest(
    [property: JsonPropertyName("model")] string Model,
    [property: JsonPropertyName("messages")] ChatMessage[] Messages,
    [property: JsonPropertyName("temperature")] double Temperature = 0.3);

record Choice([property: JsonPropertyName("message")] ChatMessage Message);
record ChatResponse([property: JsonPropertyName("choices")] Choice[] Choices);

// --- Llamada ---
using var http = new HttpClient
{
    BaseAddress = new Uri("http://localhost:1234"),
    Timeout = TimeSpan.FromMinutes(5)   // la inferencia local puede tardar
};

var request = new ChatRequest(
    Model: "qwen3-8b",                  // identificador del modelo cargado en LM Studio
    Messages: new[]
    {
        new ChatMessage("system", "Sos un asistente conciso que responde en español."),
        new ChatMessage("user",   "Explicame qué es la cuantización en una frase.")
    });

var resp = await http.PostAsJsonAsync("/v1/chat/completions", request);
resp.EnsureSuccessStatusCode();

var result = await resp.Content.ReadFromJsonAsync<ChatResponse>();
Console.WriteLine(result!.Choices[0].Message.Content);
```

> **El `model`** debe coincidir con el identificador que muestra LM Studio (pestaña Developer o `GET /v1/models`). Si ya hay un solo modelo cargado, muchos clientes aceptan cualquier valor.

### 9.2 — Con una librería compatible con OpenAI

Usás el SDK oficial **`OpenAI`** de .NET (NuGet `OpenAI`), configurando el endpoint a localhost. Mismo código que usarías contra OpenAI real.

```csharp
// dotnet add package OpenAI
using OpenAI;
using OpenAI.Chat;
using System.ClientModel;

var options = new OpenAIClientOptions
{
    Endpoint = new Uri("http://localhost:1234/v1")   // 👈 apuntamos a LM Studio
};

// La API local no valida la key: cualquier string sirve.
var client = new ChatClient(
    model: "qwen3-8b",
    credential: new ApiKeyCredential("lm-studio"),
    options: options);

ChatCompletion completion = await client.CompleteChatAsync(
    new SystemChatMessage("Sos un asistente conciso en español."),
    new UserChatMessage("Dame tres ventajas de correr modelos localmente."));

Console.WriteLine(completion.Content[0].Text);
```

> También funciona con **Microsoft.Extensions.AI** / **Semantic Kernel**: ambos aceptan un cliente OpenAI con endpoint custom, así que apuntás a `localhost:1234/v1` y usás sus abstracciones (chat, embeddings, RAG, function calling) contra el modelo local.

### 9.3 — Streaming (respuesta token a token)

Para UX de "se va escribiendo", usás streaming. Con el SDK de OpenAI:

```csharp
await foreach (StreamingChatCompletionUpdate update
               in client.CompleteChatStreamingAsync(
                   new UserChatMessage("Contame un chiste corto sobre programadores.")))
{
    foreach (var part in update.ContentUpdate)
        Console.Write(part.Text);   // imprime cada fragmento a medida que llega
}
```

Con `HttpClient` pelado, mandás `"stream": true` y leés el cuerpo como **SSE** (líneas `data: {...}`), parseando cada delta hasta recibir `data: [DONE]`.

> A partir de acá el apunte sube de nivel: configuración real con DI, la abstracción moderna `Microsoft.Extensions.AI`, **tool calling**, un **RAG propio completo** y un **chatbot con historial en ASP.NET Core**. Es la parte "para cuando ya querés hacer algo serio".

### 9.4 — Configuración e inyección de dependencias (base de una app real)

No hardcodees la URL ni el modelo. Ponelos en `appsettings.json` y registrá el cliente en el contenedor de DI una sola vez.

```json
// appsettings.json
{
  "LmStudio": {
    "Endpoint": "http://localhost:1234/v1",
    "ChatModel": "qwen3-8b",
    "EmbeddingModel": "nomic-embed-text-v1.5",
    "ApiKey": "lm-studio"
  }
}
```

```csharp
// Program.cs (ASP.NET Core / Generic Host)
using OpenAI;
using OpenAI.Chat;
using System.ClientModel;

var builder = WebApplication.CreateBuilder(args);

// Tipado fuerte de la config
builder.Services.Configure<LmStudioOptions>(
    builder.Configuration.GetSection("LmStudio"));

// Registramos el ChatClient del SDK OpenAI apuntando a LM Studio
builder.Services.AddSingleton(sp =>
{
    var cfg = builder.Configuration.GetSection("LmStudio").Get<LmStudioOptions>()!;
    var options = new OpenAIClientOptions { Endpoint = new Uri(cfg.Endpoint) };
    return new ChatClient(cfg.ChatModel, new ApiKeyCredential(cfg.ApiKey), options);
});

var app = builder.Build();
// ...

public sealed class LmStudioOptions
{
    public string Endpoint { get; set; } = "http://localhost:1234/v1";
    public string ChatModel { get; set; } = "";
    public string EmbeddingModel { get; set; } = "";
    public string ApiKey { get; set; } = "lm-studio";
}
```

> El SDK de OpenAI maneja su `HttpClient` internamente y `ChatClient` es *thread-safe*: registralo como **singleton**. Si en cambio usás `HttpClient` pelado (9.1), ahí sí registralo con `IHttpClientFactory` (`builder.Services.AddHttpClient(...)`).

### 9.5 — `Microsoft.Extensions.AI`: la abstracción recomendada

> **`Microsoft.Extensions.AI`:** capa de abstracción oficial de .NET que define interfaces neutrales como **`IChatClient`** e **`IEmbeddingGenerator`**. Tu código depende de la interfaz, no del proveedor: hoy LM Studio, mañana OpenAI o Azure, sin tocar la lógica.

Ventaja real: te da un **pipeline** componible (logging, telemetría, caché, **invocación automática de funciones**) que se enchufa con un `using`.

```csharp
// dotnet add package Microsoft.Extensions.AI
// dotnet add package Microsoft.Extensions.AI.OpenAI   (preview)
using Microsoft.Extensions.AI;
using OpenAI;
using OpenAI.Chat;
using System.ClientModel;

var options = new OpenAIClientOptions { Endpoint = new Uri("http://localhost:1234/v1") };

// Convertimos el ChatClient de OpenAI en un IChatClient neutral
IChatClient chat =
    new ChatClient("qwen3-8b", new ApiKeyCredential("lm-studio"), options)
        .AsIChatClient()                 // adaptador de Microsoft.Extensions.AI.OpenAI
        .AsBuilder()
        .UseFunctionInvocation()         // resuelve tool calls automáticamente (ver 9.6)
        .UseLogging()
        .Build();

ChatResponse resp = await chat.GetResponseAsync(
[
    new ChatMessage(ChatRole.System, "Sos un asistente conciso en español."),
    new ChatMessage(ChatRole.User,   "¿Qué ventaja tiene IChatClient?")
]);

Console.WriteLine(resp.Text);
```

Registro en DI (reemplaza al singleton crudo de 9.4 si adoptás esta abstracción):

```csharp
builder.Services
    .AddChatClient(sp =>
    {
        var cfg = sp.GetRequiredService<IOptions<LmStudioOptions>>().Value;
        var opts = new OpenAIClientOptions { Endpoint = new Uri(cfg.Endpoint) };
        return new ChatClient(cfg.ChatModel, new ApiKeyCredential(cfg.ApiKey), opts)
                   .AsIChatClient();
    })
    .UseFunctionInvocation()
    .UseLogging();
// Ahora inyectás IChatClient en cualquier servicio/controlador.
```

> ⚠️ Estos paquetes evolucionan rápido. Si tu versión no encuentra `AsIChatClient()`, probá el nombre anterior `AsChatClient()`. La forma de uso (interfaces + pipeline) se mantiene.

### 9.6 — Function / tool calling desde .NET

Querés que el modelo pueda **llamar funciones de tu código** (consultar una base, el clima, una API interna). Con `Microsoft.Extensions.AI` es casi automático: declarás métodos, los registrás como herramientas y el pipeline se encarga del ida y vuelta.

> **Tool calling (function calling):** el modelo no ejecuta nada por sí mismo; *pide* invocar una función con ciertos argumentos. Tu runtime la ejecuta y le devuelve el resultado para que redacte la respuesta final. `UseFunctionInvocation()` automatiza ese ciclo.

```csharp
using Microsoft.Extensions.AI;
using System.ComponentModel;

// 1) Tus funciones, anotadas para que el modelo entienda qué hacen
[Description("Devuelve el clima actual de una ciudad")]
static string GetWeather(
    [Description("Nombre de la ciudad, ej. 'Rosario'")] string city)
    => $"En {city} está despejado, 22 °C.";   // acá iría tu llamada real

[Description("Suma dos números")]
static double Add(double a, double b) => a + b;

// 2) Las exponés como herramientas
var chatOptions = new ChatOptions
{
    Tools =
    [
        AIFunctionFactory.Create(GetWeather),
        AIFunctionFactory.Create(Add)
    ]
};

// 3) chat es el IChatClient con .UseFunctionInvocation() del 9.5
ChatResponse resp = await chat.GetResponseAsync(
    "¿Qué temperatura hay en Rosario y cuánto es 19 + 23?",
    chatOptions);

Console.WriteLine(resp.Text);
// El modelo llamó GetWeather("Rosario") y Add(19,23) por su cuenta,
// y redactó la respuesta con los resultados.
```

> 🔧 **Requisito:** usá un modelo *instruct* con buen soporte de tool calling (gpt-oss, Qwen3, Llama 3.x, etc. — ver [§4](#4-cómo-elegir-el-modelo-según-el-hardware)). Un modelo sin esa capacidad ignorará las herramientas. Esto es independiente de [MCP](#8-integración-con-mcp-model-context-protocol): acá las herramientas viven en **tu** proceso .NET; con MCP viven en servidores que conecta LM Studio.

### 9.7 — RAG propio completo desde .NET

Implementamos la cadena de la [§7](#7-agregar-una-base-de-conocimiento-rag) en C#: generamos embeddings con LM Studio, los guardamos en un **vector store en memoria** y recuperamos por similitud coseno. Es el "hola mundo" del RAG, suficiente para entender el mecanismo y para prototipos chicos.

```csharp
// dotnet add package OpenAI
using OpenAI;
using OpenAI.Embeddings;
using System.ClientModel;

var options = new OpenAIClientOptions { Endpoint = new Uri("http://localhost:1234/v1") };
var cred    = new ApiKeyCredential("lm-studio");

var embedder = new EmbeddingClient("nomic-embed-text-v1.5", cred, options);
var chat     = new OpenAI.Chat.ChatClient("qwen3-8b", cred, options);

// --- Vector store mínimo en memoria ---
record Doc(string Text, ReadOnlyMemory<float> Vector);
var store = new List<Doc>();

async Task<ReadOnlyMemory<float>> EmbedAsync(string text)
{
    var emb = await embedder.GenerateEmbeddingAsync(text);
    return emb.Value.ToFloats();
}

static float Cosine(ReadOnlyMemory<float> a, ReadOnlyMemory<float> b)
{
    var x = a.Span; var y = b.Span;
    float dot = 0, na = 0, nb = 0;
    for (int i = 0; i < x.Length; i++) { dot += x[i] * y[i]; na += x[i] * x[i]; nb += y[i] * y[i]; }
    return dot / (MathF.Sqrt(na) * MathF.Sqrt(nb) + 1e-9f);
}

// --- 1) Indexación (una vez). En real, partí cada documento en chunks. ---
string[] chunks =
{
    "LM Studio usa el motor llama.cpp con el formato GGUF.",
    "Apple MLX solo corre en chips Apple Silicon.",
    "El endpoint de embeddings de LM Studio es /v1/embeddings."
};
foreach (var c in chunks)
    store.Add(new Doc(c, await EmbedAsync(c)));

// --- 2) Consulta ---
string pregunta = "¿Qué formato de modelo usa LM Studio en Windows?";
var qVec = await EmbedAsync(pregunta);

var top = store
    .Select(d => (d.Text, Score: Cosine(qVec, d.Vector)))
    .OrderByDescending(d => d.Score)
    .Take(2)
    .Select(d => d.Text);

string contexto = string.Join("\n- ", top);
var completion = await chat.CompleteChatAsync(
    new OpenAI.Chat.SystemChatMessage(
        "Respondé SOLO con el contexto dado. Si no está, decí que no lo sabés."),
    new OpenAI.Chat.UserChatMessage($"Contexto:\n- {contexto}\n\nPregunta: {pregunta}"));

Console.WriteLine(completion.Value.Content[0].Text);
```

> **Para producción** no uses una `List` en memoria: cambiá el store por una base vectorial real (**Postgres + pgvector**, **Qdrant**, **SQLite + sqlite-vec**, Azure AI Search). En .NET, **`Microsoft.Extensions.VectorData`** te da una interfaz común (`IVectorStore`) con conectores para esas bases, y el `IEmbeddingGenerator` de [9.5](#95--microsoftextensionsai-la-abstracción-recomendada) reemplaza al `EmbeddingClient` crudo. La lógica (chunking, top-K, prompt) es la misma que acá.

### 9.8 — Chatbot con historial en ASP.NET Core

Juntamos todo: una **minimal API** que mantiene el historial por sesión y devuelve la respuesta. El LLM no recuerda nada entre requests, así que el historial lo guardás vos (acá en memoria; en real, Redis/DB).

```csharp
using Microsoft.Extensions.AI;
using System.Collections.Concurrent;

// chat = IChatClient registrado en DI (ver 9.5)
var app = builder.Build();

// historial por sesión (demo: en memoria)
var sessions = new ConcurrentDictionary<string, List<ChatMessage>>();

app.MapPost("/chat/{sessionId}", async (
    string sessionId, ChatInput input, IChatClient chat) =>
{
    var history = sessions.GetOrAdd(sessionId, _ =>
    [
        new ChatMessage(ChatRole.System, "Sos un asistente de soporte conciso en español.")
    ]);

    history.Add(new ChatMessage(ChatRole.User, input.Message));

    var response = await chat.GetResponseAsync(history,
        new ChatOptions { Temperature = 0.3f });

    history.AddMessages(response);          // agrega la respuesta del asistente al historial
    return Results.Ok(new { reply = response.Text });
});

// Variante con streaming (SSE): el cliente ve la respuesta escribirse
app.MapPost("/chat/{sessionId}/stream", (
    string sessionId, ChatInput input, IChatClient chat, HttpResponse http) =>
{
    http.Headers.ContentType = "text/event-stream";
    var history = sessions.GetOrAdd(sessionId, _ =>
        [ new ChatMessage(ChatRole.System, "Sos un asistente conciso en español.") ]);
    history.Add(new ChatMessage(ChatRole.User, input.Message));

    return Task.Run(async () =>
    {
        await foreach (var update in chat.GetStreamingResponseAsync(history))
            await http.WriteAsync($"data: {update.Text}\n\n");
    });
});

app.Run();

record ChatInput(string Message);
```

> Tres cuidados con el historial: (1) **crece sin parar** → recortá los mensajes viejos o resumilos para no pasarte de la ventana de contexto; (2) **persistilo** fuera de memoria si tenés varias instancias; (3) si activaste tool calling, el historial también acumula los mensajes de tool — `AddMessages(response)` ya los incluye.

### Buenas prácticas (resumen)

| Tema | Recomendación |
|---|---|
| **Abstracción** | Para apps nuevas, programá contra **`IChatClient`/`IEmbeddingGenerator`** (Microsoft.Extensions.AI): cambiás de proveedor sin tocar la lógica. |
| **Timeouts** | La inferencia local (CPU o contexto largo) tarda. Subí el `Timeout` (5 min es un punto de partida). Con streaming, el *time-to-first-token* incluye la carga del modelo. |
| **Streaming** | Usalo para UX: la primera respuesta tarda (carga + *prompt processing*), pero el usuario ve avance. |
| **Configuración** | `Endpoint`, `model`, `ApiKey` en `appsettings`. En LM Studio cambiás de modelo seguido; no lo hardcodees. |
| **DI / ciclo de vida** | `ChatClient`/`IChatClient` como **singleton** (thread-safe). Con `HttpClient` pelado, `IHttpClientFactory`. |
| **Historial** | Lo gestionás vos: recortá/resumí para no exceder el contexto; persistilo fuera de memoria si escalás. |
| **Tool calling** | Necesita un modelo *instruct* con soporte real de tools. Con `UseFunctionInvocation()` el ciclo es automático. |
| **RAG** | Para prototipo, vector store en memoria; para producción, pgvector/Qdrant/SQLite-vec vía `Microsoft.Extensions.VectorData`. |
| **Manejo de errores** | Si el modelo no está cargado, la API responde error: capturalo y mostrá un mensaje claro. |
| **Seguridad** | Si exponés LM Studio en red (`--bind 0.0.0.0`), activá los API tokens y mandá `Authorization: Bearer` desde .NET. |

---

## 10. Dockerizar / despliegue headless

### Lo primero: la app de escritorio NO se containeriza

La GUI de LM Studio es una app de escritorio (Electron) y **no se dockeriza**. Lo que **sí** se puede correr sin GUI y en contenedor es el **núcleo headless**.

### `lms` vs `llmster` vs LM Studio

> **LM Studio:** la app de escritorio completa (GUI + servidor + runtime).
>
> **`llmster`:** el **daemon headless** — el mismo núcleo de LM Studio empaquetado como **servicio sin GUI**, pensado para servidores Linux, instancias cloud, GPU rigs y CI.
>
> **`lms`:** la **CLI** que controla a cualquiera de los dos (la app o el daemon). Si nada está corriendo, los comandos `lms` arrancan el backend automáticamente.

Instalación del headless (servidor Linux):

```bash
curl -fsSL https://lmstudio.ai/install.sh | bash   # Mac/Linux
# Windows: irm https://lmstudio.ai/install.ps1 | iex
```

Control del daemon:

```bash
lms daemon up      # iniciar el daemon
lms daemon down    # detenerlo
lms server start --port 1234   # levantar la API
```

### Imagen Docker oficial (estado actual verificado)

| Dato | Valor (verificado 2026-06-14) |
|---|---|
| **Imagen** | `lmstudio/llmster-preview` (tag `:cpu`) |
| **Estado** | **Technical Preview** |
| **Hardware** | **CPU-only** (sin GPU) |
| **Arquitectura** | **Solo x86 (amd64)** — ARM64 no soportado en el preview |
| **Puerto** | **1234** (`http://<host>:1234`) |
| **Tamaño** | ~370 MB. Trae `lms` preinstalado. |

> ⚠️ El preview no tuvo cambios recientes (la página figura actualizada hace ~10 meses). Tomalo como **experimental**, no como base de producción.

### Ejemplo `docker run`

```bash
docker run -d -p 1234:1234 lmstudio/llmster-preview:cpu
```

Después interactuás con la CLI apuntando al contenedor para descargar/cargar modelos:

```bash
lms get qwen3-4b --host localhost --port 1234
lms load qwen3-4b --host localhost --port 1234
```

### `docker-compose` mínimo

```yaml
services:
  llmster:
    image: lmstudio/llmster-preview:cpu
    ports:
      - "1234:1234"
    volumes:
      - lmstudio-models:/root/.lmstudio   # persistir modelos descargados
    restart: unless-stopped

volumes:
  lmstudio-models:
```

### Cuándo LM Studio headless y cuándo una alternativa

| Escenario | Recomendación |
|---|---|
| Dev local, prototipo, CI en CPU x86 | **LM Studio headless / `llmster`** funciona, pero está en preview. |
| **Producción dockerizada con GPU** | ⚠️ La imagen oficial es **CPU-only**. Para GPU en contenedor conviene **Ollama** o el **server de llama.cpp** (`llama-server`), que tienen imágenes con soporte CUDA maduras. |
| Necesitás GUI + descubrimiento de modelos cómodo | LM Studio app de escritorio (no contenedor). |
| API OpenAI-compatible estable en server Linux con GPU | **Ollama** o **llama.cpp server** (ambos OpenAI-compatible). |

> **Resumen práctico:** para *desarrollar* y *consumir desde .NET* en tu máquina, LM Studio es ideal. Para *desplegar en producción dockerizada con GPU*, hoy conviene Ollama o llama.cpp server; reservá LM Studio headless para CPU/CI o cuando salga de preview.

*Fuentes: [lmstudio-vs-llmster-vs-lms](https://lmstudio.ai/docs/app/basics/lmstudio-vs-llmster-vs-lms), [headless_llmster](https://lmstudio.ai/docs/developer/core/headless_llmster), [hub.docker.com/r/lmstudio/llmster-preview](https://hub.docker.com/r/lmstudio/llmster-preview).*

---

## 11. Escenario de producción: modelo dockerizado con base de conocimiento y API para .NET

> Esta sección une todo lo anterior en **un escenario de producción concreto y de punta a punta**: el modelo corriendo en un contenedor, su **base de conocimiento (RAG)** persistida en un vector store, y una **API OpenAI-compatible** que tu app .NET consume. Reusa lo ya visto — el runtime dockerizado de la [§10](#10-dockerizar--despliegue-headless), la cadena RAG de la [§7](#7-agregar-una-base-de-conocimiento-rag) y el cliente .NET de la [§9](#9-consumir-el-modelo-local-desde-net) — y agrega el "pegamento" que faltaba: arquitectura, generación de la imagen, configuración del contenedor según el modelo, preparación de la KB y dimensionamiento de usuarios. **El código .NET de la [§9](#9-consumir-el-modelo-local-desde-net) no cambia**: sigue siendo una API OpenAI-compatible; solo cambian el `Endpoint` y la infraestructura que la rodea.

### 11.1 Arquitectura del stack

La idea es separar responsabilidades en contenedores distintos, cada uno con su ciclo de vida y sus recursos. Una consulta RAG fluye así:

```
                                   ┌─────────────────────────────────────────────┐
                                   │                  Docker network              │
  ┌──────────────┐                 │   ┌───────────────────────────────────────┐ │
  │  Cliente      │   HTTP/JSON     │   │  runtime del modelo (CONTENEDOR 1)    │ │
  │  .NET (§9)    │────────────────►│   │  llmster + lms  (o llama-server /     │ │
  │  IChatClient  │  /v1/chat/...   │   │  ollama en el camino GPU)             │ │
  │  IEmbeddingGen│◄────────────────│   │  expone OpenAI-compat :1234           │ │
  └──────┬───────┘    (opcional)    │   │  /v1/chat/completions /v1/embeddings  │ │
         │            reverse proxy │   └──────────────┬────────────────────────┘ │
         │            (nginx/Caddy/ │                  │ embeddings                │
         │             Traefik):    │                  ▼                           │
         │            TLS + API key │   ┌───────────────────────────────────────┐ │
         │            + rate-limit  │   │  vector store (CONTENEDOR 2)          │ │
         │                          │   │  Qdrant / Postgres+pgvector           │ │
         └─────────────────────────┘   │  guarda y busca por similitud         │ │
                                        └──────────────┬────────────────────────┘ │
                                                       ▲ upsert de vectores        │
                                        ┌──────────────┴────────────────────────┐ │
                                        │  servicio de ingesta (CONTENEDOR 3)   │ │
                                        │  job/worker .NET: lee docs → chunking │ │
                                        │  → POST /v1/embeddings → upsert store │ │
                                        └───────────────────────────────────────┘ │
                                        (volúmenes: modelos / datos del store)     │
                                   └─────────────────────────────────────────────┘
```

Qué corre en cada contenedor:

| Contenedor | Qué corre | Puerto | Volumen | Notas |
|---|---|---|---|---|
| **Runtime del modelo** | `llmster` + `lms` (camino A) **o** `llama-server`/`ollama` (camino B). Expone la API OpenAI-compatible. | `1234` (LM Studio), `8080` (llama-server), `11434` (ollama) | modelos (`/root/.lmstudio`) | Es el que consume CPU/GPU/RAM. **Single-user** en el caso LM Studio (ver caveats en §11.2). |
| **Vector store** | Base vectorial (Qdrant, Postgres+pgvector, etc.). | según motor | datos del índice | Persistente. Sobrevive a reinicios del runtime. |
| **Servicio de ingesta** | Worker/job .NET que indexa documentos: chunking → embeddings (vía el runtime) → upsert al store. | — | docs de entrada | Se ejecuta **una vez** por lote de docs (o por cron), no de forma continua. |
| **Reverse proxy** *(opcional pero recomendado al exponer)* | nginx / Caddy / Traefik. | `443` | certificados | Pone TLS, **autenticación** y rate-limiting **delante** del runtime, que no trae nada de eso por defecto. |

Flujo de una consulta del usuario (fase de **consulta**, no de indexación):

1. El cliente .NET arma la pregunta y pide su **embedding** al runtime (`POST /v1/embeddings`).
2. Con ese vector, consulta el **vector store** los top-K chunks más parecidos.
3. Inyecta esos chunks como contexto y llama `POST /v1/chat/completions` contra el **mismo runtime**.
4. El runtime infiere y devuelve la respuesta (con streaming si querés). Si hay reverse proxy, todo pasa por él (TLS + `Authorization: Bearer`).

> La **indexación** (leer docs, partir en chunks, embeddings, upsert) la hace el **contenedor de ingesta** por separado, no en el camino de la request del usuario. Ver la cadena RAG en la [§7](#7-agregar-una-base-de-conocimiento-rag).

### 11.2 Dos caminos para el runtime del modelo

No hay una sola respuesta: depende de si tenés **GPU** y de cuánta madurez/concurrencia necesitás. Los dos caminos exponen API OpenAI-compatible, así que **tu .NET es el mismo** (solo cambia `Endpoint` y el puerto).

#### (A) Camino LM Studio nativo — `lmstudio/llmster-preview:cpu`

Útil para **CPU**, **CI** y **previews**. Es lo que ya describimos en la [§10](#10-dockerizar--despliegue-headless): imagen ~370 MB, x86/amd64, puerto 1234, `lms` preinstalado, persistencia en `/root/.lmstudio`.

> ⚠️ **Caveats explícitos (no los tapes):**
> - Es **Technical Preview** y **CPU-only**: sin aceleración GPU, la inferencia de modelos medianos/grandes es lenta.
> - **Solo x86 (amd64)** — no hay ARM64 en el preview.
> - El server de LM Studio está pensado como **single-user / un proceso a la vez**; no es un servidor multiusuario concurrente.
> - La página del preview no se actualiza hace meses. **No la tomes como base de producción tal cual.**
> - Ideal para: validar tu integración .NET, pipelines de CI sin GPU, demos reproducibles, o entornos donde justamente no querés GPU.

#### (B) Camino producción con GPU — `llama-server` (llama.cpp) u Ollama

Cuando necesitás **GPU real en contenedor**, hoy conviene un runtime con imagen **CUDA madura**:

- **`ghcr.io/ggml-org/llama.cpp:server-cuda`** — el mismo `llama-server` compilado con CUDA 12 (hay tag `server-cuda13` para CUDA 13). Corre el **mismo motor llama.cpp / GGUF** que usa LM Studio por debajo, pero con build GPU oficial. Puerto 8080.
- **`ollama/ollama`** — runtime con gestión de modelos cómoda (`ollama pull`), imagen oficial con soporte NVIDIA. Puerto 11434.

Ambos exponen **OpenAI-compatible** (`/v1/chat/completions`, `/v1/embeddings`), así que **el código .NET de la [§9](#9-consumir-el-modelo-local-desde-net) no cambia**: solo apuntás `Endpoint` a `http://<host>:8080/v1` o `http://<host>:11434/v1`. Requieren **NVIDIA Container Toolkit** instalado en el host y `--gpus all` en el `docker run`.

| Criterio | **LM Studio / llmster** | **Ollama** | **llama-server (llama.cpp)** |
|---|---|---|---|
| **GPU en contenedor** | ❌ imagen oficial CPU-only (preview) | ✅ imagen CUDA oficial (`--gpus all`) | ✅ imagen CUDA oficial (`server-cuda`, `-ngl 99`) |
| **Concurrencia / batching** | Single-user, un proceso a la vez | Cola de requests; varios modelos | **Batching real** (`--parallel`, *continuous batching*) |
| **Madurez para prod** | Preview, experimental | Madura, muy usada | Madura, es el motor base de todos |
| **OpenAI-compat** | ✅ (+ Anthropic + REST nativa) | ✅ | ✅ |
| **Gestión de modelos** | `lms get/load`, GUI para descubrir | `ollama pull` (registry propio) | Manual (descargás el GGUF vos) |
| **Cuándo elegirlo** | Dev local, CI/CPU, demos, o cuando salga de preview | Producción con GPU y operación simple, multimodelo | Producción con GPU exigente: máximo control de batching, contexto y rendimiento |

> **Regla práctica:** para *desarrollar* y *consumir desde .NET*, LM Studio. Para *producción dockerizada con GPU*, hoy **Ollama** (operación simple) o **llama-server** (máximo rendimiento/control). Como los tres son OpenAI-compatible, podés desarrollar contra LM Studio y desplegar contra Ollama/llama-server sin tocar la lógica de tu app.

### 11.3 Generar la imagen Docker (camino A)

#### `docker run` básico contra la imagen oficial

```bash
docker run -d --name llmster \
  -p 1234:1234 \
  -v lmstudio-models:/root/.lmstudio \
  lmstudio/llmster-preview:cpu
```

Después le decís a la CLI que descargue y cargue el modelo **apuntando al contenedor**:

```bash
lms get  qwen3-8b --host localhost --port 1234
lms load qwen3-8b --host localhost --port 1234 --context-length=8192 --ttl=3600
```

El problema de esto: cada vez que recreás el contenedor, **volvés a cargar el modelo a mano**. Para producción querés que **arranque ya listo**.

#### Dockerfile que extiende la base y pre-carga el modelo

El patrón es: extender `lmstudio/llmster-preview:cpu`, definir qué modelos querés (chat + embeddings), y un **entrypoint** que levanta el server y carga el modelo al arrancar.

```dockerfile
# Dockerfile
FROM lmstudio/llmster-preview:cpu

# Modelos que va a usar este contenedor (chat + embeddings para RAG)
ENV CHAT_MODEL="qwen3-8b" \
    EMBED_MODEL="nomic-embed-text-v1.5" \
    CONTEXT_LENGTH="8192" \
    TTL="0"

# --- OPCIÓN 1: bakear los modelos EN la imagen (build) ---
# Los descarga durante el build; la imagen queda autocontenida pero PESADA.
RUN lms get ${CHAT_MODEL} && lms get ${EMBED_MODEL}

COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

EXPOSE 1234
HEALTHCHECK --interval=30s --timeout=5s --start-period=120s --retries=5 \
  CMD curl -fsS http://localhost:1234/v1/models || exit 1

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
```

```bash
# entrypoint.sh
#!/usr/bin/env bash
set -euo pipefail

# 1) Levantar la API (en background) con CORS habilitado
lms server start --port 1234 --cors &

# 2) Esperar a que el server responda
until curl -fsS http://localhost:1234/v1/models >/dev/null 2>&1; do
  echo "esperando al server de LM Studio..."; sleep 2
done

# 3) Si los modelos NO están bakeados (volumen montado), descargarlos ahora
lms get "${CHAT_MODEL}"  || true
lms get "${EMBED_MODEL}" || true

# 4) Cargar a memoria con la config del contenedor
#    --gpu=0 porque la imagen oficial es CPU-only; --ttl=0 = sin auto-unload
lms load "${CHAT_MODEL}"  --gpu=0 --context-length="${CONTEXT_LENGTH}" --ttl="${TTL}"
lms load "${EMBED_MODEL}" --gpu=0 --ttl="${TTL}"

echo "runtime listo: ${CHAT_MODEL} + ${EMBED_MODEL}"

# 5) Mantener el contenedor vivo siguiendo al proceso del server
wait
```

> 💡 **Antes de cargar**, validá memoria sin gastarla: `lms load <modelo> --estimate-only` te previsualiza los requisitos. Útil en el build/CI para fallar temprano si el modelo no entra.

#### "Modelo bakeado en la imagen" vs "modelo en volumen montado"

| Patrón | Cómo | Ventaja | Trade-off |
|---|---|---|---|
| **Bakeado (en el build)** | `RUN lms get ...` en el Dockerfile | Imagen **autocontenida**: arranca offline, despliegue reproducible (mismo digest = mismo modelo). | Imagen **enorme** (varios GB) y lenta de pushear/pullear; cambiar de modelo = rebuild. |
| **En volumen montado** | `-v lmstudio-models:/root/.lmstudio` + `lms get` en el entrypoint | Imagen **chica**; el modelo se descarga una vez y se **reusa en caché** entre contenedores/recreaciones; cambiás de modelo sin rebuild. | Primer arranque **descarga** (necesita red); hay que gestionar/persistir el volumen. |

> **Recomendación:** para **CI/efímero/air-gapped**, bakeá. Para **servidores estables** donde varios contenedores comparten cache de modelos, **volumen montado**. Híbrido común: imagen chica + volumen, y un *warm-up job* que hace `lms get` una vez antes de poner el servicio en rotación.

### 11.4 Configurar el contenedor para el modelo adecuado

Estos son los parámetros que definen **qué modelo corre y cómo**. La mayoría se setea con flags de `lms load` (dentro del entrypoint) y con opciones de Docker (memoria/CPU/GPU).

| Parámetro | Dónde se setea | Para qué sirve | Recomendación |
|---|---|---|---|
| **Límite de RAM** | `--memory` (run) / `mem_limit` (compose) | Techo de RAM del contenedor. Si el modelo + KV-cache lo superan, el OOM-killer lo mata. | RAM del modelo (Q4_K_M) + headroom de contexto + ~2 GB. |
| **CPUs** | `--cpus` / `cpus:` | Núcleos disponibles para inferencia CPU. | Cuantos más, mejor throughput en CPU-only. |
| **GPU** | `--gpus all` (run) | Pasa la GPU al contenedor (**solo camino B**, requiere NVIDIA Container Toolkit). | En camino A es inútil (imagen CPU-only). |
| **`--gpu=max\|auto\|0.0-1.0`** | flag de `lms load` | Fracción del modelo en GPU. `max` = todo lo posible; `0` = CPU puro. | Camino A: `--gpu=0`. Camino B (con `llmster` GPU): `--gpu=max`. |
| **`--context-length`** | flag de `lms load` | Ventana de contexto del modelo cargado. Más contexto = más KV-cache = más RAM. | `8192` razonable en CPU; subí solo si tu caso lo pide. |
| **`--ttl`** | flag de `lms load` | *Keep-alive*: segundos de inactividad antes de descargar el modelo. | `--ttl=0` en producción = **siempre cargado** (sin cold-start por request). |
| **`--cors`** | flag de `lms server start` | Habilita CORS (si lo llama un navegador/SPA). | Activalo si hay frontend web directo. |
| **Variables de entorno** | `-e` / `environment:` | Inyectar `CHAT_MODEL`, `EMBED_MODEL`, etc. al entrypoint. | Parametrizá el modelo por env, no lo hardcodees en la imagen. |
| **Volumen de modelos** | `-v vol:/root/.lmstudio` | Persistir descargas entre recreaciones. | Siempre, salvo que bakees todo. |
| **Healthcheck** | `HEALTHCHECK` / `healthcheck:` | `curl` a `/v1/models`: marca *healthy* cuando la API responde. | `start-period` largo (≥120s): cargar el modelo tarda. |

Ejemplo de `docker-compose.yml` (camino A, CPU-only, con el vector store del RAG; el esquema y el *seed* del store se detallan en §11.5):

```yaml
services:
  llmster:
    build: .                       # usa el Dockerfile de §11.3
    image: mi-llmster:qwen3-8b
    ports:
      - "1234:1234"
    environment:
      CHAT_MODEL: "qwen3-8b"
      EMBED_MODEL: "nomic-embed-text-v1.5"
      CONTEXT_LENGTH: "8192"
      TTL: "0"
    mem_limit: 12g                 # 8B Q4_K_M (~5 GB) + KV-cache + headroom
    cpus: "6.0"
    volumes:
      - lmstudio-models:/root/.lmstudio
    healthcheck:
      test: ["CMD", "curl", "-fsS", "http://localhost:1234/v1/models"]
      interval: 30s
      timeout: 5s
      start_period: 120s
      retries: 5
    restart: unless-stopped

  qdrant:                          # vector store del RAG (esquema/seed en §11.5)
    image: qdrant/qdrant
    ports:
      - "6333:6333"                # REST / dashboard
      - "6334:6334"                # gRPC (upserts masivos del ingestor)
    volumes:
      - qdrant-data:/qdrant/storage         # la KB persiste acá
      - ./seed/snapshots:/qdrant/snapshots  # patrón "seed" (§11.5)
    restart: unless-stopped

  # ingestor: job de indexación (one-shot). Se corre con
  # `docker compose run --rm ingestor` cuando cambian los docs. Ver §11.5.

volumes:
  lmstudio-models:
  qdrant-data:
```

Y el bloque que **agregás** al servicio del runtime en el **camino B con GPU** (con `llama-server`, no LM Studio):

```yaml
  llama-server:
    image: ghcr.io/ggml-org/llama.cpp:server-cuda
    command: >
      -m /models/qwen3-30b-a3b-q4_k_m.gguf
      --host 0.0.0.0 --port 8080
      --n-gpu-layers 99 --ctx-size 16384 --parallel 4
    ports:
      - "8080:8080"
    volumes:
      - ./models:/models
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    # equivalente en `docker run`: --gpus all
```

**Recomendación concreta de modelo + cuantización + contexto** (apoyada en la [§4](#4-cómo-elegir-el-modelo-según-el-hardware) y el [Anexo A](#anexo-a--tabla-maestra-de-modelos-de-referencia)):

| Escenario | Modelo de chat | Cuant. | Context | Embeddings | RAM/VRAM aprox. | Por qué |
|---|---|---|---|---|---|---|
| **CPU-only modesto** (≤16 GB) | **Qwen3 8B** | Q4_K_M | 8192 | nomic-embed-text-v1.5 | ~5 GB + ctx | Tool calling fuerte, entra holgado, lento pero usable en CPU. |
| **CPU-only con RAM** (32 GB) | **gpt-oss-20b** (MoE) o **Qwen3 30B-A3B** (MoE) | Q4_K_M | 8192 | nomic-embed-text-v1.5 | ~12–18 GB + ctx | Los MoE **activan pocos parámetros por token** → rinden como uno chico en velocidad. Ideal para CPU. |
| **GPU producción** (24 GB VRAM) | **Qwen3 30B-A3B** (MoE) o **Qwen3 32B** | Q4_K_M | 16384 | nomic-embed-text-v1.5 / bge-m3 | ~18–19 GB + ctx | Con `llama-server`/Ollama y `--gpus all`, contexto más largo y batching real. |

> En CPU-only, los **MoE** (gpt-oss-20b, Qwen3 30B-A3B) son la jugada inteligente: tenés el conocimiento de un modelo grande con la velocidad de uno chico. Para chatbots con **herramientas/MCP**, priorizá *instruct* con buen tool calling (gpt-oss, Qwen3, Llama 3.x).

### 11.5 La base de conocimiento (RAG) en producción

Hasta acá la base de conocimiento la armábamos "a mano" desde la GUI ([§7](#7-agregar-una-base-de-conocimiento-rag)) o con una `List` en memoria desde .NET ([§9.7](#97--rag-propio-completo-desde-net)). Eso sirve para entender y prototipar, pero **no sobrevive a un reinicio del contenedor ni escala**. En un despliegue dockerizado, la KB deja de ser una variable en RAM y pasa a ser **un servicio más del stack**: un vector store persistente, poblado por un proceso de ingesta separado.

> **Idea central que NO hay que perder de vista:** el modelo de chat **nunca "aprende" tus documentos**. No hay fine-tuning ni reentrenamiento. La base de conocimiento vive **íntegramente en el vector store**; el LLM solo recibe, en cada pregunta, los fragmentos relevantes pegados en el prompt. Cambiar de modelo de chat no te hace perder la KB; borrar el volumen del vector store, sí.

#### Las dos fases en un despliegue real

En la GUI las dos fases de RAG ([§7](#7-agregar-una-base-de-conocimiento-rag)) ocurren juntas y ocultas. En producción se **separan físicamente** en dos procesos con ciclos de vida distintos:

| Fase | Cuándo corre | Quién la ejecuta | Qué hace |
|---|---|---|---|
| **Ingesta / indexación** | **Offline.** Una vez, o cada vez que cambian los documentos. | Un contenedor **"ingestor"** (job batch, no expuesto). | Lee docs → chunking → pide embeddings al runtime → **persiste** en el vector store. |
| **Consulta** | **Online, en caliente.** En cada request del usuario. | La **API .NET** (servicio long-running). | Embeddea la pregunta → recupera top-K → arma prompt con contexto → llama al chat model. |

**Fase de ingesta (el contenedor "ingestor").** Es un proceso que arranca, hace su trabajo y **termina** (no es un servidor). Su pipeline:

```text
documentos (PDF/MD/TXT en un volumen o bucket)
   → parser (extrae texto plano)
   → chunking (≈500 tokens, overlap ≈100)
   → POST /v1/embeddings  (model = nomic-embed-text-v1.5)   ← contra el runtime
   → upsert en el vector store  (vector + texto + metadata de la fuente)
```

Clave: el ingestor le pega al **mismo `/v1/embeddings`** que usa la fase de consulta (mismo modelo, misma dimensión — si no, los vectores no son comparables). El runtime (llmster/Ollama/llama-server, ver [§10](#10-dockerizar--despliegue-headless)) tiene que tener el **embeddings model cargado** antes de correr el ingestor.

**Fase de consulta (la API .NET).** Es exactamente la cadena de la [§9.7](#97--rag-propio-completo-desde-net) / [§9.8](#98--chatbot-con-historial-en-aspnet-core), pero con el store en memoria reemplazado por el conector del vector store real. No la repetimos acá: el código de embeddear la pregunta, recuperar top-K y armar el prompt es **idéntico**; solo cambia de dónde salen los chunks (de `IVectorStore` en vez de una `List`).

> En criollo: **el ingestor escribe la KB; la API la lee.** Pueden escalar por separado (la API a N réplicas; el ingestor corre cuando hace falta) y comparten un único punto de verdad: el vector store.

#### Elección y operación del vector store en contenedor

| Vector store | Dockerizar | Escala | Cuándo usarlo |
|---|---|---|---|
| **SQLite + sqlite-vec** | Trivial (es un archivo, no hay servicio). | Baja-media. Single-node, sin concurrencia de escritura. | KB chica/embebida, una sola instancia de API, demos, edge. |
| **Postgres + pgvector** | Fácil (imagen `pgvector/pgvector`). Si ya tenés Postgres, es **un `CREATE EXTENSION`**. | Media-alta. Transaccional, joins con tus datos relacionales. | Cuando ya usás Postgres y querés la KB **junto** a tus datos de negocio. Índices `hnsw` o `ivfflat`. |
| **Qdrant** | Fácil (imagen `qdrant/qdrant`, sin tuning). | Alta. Diseñado para vectores, HNSW nativo, filtros por metadata rápidos, **snapshots**. | KB grande, filtrado pesado por metadata, o cuando el RAG es el caso de uso principal. |

> **Recomendación por defecto para este escenario:** **Qdrant** si la KB es el corazón del sistema (HNSW nativo, snapshots para seed, escala sin dolor), o **Postgres + pgvector** si ya tenés Postgres en el stack (un servicio menos que operar). Para una KB chica de una sola instancia, **SQLite + sqlite-vec** es lo más simple. Los tres tienen conector para **`Microsoft.Extensions.VectorData`**, así que el código .NET cambia poco entre ellos.

**El servicio del vector store ya está en el `docker-compose` de §11.4** (el contenedor `qdrant`, con su volumen `qdrant-data` para `/qdrant/storage` y el bind de `snapshots` para el patrón *seed*). Si preferís **Postgres + pgvector** en vez de Qdrant, reemplazá ese servicio por:

```yaml
services:
  pgvector:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_DB: kb
      POSTGRES_USER: rag
      POSTGRES_PASSWORD: ${PG_PASSWORD}
    ports:
      - "5432:5432"
    volumes:
      - pg-data:/var/lib/postgresql/data
      - ./seed/initdb:/docker-entrypoint-initdb.d  # *.sql de seed corren al primer arranque
    restart: unless-stopped

volumes:
  pg-data:
```

**Esquema de la colección / tabla.** Los campos mínimos son los mismos en cualquier store:

| Campo | Tipo | Para qué |
|---|---|---|
| `id` | UUID / int | Clave única del chunk. |
| `text` | texto | El contenido del chunk (lo que se inyecta en el prompt). |
| `embedding` / `vector` | vector(768) | El embedding del chunk (dimensión de nomic-embed-text-v1.5). |
| `source` | texto | Archivo/URL de origen (para citar la fuente en la respuesta). |
| `chunk_index` / `page` | int | Posición dentro del documento (trazabilidad). |
| `metadata` | json | Extra: sección, fecha, versión del doc, tags para filtrar. |

En **pgvector** eso es una tabla con índice vectorial:

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE kb_chunks (
    id          BIGSERIAL PRIMARY KEY,
    text        TEXT        NOT NULL,
    embedding   vector(768) NOT NULL,          -- 768 = nomic-embed-text-v1.5
    source      TEXT,
    chunk_index INT,
    metadata    JSONB
);

-- Índice HNSW con métrica coseno (vector_cosine_ops):
CREATE INDEX ON kb_chunks USING hnsw (embedding vector_cosine_ops);
```

En **Qdrant**, la colección equivalente:

```bash
curl -X PUT http://localhost:6333/collections/kb_chunks \
  -H "Content-Type: application/json" \
  -d '{
    "vectors": { "size": 768, "distance": "Cosine" }
  }'
# id, text, source, chunk_index y metadata van en el "payload" de cada punto.
```

#### Persistencia, versionado y KB pre-cargada (seed)

**Cómo persiste.** La KB vive en el **volumen del vector store** (`qdrant-data` o `pg-data`). Mientras el volumen exista, la KB sobrevive a reinicios, actualizaciones de imagen y recreación del contenedor. **Sin volumen, la KB se evapora en cada `docker compose down`** — el error clásico.

**Cómo se versiona / re-indexa.** La KB es un derivado de tus documentos + el modelo de embeddings. Reglas:

- Si **cambian los documentos**: corré el ingestor de nuevo. Hacé **upsert por `id` estable** (p. ej. hash del archivo + `chunk_index`) para no duplicar; borrá los chunks de las fuentes que desaparecieron.
- Si **cambiás el modelo de embeddings** (o su dimensión): hay que **re-indexar todo**. Vectores de modelos distintos no son comparables. Estrategia segura: indexar en una **colección/tabla nueva** (`kb_chunks_v2`) y hacer el switch atómico cuando termina.
- Versioná la KB con un campo `kb_version` en metadata o en el nombre de la colección, así podés rollbackear.

**Patrón "KB pre-cargada / seed".** Para que el stack levante con la base **ya poblada** (sin esperar a que corra el ingestor), embebés un snapshot/dump y lo restaurás al primer arranque:

- **Qdrant — snapshot.** Generás un snapshot de la colección ya indexada y lo montás. Al levantar, restaurás:

  ```bash
  # 1) Generar snapshot (en el entorno donde ya indexaste)
  curl -X POST http://localhost:6333/collections/kb_chunks/snapshots

  # 2) Lo dejás en ./seed/snapshots (montado en /qdrant/snapshots) y restaurás al arrancar:
  curl -X PUT \
    "http://localhost:6333/collections/kb_chunks/snapshots/recover" \
    -H "Content-Type: application/json" \
    -d '{ "location": "file:///qdrant/snapshots/kb_chunks.snapshot" }'
  ```

- **pgvector — dump SQL.** Dejás un `pg_dump` (o un `.sql` con los `INSERT`) en `/docker-entrypoint-initdb.d`: Postgres lo ejecuta automáticamente **solo en el primer arranque** (cuando el volumen está vacío). Útil para CI y para arrancar entornos nuevos con la KB lista.

> El seed te da arranques **reproducibles y rápidos**: el contenedor levanta con la KB del último build, sin depender del runtime de embeddings ni de los documentos fuente en ese momento. La ingesta incremental queda para las actualizaciones.

#### Dimensiones y parámetros prácticos

`nomic-embed-text-v1.5` produce vectores de **768 dimensiones** (está entrenado con *Matryoshka Representation Learning*, así que se puede **truncar** a 512/256/128/64 para ahorrar memoria con poca pérdida — pero si truncás, **declará la dimensión truncada en el esquema** y aplicala consistente en ingesta y consulta). Soporta hasta **8192 tokens** de contexto por entrada, así que un chunk de ~500 tokens le sobra.

| Parámetro | Valor recomendado | Por qué |
|---|---|---|
| **Modelo de embeddings** | `nomic-embed-text-v1.5` | Liviano, CPU-friendly, OpenAI-compatible vía `/v1/embeddings`. |
| **Dimensión del vector** | **768** (truncable a 512/256/…) | Default del modelo; la columna/colección se define con este tamaño. |
| **Métrica de distancia** | **Coseno** | Estándar para embeddings de texto; insensible a la magnitud. |
| **Índice** | **HNSW** (Qdrant nativo, pgvector `hnsw`) | Mejor recall/latencia en consulta. `ivfflat` (pgvector) si la KB es enorme y priorizás memoria/tiempo de build. |
| **Tamaño de chunk** | **~500 tokens** | Equilibrio entre precisión de recuperación y contexto suficiente. |
| **Overlap** | **~100 tokens** | Evita cortar ideas justo en el borde de dos chunks. |
| **Top-K** | **4–8** | Suficiente contexto sin saturar la ventana del chat model ni meter ruido. |

> Nota sobre nomic: el modelo usa **prefijos de tarea** (`search_document:` al indexar, `search_query:` al consultar). Si tu cliente/conector no los agrega solo, prependelos vos en ingesta y en consulta — mejora la calidad de la recuperación.

#### Pseudocódigo del flujo de ingesta

```text
ENTRADA: carpeta/volumen con documentos, runtime con embeddings model cargado,
         vector store levantado y colección creada (size=768, Cosine)

para cada documento en la fuente:
    texto   = extraerTexto(documento)              # parser PDF/MD/TXT
    chunks  = partir(texto, tamaño=500, overlap=100)
    lote    = []
    para cada (i, chunk) en chunks:
        # search_document: => prefijo de tarea de nomic al indexar
        vec = POST /v1/embeddings(model="nomic-embed-text-v1.5",
                                  input="search_document: " + chunk)   # 768 dims
        id  = hash(documento.ruta) + ":" + i        # id estable => upsert idempotente
        lote.añadir({ id, text: chunk, embedding: vec,
                      source: documento.ruta, chunk_index: i, metadata: {...} })
    vectorStore.UPSERT(lote)                         # idempotente: re-correr no duplica

al terminar:
    borrar del store los chunks cuya 'source' ya no existe   # limpieza de docs eliminados
    (opcional) generar snapshot/dump => artefacto de "seed"
SALIDA: el ingestor TERMINA (no es un servidor)
```

Del lado .NET, el ingestor usa **`Microsoft.Extensions.VectorData`** (`IVectorStore`) + el `IEmbeddingGenerator` de la [§9.5](#95--microsoftextensionsai-la-abstracción-recomendada) — no la `List` en memoria de la [§9.7](#97--rag-propio-completo-desde-net). Fragmento del lado ingesta (el modelo del registro + el upsert; la generación de embeddings y el chunking son la misma cadena de la §7/§9.7):

```csharp
// dotnet add package Microsoft.Extensions.VectorData.Abstractions
// dotnet add package <conector>   (Qdrant.Client / Npgsql + pgvector / sqlite-vec)
using Microsoft.Extensions.VectorData;

// 1) El registro mapea 1:1 al esquema de la tabla/colección
sealed class KbChunk
{
    [VectorStoreKey]                                   public ulong Id { get; set; }
    [VectorStoreData]                                  public string Text { get; set; } = "";
    [VectorStoreData(IsIndexed = true)]                public string Source { get; set; } = "";
    [VectorStoreData]                                  public int ChunkIndex { get; set; }
    [VectorStoreVector(768, DistanceFunction = DistanceFunction.CosineSimilarity)]
    public ReadOnlyMemory<float> Embedding { get; set; }   // 768 = nomic-embed-text-v1.5
}

// 2) collection viene del conector (Qdrant/pgvector); embedder es el IEmbeddingGenerator de §9.5
await collection.EnsureCollectionExistsAsync();

foreach (var (chunk, i) in Chunk(text, size: 500, overlap: 100).Select((c, i) => (c, i)))
{
    var vec = await embedder.GenerateAsync($"search_document: {chunk}");   // prefijo nomic
    await collection.UpsertAsync(new KbChunk
    {
        Id = StableId(path, i),          // id estable => idempotente, no duplica al re-indexar
        Text = chunk, Source = path, ChunkIndex = i,
        Embedding = vec.Vector
    });
}
```

> La fase de **consulta** usa el mismo `collection` con `collection.SearchAsync(queryVector, top: K)` y de ahí en más es la cadena de la [§9.7](#97--rag-propio-completo-desde-net)/[§9.8](#98--chatbot-con-historial-en-aspnet-core) (armar prompt con el contexto + `IChatClient`). Recordá el prefijo `search_query:` al embeddear la pregunta.

#### Preparar la KB de punta a punta

1. **Cargá el embeddings model en el runtime.** Contra el contenedor del runtime ([§10](#10-dockerizar--despliegue-headless)): `lms get nomic-embed-text-v1.5 --host <runtime> --port 1234` y `lms load nomic-embed-text-v1.5 --host <runtime> --port 1234`. Verificá con `GET /v1/models` que aparezca. Puede convivir con el chat model en el mismo runtime, o ir en un runtime separado.
2. **Levantá el vector store y creá la colección/tabla.** `docker compose up -d qdrant` (o `pgvector`), luego creá la colección `kb_chunks` con `size=768`, `Cosine` e índice HNSW (curl o el `EnsureCollectionExistsAsync()` del ingestor).
3. **Corré el contenedor ingestor.** Apunta sus documentos (volumen/bucket), el `/v1/embeddings` del runtime y el vector store. Hace chunking → embeddings → upsert y **termina**. (Es un job batch, no un servicio.)
4. **Verificá el vector store.** Que la colección tenga puntos: en Qdrant `GET /collections/kb_chunks` (mirá `points_count`); en pgvector `SELECT count(*) FROM kb_chunks;`. Probá una búsqueda con un embedding de prueba y revisá que los top-K tengan sentido.
5. **(Opcional) Generá el snapshot/dump de seed.** Para que entornos nuevos arranquen con la KB ya poblada (snapshot de Qdrant o `pg_dump`).
6. **Conectá la API .NET.** Configurá en `appsettings.json` el endpoint del vector store y el `EmbeddingModel` (ya está en la config de la [§9.4](#94--configuración-e-inyección-de-dependencias-base-de-una-app-real)), registrá el `IVectorStore` y el `IEmbeddingGenerator` en DI, y levantá la API. A partir de acá la fase de consulta es la cadena de la [§9.7](#97--rag-propio-completo-desde-net)/[§9.8](#98--chatbot-con-historial-en-aspnet-core).

> **Regla de oro de operación:** el ingestor y la API **comparten el modelo de embeddings y la dimensión**. Si cambiás uno, re-indexás. La KB es persistente y versionable; el chat model es intercambiable y no guarda nada.

*Fuentes: [nomic-embed-text-v1.5 (Hugging Face)](https://huggingface.co/nomic-ai/nomic-embed-text-v1.5), [Nomic Embed Matryoshka](https://www.nomic.ai/news/nomic-embed-matryoshka), [LM Studio — text embeddings](https://lmstudio.ai/docs/text-embeddings), [Qdrant — instalación y Docker](https://qdrant.tech/documentation/guides/installation/), [Qdrant — snapshots](https://qdrant.tech/documentation/concepts/snapshots/), [pgvector](https://github.com/pgvector/pgvector), [Microsoft.Extensions.VectorData](https://learn.microsoft.com/en-us/dotnet/ai/microsoft-extensions-vector-data), [sqlite-vec](https://github.com/asg017/sqlite-vec).*

### 11.6 Capacity planning: ¿cuántos usuarios aguanta?

Antes de meter LM Studio en un escenario de producción tenés que poder responder dos preguntas con números (aunque sean aproximados): **¿cuánto tarda una consulta?** y **¿cuántas consultas en paralelo aguanto?** Esta parte te da el modelo mental, los supuestos y unas tablas para dimensionar.

> ⚠️ **El elefante en la habitación:** como vimos en la [§6](#6-el-servidor-de-api-local), el servidor de LM Studio está pensado como **single-user / un proceso a la vez**. No está documentado como servidor multiusuario concurrente: por defecto **procesa las requests en serie**. O sea, si te entran 10 requests juntas, la 10ma espera a que terminen las 9 anteriores. **La "concurrencia real" de una sola instancia de LM Studio es ≈ 1.** Todo lo que sigue parte de esa verdad incómoda: para subir concurrencia necesitás **poner una cola adelante**, **levantar varias réplicas** o **migrar a un motor con batching** (llama.cpp server, vLLM, Ollama). Lo desarrollamos abajo.

#### Modelo mental del cálculo

La latencia de **una** consulta se descompone en dos tiempos:

```
latencia ≈ time_to_first_token (TTFT)  +  (tokens_de_salida ÷ tokens_por_segundo)
            └── procesar el prompt ──┘     └──────── generación ────────┘
```

- **TTFT (time-to-first-token):** lo que tarda en *digerir* el prompt (system + historial + contexto RAG) antes de escupir el primer token. Crece con la longitud del prompt. En contextos RAG largos (1–2K tokens) puede pesar bastante. *Ojo:* con un modelo recién cargado, el TTFT del primer request **también incluye la carga del modelo a VRAM/RAM** (segundos a decenas de segundos) — por eso conviene tener el modelo precargado y con un `--ttl` alto.
- **Generación:** los tokens de respuesta salen a un ritmo de `tokens_por_segundo` (tok/s), que depende casi todo del **hardware + tamaño del modelo + cuantización**.

Y el throughput **agregado** (lo que importa para "cuántos usuarios"):

```
throughput_agregado = depende de si hay batching
   • SIN batching (LM Studio en serie):  ≈ throughput de UNA request (la siguiente espera)
   • CON batching (vLLM/llama.cpp/Ollama): suma muchas requests en paralelo en la misma GPU
```

Términos que vas a ver en las tablas:

> **Throughput:** cuánto trabajo despacha el sistema por unidad de tiempo. Se mide en **tokens/seg** (a nivel motor) o en **consultas/min** (a nivel servicio).
>
> **Tokens/seg (tok/s):** velocidad de generación. La métrica reina del rendimiento de inferencia. "Generación" (decode) y "prompt processing" (prefill) tienen tok/s distintos; el prefill suele ser más rápido por token.
>
> **Latencia p50 / p95:** tiempo de respuesta. El **p50** (mediana) es el caso típico; el **p95** es "el 95% de las requests tardan menos que esto". En sistemas en serie, cuando se forma cola **el p95 se dispara** aunque el p50 se vea bien — es la trampa clásica de medir solo el promedio.
>
> **Concurrencia:** cuántas requests se están procesando **al mismo tiempo**. En LM Studio crudo ≈ 1. Con batching, decenas o cientos.
>
> **RPS / RPM / QPS:** *requests (o queries) per second / per minute*. La tasa de llegada de consultas. `RPM = RPS × 60`. "QPS" y "RPS" son lo mismo acá.

#### Supuestos explícitos (leelos antes de creerte las tablas)

Los números de abajo son **estimaciones de orden de magnitud**, NO benchmarks que corrí en tu máquina. Sirven para dimensionar y discutir, no para un SLA. Dependen muchísimo del hardware exacto, la cuantización, el largo del contexto y la versión del runtime. **Medí en tu hardware con `lms log stream` o un script de carga antes de comprometer nada.**

Supuestos usados:

| Supuesto | Valor asumido |
|---|---|
| **Longitud media de respuesta** | **400 tokens** (rango 300–500). |
| **Prompt + contexto** | **~1.5K tokens** (system + historial corto + 2–4 chunks de RAG). |
| **Cuantización** | **Q4_K_M** en todos los modelos. |
| **TTFT** | absorbido dentro de la latencia (estimado por el prefill del prompt de ~1.5K tok). |
| **Modo serie (LM Studio)** | 1 instancia procesa 1 request por vez; el resto espera. |
| **Modo batching (vLLM/llama.cpp/Ollama)** | varias requests comparten la GPU; el tok/s **por request** baja un poco, pero el **agregado** sube fuerte. |
| **Latencia objetivo para "usuarios concurrentes razonables"** | que el usuario empiece a ver respuesta rápido (streaming) y la latencia p95 se mantenga **tolerable (≲ 15–20 s)** bajo carga. |
| **"Consultas/día"** | extrapolación lineal de las consultas/min asumiendo uso sostenido; en la práctica el tráfico es a ráfagas, así que tomalo como **techo teórico**, no como demanda real. |

#### Tabla principal — capacidad por hardware × modelo

> Lee las columnas así: **tok/s** = velocidad de generación; **latencia/consulta** = para una respuesta media de 400 tok con prompt de ~1.5K; **consultas/min (serie)** = lo que despacha **1 sola instancia en serie**; **usuarios concurrentes razonables** = cuántos usuarios *activos a la vez* tolera manteniendo el p95 aceptable; **consultas/día** = techo teórico con uso sostenido.

| # | Hardware | Modelo | tok/s aprox. (gen.) | Latencia/consulta (~400 tok) | Consultas/min (1 instancia, **serie**) | Usuarios concurrentes razonables | Consultas/día (techo) | Modo |
|---|---|---|---|---|---|---|---|---|
| 1 | **CPU-only** (imagen `llmster-preview:cpu`, x86, sin GPU) | Llama 3.2 **3B** | ~8–15 | ~30–55 s | ~1–2 | **1** (uso esporádico) | ~1.5–3 K | 🐌 **serie** |
| 2 | **CPU-only** (misma imagen) | Qwen3 **8B** | ~3–7 | ~60–130 s | ~0.5–1 | **1** (apenas) | ~700–1.5 K | 🐌 **serie** |
| 3 | **GPU media** (RTX 4090 / 24 GB) | gpt-oss-**20b** (MoE) | ~50–90 | ~5–9 s | ~7–12 | **3–6** | ~10–17 K | **serie** (LM Studio) |
| 4 | **GPU media** (RTX 4090 / 24 GB) | Qwen3 **30B-A3B** (MoE) | ~120–190 | ~3–5 s | ~12–20 | **5–10** | ~17–28 K | **serie** (LM Studio) |
| 5 | **GPU media** (RTX 4090 / 24 GB) | Qwen3 **8B** denso | ~90–110 | ~4–6 s | ~10–15 | **4–8** | ~14–22 K | **serie** (LM Studio) |
| 6 | **GPU alta / server** (A100 80 GB / H100, 1×) | Llama 3.3 **70B** Q4 | ~30–45 (por request) | ~10–15 s | ~4–6 | **2–4** | ~6–9 K | **serie** (LM Studio) |
| 7 | **GPU alta / server** (A100/H100) **con batching** | Qwen3 8B / gpt-oss-20b | **agregado: cientos–miles tok/s** | ~5–10 s/req (estable) | **~60–200+** | **50–250+** | **cientos de miles** | 🚀 **batching** (vLLM/llama.cpp) |
| 8 | **GPU server multi-GPU** (4× A100) **con batching** | Llama 70B Q4/FP8 | **agregado ~2.000+ tok/s** | ~10–20 s/req bajo carga | **~80–150** | **~100–256** | **cientos de miles** | 🚀 **batching** (vLLM) |

**Cómo leer la diferencia clave:** las filas **1–6 asumen LM Studio en serie** — la columna "usuarios concurrentes razonables" NO significa que LM Studio atienda esos N en paralelo, sino cuántos usuarios *activos* podés tener **detrás de una cola** antes de que la espera p95 se vuelva intolerable (porque cada uno consume ~latencia/consulta de tiempo de servicio). Las filas **7–8 asumen un motor con continuous batching**: ahí sí hay concurrencia real dentro de la GPU, y por eso el salto de capacidad es de uno o dos órdenes de magnitud.

> 💡 **Por qué los MoE rompen la tabla a su favor:** *gpt-oss-20b* y *Qwen3 30B-A3B* activan solo ~3B de params por token, así que generan a velocidad de un modelo chico pero responden con calidad de uno grande. Para un escenario de producción con GPU media son, casi siempre, **el mejor punto de equilibrio velocidad/calidad/concurrencia**.

> 🐌 **Por qué la fila CPU es tan floja:** la imagen Docker oficial es **CPU-only preview** ([§10](#10-dockerizar--despliegue-headless)). Sin GPU, un 8B se arrastra a pocos tok/s y una sola respuesta puede tardar **uno o dos minutos**. Sirve para dev/CI o un bot interno de bajísimo tráfico — **no** para servir usuarios concurrentes. Para producción con concurrencia: GPU + (idealmente) un motor con batching.

#### Cómo escalar la concurrencia (las palancas)

Como una instancia de LM Studio ≈ 1 request a la vez, subir concurrencia es **arquitectura alrededor del modelo**, no un flag mágico:

| Palanca | Qué hace | Cuánto sube la concurrencia | Costo / contra |
|---|---|---|---|
| **Cola + worker con backpressure** delante de LM Studio | Encolás requests y las despachás de a una; rechazás/diferís cuando la cola se llena. | No sube el throughput, pero **evita el colapso** y te da un p95 predecible. | El usuario espera en cola; necesitás UX de "procesando…". |
| **N réplicas del contenedor + balanceador** | N instancias = **N requests en paralelo**. Round-robin / least-connections adelante. | **×N lineal** (con N GPUs o N slots de CPU). | N× hardware/VRAM; cada réplica carga su copia del modelo. |
| **Migrar a motor con continuous batching** (vLLM, llama.cpp `--parallel`, Ollama `OLLAMA_NUM_PARALLEL`) | Mete varias requests en el mismo paso de GPU. | **Decenas a cientos** en una sola GPU (la vía más eficiente en hardware). | Salís de LM Studio para servir; más tuning. (Seguís usándolo para dev — siguen siendo OpenAI-compatible.) |
| **Modelo más chico / MoE** | Menos cómputo por token → más tok/s → cada request libera el slot antes. | Sube RPM por instancia (efecto multiplicador sobre todo lo demás). | Algo menos de calidad (mitigable con buen RAG/prompting). |
| **Limitar `max_tokens`** | Acotás la respuesta (p. ej. 300 en vez de 800). | Baja la latencia/consulta → más RPM. | Respuestas más cortas; cuidá no truncar lo útil. |
| **Cache de respuestas / embeddings** | Reusás resultados de prompts repetidos y embeddings ya calculados (RAG). | Las requests cacheadas cuestan ~0 → "concurrencia gratis" en lo repetido. | Solo ayuda si hay repetición; invalidación a tu cargo. |

Fórmula simple para combinar las dos palancas de escala horizontal:

```
usuarios_soportados ≈ (RPM_por_instancia × instancias) ÷ RPM_por_usuario
```

Donde `RPM_por_instancia` lo sacás de la tabla principal (columna "consultas/min, serie") y `RPM_por_usuario` es cuántas consultas hace un usuario activo por minuto (ojo: un usuario "interactivo" rara vez supera **1–2 consultas/min**, porque también lee y piensa).

#### Regla práctica de dimensionamiento

El insumo que vas a tener del negocio suele ser: **"espero X usuarios activos que hacen Y consultas/hora"**. Convertilo y compará contra una instancia:

```
1) Demanda pico (RPM) ≈ X_usuarios_activos × (Y_consultas_hora ÷ 60) × factor_pico
        (usá factor_pico ≈ 2–3: el tráfico no es uniforme, llega a ráfagas)

2) Capacidad de 1 instancia (RPM) = de la tabla principal (col. "consultas/min, serie")

3) Instancias ≈ ceil( Demanda_pico_RPM ÷ Capacidad_por_instancia_RPM )
        Si "Instancias" empieza a ser grande (≳ 4–6) → migrá a batching en vez de apilar réplicas.
```

**Ejemplo A — Equipo interno chico (lo que más te va a tocar).** *"20 usuarios internos, consultas esporádicas, ~3 consultas/hora cada uno."*

- Demanda media = 20 × (3 ÷ 60) = **1 RPM**. Con factor pico ×3 ≈ **3 RPM**.
- 1 RTX 4090 con **Qwen3 30B-A3B** (fila 4) da **~12–20 RPM**.
- **Veredicto:** **1 sola instancia GPU media alcanza y sobra.** Ni hace falta batching. Poné una **cola simple** por si dos personas mandan justo al mismo tiempo, y listo. (Si todo es CPU-only, sería ajustado pero usable para uso tan esporádico — con paciencia.)

**Ejemplo B — Chatbot de soporte de tráfico medio.** *"~150 usuarios concurrentes en hora pico, 1 consulta/min cada uno."*

- Demanda pico ≈ 150 × 1 = **150 RPM**.
- 1 RTX 4090 con gpt-oss-20b en serie (fila 3) da **~7–12 RPM** → harían falta **~13–20 réplicas en serie**. Inviable por costo.
- **Veredicto:** **migrá a continuous batching.** Una o dos GPUs con **vLLM** sirviendo un 8B/20B (fila 7) absorben 150 RPM cómodas con buena latencia. Acá LM Studio en serie ya no es la herramienta; usalo para *desarrollar* y desplegá con vLLM/llama.cpp server (que siguen siendo OpenAI-compatible, así que **tu código .NET no cambia** — solo el `Endpoint`).

**Ejemplo C — API pública de alto volumen.** *"API pública, objetivo 1000 req/min sostenidas."*

- Demanda = **1000 RPM** sostenidas (≈ 17 RPS).
- Imposible en serie (serían **~80–140 instancias** de LM Studio).
- **Veredicto:** **batching obligatorio + escalado horizontal.** vLLM con continuous batching en GPUs server (A100/H100), **N réplicas detrás de un balanceador** (fila 7/8), autoscaling por profundidad de cola, cache de respuestas/embeddings para lo repetido, y `max_tokens` acotado. LM Studio queda **fuera del path de producción** (no fue diseñado para esto); su lugar es el laboratorio donde prototipás y validás el modelo/prompt antes de portarlo al motor de serving.

> **La síntesis honesta:** LM Studio es ideal para **desarrollar, prototipar y servir poquitos usuarios internos con GPU**. Para concurrencia real (decenas/cientos de usuarios o una API pública) la respuesta no es "afinar LM Studio", es **poner una cola + réplicas** o, mejor, **mover el serving a un motor con continuous batching**. Como todos son OpenAI-compatible, el salto te cuesta cambiar una URL, no reescribir la app .NET.

*Fuentes (cifras de throughput como referencia de orden de magnitud, verificadas 2026-06-14): [Home GPU LLM Leaderboard](https://awesomeagents.ai/leaderboards/home-gpu-llm-leaderboard/), [hardware-corner GPU ranking for local LLMs](https://www.hardware-corner.net/gpu-ranking-local-llm/), [SitePoint M3 Max vs RTX 4090 (2026)](https://www.sitepoint.com/mac-m3-max-vs-rtx-4090-local-llm-performance-showdown-2026/), [vLLM PagedAttention & continuous batching (RunPod)](https://www.runpod.io/articles/guides/vllm-pagedattention-continuous-batching), [vLLM throughput guide 2026](https://blog.easecloud.io/ai-cloud/increase-throughput-with-vllm-serving/).*

### 11.7 Cómo armarlo de punta a punta

1. **Elegí el camino** según hardware: A (`llmster-preview:cpu`) para CPU/CI/preview, o B (`llama-server`/Ollama con CUDA) si tenés GPU. *(§11.2)*
2. **Elegí modelo + cuantización + context-length** con la tabla de §11.4 y validá memoria con `lms load --estimate-only` (o el cálculo de la [§4](#4-cómo-elegir-el-modelo-según-el-hardware)).
3. **Escribí el `Dockerfile`** extendiendo la base, decidiendo **bakeado vs volumen** para el modelo, y el **`entrypoint.sh`** (`lms server start --cors` → esperar `/v1/models` → `lms get` → `lms load`). *(§11.3)*
4. **Agregá el `HEALTHCHECK`** (`curl` a `/v1/models`) con `start-period` largo, para que el orquestador no rote tráfico antes de que el modelo esté cargado.
5. **Definí el `docker-compose`** con los tres contenedores: runtime, vector store (Qdrant/pgvector) y el job de **ingesta**; seteá `mem_limit`/`cpus` (y `--gpus all` en camino B) y los **volúmenes** persistentes. *(§11.4)*
6. **Buildeá y levantá:** `docker compose build && docker compose up -d`. Verificá: `curl http://localhost:1234/v1/models` debe listar tus modelos.
7. **Corré la ingesta una vez** para indexar tus documentos en el vector store (chunking → `/v1/embeddings` → upsert). El detalle paso a paso de la KB está en §11.5.
8. **Apuntá tu app .NET** al runtime cambiando solo `Endpoint` en `appsettings.json` (camino A: `…:1234/v1`; B: `…:8080/v1` o `…:11434/v1`). El resto del código de la [§9](#9-consumir-el-modelo-local-desde-net) queda igual.
9. **Poné un reverse proxy** (nginx/Caddy/Traefik) delante con **TLS + auth + rate-limit** antes de exponer fuera de la red interna.
10. **Dimensioná** la cantidad de instancias/hardware con la tabla y la regla de §11.6 según los usuarios/consultas que esperás.

> 🔒 **Seguridad (no negociable):** el runtime **no trae autenticación por defecto** y bindea `127.0.0.1`. Si lo exponés en red (`--bind 0.0.0.0` o publicando el puerto), **activá los API tokens** (Developer Page → Server Settings, opcionales desde 0.4.0) y mandá `Authorization: Bearer <token>` desde .NET. Sin auth, cualquiera en la red usa tu modelo. Lo más seguro: **no publicar el puerto del runtime**, dejarlo solo en la red interna de Docker, y exponer únicamente el **reverse proxy** con TLS y autenticación propia. Y si conectás MCP, recordá la advertencia de la [§8](#8-integración-con-mcp-model-context-protocol): los servidores MCP pueden ejecutar código arbitrario.

*Fuentes: [lmstudio-vs-llmster-vs-lms](https://lmstudio.ai/docs/app/basics/lmstudio-vs-llmster-vs-lms), [headless_llmster](https://lmstudio.ai/docs/developer/core/headless_llmster), [hub.docker.com/r/lmstudio/llmster-preview](https://hub.docker.com/r/lmstudio/llmster-preview), [cli/server-start](https://lmstudio.ai/docs/cli/server-start), [authentication](https://lmstudio.ai/docs/developer/core/authentication), [ollama docker](https://docs.ollama.com/docker), [llama.cpp docker](https://github.com/ggml-org/llama.cpp/blob/master/docs/docker.md).*

---

## 12. Resumen y ruta de aprendizaje recomendada

Checklist progresivo para llegar de cero a un chatbot .NET con RAG y herramientas:

- [ ] **1. GUI:** instalá LM Studio, descargá un modelo chico (Q4_K_M) y hacé tu primer chat. *(§2, §5)*
- [ ] **2. Modelo según hardware:** elegí el tamaño y la cuantización que entra cómodo en tu RAM/VRAM. Para tools, modelo *instruct* con buen tool calling. *(§4, Anexo A)*
- [ ] **3. Servidor API:** `lms server start --port 1234` y probá un `curl` a `/v1/chat/completions`. *(§6)*
- [ ] **4. Consumo desde .NET:** apuntá `HttpClient`/SDK OpenAI a `localhost:1234`, primero sin streaming, después con streaming. *(§9)*
- [ ] **5. RAG:** empezá con el document chat integrado; después armá tu RAG propio con `/v1/embeddings` + un vector store. *(§7)*
- [ ] **6. MCP:** agregá un servidor MCP vía `mcp.json` para darle herramientas al modelo (cuidado con la seguridad). *(§8)*
- [ ] **7. Headless/Docker:** cuando funcione local, evaluá `llmster`/Docker para CPU/CI, o Ollama/llama.cpp si necesitás GPU en producción. *(§10)*
- [ ] **8. Producción:** armá el stack dockerizado (runtime + vector store + ingesta), prepará, persistí y *seedeá* la KB, asegurá con reverse proxy/API tokens y dimensioná usuarios/consultas. *(§11)*

---

# Anexos

## Anexo A — Tabla maestra de modelos de referencia

> Vigentes a junio 2026. Los nombres de familias nuevas cambian seguido — verificá en [lmstudio.ai/models](https://lmstudio.ai/models). RAM/VRAM = referencia para **Q4_K_M** (sumá headroom de contexto).

| Modelo | Parámetros | Formato | RAM/VRAM mín. (Q4_K_M) | Cuant. recomendada | Fortaleza principal |
|---|---|---|---|---|---|
| **Llama 3.2 3B** | 3B | GGUF | ~3 GB | Q4_K_M | Chat general + tool calling chico |
| **Qwen3 4B** | 4B | GGUF | ~3 GB | Q4_K_M | Razonamiento, multilingüe, MCP |
| **Phi-4-mini** | 3.8B | GGUF | ~3 GB | Q4_K_M | Razonamiento/código, function calling |
| **Gemma 3 4B** | 4B | GGUF | ~3 GB | Q4_K_M | Multimodal, 128K contexto |
| **Qwen3 8B** | 8B | GGUF | ~5 GB | Q4_K_M | Chatbot/agentes, tool calling fuerte |
| **Llama 3.1 / 3.3 8B** | 8B | GGUF | ~5 GB | Q4_K_M | Chat general, gran ecosistema |
| **Gemma 3 12B** | 12B | GGUF | ~7–8 GB | Q4_K_M | Razonamiento + multimodal |
| **Phi-4 (full)** | 14B | GGUF | ~8 GB | Q4_K_M | Matemática/ciencia/razonamiento |
| **gpt-oss-20b** | 20B (MoE) | GGUF | ~12–14 GB | Q4_K_M | **Tool use nativo** (browsing, Python) |
| **Devstral 24B** | 24B | GGUF | ~14 GB | Q4_K_M | **Código / agentes de software** |
| **Qwen3 30B-A3B** | 30B/3B act. (MoE) | GGUF | ~18 GB | Q4_K_M | Muy rápido para su capacidad |
| **Qwen3 32B** | 32B | GGUF | ~19 GB | Q4_K_M | Razonamiento líder en su tamaño |
| **Gemma 3 27B** | 27B | GGUF | ~16 GB | Q4_K_M | "Mejor local serio sin multi-GPU" |
| **Llama 3.3 70B** | 70B | GGUF | ~39–42 GB | Q4_K_M | Chat de alta calidad |
| **gpt-oss-120b** | 120B (MoE) | GGUF | VRAM alta | Q4_K_M | Razonamiento de frontera + tool calling |
| **Qwen3 235B-A22B** | 235B/22B act. (MoE) | GGUF | VRAM muy alta | Q4_K_M | Razonamiento top open-source |
| **nomic-embed-text-v1.5** | ~137M | GGUF | <1 GB | — | **Embeddings** (RAG), CPU-friendly |
| **bge-m3** | — | GGUF | ~1 GB | — | **Embeddings** multilingües |

*Fuentes: [lmstudio.ai/models](https://lmstudio.ai/models), [OpenAI gpt-oss](https://openai.com/index/introducing-gpt-oss/), [Qwen3](https://github.com/QwenLM/Qwen3), [nomic-embed v1.5](https://huggingface.co/nomic-ai/nomic-embed-text-v1.5-GGUF).*

## Anexo B — Referencia de comandos `lms`

> Antes de usar `lms`, ejecutá LM Studio (o `llmster`) al menos una vez. Bootstrap del PATH: `~/.lmstudio/bin/lms bootstrap` (Win: `cmd /c %USERPROFILE%/.lmstudio/bin/lms.exe bootstrap`).

| Comando | Descripción | Ejemplo |
|---|---|---|
| `lms get` | Buscar y descargar un modelo | `lms get qwen3-8b` |
| `lms ls` | Listar modelos en disco | `lms ls` |
| `lms ps` | Listar modelos cargados en memoria | `lms ps` |
| `lms load` | Cargar modelo a memoria | `lms load qwen3-8b --gpu=max --context-length=8192` |
| `lms unload` | Descargar modelo(s) | `lms unload --all` |
| `lms import` | Importar un archivo de modelo | `lms import ./modelo.gguf` |
| `lms server start` | Levantar la API | `lms server start --port 1234 --cors` |
| `lms server stop` | Detener la API | `lms server stop` |
| `lms server status` | Estado del servidor | `lms server status` |
| `lms chat` | REPL de chat en terminal | `lms chat` |
| `lms log stream` | Monitorear logs de requests | `lms log stream` |
| `lms daemon up` | Iniciar el daemon headless | `lms daemon up` |
| `lms daemon down` | Detener el daemon | `lms daemon down` |
| `lms runtime` | Gestionar/actualizar runtime | `lms runtime --help` |

> `lms load` acepta `--gpu=max|auto|0.0-1.0`, `--context-length`, `--ttl` y `--estimate-only` (previsualiza requisitos de memoria sin cargar). Ayuda: `lms <cmd> --help`.
>
> *Nota: `lms daemon status`/`update` aparecen en algunas referencias pero no están confirmados textualmente en la doc oficial; sí lo están `up`/`down`.*

*Fuentes: [docs/cli](https://lmstudio.ai/docs/cli), [cli/server-start](https://lmstudio.ai/docs/cli/server-start).*

## Anexo C — Referencia de endpoints de la API

Base URL por defecto: **`http://localhost:1234`**

| Método | Ruta | Propósito | Payload de ejemplo |
|---|---|---|---|
| `GET` | `/v1/models` | Listar modelos | — |
| `POST` | `/v1/chat/completions` | Chat (OpenAI) | `{"model":"qwen3-8b","messages":[{"role":"user","content":"hola"}]}` |
| `POST` | `/v1/completions` | Completado de texto | `{"model":"qwen3-8b","prompt":"Había una vez"}` |
| `POST` | `/v1/embeddings` | Embeddings (RAG) | `{"model":"nomic-embed-text-v1.5","input":"texto"}` |
| `POST` | `/v1/responses` | Responses API (OpenAI) | `{"model":"qwen3-8b","input":"hola"}` |
| `POST` | `/v1/messages` | Chat formato **Anthropic** | `{"model":"qwen3-8b","max_tokens":256,"messages":[{"role":"user","content":"hola"}]}` |
| `GET` | `/api/v0/models` | REST nativa (legacy, 0.3.6+) | — |
| `POST` | `/api/v0/chat/completions` | REST nativa (legacy) | igual que `/v1/chat/completions` |
| `*` | `/api/v1/*` | **REST nativa nueva** (0.4.0+, recomendada) | chats con estado, gestión de modelos, auth |

> Streaming: agregar `"stream": true` al body en los endpoints de chat/completions.

*Fuentes: [openai-compat](https://lmstudio.ai/docs/developer/openai-compat), [rest/endpoints](https://lmstudio.ai/docs/developer/rest/endpoints), [anthropic-compat](https://lmstudio.ai/docs/developer/anthropic-compat).*

## Anexo D — Glosario completo

| Término | Definición breve |
|---|---|
| **AVX2** | Set de instrucciones de CPU requerido por LM Studio en x64. |
| **Chunk** | Fragmento de un documento usado en RAG. |
| **Context window (ventana de contexto)** | Máximo de tokens (entrada+salida) que el modelo maneja a la vez. |
| **CORS** | Permiso para que un navegador llame la API desde otro origen (`--cors`). |
| **Cuantización** | Reducir la precisión de los pesos para ahorrar memoria/velocidad. |
| **Embedding** | Vector numérico que representa el significado de un texto. |
| **Endpoint OpenAI-compatible** | API con las mismas rutas/formato que OpenAI; se cambia solo la BaseAddress. |
| **GGUF** | Formato de modelo cuantizado del motor llama.cpp (multiplataforma). |
| **Headless** | Ejecución sin interfaz gráfica (servidor). |
| **Inferencia** | Ejecutar el modelo para generar una respuesta. |
| **K-quant** | Cuantización por bloques que ajusta precisión por capa (variantes `_K_M`). |
| **KV-cache** | Memoria intermedia de atención; crece con el contexto. |
| **llama.cpp** | Motor de inferencia que usa GGUF; multiplataforma. |
| **`llmster`** | Daemon headless de LM Studio para servidores/CI. |
| **`lms`** | CLI de LM Studio. |
| **LLM** | Large Language Model (modelo de lenguaje grande). |
| **MCP** | Model Context Protocol: estándar de Anthropic para dar herramientas/recursos a los modelos. |
| **MCP host/client** | Rol en que LM Studio se conecta a servidores MCP para darle tools al modelo. |
| **MLX** | Motor/formato de Apple para Apple Silicon. |
| **MoE (Mixture of Experts)** | Modelo con muchos parámetros totales pero pocos activos por token. |
| **Modelo** | Archivo con los pesos de la red neuronal entrenada. |
| **Parámetros (7B/70B)** | Cantidad de pesos del modelo ("B" = mil millones). |
| **Q4_K_M** | Nivel de cuantización "sweet spot" (≈4.5 bits, ~92–95% de calidad). |
| **RAG** | Retrieval-Augmented Generation: responder usando documentos recuperados. |
| **Retrieval (recuperación)** | Traer los chunks más relevantes para una consulta. |
| **SSE (Server-Sent Events)** | Mecanismo de streaming de la respuesta token a token. |
| **System prompt** | Instrucción inicial que define el rol/comportamiento del bot. |
| **Temperature** | Parámetro de aleatoriedad de la generación (0 = determinista). |
| **Token** | Unidad mínima de texto (≈ ¾ de palabra). |
| **Tool / function calling** | Capacidad del modelo de invocar funciones/herramientas externas. |
| **Vector store** | Base de datos de embeddings con búsqueda por similitud. |

---

*Apunte generado el 2026-06-14. Datos volátiles (versión 0.4.16, endpoints, imagen Docker, catálogo de modelos) verificados contra la documentación oficial de LM Studio y fuentes citadas; reverificá en [lmstudio.ai/docs](https://lmstudio.ai/docs) antes de decisiones críticas, porque el proyecto cambia rápido.*
