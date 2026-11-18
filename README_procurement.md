# Procurement Spend Optimization — README

## Objectif et livrables

Étude de cas Data Science / Procurement sur les commandes de l’État de Californie, exercices 2012–2013 à 2014–2015. Question métier : **comment viser 5 % de réduction des dépenses l’année suivante sans compromettre la qualité du service ?**

- `analyse_procurement_completee (1).ipynb` : notebook d’origine, **code non modifié**.
- `01_synthese_executive.pdf` : synthèse décisionnelle d’une page.
- `02_journal_decisions.pdf` : journal des décisions d’une page.

## Exécution

Le notebook a été développé dans **Google Colab** et utilise un montage Google Drive (`drive.mount`). Mettre à jour `CSV_PATH` dans le notebook pour pointer vers le fichier CSV source, puis exécuter les cellules dans l’ordre. Bibliothèques : `pandas`, `numpy`, `matplotlib`, `seaborn`, `IPython`. Les cellules de texte existantes sont conservées telles quelles, conformément à la demande ; ce README explicite la démarche.

## 1. Préparation : doublons et contrôles

**Doublons.** L’observation initiale de lignes strictement identiques, notamment avec des quantités identiques, a fait suspecter des duplications. L’analyse a mesuré les occurrences supplémentaires et leur incidence sur le montant des achats : **59 874 849,55 $** sur l’ensemble des trois années, soit environ 0,04 % des dépenses déclarées. Une suppression hypothétique a aussi été comparée au budget et à l’objectif de 5 %. **Décision : conserver ces lignes**, car le fichier n’offre pas d’identifiant unique de ligne de commande ; deux lignes identiques peuvent représenter deux acquisitions réelles. Ce montant n’est pas une économie acquise. Sa comparaison approximative avec la cible annuelle de 2,33 Md$ donne ~2,57 %, mais les périmètres temporels diffèrent : elle ne doit pas être présentée comme une contribution réalisable à l’objectif.

**Contrôle de cohérence des montants.** Pour les lignes vérifiables, comparaison entre `Total Price` et `Quantity × Unit Price`, avec une tolérance de **0,01 $**. Analyse des taux et montants d’écarts par année, de leur signe, des types et méthodes d’acquisition et des descriptions. La baisse observée des incohérences au fil des années peut suggérer une amélioration de la qualité de saisie ; **le dataset ne prouve pas l’existence d’une mesure corrective**.

**Hypothèse remises/taxes.** Des écarts positifs ou négatifs auraient pu être expliqués par des taxes, remises, frais de transport ou ajustements. Le notebook recherche des indices textuels (`discount`, `rebate`, `tax`, `freight`, etc.) et examine la distribution des écarts. Leur concentration près de zéro n’appuie pas une explication générale par des taux fixes de remise/taxe ; l’hypothèse n’est donc **pas retenue comme justification automatique**. Note : le graphique intitulé « écarts relatifs » trace en réalité la variable d’écart **monétaire** ; il ne faut pas l’interpréter comme un histogramme de pourcentages.

**Imputation.** Si `Total Price` est manquant et que `Quantity` et `Unit Price` sont positifs, imputation par leur produit. Si `Total Price` est présent, conservation de ce montant même si la quantité ou le prix unitaire ne permettent pas la vérification. Exclusion seulement des lignes dont le montant total reste manquant. **344 504 lignes initiales ; 344 474 conservées ; 30 exclues.** Les montants discordants existants ne sont pas automatiquement remplacés par le produit quantité × prix : cela éviterait de gommer d’éventuels ajustements métier.

**Bases analytiques distinctes.** `df_clean` sert aux agrégations de dépenses (344 474 lignes) ; `df_prix` sert aux comparaisons unitaires (331 273 lignes, quantités/prix positifs et aucune incohérence constatée).

## 2. Analyse exploratoire : du macro au micro

