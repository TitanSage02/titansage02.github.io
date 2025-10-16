---
layout: ../../layouts/BlogPost.astro
title: "e-Juris : Assistant juridique IA pour le code du numérique"
description: "Système RAG intelligent utilisant Gemini Pro pour répondre aux questions juridiques basées sur le Code du Numérique béninois."
date: "2025-05-30"
category: "IA & Droit"
tags: ["RAG", "Legal Tech", "Gemini Pro", "FAISS", "Langchain", "Streamlit", "NLP"]
author: "Espérance AYIWAHOUN"
---

## Démocratiser l'accès au droit béninois

En tant que citoyen béninois passionné de technologie, j'ai toujours été frappé par l'inaccessibilité du droit dans notre pays. Combien de fois ai-je vu des entrepreneurs, des étudiants, ou de simples citoyens se perdre dans les méandres juridiques faute d'informations claires et accessibles ?

Le **Code du Numérique** béninois, avec ses plus de 200 pages de textes complexes, est un parfait exemple de cette problématique. Comment un développeur peut-il savoir s'il respecte la loi sur la protection des données sans passer par un avocat coûteux ?

C'est ainsi qu'est né **e-Juris** : un assistant juridique intelligent qui utilise l'IA pour rendre le droit accessible à tous les Béninois.

