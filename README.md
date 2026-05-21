# RAG Assistant – IA Generative

Systeme de Question-Reponse intelligent base sur la technique RAG (Retrieval-Augmented Generation). Il permet d'importer des documents PDF, de les indexer, puis de poser des questions en langage naturel. Les reponses sont generees par le modele Mistral en s'appuyant uniquement sur le contenu de vos documents.

## Comment ca fonctionne

Le systeme suit un pipeline en 7 etapes :

### Phase d'indexation (a l'import d'un PDF)

1. **Extraction du texte** – Le texte brut est extrait de chaque page du PDF avec pdfplumber.
2. **Decoupage en chunks** – Le texte est divise en petits fragments (500 caracteres par defaut) avec un chevauchement entre fragments pour ne pas couper les idees.
3. **Generation des embeddings** – Chaque chunk est transforme en vecteur numerique par le modele `all-MiniLM-L6-v2` (sentence-transformers). Ce vecteur represente le sens du texte.
4. **Stockage dans ChromaDB** – Les vecteurs et les textes sont stockes dans une base vectorielle locale persistante.

### Phase de question-reponse (a chaque question)

5. **Recherche par similarite** – La question est aussi convertie en vecteur, puis comparee aux chunks stockes pour trouver les passages les plus proches (top-k × 3 candidats).
6. **Re-ranking** – Un cross-encoder (`ms-marco-MiniLM-L-6-v2`) re-classe les candidats par pertinence reelle et ne garde que les top-k meilleurs. Cette etape est optionnelle mais ameliore significativement la precision.
7. **Generation de la reponse** – Les passages selectionnes + l'historique de conversation sont envoyes a l'API Mistral, qui genere une reponse basee uniquement sur le contexte fourni.

```
PDF → Extraction → Chunking → Embeddings → ChromaDB
                                                 ↓
Question → Embedding → Recherche → Re-ranking → Mistral API → Reponse
```

## Prerequis

- Python 3.10+
- Une cle API Mistral (a mettre dans un fichier `.env`)

### Installation

```bash
pip install -r requirements.txt
```

### Configuration

Creer un fichier `.env` a la racine du projet :

```
MISTRAL_API_KEY=votre_cle_api
```

## Lancement

```bash
streamlit run app.py
```

Ouvrir `http://localhost:8501` dans le navigateur.

## Utilisation

1. **Importer** vos PDFs via la barre laterale
2. Cliquer sur **Indexer** pour lancer l'indexation
3. **Poser vos questions** dans le chat
4. Les reponses s'affichent avec les **sources** (nom du document + numero de page)

### Parametres ajustables

| Parametre | Description | Defaut |
|---|---|---|
| Taille des chunks | Nombre de caracteres par fragment | 500 |
| Chevauchement | Caracteres partages entre chunks consecutifs | 50 |
| Top-k passages | Nombre de passages recuperes pour la reponse | 5 |
| Memoire (tours) | Nombre de tours de conversation gardes en memoire | 5 |
| Re-ranking | Active le re-classement par cross-encoder | Active |

## Structure du projet

```
iagenerative/
├── app.py                    # Interface Streamlit
├── rag/
│   ├── document_loader.py    # Extraction texte PDF (pdfplumber)
│   ├── chunker.py            # Decoupage en chunks (LangChain)
│   ├── embeddings.py         # Embeddings (all-MiniLM-L6-v2)
│   ├── vector_store.py       # Base vectorielle (ChromaDB)
│   ├── reranker.py           # Re-ranking (cross-encoder)
│   ├── generator.py          # Generation via API Mistral
│   ├── memory.py             # Memoire de conversation
│   └── pipeline.py           # Orchestration du pipeline
├── data/pdfs/                # PDFs importes
├── chroma_db/                # Stockage vectoriel persistant
├── .env                      # Cle API Mistral
└── requirements.txt
```

## Technologies utilisees

| Composant | Technologie |
|---|---|
| Extraction PDF | pdfplumber |
| Chunking | LangChain RecursiveCharacterTextSplitter |
| Embeddings | sentence-transformers/all-MiniLM-L6-v2 |
| Re-ranking | cross-encoder/ms-marco-MiniLM-L-6-v2 |
| Base vectorielle | ChromaDB |
| LLM | Mistral API (mistral-small-latest) |
| Memoire | Fenetre glissante de conversation |
| Interface | Streamlit |
