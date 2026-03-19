# 🔧 TP1 — Construire et interroger une base vectorielle

> **Durée** : 2 heures
> **Prérequis** : avoir lu le [cours](cours.md) et répondu à l'auto-évaluation (≥ 6/8)
> **Environnement** : Google Colab (navigateur uniquement, aucune installation locale)

---

## Ce que vous allez faire

À la fin de ce TP, vous aurez :
- créé une base vectorielle en Python avec ChromaDB
- transformé du texte en vecteurs
- effectué des recherches par similarité sémantique
- comparé les résultats avec une recherche SQL classique
- identifié les limites de la recherche vectorielle

---

## Avant de commencer

### 1. Créez votre cahier Google Colab

1. Allez sur [colab.research.google.com](https://colab.research.google.com)
2. Cliquez sur **Nouveau notebook**
3. Renommez-le : **`TP1_VECTO_NOM_Prenom`**
4. Vérifiez que vous êtes bien en **Python 3** (menu Exécution → Modifier le type d'exécution)

### 2. Mettez à jour votre document Google Docs

- Collez le **lien de partage** de votre cahier Colab dans votre document de suivi (section « Lien Colab TP1 »)
- Vérifiez que votre enseignant a bien accès au document de suivi en **Commentateur**

> À chaque étape marquée 📝, vous devez écrire votre réponse dans votre document Google Docs, section **PARTIE 2 — TP1**.

---

## Étape 1 — Mise en place de l'environnement (15 min)

### Cellule 1 — Installation des bibliothèques

Copiez ce code dans la première cellule de votre cahier Colab et exécutez-la (bouton ▶ ou `Ctrl+Entrée`).

```python
# Installation des bibliothèques
# chromadb : base vectorielle légère, fonctionne sans serveur
# sentence-transformers : modèles de transformation texte → vecteur

!pip install chromadb sentence-transformers -q

# Vérification
import chromadb
from sentence_transformers import SentenceTransformer

print(f"ChromaDB version : {chromadb.__version__}")
print("Installation réussie !")
```

L'installation prend environ 1-2 minutes. Si vous voyez des avertissements en orange, c'est normal — seules les erreurs en rouge sont bloquantes.

### Cellule 2 — Chargement du modèle de transformation

```python
# Chargement du modèle all-MiniLM-L6-v2
# - Modèle libre et gratuit
# - Taille : ~80 Mo (téléchargé automatiquement)
# - Produit des vecteurs de 384 dimensions

modele = SentenceTransformer('all-MiniLM-L6-v2')

# Test : transformer une phrase en vecteur
vecteur_test = modele.encode("Bonjour, ceci est un test")
print(f"Dimensions du vecteur : {len(vecteur_test)}")
print(f"5 premières valeurs : {vecteur_test[:5]}")
```

Le téléchargement du modèle prend environ 30 secondes. Vous devriez voir s'afficher `Dimensions du vecteur : 384`.

### Cellule 3 — Vérification avec deux phrases

```python
# Vérifions que le modèle capture bien le sens
import numpy as np

phrase_a = "Le réseau ne fonctionne pas"
phrase_b = "La connexion internet est coupée"
phrase_c = "La commande de fournitures est en retard"

vec_a = modele.encode(phrase_a)
vec_b = modele.encode(phrase_b)
vec_c = modele.encode(phrase_c)

# Calcul de la similarité cosinus entre les phrases
def similarite_cosinus(v1, v2):
    return np.dot(v1, v2) / (np.linalg.norm(v1) * np.linalg.norm(v2))

sim_ab = similarite_cosinus(vec_a, vec_b)
sim_ac = similarite_cosinus(vec_a, vec_c)

print(f"Similarité entre A et B : {sim_ab:.3f}")
print(f"  A = \"{phrase_a}\"")
print(f"  B = \"{phrase_b}\"")
print()
print(f"Similarité entre A et C : {sim_ac:.3f}")
print(f"  A = \"{phrase_a}\"")
print(f"  C = \"{phrase_c}\"")
```

📝 **Étape 1 dans votre document** — Notez les deux scores de similarité obtenus. Le résultat est-il cohérent avec ce que vous attendiez ? Expliquez en 2 phrases.

---

## Étape 2 — Créer une base vectorielle (20 min)

Vous allez créer une base vectorielle qui contient la FAQ d'un service de support informatique. C'est exactement le type de base que vous pourriez être amené à construire en stage ou en entreprise.

### Cellule 4 — Création de la base et insertion des documents

```python
# Initialisation de ChromaDB (stockage en mémoire)
client = chromadb.Client()

# Création d'une collection (équivalent d'une "table" en SQL)
collection = client.create_collection(
    name="faq_support",
    metadata={"hnsw:space": "cosine"}  # On utilise la similarité cosinus
)

# 15 documents de FAQ — un service de support informatique typique
documents = [
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

# Identifiants uniques (comme une clé primaire en SQL)
identifiants = [f"faq_{i:03d}" for i in range(len(documents))]

# Métadonnées (comme des colonnes supplémentaires en SQL)
metadonnees = [
    {"categorie": "compte",     "priorite": "haute"},
    {"categorie": "materiel",   "priorite": "haute"},
    {"categorie": "reseau",     "priorite": "moyenne"},
    {"categorie": "materiel",   "priorite": "basse"},
    {"categorie": "donnees",    "priorite": "haute"},
    {"categorie": "messagerie", "priorite": "moyenne"},
    {"categorie": "reseau",     "priorite": "basse"},
    {"categorie": "securite",   "priorite": "haute"},
    {"categorie": "materiel",   "priorite": "basse"},
    {"categorie": "materiel",   "priorite": "basse"},
    {"categorie": "materiel",   "priorite": "moyenne"},
    {"categorie": "donnees",    "priorite": "moyenne"},
    {"categorie": "logiciel",   "priorite": "basse"},
    {"categorie": "donnees",    "priorite": "basse"},
    {"categorie": "securite",   "priorite": "haute"},
]

# Insertion dans la base vectorielle
collection.add(
    documents=documents,
    ids=identifiants,
    metadatas=metadonnees
)

print(f"Documents insérés : {collection.count()}")
print(f"Catégories : {sorted(set(m['categorie'] for m in metadonnees))}")
```

Vous devriez voir `Documents insérés : 15`.

📝 **Étape 2 dans votre document** — Combien de catégories différentes sont présentes dans la base ? Listez-les. Quel parallèle faites-vous entre les `metadonnees` et une table SQL ?

---

## Étape 3 — Recherche vectorielle vs recherche par mots-clés (35 min)

C'est le cœur du TP : vous allez voir concrètement la différence entre une recherche SQL classique et une recherche vectorielle.

### Cellule 5 — Fonction de recherche vectorielle

```python
def recherche_vectorielle(question, nb_resultats=3):
    """Recherche les documents les plus proches sémantiquement."""
    resultats = collection.query(
        query_texts=[question],
        n_results=nb_resultats
    )
    
    print(f"🔍 Question : « {question} »")
    print(f"   {nb_resultats} résultats trouvés :\n")
    
    for i in range(len(resultats['documents'][0])):
        doc = resultats['documents'][0][i]
        distance = resultats['distances'][0][i]
        meta = resultats['metadatas'][0][i]
        # ChromaDB retourne la distance cosinus (0 = identique, 2 = opposé)
        similarite = 1 - distance
        print(f"   Résultat {i+1} — similarité : {similarite:.2f} [{meta['categorie']}]")
        print(f"   → {doc}")
        print()
```

### Cellule 6 — Fonction de recherche par mots-clés (simule SQL)

```python
def recherche_mots_cles(mot_cle):
    """Simule une recherche SQL : WHERE description LIKE '%mot%'"""
    resultats = []
    for i, doc in enumerate(documents):
        if mot_cle.lower() in doc.lower():
            resultats.append(doc)
    
    print(f"🔍 Mot-clé : « {mot_cle} »")
    if resultats:
        print(f"   {len(resultats)} résultat(s) trouvé(s) :\n")
        for doc in resultats:
            print(f"   → {doc}")
            print()
    else:
        print("   ❌ Aucun résultat trouvé.\n")
```

### Cellule 7 — Comparaison directe

```python
# === TEST 1 : L'utilisateur décrit un problème avec ses propres mots ===
print("=" * 70)
print("TEST 1 — L'utilisateur dit : « J'ai oublié mon code d'accès »")
print("=" * 70)
print()

print("--- Recherche par mots-clés ---")
recherche_mots_cles("code d'accès")

print("--- Recherche vectorielle ---")
recherche_vectorielle("J'ai oublié mon code d'accès")
```

### Cellule 8 — Vos expérimentations

```python
# === TEST 2 à 6 : Testez ces requêtes et observez les résultats ===

# TEST 2 — Reformulation d'un problème matériel
print("=" * 70)
print("TEST 2")
print("=" * 70)
recherche_mots_cles("PC ne s'allume plus")
recherche_vectorielle("Mon PC ne s'allume plus, que faire ?")

# TEST 3 — Question en langage courant
print("=" * 70)
print("TEST 3")
print("=" * 70)
recherche_mots_cles("envoyer gros fichier")
recherche_vectorielle("Comment envoyer un gros fichier à un collègue ?")

# TEST 4 — Description par symptômes
print("=" * 70)
print("TEST 4")
print("=" * 70)
recherche_mots_cles("écran reste noir")
recherche_vectorielle("L'écran reste noir quand j'appuie sur le bouton")

# TEST 5 — Question ambiguë
print("=" * 70)
print("TEST 5")
print("=" * 70)
recherche_vectorielle("J'ai un problème de sécurité")

# TEST 6 — Requête HORS SUJET (attention à ce qui se passe)
print("=" * 70)
print("TEST 6 — HORS SUJET")
print("=" * 70)
recherche_vectorielle("Quelle est la recette de la quiche lorraine ?")
```

📝 **Étape 3 dans votre document** — Remplissez le tableau suivant pour chaque test :

```
| Test | Requête résumée         | Résultat mots-clés | Résultat vectoriel | Pertinent ? |
|------|-------------------------|--------------------|--------------------|-------------|
| 1    | Code d'accès oublié     | ...                | ...                | ...         |
| 2    | PC ne s'allume plus     | ...                | ...                | ...         |
| 3    | Envoyer gros fichier    | ...                | ...                | ...         |
| 4    | Écran reste noir        | ...                | ...                | ...         |
| 5    | Problème de sécurité    | ...                | ...                | ...         |
| 6    | Recette quiche lorraine | (pas testé)        | ...                | ...         |
```

Puis répondez à cette question : **Le test 6 (hors sujet) a-t-il quand même retourné des résultats ? Si oui, quel est le score de similarité du meilleur résultat ? Pourquoi est-ce problématique pour une application en production ?**

---

## Étape 4 — Filtrage par métadonnées (20 min)

En SQL, vous combinez souvent des conditions : `WHERE categorie = 'réseau' AND priorite = 'haute'`. Les bases vectorielles permettent la même chose : recherche sémantique **+ filtrage** sur les métadonnées.

### Cellule 9 — Recherche filtrée par catégorie

```python
# Recherche sémantique limitée à la catégorie "securite"
print("=== Recherche filtrée : catégorie 'securite' uniquement ===\n")

resultats = collection.query(
    query_texts=["Je pense avoir reçu un virus"],
    n_results=3,
    where={"categorie": "securite"}  # Filtre sur les métadonnées
)

for i in range(len(resultats['documents'][0])):
    doc = resultats['documents'][0][i]
    distance = resultats['distances'][0][i]
    similarite = 1 - distance
    print(f"  Résultat {i+1} — similarité : {similarite:.2f}")
    print(f"  → {doc}\n")
```

### Cellule 10 — Recherche filtrée par priorité

```python
# Recherche sémantique limitée aux problèmes de priorité haute
print("=== Problèmes prioritaires pour 'perte de données' ===\n")

resultats = collection.query(
    query_texts=["J'ai perdu des fichiers importants"],
    n_results=3,
    where={"priorite": "haute"}
)

for i in range(len(resultats['documents'][0])):
    doc = resultats['documents'][0][i]
    meta = resultats['metadatas'][0][i]
    distance = resultats['distances'][0][i]
    similarite = 1 - distance
    print(f"  [{meta['categorie']}] similarité : {similarite:.2f}")
    print(f"  → {doc}\n")
```

### Cellule 11 — Comparaison avec et sans filtre

```python
# Même question, AVEC et SANS filtre — observez la différence
print("=== SANS filtre ===\n")
recherche_vectorielle("Je pense avoir reçu un virus")

print("=== AVEC filtre (catégorie = securite) ===\n")
resultats = collection.query(
    query_texts=["Je pense avoir reçu un virus"],
    n_results=3,
    where={"categorie": "securite"}
)
for i in range(len(resultats['documents'][0])):
    doc = resultats['documents'][0][i]
    distance = resultats['distances'][0][i]
    similarite = 1 - distance
    print(f"  Résultat {i+1} — similarité : {similarite:.2f}")
    print(f"  → {doc}\n")
```

📝 **Étape 4 dans votre document** — Décrivez un cas d'utilisation professionnel concret (en 4-5 lignes) dans lequel le filtrage par métadonnées combiné à la recherche vectorielle serait indispensable. Exemple : dans une application de support multi-sites, filtrer par site géographique avant de chercher la solution.

---

## Étape 5 — Bilan du TP1 (10 min)

📝 **Étape 5 dans votre document** — Répondez à ces 4 questions :

**5.1 — Ce que j'ai compris** : listez 3 concepts que vous maîtrisez maintenant.

**5.2 — Ce qui reste flou** : notez 1 ou 2 points que vous aimeriez approfondir.

**5.3 — SQL vs vectoriel** : en une phrase, dans quel cas choisiriez-vous une base vectorielle plutôt qu'une base SQL ?

**5.4 — Limites** : donnez un exemple concret dans lequel faire confiance aveuglément aux résultats de la recherche vectorielle serait problématique.

---

## Checklist de fin de TP1

Avant de quitter, vérifiez que vous avez bien :

- [ ] Un cahier Colab **`TP1_VECTO_NOM_Prenom`** avec toutes les cellules exécutées sans erreur
- [ ] Le lien de votre cahier Colab dans votre document Google Docs
- [ ] Les réponses aux 5 étapes dans votre document Google Docs :
  - [ ] Étape 1 : scores de similarité + commentaire
  - [ ] Étape 2 : catégories + parallèle SQL
  - [ ] Étape 3 : tableau des 6 tests + analyse du test hors sujet
  - [ ] Étape 4 : cas d'utilisation professionnel du filtrage
  - [ ] Étape 5 : bilan personnel (4 questions)

---

➡️ **Suite** : [TP2 — Construire un moteur RAG de support technique](tp2.md)
