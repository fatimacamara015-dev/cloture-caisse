# Clôture de caisse : spec

## Problème et utilisateurs
Chaque soir, le caissier compare ce qu'il a encaissé (espèces, Wave, Orange Money, carte) aux ventes du système. Sur papier, les écarts sont mal suivis d'un jour à l'autre et le titulaire n'a pas de vue d'ensemble.

- **Caissier :** clôturer sa journée vite, sans erreur de saisie.
- **Titulaire :** valider chaque clôture et retrouver vite, dans tout l'historique, les journées qui posent problème.

## Hypothèses
**Confirmées par le recruteur**
- Une seule caisse et un seul caissier (simplification pour l'exercice).
- Tout écart est signalé, quel que soit son montant : aucune tolérance.
- La justification d'un écart se fait hors de l'app, entre le titulaire et le caissier.
- Le caissier ne voit pas les ventes du système avant de saisir. **Pourquoi :** pour qu'il ne puisse pas ajuster sa saisie au total attendu. Son comptage reste ainsi indépendant.
- Le titulaire peut consulter tout l'historique.

**Mes décisions (laissées à mon choix)**
- Montants en FCFA entiers, sans décimales.
- Le caissier saisit les espèces **hors fond de caisse**.
- Les ventes du système ne comptent que la **part payée par le client** ; le tiers payant (IPM, assurances, mutuelles) en est exclu.
- Données fictives : environ 60 journées, ventilées par moyen de paiement, avec des écarts variés (manques, excédents, écarts qui se compensent, journées sans écart).

## Règle de calcul
- **Écart = encaissé − attendu**, pour chaque moyen de paiement. Négatif = **manque**, positif = **excédent**.
- Une journée est « **avec écart** » dès qu'**une ligne** est différente de 0, même si le total est nul. Exemple : espèces −2 000 et Wave +2 000, souvent un paiement Wave enregistré en espèces.
- **Écart de la journée** (utilisé pour le tri) = somme des valeurs absolues des écarts par ligne. Dans l'exemple, il vaut 4 000 et non 0, ce qui évite de cacher les écarts qui se compensent.

## Rôles
| | Caissier | Titulaire |
|---|---|---|
| Saisir les 4 montants du jour | Oui | Non |
| Soumettre la clôture | Oui | Non |
| Modifier une clôture soumise | Non | Non |
| Voir les ventes du système et les écarts | **Non** | Oui |
| Valider une clôture | Non | Oui |
| Consulter et filtrer tout l'historique | Non | Oui |

## Parcours principal
1. Le caissier ouvre la clôture du jour.
2. Il saisit espèces, Wave, Orange Money et carte. Aucun total du système n'est affiché.
3. Il soumet. Il voit seulement « Clôture envoyée au titulaire ». Sa saisie est verrouillée.
4. Le titulaire arrive sur l'historique. Le bandeau lui indique les clôtures à valider.
5. Il ouvre une clôture et voit le tableau attendu / encaissé / écart par moyen de paiement, avec le total.
6. S'il y a un écart, il en parle avec le caissier hors de l'app, puis valide.
7. La clôture passe en « validée », en lecture seule.

## Historique du titulaire
Le besoin : trouver vite **ce qui reste à traiter** et **ce qui pose problème**.

- **Bandeau :** nombre de clôtures à valider, nombre de journées avec écart et total des manques sur la période filtrée.
- **Filtres combinables :**
  - Période : 7 derniers jours, 30 derniers jours, tout, ou dates personnalisées.
  - Statut : à valider / validée.
  - Écart : tous / avec écart / sans écart / manque uniquement / excédent uniquement.
  - Moyen de paiement en écart : espèces, Wave, Orange Money, carte.
- **Tri :** date (la plus récente d'abord par défaut) ou écart de la journée (le plus grand d'abord).
- **Raccourci :** un clic sur un chiffre du bandeau applique le filtre correspondant.
- **Ligne de l'historique :** date, statut, badge d'écart, moyens de paiement concernés, écart total.

## États par écran
**Saisie (caissier)**
- *Normal :* formulaire vide, prêt à saisir.
- *Erreur :* champ vide, montant négatif, décimal ou non numérique. Le message s'affiche sous le champ et la soumission est bloquée. Le 0 est accepté ; « 50 000 » avec espaces est compris.
- *Déjà soumise :* « La clôture du jour a déjà été envoyée ». Aucune nouvelle saisie possible.

**Détail d'une clôture (titulaire)**
- *En attente de validation :* tableau des montants, bouton « Valider ».
- *Écart détecté :* badge « Écart », lignes concernées mises en évidence avec « manque » ou « excédent ».
- *Clôture validée :* lecture seule, avec la date de validation.

**Historique (titulaire)**
- *Vide :* « Aucune clôture pour l'instant ».
- *Filtre sans résultat :* « Aucune journée ne correspond à ces filtres », avec un bouton « Réinitialiser les filtres ».

## Critères d'acceptation
1. Étant donné un caissier sur l'écran de saisie, alors aucun total du système n'est affiché.
2. Étant donné 50 000 FCFA saisis en espèces pour 52 000 attendus, quand le titulaire ouvre la clôture, alors la ligne espèces affiche −2 000 FCFA (manque).
3. Étant donné espèces −2 000 et Wave +2 000, alors le total affiche 0, la journée est marquée « avec écart » et son écart de journée vaut 4 000.
4. Étant donné des montants égaux aux ventes du système, alors aucun écart n'est signalé.
5. Étant donné un champ vide, négatif ou décimal, quand le caissier soumet, alors la soumission est bloquée avec un message sous le champ. Un 0 est accepté.
6. Étant donné une clôture déjà soumise pour aujourd'hui, quand le caissier ouvre l'app, alors il voit « déjà soumise » et ne peut ni la modifier ni en créer une autre.
7. Étant donné une clôture soumise, quand le titulaire la valide, alors son statut passe à « validée », la date de validation s'affiche et la clôture n'est plus modifiable.
8. Étant donné les filtres « à valider » et « avec écart », alors seules les journées qui remplissent les deux conditions sont affichées.
9. Étant donné le filtre « moyen de paiement en écart = Wave », alors seules les journées avec un écart sur Wave sont affichées.
10. Étant donné le tri par écart, alors la journée avec le plus grand écart de journée apparaît en premier.
11. Étant donné un filtre sans résultat, alors un message explicite et un bouton de réinitialisation s'affichent.

## Ce que je ne fais pas, et pourquoi
- **Justification ou commentaire dans l'app :** la justification se fait hors de l'app (confirmé).
- **Refus ou renvoi pour recomptage :** ça ajoute un aller-retour de statuts sans intérêt pour une première version.
- **Modification après soumission ou validation :** elle casserait le comptage à l'aveugle et la fiabilité de l'historique.
- **Tiers payant, fond de caisse, autres moyens de paiement (Free Money, chèque) :** je les traite par hypothèse pour garder un calcul simple et vérifiable.
- **Plusieurs caisses ou caissiers, vraie authentification, backend, export comptable :** hors du périmètre de l'exercice.

## Limites connues
Les données restent dans le navigateur (localStorage) et le rôle se choisit avec un simple sélecteur. Le masquage des ventes pour le caissier n'existe que dans l'interface : il n'est pas sécurisé. En production, il devrait être appliqué côté serveur.