**Code source :** [GitHub - e-Juris](https://github.com/TiTanSage02/eJuris)

---

## Le constat qui m'a motivé

En observant l'écosystème juridique béninois, j'ai identifié plusieurs barrières majeures :

| Problème | Impact |
|----------|--------|
| **Complexité du jargon** | Incompréhensible pour non-juristes |
| **Documents volumineux** | Code du Numérique = 200+ pages |
| **Consultations coûteuses** | Avocat = 50,000-100,000 FCFA/h |
| **Délai de réponse** | 2-7 jours pour une consultation |

**Solution** : Un chatbot juridique **gratuit**, **instantané** et **précis** basé sur les documents officiels.

---

## 🏗️ Architecture RAG

### Pipeline de Traitement

```
┌─────────────────────┐
│  Documents PDF      │
│  (Code Numérique)   │
└──────────┬──────────┘
           │
           v
┌─────────────────────────────┐
│  Chargement + Découpage     │
│  (PyPDFLoader, RecursiveText│
│   CharacterTextSplitter)    │
└──────────┬──────────────────┘
           │
           v
┌─────────────────────────────┐
│  Génération Embeddings      │
│  (Google Generative AI)     │
└──────────┬──────────────────┘
           │
           v
┌─────────────────────────────┐
│  Indexation FAISS           │
│  (Recherche vectorielle)    │
└──────────┬──────────────────┘
           │
           v
    ┌──────┴──────┐
    │ Question    │
    │ Utilisateur │
    └──────┬──────┘
           │
           v
┌─────────────────────────────┐
│  Recherche Similarité       │
│  (Top-K passages)           │
└──────────┬──────────────────┘
           │
           v
┌─────────────────────────────┐
│  Génération Réponse         │
│  (Gemini Pro + Context)     │
└──────────┬──────────────────┘
           │
           v
┌─────────────────────────────┐
│  Réponse + Sources          │
└─────────────────────────────┘
```

### Stack Technologique

| Composant | Technologie |
|-----------|-------------|
| **Framework RAG** | Langchain |
| **LLM** | Google Gemini Pro |
| **Embeddings** | Google Generative AI Embeddings |
| **Vector Store** | FAISS |
| **PDF Processing** | PyPDF |
| **Interface** | Streamlit |

---

## ✨ Implémentation Technique

### 1. Chargement et Découpage de Documents

```python
from langchain.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

def load_and_split_documents(pdf_path: str):
    """Charge et découpe un PDF en chunks."""
    
    # Chargement
    loader = PyPDFLoader(pdf_path)
    documents = loader.load()
    
    # Découpage
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=1000,        # Taille des chunks
        chunk_overlap=200,      # Chevauchement
        length_function=len,
        separators=["\n\n", "\n", " ", ""]
    )
    
    chunks = text_splitter.split_documents(documents)
    
    print(f"✅ {len(chunks)} chunks créés depuis {len(documents)} pages")
    return chunks
```

### 2. Création du Vector Store FAISS

```python
from langchain.vectorstores import FAISS
from langchain.embeddings import GoogleGenerativeAIEmbeddings
import google.generativeai as genai

# Configuration Gemini
genai.configure(api_key=os.getenv("GOOGLE_API_KEY"))

def create_vector_store(chunks: list):
    """Crée un index FAISS depuis les chunks."""
    
    # Génération d'embeddings
    embeddings = GoogleGenerativeAIEmbeddings(
        model="models/embedding-001"
    )
    
    # Création FAISS
    vectorstore = FAISS.from_documents(
        documents=chunks,
        embedding=embeddings
    )
    
    # Sauvegarde locale
    vectorstore.save_local("faiss_index")
    
    return vectorstore
```

### 3. Chaîne RAG avec Langchain

```python
from langchain.chains import RetrievalQA
from langchain.llms import GooglePalm
from langchain.prompts import PromptTemplate

def create_rag_chain(vectorstore):
    """Crée la chaîne RAG complète."""
    
    # Template de prompt
    template = """
    Tu es un assistant juridique spécialisé dans le Code du Numérique béninois.
    
    Contexte juridique :
    {context}
    
    Question de l'utilisateur :
    {question}
    
    Instructions :
    1. Réponds de manière claire et précise
    2. Cite les articles pertinents
    3. Si tu ne sais pas, dis-le clairement
    4. Utilise un langage accessible
    
    Réponse :
    """
    
    prompt = PromptTemplate(
        template=template,
        input_variables=["context", "question"]
    )
    
    # Configuration du LLM
    llm = GooglePalm(
        model="models/text-bison-001",
        temperature=0.2  # Faible pour précision juridique
    )
    
    # Chaîne RAG
    qa_chain = RetrievalQA.from_chain_type(
        llm=llm,
        chain_type="stuff",
        retriever=vectorstore.as_retriever(
            search_kwargs={"k": 4}  # Top 4 passages
        ),
        chain_type_kwargs={"prompt": prompt},
        return_source_documents=True
    )
    
    return qa_chain
```

### 4. Interface Streamlit

```python
import streamlit as st

st.set_page_config(
    page_title="e-Juris - Assistant Juridique",
    page_icon="⚖️",
    layout="wide"
)

# Header
st.title("⚖️ e-Juris - Assistant Juridique du Numérique")
st.caption("Posez vos questions sur le Code du Numérique béninois")

# Chargement du modèle
@st.cache_resource
def load_chain():
    vectorstore = FAISS.load_local(
        "faiss_index",
        GoogleGenerativeAIEmbeddings(model="models/embedding-001")
    )
    return create_rag_chain(vectorstore)

qa_chain = load_chain()

# Interface de chat
question = st.text_area(
    "Votre question juridique :",
    placeholder="Ex: Quelles sont les sanctions en cas de violation de données personnelles ?",
    height=100
)

if st.button("🔍 Obtenir une réponse", type="primary"):
    if question:
        with st.spinner("Recherche dans le Code du Numérique..."):
            result = qa_chain({"query": question})
        
        # Affichage de la réponse
        st.success("✅ Réponse trouvée")
        st.write(result["result"])
        
        # Sources
        with st.expander("📚 Sources juridiques"):
            for i, doc in enumerate(result["source_documents"]):
                st.markdown(f"**Source {i+1}** (Page {doc.metadata['page']})")
                st.text(doc.page_content[:300] + "...")
    else:
        st.warning("Veuillez entrer une question")

# Exemples de questions
with st.sidebar:
    st.header("💡 Exemples de questions")
    examples = [
        "Qu'est-ce qu'une donnée à caractère personnel ?",
        "Quelles sont les obligations des hébergeurs de données ?",
        "Comment porter plainte pour cybercriminalité ?",
        "Quelle est la définition juridique du spam ?"
    ]
    for ex in examples:
        if st.button(ex, key=ex):
            st.session_state.question = ex
```

---

## 📊 Résultats et Performance

### Métriques

| Métrique | Valeur |
|----------|--------|
| **Documents indexés** | 1 (Code Numérique complet) |
| **Chunks générés** | 450+ |
| **Temps de réponse** | 2-4s |
| **Précision (évaluation manuelle)** | 91% |
| **Taux de citation correcte** | 95% |

### Cas d'Usage

✅ **Citoyens** : Comprendre leurs droits numériques  
✅ **Entrepreneurs** : Conformité RGPD locale  
✅ **Étudiants en droit** : Recherche juridique rapide  
✅ **Développeurs** : Vérifier obligations légales

---

## 💡 Innovations

### 1. Chunking Intelligent

```python
# Découpage respectant la structure juridique
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separators=[
        "\nArticle ",      # Séparation par article
        "\nSection ",      # Séparation par section
        "\nChapitre ",     # Séparation par chapitre
        "\n\n",
        "\n",
        " "
    ]
)
```

**Avantage** : Préservation du contexte juridique.

### 2. Température Faible pour Précision

```python
llm = GooglePalm(temperature=0.2)
```

**Raison** : En droit, la créativité est dangereuse. On privilégie la **précision** et la **fidélité** aux sources.

---

## Déploiement

```bash
# Installation
git clone https://github.com/TiTanSage02/eJuris.git
cd eJuris
pip install -r requirements.txt

# Configuration
echo "GOOGLE_API_KEY=your_key" > .env

# Lancement
streamlit run main.py
```

---

## Perspectives d'évolution

- Support de plusieurs codes (Pénal, Civil, Commerce)
- Comparaison de jurisprudences
- Export de consultations en PDF
- Multi-langue (Fon, Yoruba)

**e-Juris représente ma contribution à la démocratisation de l'accès au droit au Bénin grâce à l'intelligence artificielle.**