# 🗃️ Atelier — Bases de données vectorielles

**BTS SIO option SLAM — 2ème année**

---

## De quoi parle cet atelier ?

Vous savez interroger une base de données SQL avec des requêtes précises. Mais comment faire quand un utilisateur pose une question en langage naturel, avec ses propres mots, sans connaître les termes exacts stockés en base ?

Les **bases de données vectorielles** répondent à ce problème : elles comparent le **sens** des textes, pas leur orthographe. C'est la technologie derrière les moteurs de recherche intelligents, les chatbots d'entreprise et les systèmes de recommandation.

Dans cet atelier, vous allez **construire de vos propres mains** un moteur de recherche sémantique, puis un assistant de support technique capable de retrouver la bonne réponse dans une base de connaissances.

---

## Organisation

| | Contenu | Durée | Fichier |
|---|---------|-------|---------|
| 📖 | **Cours — Les concepts clés** | ~45 min | [cours.md](cours.md) |
| ✅ | **Auto-évaluation** | ~15 min | Intégrée en fin de cours |
| 🔧 | **TP1 — Construire et interroger une base vectorielle** | 2h | [tp1.md](tp1.md) |
| 🚀 | **TP2 — Construire un moteur RAG de support technique** | 2h | [tp2.md](tp2.md) |

---

## Ce dont vous avez besoin

- Un **navigateur web** (Chrome, Firefox, Edge)
- Un **compte Google** (pour Google Colab et Google Docs)
- **Aucune installation** sur votre poste — tout se fait à distance

---

## Votre document de suivi

Tout au long de l'atelier, vous répondrez aux questions et noterez vos observations dans un **document Google Docs** personnel.

### Comment le créer

1. Allez sur [docs.google.com](https://docs.google.com) et créez un nouveau document
2. Nommez-le : **`VECTO_NOM_Prenom`** (exemple : `VECTO_DUPONT_Marie`)
3. En haut de votre document, écrivez :
   - Votre **nom et prénom**
   - La **date**
   - Le lien vers votre **cahier Google Colab** du TP1 (vous le créerez au début du TP1)
   - Le lien vers votre **cahier Google Colab** du TP2 (vous le créerez au début du TP2)
4. **Partagez** le document avec votre enseignant (accès **Commentateur**) dès le début de la première séance

> ⚠️ **Important** : ce document est votre fil conducteur. Chaque question marquée 📝 dans le cours et les TP doit recevoir une réponse dans ce document. L'enseignant suit votre progression grâce à lui.

### Structure attendue du document

```
ATELIER BASES DE DONNÉES VECTORIELLES
Nom : ...
Prénom : ...
Date : ...
Lien Colab TP1 : [à compléter]
Lien Colab TP2 : [à compléter]

────────────────────────────────────
PARTIE 1 — COURS : AUTO-ÉVALUATION
────────────────────────────────────
Q1 — ...
Q2 — ...
(...)

────────────────────────────────────
PARTIE 2 — TP1
────────────────────────────────────
Étape 1 — ...
Étape 2 — ...
(...)

────────────────────────────────────
PARTIE 3 — TP2
────────────────────────────────────
Étape 6 — ...
Étape 7 — ...
(...)
```

---

## Liens utiles

- [Google Colab](https://colab.research.google.com) — votre environnement de code Python
- [Documentation ChromaDB](https://docs.trychroma.com/) — la base vectorielle utilisée dans les TP
- [Sentence Transformers](https://www.sbert.net/) — les modèles de transformation texte → vecteur

---

## Prérequis

Avant de commencer, vérifiez que vous maîtrisez :

- ✅ Les bases de SQL : `SELECT`, `WHERE`, `JOIN`, `INSERT`
- ✅ Les fondamentaux de Python : variables, listes, dictionnaires, fonctions, boucles
- ✅ Le principe d'une API REST (requête → réponse)

Si l'un de ces points n'est pas acquis, signalez-le à votre enseignant en début de séance.
