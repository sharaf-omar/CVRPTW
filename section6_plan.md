# Section 6 « Solution method » : plan proposé

*Version 1, 21/10/2026. Plan à valider avant rédaction. La section elle-même sera rédigée en anglais.*

## 1. Méthode recommandée : ALNS hybride (avec recherche locale)

**ALNS** (Adaptive Large Neighbourhood Search, Ropke et Pisinger 2006) : à chaque itération on **détruit** une partie de la solution (on retire q clients), on la **répare** (on les réinsère au mieux), puis une **recherche locale** courte améliore le résultat. Le choix des opérateurs s'adapte selon leurs succès passés, et un critère de **recuit simulé** décide si on accepte la nouvelle solution.

**Pourquoi celle-là :**
- **Fenêtres de temps et trafic :** la réparation insère client par client en testant la faisabilité avec la règle 2.7 d'Omar. Le trafic par créneaux (ρ) ne change rien à l'algorithme, seulement à l'évaluation. C'est la méthode la plus simple à rendre cohérente avec notre modèle.
- **Qualité :** sur Solomon / Homberger, ALNS atteint en général un écart de 1 à 3 % aux BKS. On a donc de la marge par rapport au seuil de 7-10 %.
- **Taille :** comme on ne détruit que q clients par itération, une itération coûte environ O(q·n) et non O(n²). Avec des voisinages granulaires (on ne teste que les 20 à 40 plus proches voisins), ça reste faisable en Python jusqu'à quelques milliers de clients.
- **Consigne :** elle couvre exactement les puces demandées : définition des solutions, voisinages (LNS + recherche locale), opérateurs (insertion, échange…) et mécanisme de guidage (poids adaptatifs + recuit).

**Alternatives à citer en une ligne chacune :**
- **HGS** (Vidal 2013) : meilleure qualité, mais croisement et gestion de pénalités lourds à coder et à expliquer.
- **Tabou** : moins efficace à grande échelle.
- **Colonies de fourmis / algo génétique simple** : écarts plus élevés.
- **PyVRP** : uniquement comme référence de comparaison, pas comme solveur (sinon ce n'est plus notre méthode).

## 2. Plan de la section (en anglais dans le notebook)

- **6.1 Choice of method and justification** : un court paragraphe et un tableau de 4 lignes (ALNS, HGS, Tabou, ACO).
- **6.2 Solution representation and evaluation** :
  - une solution est une liste de routes ;
  - les heures sont recalculées avec la règle 2.7 ;
  - coût = coût free-flow + pénalité sur les clients non servis.
- **6.3 Initial solution** : insertion gloutonne (type Solomon I1) en respectant les fenêtres.
- **6.4 Neighbourhoods and operators** :
  - destruction : aléatoire, pire coût, Shaw (clients proches), retrait d'une route ;
  - réparation : glouton, regret-2/3 ;
  - recherche locale : relocate, swap, 2-opt*, or-opt, limités aux k plus proches voisins.
- **6.5 Guidance mechanisms** : poids adaptatifs par roulette (scores σ1/σ2/σ3, mise à jour par segments), acceptation par recuit (température initiale et refroidissement), bruit dans la réparation.
- **6.6 Algorithm and stopping criterion** : pseudocode de 10 à 15 lignes. On s'arrête à une limite de temps proportionnelle à n ou après N itérations sans amélioration.

Longueur visée : comme la Section 1, sans code. Une seule référence nouvelle : Ropke et Pisinger 2006 (à lire avant de la citer).

## 3. Choix à trancher en équipe

1. **Méthode** : ALNS hybride (recommandé) ou HGS ?
2. **Fenêtres et capacité** : on les garde strictement dures pendant la recherche, avec une pénalité seulement pour les clients non insérés (recommandé, plus simple), ou on autorise des solutions infaisables pénalisées (plus performant mais plus complexe) ?
3. **Nombre de véhicules** : m strict (choix d'Omar) ou minimisé d'abord ? ⚠️ Les BKS Solomon / Homberger publiées minimisent souvent **d'abord les véhicules, puis la distance**. Il faut vérifier quelles BKS vrplib fournit, sinon nos écarts ne seront pas comparables. C'est lié à la décision ouverte n°2 de NOTATION.md.
4. **Objectif** : free-flow (défaut) ou temps vécu ? Pour comparer aux BKS, il faut ρ = 1 et l'objectif free-flow.
5. **Valeurs de départ des paramètres** : taille de destruction q (par exemple 10 à 40 % de n, plafonnée), température, nombre de plus proches voisins k. Ces valeurs seront calibrées dans le livrable final.
6. **Rapidité de l'évaluation avec trafic** : les astuces classiques en O(1) (marges de temps de Savelsbergh) ne s'appliquent plus telles quelles. On recalcule donc la fin de la route seulement, et on s'arrête dès que les heures ne changent plus.
7. **Rédaction** : qui écrit la section 6 ?
