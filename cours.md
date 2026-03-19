# 📖 Cours — Les concepts clés des bases de données vectorielles

> **Durée estimée** : 45 minutes de lecture active
> **Objectif** : comprendre pourquoi et comment on passe du SQL classique aux bases vectorielles
> **À faire** : répondez aux questions 📝 dans votre document Google Docs au fur et à mesure

---

## 1. Le problème de départ : les limites du SQL face au langage naturel

Vous avez l'habitude d'écrire des requêtes SQL pour interroger une base de données. C'est efficace quand on connaît exactement ce qu'on cherche :

```sql
SELECT * FROM tickets WHERE categorie = 'réseau' AND statut = 'ouvert';
```

Cette requête fonctionne parce que les valeurs sont **structurées** et **exactes** : le mot `réseau` dans la requête correspond exactement au mot `réseau` dans la colonne.

Mais imaginez maintenant un utilisateur non-technique qui tape dans un champ de recherche :

> *« internet ne marche plus depuis ce matin »*

Votre requête SQL ne trouvera rien, parce que :
- l'utilisateur a écrit « internet », pas « réseau »
- il a écrit « ne marche plus », pas « panne » ni « incident »
- il a ajouté « depuis ce matin », information temporelle que la base n'exploite pas

**Le problème fondamental** : SQL compare des **chaînes de caractères**, pas des **significations**. Pour un SGBD relationnel, les mots « internet » et « réseau » sont aussi différents que « internet » et « pizza ».

### Le quotidien d'un développeur SLAM face à ce problème

En stage ou en entreprise, vous serez confronté à cette situation. Un client ou un chef de projet vous demandera : *« On voudrait un champ de recherche intelligent dans l'application de gestion des tickets »*. Avec SQL seul, vous pouvez proposer un `LIKE '%mot%'` ou une recherche plein texte (`FULLTEXT`), mais ces approches restent limitées aux mots exacts ou à leurs variantes orthographiques.

C'est là qu'interviennent les **bases de données vectorielles**.

### Schéma comparatif

```
RECHERCHE SQL CLASSIQUE                    RECHERCHE VECTORIELLE
─────────────────────                      ─────────────────────

Utilisateur tape :                         Utilisateur tape :
"internet ne marche plus"                  "internet ne marche plus"
         │                                          │
         ▼                                          ▼
┌─────────────────────┐                   ┌─────────────────────┐
│  WHERE description   │                   │  Transformation en  │
│  LIKE '%internet%'   │                   │  vecteur numérique  │
└─────────────────────┘                   └─────────────────────┘
         │                                          │
         ▼                                          ▼
  Comparaison lettre                        Comparaison du SENS
  par lettre                                avec tous les documents
         │                                          │
         ▼                                          ▼
  ❌ "panne réseau"                         ✅ "panne réseau"
     → pas trouvé                              → trouvé ! (même sens)
  ❌ "connexion coupée"                     ✅ "connexion coupée"
     → pas trouvé                              → trouvé ! (même sens)
  ✅ "internet lent"                        ✅ "internet lent"
     → trouvé (mot exact)                      → trouvé (mot + sens)
```

📝 **Question Q1** — Vous développez une application de gestion de tickets pour un service informatique. Un utilisateur saisit « mon écran est tout noir ». Expliquez en 2-3 phrases pourquoi une recherche `WHERE description LIKE '%écran noir%'` pourrait ne pas retrouver un ticket existant intitulé « Moniteur éteint après mise à jour ».

---

## 2. Les vecteurs d'apprentissage : transformer du texte en nombres

### L'idée centrale

Un **vecteur d'apprentissage** (en anglais : *embedding*) est une liste de nombres décimaux qui représente le **sens** d'un texte. Un modèle d'IA, entraîné sur des milliards de phrases, a appris à placer les textes qui parlent du même sujet **proches les uns des autres** dans un espace mathématique.

Concrètement, le texte `"panne réseau"` est transformé en quelque chose comme :

```
[0.23, -0.41, 0.67, 0.12, -0.55, 0.89, ..., 0.34]
     384 nombres au total (pour le modèle all-MiniLM-L6-v2)
```

Et le texte `"internet coupé"` donne un vecteur **très similaire** :

