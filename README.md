# ECGAssistant — Sistema de Apoyo al Diagnóstico Cardíaco



Proyecto final del Bootcamp de Inteligencia Artificial (KeepCoding ED.4)



ECGAssistant es un sistema de apoyo al diagnóstico cardíaco que combina

deep learning, modelos de lenguaje y recuperación de información para

clasificar señales ECG y generar explicaciones clínicas en español.



---



## Pipeline del sistema

```

Señal ECG → CNN clasifica → Normal/Anormal → RAG busca contexto → LLM genera explicación

```



1. **CNN 1D** clasifica latidos cardíacos como normales o anormales

2. **RAG con FAISS** recupera contexto clínico relevante de documentos médicos

3. **Gemma 3 4B fine-tuneado con QLoRA** genera explicaciones en español



---



## Resultados

| Métrica | Valor |
|---|---|
| F1-score CNN (test) | 0.9867 |
| Accuracy CNN (test) | 99% |
| AUC-ROC | 0.9988 |
| Recall clase anormal | 0.99 |

## Dataset

| | |
|---|---|
| Nombre | MIT-BIH Arrhythmia Database |
| Latidos procesados | 73.737 |



---



## Estructura del repositorio

```

Proyecto_Final_ECGAssistant/

│

├── notebooks/

│   ├── 01_definicion_problema.ipynb

│   ├── 02_recoleccion_datos.ipynb

│   ├── 03_preprocesamiento.ipynb

│   ├── 04_eda.ipynb

│   ├── 05_entrenamiento_cnn.ipynb

│   ├── 06_evaluacion_modelo.ipynb

│   ├── 07_finetuning_gemma.ipynb

│   ├── 08_rag_faiss.ipynb

│   ├── 09_pipeline_completo.ipynb

│   └── 10_conclusiones.ipynb

│

├── rag_documents/

│   └── (documentos clínicos en formato .md)

│

├── models/

│   └── ecg_cnn.pt

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

- GPU recomendada para los notebooks 07, 08 y 09 (Colab con GPU T4 o superior)

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

3. Ejecuta los notebooks en orden del 01 al 10

4. El notebook 03 genera los archivos `.npy` necesarios para los siguientes

5. El notebook 07 genera la carpeta `gemma3_qlora/` necesaria para el 09

6. El notebook 08 genera `faiss_index.bin` y `chunks.json` necesarios para el 09



### Ejecución local

```bash

# Los notebooks 01-06 y 10 funcionan en CPU sin problema

# Los notebooks 07, 08 y 09 requieren GPU con al menos 8GB de VRAM

jupyter notebook notebooks/

```



---



## Dataset



**MIT-BIH Arrhythmia Database** — PhysioNet

https://physionet.org/content/mitdb/1.0.0/



- 48 registros de pacientes diferentes

- Frecuencia de muestreo: 360 Hz

- ~110.000 latidos anotados manualmente por cardiólogos

- 18 tipos de arritmias diferentes

- Datos anonimizados, uso libre para investigación



El dataset se descarga automáticamente ejecutando el notebook 02.



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