Budget de référence **2014–2015 : 46 541 293 689 $** ; cible **5 % : 2 327 064 684 $** ; budget cible ~44,21 Md$. Analyse des catégories de dépenses, concentration, variations interannuelles et fournisseurs. Puis descente en granularité : `Segment Title` → `Commodity Title` → département, commande, fournisseur et description. Focus sur **Healthcare Services**, ainsi que sur les évolutions d’**Environmental Services**. Exploration des commandes de **Public health administration** et **Environmental technology advisory services**. Les dépenses unitaires ou commandes exceptionnellement élevées, ainsi que les champs « Unknown », sont **des priorités d’audit**, non des erreurs avérées ni des économies immédiatement récupérables.

## 3. Trois leviers de réduction — résultats du notebook

| Levier | Calcul / hypothèse | Potentiel brut | Part de la cible 5 % | Confiance |
|---|---|---:|---:|---|
| Biens potentiellement non récurrents | IT/NON-IT Goods, baisse en 2014 puis hausse en 2015 ; proposer 80 % de la baisse de 2014, plafonnée au retour au niveau 2014 | **144 231 626,54 $** | **6,20 %** | Faible |
| Harmonisation des prix fournisseurs | Même `Normalized UNSPSC` + `Item Name` ; fournisseurs distincts ; appliquer le prix unitaire minimum observé aux quantités comparées | **286 870 202,80 $** | **12,33 %** | Faible à moyenne |
| Poursuite des baisses historiques | Départements avec deux baisses consécutives et décroissantes ; projeter 75 % du taux de baisse 2015 | **1 329 269 408,20 $** | **57,12 %** | Faible à moyenne |
| **Somme brute** | **Chevauchements possibles : ne pas interpréter comme gain net** | **1 760 371 237,54 $** | **75,65 %** | À valider |

**Lecture de l’objectif :** 1,760 Md$ représentent une réduction brute théorique de **3,78 %** du budget de référence, contre **5 %** demandés. Écart brut : **566 693 447 $**. Ces chiffres sont issus des sorties enregistrées dans le notebook, pas de nouvelles exécutions.

## 4. Hypothèses, confiance et limites

1. **Biens :** `Acquisition Type = Goods` inclut aussi des consommables ; le caractère durable et la possibilité de différer le renouvellement ne sont pas établis. Vérifier besoins, durée de vie, stock, sécurité et qualité du service.
2. **Prix :** même code UNSPSC et même nom ne garantissent pas la même unité, le même conditionnement ou la même spécification. Le prix minimum peut être lié à un volume, une date ou un contrat particulier. Vérifier ces dimensions et la capacité du fournisseur avant négociation.
3. **Tendance :** des dépenses en baisse peuvent refléter la fin d’un projet ou un changement de périmètre, pas une efficience reproductible. Le coefficient de 75 % est un scénario, non une prévision validée.
4. **Addition :** les leviers peuvent toucher les mêmes lignes de dépenses. Les montants sont **bruts et non dédupliqués**. Ils ne peuvent être annoncés comme économies réalisées ou garanties.
5. **Périmètre :** le notebook est une étude historique, non une preuve des budgets, contrats ou besoins de l’année suivante.

## 5. Plan de validation avant décision

- Prioriser les plus grosses lignes / commandes, notamment celles à classification ou description « Unknown », et vérifier leur nature avec les départements.
- Pour le benchmark fournisseurs, vérifier unité, caractéristiques, volumes, contrats et délais ; demander des offres comparables.
- Pour les biens, confirmer quels investissements peuvent réellement être différés sans risque opérationnel.
- Pour les départements en baisse, demander l’explication métier de la tendance et valider une nouvelle cible réaliste.
- Identifier les chevauchements entre leviers, établir une assiette unique et calculer les **économies nettes sécurisées**, en suivant des indicateurs de qualité de service.

## Conclusion

L’analyse identifie **des pistes chiffrées et hiérarchisées**, mais ne démontre pas encore que la cible de 5 % sera atteinte. La stratégie la plus directement négociable est le benchmark de prix, sous réserve de comparabilité ; les achats potentiellement non récurrents et les extrapolations historiques exigent une validation métier plus forte. Les doublons potentiels restent une piste de vérification de qualité, **pas un quatrième levier d’économies comptabilisable**.