```
[0.21, -0.39, 0.65, 0.14, -0.53, 0.87, ..., 0.32]
     ↑       ↑      ↑   → valeurs proches = sens proches
```

Tandis que `"recette de cuisine"` donne un vecteur **très différent** :

```
[0.78, 0.55, -0.12, 0.91, 0.33, -0.44, ..., -0.67]
     ↑       ↑       ↑   → valeurs éloignées = sens éloignés
```

### Analogie avec une carte géographique

Pensez à une carte de France. Lyon et Saint-Étienne ont des coordonnées GPS proches parce qu'elles sont géographiquement proches. Paris et Lyon sont plus éloignées. Paris et Tokyo sont très éloignées.

Les vecteurs d'apprentissage fonctionnent exactement pareil, mais dans un espace à 384 dimensions au lieu de 2 :

```
Espace vectoriel simplifié (2D pour l'illustration)
─────────────────────────────────────────────────

          panne réseau ●──── ● connexion coupée
                       │
                       │ (très proches = même sens)
                       │
          wifi cassé ●─┘

                                    ● recette gâteau
                                    │
                                    │ (très proches entre eux,
                                    │  très loin du réseau)
                                    │
                             ● menu du jour ──── ● ingrédients
```

### Ce qu'il faut retenir

- Un vecteur capture le **sens**, pas l'orthographe
- Deux textes peuvent avoir des mots **totalement différents** mais des vecteurs **très proches** (« panne réseau » ↔ « internet coupé »)
- Le modèle a appris ces proximités en lisant des milliards de textes : il sait que « réseau » et « internet » apparaissent souvent dans les mêmes contextes
- Vous n'avez **pas besoin de comprendre les maths** derrière le modèle pour l'utiliser — la bibliothèque Python fait le travail

📝 **Question Q2** — Un vecteur contient 384 nombres. Pourquoi le modèle n'utilise-t-il pas simplement 2 ou 3 nombres (comme des coordonnées GPS) ? Répondez en une phrase.

---

## 3. La similarité cosinus : mesurer la proximité du sens

Une fois les textes transformés en vecteurs, il faut pouvoir **mesurer** à quel point deux vecteurs sont proches. C'est le rôle de la **similarité cosinus**.

### Le principe

La similarité cosinus mesure l'**angle** entre deux vecteurs. Plus l'angle est petit, plus les vecteurs pointent dans la même direction, plus les textes ont un sens proche.

Le résultat est un nombre entre **0 et 1** (en pratique, pour du texte) :

| Score | Signification | Exemple |
|-------|--------------|---------|
| **0.90 – 1.00** | Quasi identique | « panne réseau » ↔ « problème de connexion réseau » |
| **0.70 – 0.89** | Très lié | « panne réseau » ↔ « le wifi ne fonctionne plus » |
| **0.40 – 0.69** | Lien indirect | « panne réseau » ↔ « configurer un routeur » |
| **0.00 – 0.39** | Sans rapport | « panne réseau » ↔ « recette de quiche lorraine » |

### Visualisation simplifiée

```
                    Similarité = 0.95
                    (presque identique)
                          │
           Vecteur A ────►/
                         / angle petit
           Vecteur B ───►/
           
           
                    Similarité = 0.15
                    (sans rapport)
                          │
           Vecteur A ────────►
                         
                              ▲
                              │
           Vecteur B ─────────┘
                         angle grand
```

### En pratique

Vous n'aurez **jamais** à calculer cette formule vous-même. ChromaDB (la base vectorielle que vous utiliserez en TP) le fait automatiquement. Ce qui compte, c'est de savoir **interpréter** le score :

- Un score de **0.85** entre la question d'un utilisateur et un document de votre base → le document est probablement pertinent
- Un score de **0.25** → le document n'a probablement rien à voir → attention avant de le proposer à l'utilisateur

📝 **Question Q3** — Vous construisez un moteur de recherche pour une application de gestion de tickets. À partir de quel seuil de similarité considérez-vous qu'un résultat est assez pertinent pour être affiché à l'utilisateur ? Justifiez votre choix.

---

## 4. Architecture d'une base vectorielle

### Les trois composants

