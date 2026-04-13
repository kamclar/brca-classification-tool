# Setup prostředí

## Google Colab (doporučeno pro prototypování)

### Požadavky

- Google účet
- Google Drive pro ukládání notebooků

### Setup

1. Otevři [Google Colab](https://colab.research.google.com/)
2. Připoj Google Drive: `from google.colab import drive; drive.mount('/content/drive')`
3. Vytvoř složku: `/content/drive/MyDrive/BRCA_Classification/`

### GPU Runtime (pro LLM)

1. Runtime → Change runtime type → T4 GPU
2. Instalace Ollama:
```bash
!apt-get update && apt-get install -y zstd
!curl -fsSL https://ollama.com/install.sh | sh
!nohup ollama serve &
!ollama pull mistral
```

## Lokální vývoj

### Požadavky

- Python 3.10+
- pip nebo conda

### Instalace

```bash
# Vytvoř virtuální prostředí
python -m venv venv
source venv/bin/activate  # Linux/Mac
# nebo: venv\Scripts\activate  # Windows

# Instaluj závislosti
pip install -r requirements.txt
```

### requirements.txt

```
requests>=2.28.0
pandas>=1.5.0
biopython>=1.80
httpx>=0.24.0
```

## API klíče

### NCBI E-utilities (doporučeno)

1. Registruj se na https://www.ncbi.nlm.nih.gov/account/
2. Získej API key v nastavení účtu
3. Nastav proměnnou prostředí:
```bash
export NCBI_API_KEY="your_api_key"
```

Výhoda: 10 requests/s místo 3 requests/s

## Struktura projektu

```
brca-classification-tool/
├── notebooks/
│   ├── 01_literature_search.ipynb
│   ├── 02_acmg_engine.ipynb
│   └── 03_integration.ipynb
├── src/
│   ├── __init__.py
│   ├── gnomad.py
│   ├── clinvar.py
│   ├── pubmed.py
│   ├── llm_extraction.py
│   └── acmg_rules.py
├── tests/
│   └── test_variants.py
├── data/
│   ├── test_variants.csv
│   └── enigma_table9.csv
└── requirements.txt
```
