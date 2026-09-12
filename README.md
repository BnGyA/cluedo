# Château Guillermo — jeu d'enquête grandeur nature

Web app 100 % statique (un seul `index.html`, aucun build, aucun backend), pensée
pour être jouée sur téléphone et hébergée sur GitHub Pages.

## Arborescence

```
cluedo/
├── index.html          # toute l'app : DONNÉES en tête, puis CSS, puis JS
├── .nojekyll           # GitHub Pages sert les fichiers tels quels
├── README.md
└── assets/
    ├── maps/           # plans des étages (ground-floor.jpg, first-floor.jpg, second-floor.jpg)
    ├── print/          # accessoires à imprimer (lettre + masque de la chambre Marie-Antoinette)
    └── cards/
        ├── suspects/   # {id}.png × 7
        ├── weapons/    # {id}.png × 7
        └── pieces/     # {id}.png × 7
```

## Éditer le contenu

Tout est dans le premier bloc `<script>` de `index.html` :

- `CONFIG` : réglages (dossier des cartes, plan par défaut, taille du classement,
  `revealSolutionOnFailure`, `hintPenaltySeconds`).
- `UI` : tous les textes de l'interface, dont la lettre d'accueil (`UI.welcome`)
  et les libellés du blason (`UI.crest`).
- `suspects`, `weapons`, `pieces` : 7 cartes par axe, `{ id, name, isSolution }`.
  Mettre `isSolution: true` sur **une** carte par axe pour définir la solution.
  Les suspects portent en plus `tagline` (clin d'œil à la vie de la boîte),
  `motive` (la raison qui aurait pu pousser au meurtre) et, pour le coupable,
  `reveal` (phrase affichée sur le verdict réussi). Les pièces portent
  `history` (la vraie histoire de la pièce, hors intrigue). Toute carte qui
  porte un `tagline`, un `motive` ou une `history` reçoit un sceau dans son
  coin, dans le carnet : il ouvre sa fiche (portrait, clin d'œil, mobile ou
  histoire, barrer / restaurer pour les suspects et les armes).
- `pieces` suit une règle à part : **pas d'indice « room »**. Les sept cartes
  sont les six salles d'épreuve (même `id` que dans `rooms`) plus une pièce
  sans épreuve, la scène du crime (`isSolution: true`). Les cartes pièces ne
  se barrent pas à la main : chaque salle résolue grise la sienne dans le
  carnet (tampon « searched ») et la révélation le dit à l'équipe ; à la fin,
  seule la scène du crime reste active et l'accusation la présélectionne. `init()` signale dans la console
  toute incohérence entre `pieces` et `rooms` (salle sans carte, plusieurs
  pièces sans épreuve, solution sur une salle d'épreuve, cartes dans l'ordre
  des salles). L'ordre du tableau est l'ordre d'affichage : le garder
  alphabétique, jamais dans l'ordre de visite.
- `houseHistory` : la vraie histoire du château, un paragraphe par entrée,
  affichée sur l'écran de résultat sous le verdict (kicker et titre dans
  `UI.houseKicker` / `UI.houseTitle`).
- `rooms` : les 6 salles. Chaque salle a `name`, `floor`, `ambiance`, `map`,
  `cartons`, `inputLabel`, `inputMode`, `answers`, `clues`, `eliminates` et,
  au choix, `hints`. `clues` et `eliminates` n'ont que deux axes, `culprit`
  et `weapon`. Les réponses sont comparées après normalisation (minuscules,
  sans accents, sans espaces). `eliminates` n'est jamais affiché au joueur.

### Indices payants

Une salle peut porter `hints`, un tableau de coups de pouce dans l'ordre où
ils se débloquent. Sous le formulaire de réponse, un bloc « Need a hand? »
propose « Ask for a hint (+3 min) » ; l'équipe confirme, l'indice s'affiche
sur un carton vert-de-gris et reste visible jusqu'à la résolution de la
salle. Chaque indice ajoute `CONFIG.hintPenaltySeconds` (180 s) au chrono,
immédiatement et de façon visible (le chrono s'éclaire en bordeaux), donc au
temps final et au code de résultat. Le nombre d'indices demandés par salle
est mémorisé dans `chateau.progress` (`hints`) et survit à un
rafraîchissement ; l'écran de résultat rappelle la pénalité totale. Une salle
sans `hints` n'affiche pas le bloc. Textes dans `UI.hint*` et
`UI.hintsSummary*` ; la durée y est mise en forme par `formatPenalty`
(« 3 minutes » en toutes lettres, « 3 min » dans les boutons).

### Blason d'équipe

Sur l'écran d'accueil, l'équipe compose son blason : couleur du champ, seconde
couleur, motif, emblème et métal, avec un bouton « Surprise me » pour un tirage
aléatoire. Le dessin (couleurs, motifs, symboles SVG) est dans `CREST`, en bas
de `index.html`, indexé par des ids stables (`wine`, `tower`, `gold`…) ; les
libellés affichés sont dans `UI.crest`. Pour ajouter une option, ajouter une
entrée dans `CREST` **et** son libellé dans `UI.crest`.

### Images des cartes

Nommer les fichiers d'après les `id` des tableaux :

```
assets/cards/suspects/  governess housekeeper inventor notary gardener cook doctor
assets/cards/weapons/   candlestick dagger revolver rope poison poker billiard-cue
assets/cards/pieces/    billiard-room rotunda-bar kitchen salon bonaparte-bedroom
                        marie-antoinette-bedroom louis-xiv-bedroom
```

Format PNG, ratio conseillé 5:7 (carte à jouer). Une image absente laisse
apparaître l'icône de la carte (voir ci-dessous) sur fond papier.

### Icônes des cartes

Chaque carte a sa silhouette SVG dans `cardIcons` (fin du premier bloc
`<script>`, indexé par axe puis par `id`) : elle occupe le cadre de la carte et
de sa fiche tant que l'image PNG manque, et reste derrière l'image sinon. Une
carte sans entrée retombe sur l'icône générique de son axe (`ICON_FALLBACK`,
script principal). Pour changer une icône : coller le contenu d'un SVG 512×512
sans couleur (`<path d="…"/>`), la teinte est donnée par le CSS (`.card__icon`).

