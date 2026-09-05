🔎 Chatbot RAG — Prevención de Lavado de Activos y Financiamiento del Terrorismo (UAF Chile)

Asistente conversacional (RAG — Retrieval-Augmented Generation) que permite consultar en lenguaje natural la documentación oficial de la Unidad de Análisis Financiero (UAF) de Chile en materia de Lavado de Activos, Financiamiento del Terrorismo y Financiamiento de la Proliferación (LA/FT/FP).

Pensado como herramienta de apoyo para oficiales de cumplimiento y sujetos obligados que necesitan resolver rápidamente preguntas como:

¿Mi institución debe informar a la UAF según su tipo de actividad?
¿Qué reportes debo enviar, con qué periodicidad y contenido?
¿Qué sanciones y normativa están asociadas a un incumplimiento específico?
¿Este comportamiento de un cliente es una señal de alerta de LA/FT?
Cualquier otra consulta general sobre LA/FT/FP.

⚠️ Disclaimer: esta herramienta es un apoyo informativo basado en documentación pública de la UAF. No constituye asesoría legal vinculante. La decisión final sobre reportar una operación corresponde siempre al Oficial de Cumplimiento del sujeto obligado.

📄 Corpus de referencia

El chatbot responde exclusivamente con base en un PDF consolidado (LAFT_UAF_Chile.pdf) que reúne 7 documentos oficiales de la UAF:

Ley N°19.913 — Crea la Unidad de Análisis Financiero.
Anexos N°1 y N°2 de la Circular N°62 (UAF).
Circular N°62 (UAF) — Instrucciones de carácter general a sujetos obligados.
Señales de Alerta para Prevenir el Cohecho a Funcionarios Públicos Extranjeros (UAF, 2014).
Guía de Señales de Alerta de LA/FT/FP — Actualización 2023 (UAF).
Señales de Alerta — Instituciones Públicas (UAF, 2016).
Catálogo de Delitos Base o Precedentes de Lavado de Activos (UAF, septiembre 2025).
🧩 Arquitectura
PDF consolidado (UAF)
        │
        ▼
  Chunking (RecursiveCharacterTextSplitter, tiktoken)
        │
        ▼
  Embeddings (thenlper/gte-large, multilingüe)
        │
        ▼
  Base vectorial (ChromaDB)
        │
        ▼
  Retriever (similarity search) ──► Control dinámico de presupuesto de tokens
        │
        ▼
  LLM (HuggingFaceH4/zephyr-7b-beta, 4-bit vía bitsandbytes)
        │
        ▼
  Formulario interactivo (ipywidgets): 1 verificación determinística + 4 modos RAG guiados

Stack: LangChain 1.x (langchain-text-splitters, langchain-chroma, langchain-huggingface), ChromaDB, Sentence Transformers, Hugging Face transformers + bitsandbytes, PyMuPDF, ipywidgets.

Nota técnica: el motor de generación usa transformers + bitsandbytes (cuantización de 4 bits) en vez de llama-cpp-python, porque este último requiere compilar C++ y actualmente falla en Python 3.13 (versión que trae Google Colab por defecto). Este enfoque evita cualquier compilación local: todo se instala desde wheels precompiladas.

✨ Funcionalidades del formulario

El chatbot se usa a través de un formulario interactivo dentro del notebook (sin tocar código), con 5 preguntas:

¿Debo informar a la UAF? (según tipo de entidad) — determinístico: un menú desplegable con las ~34 categorías reales de sujetos obligados del Art. 3° de la Ley N°19.913, más ejemplos de entidades no obligadas. La respuesta (Sí/No) es una comparación directa en código, sin pasar por el modelo de lenguaje, por lo que no hay margen de error o alucinación en esta pregunta.
¿Qué reportes debo enviar? — el usuario describe su situación/operación; el modelo responde sobre tipo de informe (ROS, ROE, u otro), periodicidad y contenido.
¿Qué sanciones y normativa está asociada a los siguientes incumplimientos? — el usuario describe el/los incumplimientos específicos; el modelo responde con las sanciones y normativa aplicable a ese caso concreto (no una respuesta genérica única, ya que cada incumplimiento es distinto).
Señales de Alerta LA/FT — el usuario describe un comportamiento u operación de un cliente; el modelo evalúa de forma estructurada: señales de alerta detectadas, si corresponde reportar, tipo de informe/periodicidad, umbral aplicable y sanciones.
Pregunta general — cualquier otra consulta libre sobre LA/FT/FP.

Además:

Control dinámico de tokens: recorta automáticamente el contexto recuperado si el prompt excede la ventana del modelo, evitando errores por exceso de tokens.
Evaluación de calidad: funciones de groundedness (¿la respuesta se basa solo en el contexto?) y relevance (¿responde lo que se preguntó?), para validar el desempeño del sistema.
🚀 Cómo correrlo
Abre UAF_LAFT_RAG_Chatbot.ipynb en Google Colab.
Entorno de ejecución → Cambiar tipo de entorno de ejecución → selecciona GPU T4.
Sube LAFT_UAF_Chile.pdf al panel de archivos (o móntalo desde Google Drive; hay una celda dedicada a esto).
Entorno de ejecución → Ejecutar todo.
La celda de instalación reinicia automáticamente el entorno de Python al terminar (verás un aviso de "sesión reiniciada" — es esperado).
Vuelve a correr Ejecutar todo una segunda vez para completar el resto del notebook.
Usa el formulario interactivo que aparece al final del notebook para hacer tus consultas.

Requisitos: cuenta de Google (Colab), GPU T4 (incluida en el nivel gratuito de Colab).

📁 Estructura del repositorio
.
├── UAF_LAFT_RAG_Chatbot.ipynb   # Notebook principal
├── LAFT_UAF_Chile.pdf           # Corpus consolidado (documentación oficial UAF)
└── README.md
🛣️ Posibles mejoras futuras
Chunking por artículo/sección normativa para citas más precisas (ej. "Ley 19.913, art. 2°").
Registro (logging) de consultas y respuestas para trazabilidad y auditoría interna, especialmente las verificaciones de sujeto obligado (Pregunta 1) por su relevancia legal.
Actualización periódica del listado de sujetos obligados (SUJETOS_OBLIGADOS) si la Ley N°19.913 se reforma.
Exponer el chatbot como aplicación web independiente (Gradio/Streamlit) para uso fuera de Colab, con infraestructura persistente.
Actualización periódica del corpus documental ante cambios normativos de la UAF.
⚖️ Licencia y fuente de datos

El corpus documental proviene de publicaciones oficiales y de acceso público de la Unidad de Análisis Financiero de Chile (www.uaf.cl). Este proyecto es de carácter educativo/demostrativo y no está afiliado ni respaldado por la UAF.