Une base de données vectorielle s'appuie sur trois briques qui travaillent ensemble :

```
┌──────────────────────────────────────────────────────────────────┐
│                    BASE DE DONNÉES VECTORIELLE                   │
│                                                                  │
│  ┌─────────────────┐  ┌──────────────────┐  ┌────────────────┐  │
│  │   1. MODÈLE DE  │  │  2. INDEX        │  │ 3. MÉTADONNÉES │  │
│  │  TRANSFORMATION │  │    VECTORIEL     │  │                │  │
│  │                 │  │                  │  │                │  │
│  │  Texte → Vecteur│  │  Stocke les      │  │ Texte original │  │
│  │                 │  │  vecteurs et     │  │ + infos en plus│  │
│  │  "panne réseau" │  │  retrouve les    │  │ (catégorie,    │  │
│  │       ↓         │  │  plus proches    │  │  date, auteur, │  │
│  │  [0.23, -0.41,  │  │  rapidement      │  │  priorité...)  │  │
│  │   0.67, ...]    │  │                  │  │                │  │
│  └─────────────────┘  └──────────────────┘  └────────────────┘  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

**Composant 1 — Le modèle de transformation** transforme du texte en vecteur. Dans nos TP, nous utiliserons `all-MiniLM-L6-v2`, un modèle libre et gratuit qui produit des vecteurs de 384 dimensions. Il est assez léger pour fonctionner dans Google Colab.

**Composant 2 — L'index vectoriel** stocke les vecteurs et permet de retrouver rapidement les plus proches d'un vecteur donné. Sans cet index, il faudrait comparer le vecteur de la question à **chaque** vecteur de la base — trop lent si on a des millions de documents. Les algorithmes d'indexation (comme HNSW) organisent les vecteurs pour accélérer la recherche.

**Composant 3 — Les métadonnées** sont les informations classiques associées à chaque document : le texte original, mais aussi la catégorie, la date, la priorité, l'auteur... Elles permettent de **filtrer** les résultats après la recherche vectorielle (par exemple : chercher uniquement dans les tickets de la catégorie « réseau »).

### Le parallèle avec ce que vous connaissez en SLAM

| Concept SQL (ce que vous connaissez) | Concept vectoriel (ce que vous allez apprendre) |
|--------------------------------------|------------------------------------------------|
| Table | Collection |
| Ligne (enregistrement) | Document + son vecteur |
| Colonne | Métadonnée |
| `WHERE colonne = 'valeur'` | Filtre sur les métadonnées |
| `SELECT ... WHERE nom LIKE '%mot%'` | Recherche par similarité sémantique |
| Index B-tree (accélère les recherches exactes) | Index HNSW (accélère les recherches par proximité) |
| Clé primaire | Identifiant du document |

### Solutions existantes

| Outil | Type | Avantage | Utilisé en TP ? |
|-------|------|----------|:---:|
| **ChromaDB** | Libre, léger | Fonctionne sans serveur, idéal pour apprendre | ✅ |
| Qdrant | Libre, performant | Adapté à la production | |
| Weaviate | Libre, complet | Filtrage avancé, GraphQL | |
| Pinecone | Service en ligne (payant) | Pas d'infrastructure à gérer | |
| pgvector | Extension PostgreSQL | S'intègre dans un SGBD existant | |

Nous utiliserons **ChromaDB** dans les TP parce qu'il s'installe en une ligne de commande et fonctionne directement dans Google Colab, sans serveur externe.

📝 **Question Q4** — Vous devez intégrer une recherche sémantique dans une application Python existante qui utilise déjà PostgreSQL. Parmi les solutions ci-dessus, laquelle choisiriez-vous et pourquoi ?

---

## 5. Cas d'usage concrets pour un développeur SLAM

Les bases vectorielles ne sont pas un concept théorique. Elles sont déployées dans des applications que vous utilisez peut-être déjà.

### Cas 1 — Recherche intelligente dans une application de support

**Le besoin** : une entreprise a une base de 5 000 fiches de résolution d'incidents. Quand un technicien reçoit un nouvel appel, il veut retrouver les fiches utiles en décrivant le problème avec ses propres mots.

**Sans base vectorielle** : le technicien doit connaître les bons mots-clés, essayer plusieurs combinaisons, parcourir des résultats non pertinents.

**Avec base vectorielle** : le technicien décrit le problème en langage naturel → la base retrouve les fiches dont le **sens** est le plus proche → gain de temps, meilleure résolution.

```
Technicien saisit : "L'utilisateur n'arrive pas à ouvrir ses mails sur son téléphone"

                    ┌──────────────────────────────────────┐
                    │        RECHERCHE VECTORIELLE          │
                    └──────────────────────────────────────┘
                                     │
                    ┌────────────────┼────────────────┐
                    ▼                ▼                ▼
            ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
            │ Fiche #2341  │ │ Fiche #1892  │ │ Fiche #3105  │
            │ Config IMAP  │ │ Outlook      │ │ Certificat   │
            │ mobile       │ │ mobile sync  │ │ SSL expiré   │
            │ Sim: 0.89    │ │ Sim: 0.84    │ │ Sim: 0.71    │
            └──────────────┘ └──────────────┘ └──────────────┘
