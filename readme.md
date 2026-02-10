# 🏠 AI Real Estate RAG Agent: Inmobiliaria Melisa Schwab

Este proyecto es un ecosistema completo de **IA Aplicada** al sector inmobiliario. Utiliza un pipeline de **RAG (Retrieval-Augmented Generation)** para transformar una web estática en un asistente inteligente capaz de buscar propiedades por similitud semántica y responder a través de **Telegram** con contenido multimedia nativo.

---

## 🎥 Demo & Workflow

> **[VER VIDEO DEMO AQUÍ]***(Sustituir por link a YouTube/Loom)*

---

## 🚀 1. Pipeline de Ingesta (ETL & Vectorización)

El objetivo es automatizar la creación de una base de conocimiento actualizada en tiempo real.

* **Discovery & Extraction:** Uso de **Firecrawl** para el rastreo y **Browserless** para el renderizado dinámico de JavaScript (crucial para extraer precios y stock que no cargan en el HTML estático).
* **Normalización:** Nodo de código (JavaScript) para limpiar símbolos de moneda, convertir textos a números reales y descartar registros incompletos.
* **Vectorización:** Integración con **OpenAI (text-embedding-3-small)** para generar representaciones matemáticas del contexto de cada propiedad.
* **Persistencia:** Almacenamiento dual en **Supabase** (PostgreSQL + pgvector) permitiendo búsquedas relacionales (filtros duros) y semánticas (intención).

---

## 🤖 2. Chatbot & UX (RAG Retrieval)

Interfaz conversacional diseñada para eliminar la fricción del usuario.

* **Entrada Multimodal:** Normalización de audio (Whisper) y texto en un flujo unificado.
* **Feedback Visual:** Implementación de acciones de chat ("Escribiendo...") y mensajes de transición ("Buscando propiedades...") para mejorar la percepción de velocidad.
* **AI Agent (Structured Output):** El agente de IA está forzado a responder en **JSON puro**. Esto desacopla la lógica de negocio del renderizado visual.
* **Native Rendering:** Uso del nodo `Send Photo` de Telegram en modo **HTML**. Esto permite enviar la imagen de la propiedad en HD con una descripción persuasiva debajo, evitando los "links azules" tradicionales.

---

## 🐘 3. Arquitectura de Datos

Se diseñó un esquema en **Supabase** que gestiona la información inmobiliaria, la búsqueda vectorial y la persistencia de conversaciones.


| **Tabla**            | **Tipo**     | **Propósito**                                                               |
| -------------------- | ------------ | ---------------------------------------------------------------------------- |
| `properties`         | Relacional   | Almacena los datos estructurados y técnicos de cada propiedad.              |
| `property_documents` | Vectorial    | Almacena los fragmentos de texto y sus embeddings para búsqueda semántica. |
| `chat_logs`          | Logs/Memoria | Mantiene el historial de la conversación para dar contexto al Agente.       |

#### 📄 Ejemplos de Estructura (JSON)

**1. Tabla `properties` (Datos Estructurados)**

Representa la información "dura" extraída durante el scraping.

**JSON**

```
{
  "id": "a1b2-c3d4-e5f6",
  "titulo": "Casa 4 ambientes con pileta en San Vicente",
  "precio_valor": 160000,
  "moneda": "USD",
  "garage": 1,
  "imagen_principal": "https://cdn-images.xintelweb.com/upload/msw1052_2.jpg",
  "source_url": "https://www.melisaschwabinmobiliaria.com.ar/casa-en-venta-msw931"
}
```

**2. Tabla `property_documents` (Datos Vectoriales)**

Es el bloque de texto que la IA "lee" y su representación matemática (embedding).

**JSON**

```
{
  "property_id": "a1b2-c3d4-e5f6",
  "content": "Título: Casa 4 ambientes con pileta en San Vicente. Precio: 160k USD. Descripción: Amplia casa con churrasquera, pileta y garage...",
  "embedding": [0.0023, -0.0145, 0.8231, "... (1536 dimensiones)"],
  "metadata": {
    "precio": 160000,
    "zona": "San Vicente"
  }
}
```

**3. Tabla `chat_logs` (Memoria de Conversación)**

Guarda el intercambio entre el usuario y la IA para mantener el hilo conductor del chat.

**JSON**

```
{
  "session_id": "929364357",
  "message": {
    "type": "human",
    "content": "Hola, estoy buscando una casa en Pueblo Nuevo hasta 180 mil dólares.",
    "additional_kwargs": {},
    "response_metadata": {}
  },
  "created_at": "2026-02-09 20:14:46.948+00"
}
```

