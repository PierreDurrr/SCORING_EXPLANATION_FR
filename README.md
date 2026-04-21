
# 📊 Comment fonctionne le calcul des recommandations

Le Unified Plex Recommender (UPR) utilise un moteur de scoring multi-critères pour classer les films et séries TV en fonction des habitudes de visionnage de chaque utilisateur. Ce document explique **comment fonctionne le scoring**, notamment :

✅ Comment UPR construit le profil de goûts de l'utilisateur  
✅ Comment chaque dimension contribue au score  
✅ Comment fonctionne le *bonus d'interaction*  
✅ Comment la *décroissance temporelle* affecte les poids  
✅ Des exemples clairs et des schémas

---

# ✅ 1. Construction du profil de goûts de l'utilisateur

Pour chaque titre visionné par un utilisateur, UPR extrait :
- Les genres
- Les réalisateurs / studios
- Le casting (acteurs principaux)
- La langue
- Les mots-clés TMDb
- L'identifiant TMDb

Ces éléments deviennent des compteurs qui reflètent la fréquence à laquelle l'utilisateur regarde chaque attribut.

Exemple :
```
genres:
  action: 12.5
  romance: 3.0
acteurs:
  keanu reeves: 5.0
```

Ces valeurs incluent :
- Le multiplicateur de note
- La décroissance temporelle
- Le nombre de visionnages

---

# ✅ 2. Multiplicateur de note

Chaque élément visionné utilise sa note utilisateur/audience pour appliquer un multiplicateur :

| Note | Poids |
|------|-------|
| 0 | ×0.1 |
| 1 | ×0.2 |
| 2 | ×0.4 |
| 3 | ×0.6 |
| 4 | ×0.8 |
| 5 | ×1.0 |
| 6 | ×1.2 |
| 7 | ×1.4 |
| 8 | ×1.6 |
| 9 | ×1.8 |
| 10 | ×2.0 |

Cela signifie :
- Les titres mieux notés influencent davantage les recommandations.
- Les notes neutres → poids normal.
- Les notes basses → faible influence.

---

# ✅ 3. Décroissance temporelle

UPR réduit l'influence des visionnages plus anciens.

Formule :
```
weight *= exp(-ln(2) * days_ago / HALFLIFE)
```

La DEMI-VIE (HALFLIFE) = **90 jours**.

Signification :
- visionné hier → 100% de l'influence
- visionné il y a 3 mois → 50%
- visionné il y a 6 mois → 25%
- visionné il y a 1 an → ~6%

Cela garantit que les recommandations reflètent les **goûts actuels**, et non des habitudes obsolètes.

---

# ✅ 4. Calcul du score d'un élément

Chaque élément de la bibliothèque Plex est comparé au profil de l'utilisateur selon 5 dimensions :
- Genres
- Réalisateurs / Studios
- Acteurs
- Langue
- Mots-clés

Chaque dimension contribue à une composante pondérée.

---

## ✅ Score de genre (25%)
Pour chaque genre que possède l'élément :
```
score = sqrt(user_genre_count / user_genre_max)
```
Plusieurs genres sont moyennés ensemble.

---

## ✅ Score Réalisateur / Studio (20%)
Même méthode que le genre ; reflète les créateurs préférés de l'utilisateur.

---

## ✅ Score des acteurs (20%)
Fort bonus si l'utilisateur regarde fréquemment du contenu mettant en scène certains acteurs.

---

## ✅ Score de langue (10%)
Si la piste audio principale de l'élément correspond à une langue que l'utilisateur regarde fréquemment, il reçoit un petit bonus.

---

## ✅ Score des mots-clés (25%)
Les mots-clés TMDb représentent des thèmes tels que :
- magie
- voyage dans le temps
- dystopie
- vampire
- isekai

Cela améliore la précision thématique, notamment pour les animés.

---

# ✅ 5. Bonus d'interaction (Synergie Genre × Mots-clés)

UPR récompense les éléments qui correspondent fortement **à la fois** aux préférences de genre et de mots-clés.

Formule :
```
bonus = 0.05 × min(genre_score, keyword_score) / 0.25
```

Exemple :
- Genres : fantasy ✅
- Mots-clés : magie, aventure ✅

L'élément reçoit un bonus de synergie.

C'est particulièrement puissant pour les animés :
- "isekai"
- "magie"
- "aventure"
- "fantasy"

Tous se combinent pour faire remonter ces titres dans le classement.

---

# ✅ 6. Score final

Toutes les composantes sont additionnées :
```
final_score = genre + directeur + acteur + langue + mots-clés + bonus_interaction
```

Puis plafonné :
```
score = min(final_score, 1.0)
```

---

# ✅ 7. Exclusions de genres (Globales + Spécifiques à l'utilisateur)

Avant tout calcul, UPR retire les éléments contenant des genres exclus.

Par exemple :
```
global: spectacle
utilisateur: sport
```

→ Tout titre correspondant à l'un ou l'autre genre est **retiré des candidats**.

Cela garantit que les éléments non pertinents n'apparaissent jamais.

---

# ✅ 8. Option de randomisation

Si activée :
```
randomize_recommendations: true
```

UPR :
1. Prend le top 10% des scores les plus élevés.
2. Sélectionne aléatoirement des éléments depuis ce pool.

Cela crée de la variété entre les exécutions.

---

# ✅ Schéma — Comment UPR raisonne

```
[ Historique de visionnage ]
        ↓
[ Construction des compteurs ]
        ↓
[ Multiplicateur de note ]
        ↓
[ Décroissance temporelle ]
        ↓
[ Profil de goûts de l'utilisateur ]
        ↓
[ Calcul des scores de la bibliothèque ]
        ↓
[ Exclusions de genres ]
        ↓
[ Sélection & Randomisation ]
        ↓
[ Application des labels / listes Trakt ]
```

---

# ✅ Résumé
UPR construit un profil de goûts personnalisé basé sur ce que chaque utilisateur regarde, à quelle date il l'a regardé et combien il l'a apprécié. Chaque élément de Plex est évalué par rapport à ce profil à l'aide d'une combinaison de similarités de genre, d'acteurs, de réalisateurs, de langue et de mots-clés, enrichie par des signaux de récence et d'interaction.

Cela produit des recommandations hautement précises et personnalisées.
