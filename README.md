# 🤖 Agente ReAct con LangGraph + Azure OpenAI

Implementación completa de un agente conversacional basado en el patrón **ReAct** (Razonamiento + Actuación) usando **LangGraph** y **Azure AI Foundry** (Azure OpenAI Service).

---

## 📐 Arquitectura

```
INICIO
  └─> ENTRADA Y FILTRO (SEGURIDAD)
        └─> AGENTE / LLM  <──────────────────────────┐
              │  ↕ LEER/ESCRIBIR                      │
         MEMORIA DE CHAT                              │
              │                                       │
              ├─> HERRAMIENTAS (RAG / API Clima / Export)
              │         └─> OBSERVACIÓN / SALIDA ─────┘
              │
              └─> RESPUESTA Y FILTRO (CONTROL)
                        └─> FIN
```

### 🔄 Bucle ReAct

```
agente (DECISIÓN)
   ↓ si tool_calls
herramientas (ACCIÓN)
   ↓ resultado
agente (OBSERVACIÓN → nueva DECISIÓN)
   ↓ si respuesta final
respuesta_filtro → FIN
```

---

## 🗂️ Estructura del proyecto

```
langChain/
├── agente_react_langgraph.ipynb   # Notebook principal con el agente
├── .env                           # Variables de entorno (no subir a git)
├── .env.example                   # Plantilla de variables de entorno
├── requirements.txt               # Dependencias del proyecto
├── venv/                          # Entorno virtual Python
└── README.md                      # Este archivo
```

---

## ⚙️ Configuración

### 1. Requisitos previos

- Python 3.10+
- Una cuenta de **Azure AI Foundry** con:
  - Un deployment de modelo de chat (ej: `gpt-4o-mini`)
  - Un deployment de embeddings (ej: `text-embedding-ada-002`)