---

> **💡 Trick Técnico:** La función SQL `consultar_inmobiliaria` realiza un `LEFT JOIN` en tiempo real entre `property_documents` y `properties`. Esto permite que el Agente reciba el texto semántico junto con la URL de la imagen en una sola operación, optimizando la respuesta.

## 📂 4. Estructura del Proyecto

Para replicar este sistema, el repositorio está organizado de la siguiente manera:

* **/workflows:** Archivos `.json` exportados de n8n.
  * `01_ingesta.json`: Flujo de scraping y carga.
  * `02_chatbot.json`: Lógica de Telegram y RAG.
* **/database:** Scripts SQL para Supabase.
  * `schema.sql`: Definición de tablas.
  * `functions.sql`: Lógica de búsqueda vectorial y filtros.
* **/assets:** Diagramas de arquitectura y capturas de pantalla.

---

## 🛠️ 5. Configuración y Seguridad

1. **Importación:** En n8n, selecciona "Import from file" y sube los archivos de la carpeta `/workflows`.
2. **Credenciales:** Deberás configurar tus propios tokens para Telegram API, OpenAI y Supabase.
3. **Seguridad:** Este repositorio **no contiene API Keys**. Todas las variables sensibles están manejadas mediante el sistema de credenciales nativo de n8n.
4. **Parse Mode:** Asegúrate de que los nodos de Telegram estén configurados en modo **HTML** para que el formateo del agente (negritas `<b>`) funcione correctamente sin errores de escape.

---


## 🛠️ 6. Tecnologías & Documentación Oficial

El proyecto se sustenta sobre un stack tecnológico moderno de **AI-Orchestration** e **Infraestructura Vectorial**. A continuación, los enlaces a la documentación de las herramientas clave utilizadas:

### **Orquestación y Workflow**

* [n8n](https://docs.n8n.io/): Plataforma de automatización basada en nodos utilizada para el diseño de los pipelines de ingesta y el motor del chatbot.
* [Browserless](https://www.browserless.io/docs/): Navegador headless utilizado para el renderizado de contenido dinámico (JavaScript) durante el scraping.
* [Firecrawl](https://docs.firecrawl.dev/): Herramienta de crawling optimizada para LLMs utilizada en la fase de discovery de propiedades.

### **Inteligencia Artificial**

* [OpenAI (GPT-5-mini)](https://developers.openai.com/api/docs/models/gpt-5-mini): Modelo de lenguaje principal para la generación de respuestas persuasivas y estructuración de JSON.
* [OpenAI Embeddings](https://platform.openai.com/docs/guides/embeddings): Modelo `text-embedding-3-small` utilizado para la vectorización semántica de las propiedades.
* [OpenAI Whisper](https://platform.openai.com/docs/guides/speech-to-text): Motor de transcripción de audio a texto para procesar notas de voz de los usuarios.

### **Base de Datos y Búsqueda Vectorial**

* [Supabase](https://supabase.com/docs): Backend-as-a-Service basado en PostgreSQL utilizado para la persistencia relacional.
* [pgvector (Supabase Guide)](https://supabase.com/docs/guides/database/extensions/pgvector): Extensión de PostgreSQL para el almacenamiento y búsqueda de vectores de alta dimensionalidad.

### **Interfaz de Usuario**

* [Telegram Bot API](https://core.telegram.org/bots/api): Documentación oficial para el manejo de mensajes, envío de fotos nativas y formateo HTML/Markdown.

## 7. Próximos Pasos & Mejoras Futuras

El sistema está diseñado para ser escalable. Las siguientes iteraciones planificadas incluyen:

* **Integración Multicanal:** Adaptar los webhooks para soportar **WhatsApp Business API** (vía Twilio o Cloud API), manteniendo la misma lógica de RAG.
* **Optimización de Tokens:** Implementar una capa de caché para consultas frecuentes y refinar el *trimming* de la memoria del chat para reducir costos operativos de OpenAI.
* **Galerías Dinámicas:** Evolucionar el envío de una sola imagen a un `Media Group` (Carrusel) para mostrar múltiples fotos de la propiedad en un solo mensaje de Telegram.
* **Lead Scoring:** Integrar un nodo de CRM para calificar automáticamente al cliente según su presupuesto y comportamiento en el chat.

---

## 👤 Autor

**Luciano Oroquieta** *Full Stack Developer & AI Solutions* [LinkedIn](https://www.linkedin.com/in/luciano-oroquieta/) | [Portfolio](https://www.luciano-oroquieta.me/home) | [Email](https://www.google.com/search?q=oroquietaluciano@gmail.com)
