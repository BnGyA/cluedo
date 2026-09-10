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
    └── cards/
        ├── suspects/   # {id}.png × 7
        ├── weapons/    # {id}.png × 7
        └── pieces/     # {id}.png × 7
```

## Éditer le contenu

Tout est dans le premier bloc `<script>` de `index.html` :

- `CONFIG` : réglages (dossier des cartes, plan par défaut, taille du classement,
  `revealSolutionOnFailure`).
- `UI` : tous les textes de l'interface, dont la lettre d'accueil (`UI.welcome`)
  et les libellés du blason (`UI.crest`).
- `suspects`, `weapons`, `pieces` : 7 cartes par axe, `{ id, name, isSolution }`.
  Mettre `isSolution: true` sur **une** carte par axe pour définir la solution.
- `rooms` : les 6 salles. Chaque salle a `name`, `floor`, `ambiance`, `map`,
  `cartons`, `inputLabel`, `inputMode`, `answers`, `clues`, `eliminates`.
  Les réponses sont comparées après normalisation (minuscules, sans accents,
  sans espaces). `eliminates` n'est jamais affiché au joueur.

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
assets/cards/suspects/  butler housekeeper countess notary gardener cook doctor
assets/cards/weapons/   candlestick dagger revolver rope poison poker billiard-cue
assets/cards/pieces/    billiard-room rotunda-bar kitchen bonaparte-bedroom
                        marie-antoinette-bedroom library wine-cellar
```

Format PNG, ratio conseillé 5:7 (carte à jouer). Une image absente affiche un
placeholder propre avec le nom et une icône générique.

### Plans

`map.src` pointe vers le plan de l'étage, `map.zone` surligne la pièce en % de
l'image (`{ x, y, w, h }`). Les zones relevées sur les trois plans sont listées
en commentaire au-dessus de `rooms`. `map: null` affiche une plaque nom + étage.

## Déployer sur GitHub Pages

1. Pousser le dossier sur un dépôt GitHub.
2. Settings → Pages → Source « Deploy from a branch », branche `main`, dossier `/ (root)`.
3. L'app est servie à `https://<compte>.github.io/<dépôt>/`. Tous les chemins sont relatifs.

## Code de résultat

À la fin, chaque équipe obtient un code base64url d'un petit JSON
`{ t: nom, s: secondes, ok: accusation juste }`. Pour le décoder, dans la console
du navigateur (n'importe quelle page de l'app) :

```js
decodeResultCode('eyJ0IjoiTGVzIExpbW…')
```

Il n'y a pas de classement partagé : l'app affiche un classement local des
parties jouées sur l'appareil. Le point d'accroche pour un futur envoi vers
Google Sheet ou JSONBin est marqué `// LEADERBOARD HOOK` dans `saveResult()`.

## Stockage local

- `chateau.progress` : équipe, blason choisi (`crest`), timestamps de départ et de
  fin, nombre de salles résolues, résultat. Une sauvegarde antérieure qui n'a
  qu'une graine (`crestSeed`) est convertie au chargement, à l'identique.
- `chateau.cards` : cartes barrées.
- `chateau.results` : classement local (le blason y est conservé et affiché en
  miniature devant le nom d'équipe).
