# 🚀 TP2 — Construire un moteur RAG de support technique

> **Durée** : 2 heures
> **Prérequis** : avoir terminé le [TP1](tp1.md) (toutes les étapes validées)
> **Environnement** : Google Colab (navigateur uniquement)

---

## Ce que vous allez faire

À la fin de ce TP, vous aurez :
- enrichi une base vectorielle avec vos propres documents
- construit un moteur de type RAG (Génération Augmentée par Récupération)
- testé ses limites avec des questions pièges
- implémenté une amélioration concrète pour fiabiliser le système

---

## Avant de commencer

### 1. Créez un nouveau cahier Google Colab

1. Allez sur [colab.research.google.com](https://colab.research.google.com)
2. Créez un **Nouveau notebook**
3. Renommez-le : **`TP2_RAG_NOM_Prenom`**

### 2. Mettez à jour votre document Google Docs

- Collez le **lien de partage** de ce nouveau cahier dans votre document de suivi (section « Lien Colab TP2 »)
- Créez la section **PARTIE 3 — TP2** dans votre document

> À chaque étape marquée 📝, écrivez votre réponse dans votre document Google Docs.

---

## Étape 6 — Reconstruire et enrichir la base (25 min)

### Cellule 1 — Reconstruire la base du TP1

Comme Google Colab ne conserve pas les données entre les sessions, il faut recréer la base. Copiez et exécutez ce code :

```python
# Installation et imports
!pip install chromadb sentence-transformers -q

import chromadb

client = chromadb.Client()
collection = client.create_collection(
    name="faq_support_v2",
    metadata={"hnsw:space": "cosine"}
)

# Les 15 documents du TP1
documents_tp1 = [
    "Pour réinitialiser votre mot de passe, accédez aux paramètres de sécurité du compte et cliquez sur modifier le mot de passe.",
    "Si votre ordinateur ne démarre plus, maintenez le bouton d'alimentation enfoncé pendant 10 secondes puis redémarrez.",
    "L'installation du VPN nécessite de télécharger le client depuis le portail interne et de saisir vos identifiants professionnels.",
    "Pour connecter une imprimante réseau, allez dans les paramètres d'impression et ajoutez une imprimante par adresse IP.",
    "En cas de perte de fichiers, vérifiez la corbeille puis contactez le service de sauvegarde pour une restauration.",
    "La messagerie Outlook se configure avec le serveur IMAP de l'entreprise et nécessite une authentification à deux facteurs.",
    "Pour accéder au réseau Wi-Fi invité, demandez un code temporaire à l'accueil. Ce code expire après 24 heures.",
    "Le logiciel antivirus se met à jour automatiquement. Si une alerte apparaît, ne cliquez pas sur les liens et prévenez le support.",
    "Pour augmenter la mémoire vive de votre poste, faites une demande au service technique avec le numéro d'inventaire.",
    "La visioconférence fonctionne mieux avec un casque filaire USB et une connexion Ethernet plutôt que Wi-Fi.",
    "Si votre écran est noir mais que le voyant est allumé, vérifiez le câble vidéo et essayez un autre port HDMI ou DisplayPort.",
    "Les sauvegardes automatiques sont réalisées chaque nuit à 2h00. La rétention est de 30 jours sur le serveur NAS.",
    "Pour demander un nouveau logiciel, remplissez le formulaire de demande sur l'intranet avec la justification métier.",
    "Le partage de fichiers volumineux se fait via le serveur de fichiers interne, pas par courriel. Limite : 500 Mo par fichier.",
    "En cas de tentative de hameçonnage, ne répondez pas, transférez le courriel à securite@entreprise.fr et supprimez-le."
]

ids_tp1 = [f"faq_{i:03d}" for i in range(len(documents_tp1))]
metas_tp1 = [
    {"categorie": "compte",     "priorite": "haute",   "source": "base"},
    {"categorie": "materiel",   "priorite": "haute",   "source": "base"},
    {"categorie": "reseau",     "priorite": "moyenne",  "source": "base"},
    {"categorie": "materiel",   "priorite": "basse",   "source": "base"},
    {"categorie": "donnees",    "priorite": "haute",   "source": "base"},
    {"categorie": "messagerie", "priorite": "moyenne",  "source": "base"},
    {"categorie": "reseau",     "priorite": "basse",   "source": "base"},
    {"categorie": "securite",   "priorite": "haute",   "source": "base"},
    {"categorie": "materiel",   "priorite": "basse",   "source": "base"},
    {"categorie": "materiel",   "priorite": "basse",   "source": "base"},
    {"categorie": "materiel",   "priorite": "moyenne",  "source": "base"},
    {"categorie": "donnees",    "priorite": "moyenne",  "source": "base"},
    {"categorie": "logiciel",   "priorite": "basse",   "source": "base"},
    {"categorie": "donnees",    "priorite": "basse",   "source": "base"},
    {"categorie": "securite",   "priorite": "haute",   "source": "base"},
]

collection.add(documents=documents_tp1, ids=ids_tp1, metadatas=metas_tp1)
print(f"Base reconstruite : {collection.count()} documents")
```

### Cellule 2 — Ajoutez vos propres documents

Vous allez enrichir la base avec **10 nouvelles fiches de FAQ** que vous rédigez vous-même. C'est une compétence réelle de développeur : en entreprise, la qualité de la base de connaissances conditionne la qualité des résultats.

```python
# === RÉDIGEZ 10 NOUVELLES FICHES DE FAQ ===
#
# Consignes :
# - Chaque fiche doit décrire un problème ou une procédure informatique
# - Rédigez en français professionnel (phrases complètes)
# - Variez les sujets : accès distant, double authentification,
#   mise à jour, droits d'accès, sauvegarde cloud, formation, etc.
# - Chaque fiche doit contenir une SOLUTION ou une PROCÉDURE concrète
#
# NE PAS copier-coller depuis une IA ou depuis un camarade.

mes_documents = [
    "REMPLACEZ PAR VOTRE FICHE 1",
    "REMPLACEZ PAR VOTRE FICHE 2",
    "REMPLACEZ PAR VOTRE FICHE 3",
    "REMPLACEZ PAR VOTRE FICHE 4",
    "REMPLACEZ PAR VOTRE FICHE 5",
    "REMPLACEZ PAR VOTRE FICHE 6",
    "REMPLACEZ PAR VOTRE FICHE 7",
    "REMPLACEZ PAR VOTRE FICHE 8",
    "REMPLACEZ PAR VOTRE FICHE 9",
    "REMPLACEZ PAR VOTRE FICHE 10",
]

# Complétez les métadonnées pour chaque fiche
# Catégories possibles : compte, materiel, reseau, securite,
#                        donnees, messagerie, logiciel
# Priorités possibles : haute, moyenne, basse

mes_metas = [
    {"categorie": "À COMPLÉTER", "priorite": "À COMPLÉTER", "source": "etudiant"},
    {"categorie": "À COMPLÉTER", "priorite": "À COMPLÉTER", "source": "etudiant"},
    {"categorie": "À COMPLÉTER", "priorite": "À COMPLÉTER", "source": "etudiant"},
    {"categorie": "À COMPLÉTER", "priorite": "À COMPLÉTER", "source": "etudiant"},
    {"categorie": "À COMPLÉTER", "priorite": "À COMPLÉTER", "source": "etudiant"},
    {"categorie": "À COMPLÉTER", "priorite": "À COMPLÉTER", "source": "etudiant"},
    {"categorie": "À COMPLÉTER", "priorite": "À COMPLÉTER", "source": "etudiant"},
    {"categorie": "À COMPLÉTER", "priorite": "À COMPLÉTER", "source": "etudiant"},
    {"categorie": "À COMPLÉTER", "priorite": "À COMPLÉTER", "source": "etudiant"},
    {"categorie": "À COMPLÉTER", "priorite": "À COMPLÉTER", "source": "etudiant"},
]

mes_ids = [f"perso_{i:03d}" for i in range(len(mes_documents))]

collection.add(documents=mes_documents, ids=mes_ids, metadatas=mes_metas)
print(f"Base enrichie : {collection.count()} documents")
```

### Cellule 3 — Vérification

```python
# Afficher uniquement vos documents
mes_resultats = collection.get(where={"source": "etudiant"})
print(f"Vos documents : {len(mes_resultats['ids'])}\n")
for i, doc in enumerate(mes_resultats['documents']):
    cat = mes_resultats['metadatas'][i]['categorie']
    pri = mes_resultats['metadatas'][i]['priorite']
    print(f"  [{cat} | {pri}] {doc[:90]}...")
```

Vous devez voir `Base enrichie : 25 documents` et vos 10 fiches s'afficher.

📝 **Étape 6 dans votre document** — Listez les 10 sujets de vos fiches (une ligne par fiche). Pour chacune, indiquez la catégorie et la priorité choisies.

---

## Étape 7 — Construire le moteur RAG (35 min)

Le moteur RAG que vous allez construire suit le schéma vu dans le cours : **Rechercher → Augmenter → Générer**. Pour éviter de dépendre d'une API payante, la partie « Générer » sera simplifiée : nous construirons le message complet qui serait envoyé à une IA, et nous afficherons la réponse la plus pertinente extraite de la base.

### Cellule 4 — Fonction de recherche contextuelle

```python
def extraire_contexte(question, nb_documents=3):
    """
    Étape 1 du RAG : RECHERCHER
    Retrouve les documents les plus pertinents pour répondre à la question.
    """
    resultats = collection.query(
        query_texts=[question],
        n_results=nb_documents
    )
    
    contexte = ""
    sources = []
    for i in range(len(resultats['documents'][0])):
        doc = resultats['documents'][0][i]
        distance = resultats['distances'][0][i]
        similarite = 1 - distance
        meta = resultats['metadatas'][0][i]
        
        contexte += f"[Source {i+1} — {meta['categorie']}, similarité {similarite:.2f}]\n"
        contexte += f"{doc}\n\n"
        sources.append({
            "texte": doc,
            "similarite": similarite,
            "categorie": meta['categorie'],
            "id": resultats['ids'][0][i]
        })
    
    return contexte, sources
```

### Cellule 5 — Le moteur RAG complet

```python
def moteur_rag(question):
    """
    Moteur RAG complet :
    1. RECHERCHER : trouver les documents pertinents
    2. AUGMENTER : construire le message avec le contexte
    3. AFFICHER : présenter les résultats de manière structurée
    """
    contexte, sources = extraire_contexte(question, nb_documents=3)
    
    # --- Étape 2 : AUGMENTER ---
    # Construction du message qui serait envoyé à une IA générative
    message_systeme = (
        "Tu es un assistant de support technique. "
        "Réponds UNIQUEMENT en te basant sur les documents fournis. "
        "Si la réponse ne se trouve pas dans les documents, dis-le clairement. "
        "Ne jamais inventer d'information."
    )
    
    message_utilisateur = (
        f"DOCUMENTS DE RÉFÉRENCE :\n{contexte}\n"
        f"QUESTION : {question}\n\n"
        f"Réponds en citant les numéros de source utilisés."
    )
    
    # --- Étape 3 : AFFICHER ---
    print("╔" + "═" * 68 + "╗")
    print(f"║  QUESTION : {question[:56]:<56} ║")
    print("╠" + "═" * 68 + "╣")
    
    print("║  📚 DOCUMENTS RETROUVÉS :                                        ║")
    for i, s in enumerate(sources):
        sim_str = f"{s['similarite']:.2f}"
        cat_str = s['categorie']
        print(f"║    Source {i+1} [{cat_str:<11}] similarité : {sim_str}                  ║"[:71] + "║")
        texte_court = s['texte'][:60]
        print(f"║    → {texte_court}...  ║"[:71] + "║")
    
    print("╠" + "═" * 68 + "╣")
    
    # Réponse basée sur la meilleure source
    meilleure = sources[0]
    print("║  💡 RÉPONSE PROPOSÉE :                                           ║")
    
    # Découper la réponse en lignes de 64 caractères
    reponse = meilleure['texte']
    for j in range(0, len(reponse), 64):
        ligne = reponse[j:j+64]
        print(f"║    {ligne:<64} ║")
    
    print("║                                                                    ║"[:71] + "║")
    print(f"║    Confiance : {meilleure['similarite']:.0%} — Source : {meilleure['id']:<30}    ║"[:71] + "║")
    
    if meilleure['similarite'] < 0.3:
        print("║                                                                    ║"[:71] + "║")
        print("║  ⚠️  ATTENTION : confiance faible — la base ne contient           ║")
        print("║  probablement pas de réponse. Orientez vers un humain.            ║")
    
    print("╚" + "═" * 68 + "╝")
    print()
    
    return sources
```

### Cellule 6 — Premiers tests du moteur

```python
# Test 1 — Question courante
moteur_rag("Comment faire quand je reçois un mail bizarre ?")

# Test 2 — Question avec reformulation
moteur_rag("Mon ordinateur est très lent depuis ce matin")

# Test 3 — Question sur le partage de fichiers
moteur_rag("Comment partager un dossier de 2 Go avec mon équipe ?")
```

### Cellule 7 — Tests avec questions pièges

```python
# Test 4 — Question dont la réponse N'EST PAS dans la base
moteur_rag("Quel est le numéro de téléphone du support ?")

# Test 5 — Question hors sujet total
moteur_rag("Comment installer Python sur mon poste ?")

# Test 6 — Question ambiguë
moteur_rag("Ça ne marche pas")
```

📝 **Étape 7 dans votre document** — Pour chaque test (1 à 6), notez :
- le score de confiance du meilleur résultat
- si la réponse proposée est pertinente (oui / partiellement / non)
- pour les tests 4, 5 et 6 : expliquez en une phrase pourquoi le système se trompe ou manque d'information

---

## Étape 8 — Évaluer la qualité du système (25 min)

Vous allez maintenant prendre du recul et évaluer votre moteur RAG comme le ferait un testeur en entreprise.

### Cellule 8 — Votre propre batterie de tests

```python
# === RÉDIGEZ 5 QUESTIONS DE TEST ===
#
# Consignes :
# - 2 questions auxquelles la base DEVRAIT bien répondre
# - 2 questions auxquelles la base répondra MAL (questions pièges)
# - 1 question volontairement ambiguë
#
# Pour chaque question, notez AVANT d'exécuter le code ce que vous
# attendez comme résultat (dans votre document Google Docs).

mes_questions = [
    "REMPLACEZ PAR VOTRE QUESTION 1 (réponse attendue : bonne)",
    "REMPLACEZ PAR VOTRE QUESTION 2 (réponse attendue : bonne)",
    "REMPLACEZ PAR VOTRE QUESTION 3 (réponse attendue : mauvaise)",
    "REMPLACEZ PAR VOTRE QUESTION 4 (réponse attendue : mauvaise)",
    "REMPLACEZ PAR VOTRE QUESTION 5 (ambiguë)",
]

for q in mes_questions:
    moteur_rag(q)
```

📝 **Étape 8 dans votre document** — Remplissez ce tableau :

```
| # | Ma question | Résultat attendu | Résultat obtenu | Analyse |
|---|-------------|-------------------|-----------------|---------|
| 1 | ...         | Bonne réponse     | ...             | ...     |
| 2 | ...         | Bonne réponse     | ...             | ...     |
| 3 | ...         | Mauvaise réponse  | ...             | ...     |
| 4 | ...         | Mauvaise réponse  | ...             | ...     |
| 5 | ...         | Ambiguë           | ...             | ...     |
```

Puis répondez à ces questions :

**8.1 — Faiblesses identifiées** : listez 3 faiblesses du moteur RAG que vos tests ont révélées. Pour chaque faiblesse, donnez la question qui l'a mise en évidence.

**8.2 — Améliorations proposées** : pour chaque faiblesse, proposez une solution technique ou organisationnelle (en 2-3 lignes par faiblesse).

---

## Étape 9 — Implémenter une amélioration (20 min)

Vous allez maintenant **coder** une amélioration concrète. Choisissez **une** des trois options suivantes.

### Option A — Seuil de confiance

Si la similarité du meilleur résultat est trop faible, le système affiche un message d'avertissement au lieu de proposer une réponse potentiellement fausse.

```python
def moteur_rag_avec_seuil(question, seuil=0.4):
    """
    Version améliorée : refuse de répondre si la confiance est trop faible.
    """
    contexte, sources = extraire_contexte(question, nb_documents=3)
    meilleure = sources[0]
    
    print(f"🔍 Question : « {question} »")
    print(f"   Meilleur score : {meilleure['similarite']:.2f}")
    print()
    
    if meilleure['similarite'] < seuil:
        print(f"   ⛔ Confiance insuffisante (< {seuil})")
        print(f"   → Je ne peux pas répondre de manière fiable.")
        print(f"   → Veuillez contacter le support au 01 23 45 67 89.")
    else:
        print(f"   ✅ Confiance suffisante ({meilleure['similarite']:.2f} ≥ {seuil})")
        print(f"   → {meilleure['texte']}")
        print(f"   [Source : {meilleure['id']}]")
    print()

# Testez avec une question normale et une question hors sujet
moteur_rag_avec_seuil("Mon imprimante ne fonctionne plus")
moteur_rag_avec_seuil("Quelle est la météo à Paris ?")
moteur_rag_avec_seuil("Comment changer mon mot de passe ?")
```

### Option B — Détection de doublons avant insertion

Avant d'ajouter un document, le système vérifie s'il n'en existe pas déjà un très similaire.

```python
def ajouter_si_nouveau(texte, categorie, priorite, seuil_doublon=0.9):
    """
    Ajoute un document seulement s'il n'existe pas déjà un document
    très similaire dans la base.
    """
    # Vérifier si un document similaire existe
    resultats = collection.query(
        query_texts=[texte],
        n_results=1
    )
    
    if resultats['documents'][0]:
        distance = resultats['distances'][0][0]
        similarite = 1 - distance
        doc_existant = resultats['documents'][0][0]
        
        if similarite > seuil_doublon:
            print(f"⚠️  Doublon détecté (similarité : {similarite:.2f})")
            print(f"   Document existant : {doc_existant[:80]}...")
            print(f"   → Document NON ajouté.")
            return False
    
    # Pas de doublon : on ajoute
    nouvel_id = f"auto_{collection.count():03d}"
    collection.add(
        documents=[texte],
        ids=[nouvel_id],
        metadatas=[{"categorie": categorie, "priorite": priorite, "source": "auto"}]
    )
    print(f"✅ Document ajouté (id: {nouvel_id})")
    return True

# Tests
print("--- Test 1 : document nouveau ---")
ajouter_si_nouveau(
    "Pour configurer le proxy, allez dans les paramètres réseau du navigateur.",
    "reseau", "moyenne"
)

print("\n--- Test 2 : doublon probable ---")
ajouter_si_nouveau(
    "Réinitialisation du mot de passe : rendez-vous dans les paramètres de sécurité.",
    "compte", "haute"
)

print(f"\nTotal documents : {collection.count()}")
```

### Option C — Journal d'audit

Chaque recherche est enregistrée dans un journal pour assurer la traçabilité.

```python
from datetime import datetime

# Journal de toutes les recherches
journal = []

def moteur_rag_avec_journal(question):
    """
    Version améliorée : enregistre chaque recherche dans un journal.
    """
    contexte, sources = extraire_contexte(question, nb_documents=3)
    meilleure = sources[0]
    
    # Enregistrer dans le journal
    entree = {
        "horodatage": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        "question": question,
        "meilleur_score": round(meilleure['similarite'], 3),
        "meilleure_source": meilleure['id'],
        "categorie": meilleure['categorie'],
        "nb_resultats": len(sources)
    }
    journal.append(entree)
    
    # Afficher la réponse
    print(f"🔍 [{entree['horodatage']}] « {question} »")
    print(f"   → {meilleure['texte'][:100]}...")
    print(f"   Score : {meilleure['similarite']:.2f} | Source : {meilleure['id']}")
    print()

# Faire quelques recherches
moteur_rag_avec_journal("Mon écran ne s'allume plus")
moteur_rag_avec_journal("Comment accéder au VPN ?")
moteur_rag_avec_journal("Recette de gâteau au chocolat")

# Afficher le journal complet
print("=" * 60)
print("JOURNAL D'AUDIT")
print("=" * 60)
for e in journal:
    score_emoji = "✅" if e['meilleur_score'] > 0.4 else "⚠️"
    print(f"  {score_emoji} {e['horodatage']} | Score {e['meilleur_score']:.2f} | {e['question'][:40]}")
```

📝 **Étape 9 dans votre document** — Répondez à ces questions :

**9.1 — Choix** : quelle option avez-vous choisie (A, B ou C) et pourquoi ?

**9.2 — Test** : décrivez le test que vous avez réalisé et son résultat.

**9.3 — Limite** : quelle faiblesse du système votre amélioration ne résout-elle **pas** ?

---

## Étape 10 — Bilan final (15 min)

📝 **Étape 10 dans votre document** — Répondez à ces 4 questions de synthèse :

**10.1 — Architecture RAG** : décrivez en vos propres mots les 3 étapes de l'architecture RAG. Pour chaque étape, expliquez le rôle de la base vectorielle.

**10.2 — Avantage du RAG** : quel est l'avantage principal d'un chatbot RAG par rapport à un chatbot classique qui s'appuie uniquement sur sa mémoire d'entraînement ?

**10.3 — Posture professionnelle** : vous êtes recruté comme développeur dans une entreprise qui veut déployer un chatbot de support interne basé sur du RAG. Quelles 3 questions posez-vous avant de commencer le développement ?

**10.4 — Réflexion critique** : le moteur RAG que vous avez construit retourne toujours une réponse, même quand il ne devrait pas. En quoi est-ce un problème dans un contexte professionnel ? Proposez une règle que vous ajouteriez au cahier des charges d'un tel projet.

---

## Checklist de fin de TP2

Avant de quitter, vérifiez que vous avez bien :

- [ ] Un cahier Colab **`TP2_RAG_NOM_Prenom`** avec toutes les cellules exécutées sans erreur
- [ ] Le lien de votre cahier Colab dans votre document Google Docs
- [ ] Les réponses aux 5 étapes dans votre document Google Docs :
  - [ ] Étape 6 : liste des 10 fiches + catégories + priorités
  - [ ] Étape 7 : tableau des 6 tests avec analyse
  - [ ] Étape 8 : tableau des 5 questions personnelles + 3 faiblesses + 3 améliorations
  - [ ] Étape 9 : choix d'amélioration + test + limite identifiée
  - [ ] Étape 10 : bilan final (4 questions de synthèse)

---

## Pour aller plus loin (facultatif)

Si vous avez terminé en avance, voici des pistes d'approfondissement :

- **Connecter une vraie IA** : remplacez la réponse simplifiée par un appel à l'API Mistral (gratuite pour les petits volumes) ou à Ollama (modèle local)
- **Interface web** : créez une interface Gradio ou Streamlit pour interroger votre moteur RAG depuis un navigateur
- **Base persistante** : configurez ChromaDB avec un stockage sur disque (`PersistentClient`) pour que la base survive au redémarrage de Colab
- **Évaluation automatique** : créez un jeu de test de 20 questions avec les réponses attendues et calculez le taux de bonnes réponses de votre moteur
