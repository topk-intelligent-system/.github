

# Business Intelligence for E-commerce – Automated System
<img width="1366" height="653" alt="image" src="https://github.com/user-attachments/assets/a47f6cfb-72ef-4003-83c6-e5143c1ed105" />


## Introduction Générale

Ce projet a pour objectif de développer un **système d'intelligence économique (Business Intelligence) avancé et automatisé**, spécifiquement conçu pour le secteur du **e-commerce**.
Le système est capable de :

-   Collecter des données produits depuis des plateformes e-commerce.
    
-   Identifier les produits à fort potentiel (Top-K).
    
-   Orchestrer les étapes via des pipelines de Machine Learning et de Data Mining.
    
-   Enrichir les analyses grâce aux **LLM** (Large Language Models).
    
-   Présenter les résultats dans un **dashboard interactif**.
    

Une réflexion sur l’**architecture responsable des agents** (basée sur le **Model Context Protocol – MCP d’Anthropic**) est également intégrée.

**Ambition** : fournir une solution **de bout en bout** allant de la collecte brute jusqu’aux insights exploitables, en utilisant des approches modernes **MLOps**.

## Étape 1 : Scraping de Données – Agents A2A

### Objectif

Extraire automatiquement et de manière structurée les données produits depuis des plateformes e-commerce (Shopify en priorité).

### Architecture et Concepts

-   **Agent A2A (Agent-to-Agent)** : agent dédié à chaque plateforme (Shopify, etc.), responsable du crawling et du scraping.
    
-   **Modularité** : architecture orientée classes (`base_agent.py`, `shopify_api_agent.py`).
    
-   **Persistance** : stockage dans **MongoDB** (NoSQL).
    
-   **Robustesse** : gestion d’erreurs, pagination, rate limiting.
    

### Outils et Technologies Utilisés

-   Python 3.8+
    
-   Requests
    
-   Pymongo
    
-   Loguru
    
-   python-dotenv
    
-   MongoDB (via Docker recommandé)
    
-   Docker (conteneurisation et Kubeflow)
    

### Implémentation et Fonctionnalités

-   Extraction des produits via `/products.json` (plus stable que HTML).
    
-   Pagination automatique.
    
-   Stockage structuré dans MongoDB avec **timestamps** et **index**.
    
-   Export CSV automatique (`data/shopify_products.csv`).
    
-   Script `check_data.py` pour contrôler l’état des données.
    

### Défis et Limitations

-   L’endpoint `/products.json` peut être désactivé.
    
-   Variabilité des structures de données selon le thème Shopify.
    
-   WooCommerce écarté à ce stade (API REST + HTML scraping trop complexe).


<img width="1035" height="625" alt="image" src="https://github.com/user-attachments/assets/1c2c4f03-1b6f-4a6c-802f-f2c65bef01d5" />


----------

## Étape 2 : Analyse et Sélection des Top-K Produits

### Objectif

Attribuer un **score** aux produits pour identifier les plus attractifs.

### Méthodologie

1.  **Chargement des données** depuis MongoDB → Pandas DataFrame.
    
2.  **Calcul de métriques** : prix moyen, ratio de disponibilité, nombre d’images, longueur de description, popularité simulée.
    
3.  **Normalisation** avec `MinMaxScaler`.
    
4.  **Scoring pondéré** :


		score = (
		    w1 * available_ratio_scaled
		  + w2 * num_tags_scaled
		  + w3 * (1 - avg_price_scaled)
		  + w4 * num_images_scaled
		  + w5 * description_length_scaled
		  + w6 * popularity_score_scaled
		)




5.  **Sélection Top-K** : tri par score décroissant.
  <img width="1077" height="440" alt="image" src="https://github.com/user-attachments/assets/b176cd17-9698-4d6f-9607-408d5075ca9e" />

6.  **Visualisations** : heatmap, PCA, clustering K-Means.

   
    <img width="800" height="600" alt="image" src="https://github.com/user-attachments/assets/ec9175cb-c17b-4fcd-9dd9-9ad3e8b02046" />


    <img width="800" height="600" alt="image" src="https://github.com/user-attachments/assets/025d03ba-7dac-4943-bc24-0951e6b85e33" />

    <img width="800" height="600" alt="image" src="https://github.com/user-attachments/assets/85794a6c-05e2-4d64-9cb2-b932aba7693a" />


    ### Outils et Technologies Utilisés

-   Python, Pandas, NumPy
    
-   Scikit-learn (PCA, KMeans)
    
-   Matplotlib, Seaborn
    
-   Plotly Express (Streamlit dashboard)
    

### Résultats et Sorties

-   `outputs/products_from_mongo.csv`
    
-   `outputs/top_k.csv`
    
-   Visualisations : `heatmap_correlation.png`, `pca_clusters.png`, `pca_topk.png`

## Étape 3 : Conteneurisation et Déploiement Orchestré (Préparation pour Kubeflow Pipelines)

### Objectif

Conteneuriser les services (**scraping, analyse, enrichissement LLM**) avec **Docker** et déployer sur un cluster **Kubernetes (Minikube)**.

### Conteneurisation avec Docker

-   Images basées sur `python:3.10-slim`.
    
-   Installation via `requirements.txt`.
    
-   Exposition des scripts via `ENTRYPOINT`/`CMD`.
    

### Déploiement sur Kubernetes avec Minikube

-   **Deployments** : gèrent la scalabilité des Pods.
    
-   **Services** : exposent les API internes/externes.
    
-   **ConfigMaps & Secrets** : gestion de la configuration et credentials.
    
-   **PVC & PV** : persistance des données.
    

### Services conteneurisés

-   `scraper-agents`
    
-   `topk-analysis`
    
-   `llm-summarizer-agent`
    
-   `mcp-architecture` (conceptuel)
    

### Gestion des Volumes et de la Persistance des Données

-   MongoDB hébergée en dehors du cluster pour cette étape.
    
-   Fichiers CSV sauvegardés localement → dans KFP ils deviennent des **artefacts MinIO**.
    

### Surveillance et Maintenance (dans Minikube)

-   Logs via `kubectl logs <pod-name>`.
    
-   Redémarrage automatique des Pods en cas d’échec.
    
-   Interface `minikube dashboard` pour visualiser l’état.



    <img width="412" height="199" alt="image" src="https://github.com/user-attachments/assets/69db12bc-4150-43df-a43b-1fc501a68cbd" />


### Conclusion Partielle et Transition vers Kubeflow Pipelines

Les services sont prêts pour être intégrés dans un pipeline ML avec **Kubeflow Pipelines (KFP)** :

-   Définir les scripts comme **composants KFP**.
    
-   Créer un **pipeline global** via `@dsl.pipeline`.
    
-   Compiler et exécuter dans KFP (standalone sur Minikube).
    
-   Gérer les secrets et artefacts (ex: Hugging Face API, MongoDB URI).