```

### Cas 2 — Chatbot RAG (Génération Augmentée par Récupération)

**Le besoin** : une entreprise veut un chatbot qui réponde aux questions des employés sur les procédures internes, sans inventer de fausses réponses.

**Le problème des chatbots classiques** : une IA générative (comme ChatGPT) peut « halluciner », c'est-à-dire inventer des réponses plausibles mais fausses, parce qu'elle ne connaît pas vos documents internes.

**La solution RAG** combine une base vectorielle et une IA générative :

```
L'utilisateur pose une question
            │
            ▼
   ┌─────────────────┐
   │  1. RECHERCHER   │  La question est transformée en vecteur.
   │                  │  La base vectorielle retrouve les 3 documents
   │  Base vectorielle│  les plus pertinents.
   └────────┬─────────┘
            │ 3 documents pertinents
            ▼
   ┌─────────────────┐
   │  2. AUGMENTER    │  Les documents sont injectés dans le message
   │                  │  envoyé à l'IA : "Réponds en te basant
   │  Construction du │  UNIQUEMENT sur ces documents."
   │  message         │
   └────────┬─────────┘
            │ message enrichi
            ▼
   ┌─────────────────┐
   │  3. GÉNÉRER      │  L'IA rédige une réponse en s'appuyant
   │                  │  sur les documents fournis.
   │  IA générative   │  La réponse est traçable : on sait d'où
   │                  │  vient chaque information.
   └────────┬─────────┘
            │
            ▼
   Réponse affichée à l'utilisateur
   avec les sources citées