- (Opcional) API key de [OpenWeatherMap](https://openweathermap.org/api) para clima real

### 2. Clonar e instalar dependencias

```bash
git clone <url-del-repo>
cd langChain

python -m venv venv
venv\Scripts\activate        # Windows
# source venv/bin/activate   # Linux/Mac

pip install langgraph langchain langchain-openai langchain-community \
    langchain-core faiss-cpu tiktoken python-dotenv requests
```

### 3. Configurar variables de entorno

Crea un archivo `.env` en la raíz del proyecto con el siguiente contenido:

```env
# Azure AI Foundry — Endpoint y clave
# Encuéntralo en: Azure Portal → tu recurso Azure OpenAI → Claves y punto de conexión
AZURE_OPENAI_ENDPOINT=https://<tu-recurso>.openai.azure.com/
AZURE_OPENAI_API_KEY=<tu-clave-aqui>

# Nombre del deployment del modelo de chat
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini

# Nombre del deployment del modelo de embeddings (para RAG)
AZURE_OPENAI_EMBEDDINGS_DEPLOYMENT=text-embedding-ada-002

# Versión de la API de Azure OpenAI
AZURE_OPENAI_API_VERSION=2024-02-01

# OpenWeatherMap API Key (opcional)
OPENWEATHER_API_KEY=<tu-clave-openweather>
```

> ⚠️ **Nunca subas el archivo `.env` a git.** Está incluido en `.gitignore`.

---

## 🚀 Uso

Abre el notebook en VS Code o Jupyter:

```bash
jupyter notebook agente_react_langgraph.ipynb
```

Asegúrate de seleccionar el kernel del entorno virtual (`venv`).

### Ejecutar las celdas en orden

| Celda | Descripción |
|-------|-------------|
| 1 | Instalación de dependencias |
| 2 | Carga de variables de entorno |
| 3 | Importaciones |
| 4 | Definición del `AgentState` |
| 5 | Creación del vectorstore RAG (FAISS) |
| 6 | Definición de herramientas |
| 7 | Configuración del LLM |
| 8 | Nodos del grafo |
| 9 | Lógica de enrutamiento |
| 10 | Construcción y compilación del grafo |
| 11 | Visualización del grafo |
| 12 | Función `invocar_agente` |
| 13 | Pruebas automáticas |
| 14 | **Chat interactivo** ← modo conversación en tiempo real |
| 15 | Ver historial exportable |

---

## 🛠️ Herramientas del agente

### 📚 `busqueda_rag`
Búsqueda semántica en la base de conocimiento interna usando **FAISS + embeddings de Azure OpenAI**.
- Úsala preguntando sobre LangChain, LangGraph, FAISS o el patrón ReAct.
- Los documentos se dividen en fragmentos de 300 tokens con solapamiento de 50.

### 🌤️ `api_clima`
Obtiene el clima actual de cualquier ciudad usando la **API de OpenWeatherMap**.
- Si no hay API key configurada, devuelve datos simulados.
- Ejemplo: *"¿Qué tiempo hace en Madrid?"*

### 💾 `exportar_chat`
Exporta el historial de la conversación a un archivo `.txt` o `.json`.
- Ejemplo: *"Exporta esta conversación en formato json."*

---

## 🧠 Nodos del grafo

| Nodo | Función | Equivalente en diagrama |
|------|---------|------------------------|
| `entrada_filtro` | Valida y filtra el input del usuario | ENTRADA Y FILTRO (SEGURIDAD) |
| `agente` | LLM con tool calling (ReAct: DECISIÓN) | AGENTE / LLM |
| `herramientas` | Ejecuta RAG, Clima, Export (ACCIÓN) | HERRAMIENTAS + OBSERVACIÓN |
| `respuesta_filtro` | Formatea y valida la respuesta final | RESPUESTA Y FILTRO (CONTROL) |
| `messages` en estado | Historial completo de la conversación | MEMORIA DE CHAT |

---

## 🔒 Filtro de seguridad

El nodo `entrada_filtro` bloquea automáticamente mensajes que contengan:

```
hack, exploit, malware, virus, bomba, arma,
contraseña, password, crack, robar, ilegal,
inject, sql injection, xss, bypass
```

---

## 💬 Chat interactivo

La celda 14 lanza un bucle de chat en tiempo real:

```
💬 Chat Interactivo con el Agente ReAct
  Escribe 'salir' o 'exit' para terminar.
──────────────────────────────────────────

👤 Tú: ¿Qué es LangGraph?

🤖 Agente: LangGraph es un framework de flujos de trabajo basado en grafos...
```

El agente **mantiene memoria entre turnos** del mismo `thread_id`, por lo que puede recordar el contexto de conversaciones anteriores.

---

## 📋 Ver historial del chat

La celda 15 permite inspeccionar el historial exportable de cualquier sesión:

```python
config_memoria = {"configurable": {"thread_id": "sesion-interactiva"}}
estado_actual = grafo.get_state(config_memoria)
historial = estado_actual.values.get("chat_export", [])
```

---

## 🐛 Depuración

Para ver el flujo interno del grafo (nodos, routers, decisiones), activa el flag `VERBOSE_NODES` en la celda 8:

```python
VERBOSE_NODES = True   # muestra el flujo completo
VERBOSE_NODES = False  # chat limpio (recomendado para uso normal)
```

Recuerda re-ejecutar la celda después de cambiar el valor.

---

## 📦 Dependencias principales

| Paquete | Versión mínima | Uso |
|---------|---------------|-----|
| `langgraph` | 0.2+ | Orquestación del grafo |
| `langchain` | 0.3+ | Framework base |
| `langchain-openai` | 0.2+ | Azure OpenAI / embeddings |
| `langchain-community` | 0.3+ | FAISS, loaders |
| `faiss-cpu` | 1.7+ | Vectorstore de embeddings |
| `tiktoken` | 0.7+ | Tokenización para OpenAI |
| `python-dotenv` | 1.0+ | Carga de `.env` |
| `requests` | 2.31+ | API del clima |

---

## 📄 Licencia

MIT — libre para uso personal y educativo.