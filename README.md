# ECGAssistant — Sistema de Apoyo al Diagnóstico Cardíaco

Proyecto final del Bootcamp de Inteligencia Artificial (KeepCoding ED.4)

ECGAssistant es un sistema de apoyo al diagnóstico cardíaco que combina 
deep learning, modelos de lenguaje y recuperación de información para 
clasificar señales ECG y generar explicaciones clínicas en español.

El sistema actúa como un primer filtro de detección: analiza la señal 
eléctrica del corazón, detecta si hay anomalías y genera una explicación 
clínica comprensible para el paciente o el profesional médico. No sustituye 
el criterio de un cardiólogo, pero puede ayudar a priorizar qué pacientes 
requieren atención urgente.

---

## Pipeline del sistema
```
Señal ECG → CNN clasifica → Normal/Anormal → RAG busca contexto → LLM genera explicación
```

El sistema integra tres componentes de inteligencia artificial:

1. **CNN 1D** — analiza directamente la señal eléctrica del corazón y 
   clasifica cada latido como normal o anormal. En lugar de calcular 
   características manualmente, la red aprende sola qué patrones son 
   relevantes para detectar arritmias.

2. **RAG con FAISS** — antes de generar la explicación, el sistema 
   recupera fragmentos relevantes de documentos clínicos reales. Así 
   las respuestas del LLM están ancladas en información médica verificada 
   y no generada de memoria.

3. **Gemma 3 4B fine-tuneado con QLoRA** — genera explicaciones clínicas 
   en español adaptadas al resultado de la CNN, usando el contexto 
   recuperado por el RAG como base.

---

## Resultados

### Clasificación ECG (CNN)

| Métrica | Valor |
|---|---|
| F1-score (test) | 0.9867 |
| Accuracy (test) | 99% |
| AUC-ROC | 0.9988 |
| Recall clase anormal | 0.99 |
| Arritmias detectadas | 4.236 / 4.299 (98.5%) |
| Falsas alarmas | 51 / 14.748 (0.3%) |

El modelo supera el objetivo marcado de F1 ≥ 0.95 y demuestra una 
capacidad discriminativa casi perfecta con un AUC de 0.9988.

### Evaluación LLM (rúbricas clínicas automáticas, escala 0-3)

| Rúbrica | Descripción | Puntuación |
|---|---|---|
| Precisión | Cobertura de términos clave clínicos | 0.96 |
| Claridad | Longitud y estructura de la respuesta | 3.00 |
| Anclaje | Respuesta centrada en la pregunta | 0.98 |
| Seguridad | Presencia de disclaimers médicos | 1.00 |

---

## Dataset

El proyecto usa el **MIT-BIH Arrhythmia Database** de PhysioNet, uno de 
los datasets de referencia en investigación de arritmias cardíacas.

| | |
|---|---|
| Fuente | PhysioNet — https://physionet.org/content/mitdb/1.0.0/ |
| Registros | 48 pacientes diferentes |
| Frecuencia de muestreo | 360 Hz |
| Latidos anotados | ~110.000 por cardiólogos |
| Tipos de arritmias | 18 clases diferentes |
| Latidos procesados | 73.737 |
| Licencia | Público, anonimizado, uso libre para investigación |

El dataset se descarga automáticamente ejecutando el notebook 02.

---

## Estructura del repositorio
```
Proyecto_Final_ECGAssistant/
│
├── notebooks/
│   ├── 01_Definicion_del_problema.ipynb       ← problema, métricas y ética
│   ├── 02_Recoleccion_de_Datos.ipynb          ← descarga MIT-BIH
│   ├── 03_Preprocesamiento_de_Datos.ipynb     ← segmentación y normalización
│   ├── 04_Analisis_Exploratorio_de_Datos.ipynb ← EDA y visualizaciones
│   ├── 05_Entrenamiento_CNN.ipynb             ← arquitectura y entrenamiento
│   ├── 06_Evaluacion_del_Modelo.ipynb         ← F1, AUC, matriz de confusión
│   ├── 07_Fine_tuning_Gemma_3_con_QLoRA.ipynb ← fine-tuning del LLM
│   ├── 08_RAG_con_FAISS.ipynb                 ← índice FAISS y retrieval
│   ├── 09_Pipeline_Completo.ipynb             ← integración CNN + RAG + LLM
│   ├── 10_Evaluacion_LLM_Rubricas_Clinicas.ipynb ← evaluación del LLM
│   └── 11_Conclusiones.ipynb                 ← reflexiones y trabajo futuro
│
├── rag_documents/
│   └── (documentos clínicos en formato .md)
│
├── eval/
│   └── ecg_eval_model_dataset.json            ← dataset de evaluación LLM
│
├── gemma3_qlora/
│   └── (generado ejecutando el notebook 07, no incluido por tamaño)
│
├── data/
│   └── (generado ejecutando el notebook 03, no incluido por tamaño)
│
├── requirements.txt
└── README.md
```

> Los archivos pesados (datos .npy, modelo Gemma, índice FAISS) no están
> incluidos en el repositorio por su tamaño. Se generan ejecutando los
> notebooks en orden.

---

## Instalación y ejecución

### Requisitos

- Python 3.10+
- GPU recomendada para los notebooks 07, 08, 09 y 10 (Colab con GPU T4 o superior)
- Cuenta en Hugging Face con acceso a `google/gemma-3-4b-it`

### Instalación
```bash
git clone https://github.com/Rvs8/Proyecto_Final_ECGAssistant.git
cd Proyecto_Final_ECGAssistant
pip install -r requirements.txt
```

### Ejecución en Google Colab

1. Sube la carpeta del proyecto a Google Drive
2. Añade tu token de Hugging Face como secreto en Colab con el nombre `HF_TOKEN`
3. Ejecuta los notebooks en orden del 01 al 11
4. El notebook 03 genera los archivos `.npy` necesarios para los siguientes
5. El notebook 07 genera la carpeta `gemma3_qlora/` necesaria para el 09 y 10
6. El notebook 08 genera `faiss_index.bin` y `chunks.json` necesarios para el 09

### Ejecución local
```bash
# Los notebooks 01-06 y 11 funcionan en CPU sin problema
# Los notebooks 07, 08, 09 y 10 requieren GPU con al menos 8GB de VRAM
jupyter notebook notebooks/
```

---

## Tecnologías utilizadas

| Componente | Tecnología |
|---|---|
| Clasificación ECG | PyTorch, CNN 1D |
| Fine-tuning LLM | Hugging Face Transformers, PEFT, QLoRA |
| Modelo base LLM | Google Gemma 3 4B Instruct |
| Recuperación RAG | FAISS, sentence-transformers |
| Embeddings | paraphrase-multilingual-MiniLM-L12-v2 |
| Dataset ECG | MIT-BIH (wfdb) |
| Entorno | Google Colab, Python 3.10 |

---

## Equipo

Proyecto desarrollado en equipo siguiendo metodología SCRUM con sprints
semanales, backlog en Trello y control de versiones en GitHub.

- Roger Vergés
- Raúl Sánchez
- Matteo Romanello
- Matias Cavallo

Bootcamp de Inteligencia Artificial — KeepCoding ED.4 — 2026

---

## Aviso

Este sistema está diseñado como herramienta de apoyo al diagnóstico,
no como sustituto del criterio clínico de un profesional médico.
Todas las respuestas generadas incluyen un aviso explícito al respecto.