```

C'est exactement cette architecture que vous construirez dans le **TP2**.

### Cas 3 — Détection d'anomalies par similarité

**Le besoin** : détecter des transactions bancaires frauduleuses parmi des millions de transactions quotidiennes.

**Le principe** : chaque transaction est transformée en vecteur (montant, lieu, heure, type de commerce...). Le profil habituel d'un client est aussi un vecteur. Si une nouvelle transaction est **trop éloignée** du profil habituel, une alerte est déclenchée.

ING Bank utilise cette approche pour analyser 1,2 million d'événements par jour et détecter 73 % des incidents avant qu'ils n'aient un impact.

📝 **Question Q5** — Vous développez une application de gestion de tickets en Python avec une base PostgreSQL. Votre chef de projet vous demande d'ajouter un « champ de recherche intelligent ». Décrivez en 4-5 lignes les étapes techniques que vous suivriez pour intégrer une recherche vectorielle à l'application existante.

---

## 6. Limites et points de vigilance

Comme tout outil technique, les bases vectorielles ont des limites qu'un développeur doit connaître.

### Limite 1 — Les biais du modèle

Le modèle de transformation a été entraîné sur des textes existants (livres, sites web, articles). Si ces textes contiennent des biais (stéréotypes, sous-représentation de certains domaines), les vecteurs les reproduiront. Un modèle entraîné principalement en anglais représentera moins bien les nuances du français technique.

### Limite 2 — La similarité n'est pas la vérité

Deux textes peuvent avoir des vecteurs proches alors qu'ils ne répondent pas à la même question. Par exemple, « comment réinitialiser un mot de passe » et « comment changer un mot de passe » sont très proches vectoriellement, mais la procédure peut être différente selon le contexte.

De même, une recherche vectorielle retourne **toujours** des résultats, même si la question n'a rien à voir avec le contenu de la base. Si vous cherchez « recette de quiche lorraine » dans une base de tickets informatiques, vous obtiendrez quand même 3 résultats — ce seront simplement les documents les « moins éloignés », même s'ils n'ont aucun rapport.

### Limite 3 — L'opacité du vecteur

Contrairement à une requête SQL que vous pouvez lire et comprendre (`WHERE categorie = 'réseau'`), un vecteur de 384 nombres n'est pas interprétable par un humain. Vous ne pouvez pas regarder le vecteur `[0.23, -0.41, 0.67, ...]` et en déduire qu'il parle de « réseau ». Cette opacité rend le débogage plus difficile.

### Conséquence pratique : toujours vérifier les résultats

Dans une application professionnelle, il est indispensable de :
- afficher le **score de similarité** pour que l'utilisateur juge de la pertinence
- définir un **seuil minimal** en dessous duquel les résultats ne sont pas affichés
- prévoir un **mécanisme de retour** vers un humain quand le système n'est pas sûr

📝 **Question Q6** — Vous avez déployé un chatbot de support basé sur une base vectorielle. Un utilisateur signale que le chatbot lui a donné une réponse complètement hors sujet. Proposez deux mécanismes techniques que vous pourriez ajouter à votre code pour éviter ce type de problème.

---

## 7. Synthèse — Ce que vous devez retenir

```
┌────────────────────────────────────────────────────────────────┐
│                        À RETENIR                               │
│                                                                │
│  1. SQL compare des MOTS → base vectorielle compare du SENS   │
│                                                                │
│  2. Un vecteur = liste de nombres qui représente le sens       │
│     d'un texte (384 nombres pour le modèle qu'on utilise)      │
│                                                                │
│  3. Similarité cosinus = score de 0 (aucun rapport)            │
│     à 1 (sens identique) entre deux vecteurs                   │
│                                                                │
│  4. Architecture = modèle de transformation + index            │
│     vectoriel + métadonnées                                    │
│                                                                │
│  5. ChromaDB = base vectorielle libre, légère, idéale          │
│     pour apprendre (fonctionne dans Google Colab)              │
│                                                                │
│  6. RAG = recherche vectorielle + IA générative pour           │
│     des réponses traçables et fiables                          │
│                                                                │
│  7. Toujours vérifier les résultats : la similarité            │
│     n'est pas la vérité                                        │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Auto-évaluation — Avant de passer au TP

Répondez à ces 8 questions dans votre document Google Docs, dans la section **PARTIE 1 — COURS : AUTO-ÉVALUATION**.

**Q7** — Expliquez en une phrase la différence fondamentale entre une recherche SQL et une recherche vectorielle.

**Q8** — Un vecteur d'apprentissage capture : ☐ l'orthographe ☐ le sens ☐ la longueur d'un texte. Justifiez votre choix.

**Q9** — La similarité cosinus entre « développement Python » et « programmation en Python » sera-t-elle plus proche de 0.1 ou de 0.9 ? Pourquoi ?

**Q10** — Citez les trois composants d'une base vectorielle et donnez l'équivalent SQL de chacun.

**Q11** — Pourquoi un index vectoriel (comme HNSW) est-il nécessaire quand on a des millions de documents ?

**Q12** — Qu'est-ce que l'architecture RAG et quel problème résout-elle ?

**Q13** — Une recherche vectorielle dans une base de tickets informatiques avec la requête « recette de quiche lorraine » retournera-t-elle des résultats ? Pourquoi est-ce un problème ?

**Q14** — Vous développez un chatbot de support. Un utilisateur pose une question dont la réponse n'est pas dans votre base. Que devrait faire votre application ?

> **Seuil de passage** : vous devez avoir répondu correctement à **au moins 6 questions sur 8** pour commencer le TP1. Si vous avez un doute, relisez la section correspondante du cours.

---

➡️ **Suite** : [TP1 — Construire et interroger une base vectorielle](tp1.md)
