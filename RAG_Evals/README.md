# RAG_Evals

A Jupyter notebook project for building and evaluating a retrieval-augmented question-answering system about Power BI. The notebook loads the ebook PDF, indexes its text with FAISS, retrieves relevant passages for test questions, and generates answers with a local Ollama model. It measures answer quality with RAGAS, BLEU, ROUGE-1, BERTScore, and BARTScore.

## Folder Contents

- `Rag_metrics.ipynb` builds the RAG pipeline, runs the evaluation questions, calculates scores, and ranks model/configuration runs.
- `Power+BI+Ebook.pdf` is the source document loaded and indexed by the notebook.
- `power_bi_rag_test_questions.csv` contains evaluation questions and reference answers. The notebook expects columns named `Question` and `Expected Answer`.

Keep the PDF and CSV in this folder: the notebook loads both using paths relative to its current working directory.

## Requirements

- macOS and Python 3.11 (the version used by the notebook's current environment)
- [Ollama](https://ollama.com/download) installed and running locally
- An internet connection for installing packages and downloading model checkpoints on first use
- Sufficient memory and disk space for the language, embedding, and evaluation models

The BARTScore cell uses Apple Metal (MPS) when available and falls back to CPU.

## Installation

Open a terminal in this folder and create a virtual environment:

```bash
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

Install the notebook and Python dependencies:

```bash
python -m pip install jupyterlab pandas langchain langchain-core langchain-classic langchain-community langchain-ollama langchain-huggingface langchain-text-splitters ragas pypdf faiss-cpu sentence-transformers rouge nltk bert-score transformers torch sentencepiece
```

If `faiss-cpu` cannot be installed through pip for your Mac/Python combination, install it from conda-forge in a conda environment:

```bash
conda install -c conda-forge faiss-cpu
```

Install and start Ollama, then download the generation model used by default:

```bash
ollama pull qwen2.5:3b
```

The notebook also downloads `sentence-transformers/all-MiniLM-L6-v2` for embeddings and `facebook/bart-large-cnn` for BARTScore the first time those models are used.

## Run the Notebook

From this folder, activate the environment and start JupyterLab:

```bash
source .venv/bin/activate
jupyter lab
```

Open `Rag_metrics.ipynb`, select the `.venv` Python kernel, and run the cells from top to bottom. Ollama must be available while the notebook generates answers and runs RAGAS evaluation. Model downloads and evaluation can take several minutes.

To compare generation models, change the Ollama model in the model-configuration cell, rerun that cell and all dependent answer-generation and scoring cells using the same test questions, then update `mode_label` in the final comparison cell. The comparison table is held in memory for the current kernel session; restarting the kernel clears previous runs. Use a unique label for each configuration you want to compare.

## Metrics and Scope

- **RAGAS** checks faithfulness, context recall, and factual correctness. The current notebook uses only the first test item for this evaluation; expand `test_dataset` to include all results for a broader assessment.
- **BLEU** and **ROUGE-1 F1** measure lexical overlap with the reference answer.
- **BERTScore F1** measures semantic similarity using contextual embeddings.
- **BARTScore** scores the generated answer against the reference; in this implementation, a higher (less negative) score is better.
- The final table ranks each metric separately and sorts by mean rank, avoiding direct averaging of differently scaled metric values. A single run is only a baseline, not a model comparison.