Les silhouettes viennent de [Game-icons.net](https://game-icons.net) (licence
[CC BY 3.0](https://creativecommons.org/licenses/by/3.0/), auteurs Lorc,
Delapouite et Caro Asercion, nom d'origine indiqué avant chaque entrée). Le
tisonnier et la queue de billard sont dessinés pour le projet.

### Plans

`map.src` pointe vers le plan de l'étage, `map.zone` surligne la pièce en % de
l'image (`{ x, y, w, h }`). Les zones relevées sur les trois plans sont listées
en commentaire au-dessus de `rooms`. `map: null` affiche une plaque nom + étage.

### Accessoires à imprimer

`assets/print/marie-antoinette-letter.html` : la lettre de la reine à Fersen et
son masque, une grille de Cardan, pour la chambre Marie-Antoinette. Ouvrir le
fichier dans Chrome, cliquer « Check » (ou ajouter `#check` à l'URL) pour
vérifier à l'écran que les douze fenêtres tombent sur les bons mots, puis
imprimer en A4 paysage, échelle 100 %, sans marges ni en-têtes. La feuille 1
porte la lettre et le masque côte à côte : même passage dans l'imprimante,
donc même échelle. La feuille 2 porte le carton d'instructions et un masque
de rechange. Découper chaque pièce le long de son trait gris, puis les
fenêtres du masque au cutter, raturer à la plume les deux phrases repérées en
pointillé à l'écran, et plastifier les deux : les fenêtres deviennent des vitres. Les positions des fenêtres sont calculées depuis le rendu réel de
la lettre, la police peut donc manquer ou changer sans casser l'alignement.
Le mot caché et la mise en place sont commentés au-dessus de la salle dans
`rooms`.

## Déployer sur GitHub Pages

1. Pousser le dossier sur un dépôt GitHub.
2. Settings → Pages → Source « Deploy from a branch », branche `main`, dossier `/ (root)`.
3. L'app est servie à `https://<compte>.github.io/<dépôt>/`. Tous les chemins sont relatifs.

## Classement final (maître du jeu)

Aucun serveur : le classement final se construit sur le téléphone du maître du
jeu à partir des résultats que les équipes lui envoient.

1. À la fin de l'enquête, l'écran de résultat affiche le code de résultat de
   l'équipe et un bouton « Copy the code ». Le texte demande de l'envoyer à
   Benjamin sur Slack. Pas de partage natif : `navigator.share` faisait planter
   l'onglet sur certaines versions de Chrome (voir plus bas).
2. Le maître du jeu colle chaque code reçu dans le champ « Add a result code » :
   l'app l'ajoute à son classement (stocké en local) et affiche l'écran
   « Final ranking ». Il peut retirer une ligne (doublon, test) ou vider le
   classement.
3. Les liens `index.html#r=CODE` et `index.html#board=CODE,CODE,…` restent
   acceptés, ouverts directement ou collés dans le champ ; leurs résultats sont
   fusionnés avec le classement local. Plus aucun lien n'est généré par l'app.

L'écran est accessible à tout moment via `index.html#board` ou par le lien
discret en pied de l'écran d'accueil. Tri : accusation juste d'abord, puis
temps croissant ; les trois premiers sont mis en valeur, le premier porte une
couronne. Les doublons (même code) sont ignorés.

### Pourquoi pas de partage natif

`navigator.share()` pouvait rejeter avec autre chose qu'un `AbortError`. Le repli
`navigator.clipboard.writeText()` s'exécutait alors après un `await`, donc sans
activation utilisateur : Chrome ne résout ni ne rejette cette promesse, le bouton
restait muet, l'utilisateur recliquait, et le second `navigator.share()` tuait le
processus de rendu (`RESULT_CODE_KILLED_BAD_MESSAGE`). La copie passe maintenant
par `copyText()`, qui borne l'attente du presse-papiers et se replie sur une copie
synchrone `execCommand`.

### Code de résultat

Base64url d'un petit JSON `{ t: nom, s: secondes, ok: accusation juste, c: blason }`,
où `c` est la suite des cinq ids du blason séparés par un point
(`wine.forest.plain.tower.gold`). Un code sans `c` (ancienne version) reste
accepté, le nom s'affiche alors sans blason. Pour le décoder dans la console
du navigateur (n'importe quelle page de l'app) :

```js
decodeResultCode('eyJ0IjoiTGVzIExpbW…')
```

Le point d'accroche pour un futur envoi automatique vers un service externe
reste marqué `// LEADERBOARD HOOK` dans `saveResult()`.

## Stockage local

- `chateau.progress` : équipe, blason choisi (`crest`), timestamps de départ et de
  fin, nombre de salles résolues, indices demandés par salle (`hints`,
  `{ id de salle: nombre }`), résultat. Une sauvegarde antérieure qui n'a
  qu'une graine (`crestSeed`) est convertie au chargement, à l'identique.
- `chateau.cards` : cartes barrées par le joueur. Les pièces grisées ne sont pas
  stockées : elles se déduisent du nombre de salles résolues.
- `chateau.results` : classement local (le blason y est conservé et affiché en
  miniature devant le nom d'équipe).
- `chateau.board` : codes de résultat reçus par le maître du jeu (classement
  final). Indépendant de la partie en cours sur l'appareil.
