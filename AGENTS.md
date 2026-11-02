# Motion Lab — mode d’emploi pour agents IA

Ce document s’adresse aux agents IA (Claude Code, Codex, ou tout agent qui pilote un navigateur via Playwright / CDP). Il explique comment utiliser **Motion Lab** (`app.html`), l’éditeur de modèles d’animation de Binks Design : mettre des visuels dans des emplacements, choisir un des 258 modèles, régler le format, le style, le fond, les textes et le logo, poser des images clés, puis exporter une vidéo MP4 ou WebM (ou une image PNG).

Il existe en deux endroits, toujours identiques : `AGENTS.md` (Markdown, français puis anglais) et la page `agent/index.html`. Résumé court : `llms.txt`. Documentation pour les humains : `docs.html`.

**Important : Motion Lab a une API JavaScript publique, `window.BinksyAPI`, et les mêmes outils en WebMCP (section 0, page `mcp/`).** Préfère-la ; l’interface reste pilotable avec les sélecteurs réels décrits plus bas. N’invente pas de fonction qui n’existe pas : la liste à jour est `await BinksyAPI.describe()`.

## 0. API pour agents : `window.BinksyAPI`, WebMCP et page `/mcp/`

**Préfère cette API à l’interface.** Une fois l’app prête, `window.BinksyAPI` expose des méthodes asynchrones qui renvoient du JSON (jamais de nœud DOM) et appellent la même logique que l’interface : l’écran se met à jour et l’enregistrement automatique (IndexedDB) suit. Une erreur est rejetée avec `{ code, message }`. Les mêmes outils sont enregistrés en **WebMCP** (`document.modelContext.registerTool`, repli `navigator.modelContext`) quand le navigateur le permet ; une seule table de définitions sert les deux. Page de connexion : `mcp/` (https://motion.binksdesign.com/mcp/).

```js
await BinksyAPI.ready();
const info = await BinksyAPI.describe(); // { app, version, apiVersion: "1", description, tools: [...] }
```

| Outil WebMCP | Méthode | Rôle |
|---|---|---|
| `motionlab_describe` | `BinksyAPI.describe()` | Décrit l’app et la liste des outils. |
| `motionlab_get_state` | `BinksyAPI.getState()` | Lit l’état : modèle, format, durée, emplacements, fond, textes, logo, images clés. |
| `motionlab_lister_modeles` | `BinksyAPI.listTemplates()` | Liste / cherche les 258 modèles. |
| `motionlab_lire_modele` | `BinksyAPI.getTemplate()` | Réglages d’un modèle (chemins, bornes, options). |
| `motionlab_choisir_modele` | `BinksyAPI.selectTemplate()` | Choisit le modèle. |
| `motionlab_choisir_format` | `BinksyAPI.setRatio()` | Format du cadre (16:9, 4:3, 1:1, 4:5, 3:4, 9:16). |
| `motionlab_duree` | `BinksyAPI.setDuration()` | Durée de la boucle (2 à 40 s). |
| `motionlab_nombre_images` | `BinksyAPI.setImageCount()` | Nombre d’images utilisées par le modèle. |
| `motionlab_regler` | `BinksyAPI.setValues()` | Règle des valeurs par chemin (tp.…, bg.…, logo.…, tx.…). |
| `motionlab_fond` | `BinksyAPI.setBackground()` | Fond couleur, dégradé ou image fournie. |
| `motionlab_importer` | `BinksyAPI.importer()` | Importe images / vidéos dans les emplacements, ou un projet .json. |
| `motionlab_vider` | `BinksyAPI.clearSlots()` | Vide des emplacements. |
| `motionlab_exemples` | `BinksyAPI.loadExamples()` | Remet les visuels d’exemple (tests seulement). |
| `motionlab_ajouter_texte` | `BinksyAPI.addText()` | Ajoute un calque texte (texte fourni / validé). |
| `motionlab_supprimer_texte` | `BinksyAPI.removeText()` | Supprime un calque texte. |
| `motionlab_logo` | `BinksyAPI.setLogo()` | Pose, place, règle ou retire le logo fourni. |
| `motionlab_image_cle` | `BinksyAPI.setKeyframe()` | Pose / modifie / retire une image clé à un instant (0–1). |
| `motionlab_temps` | `BinksyAPI.setTime()` | Tête de lecture et lecture / pause. |
| `motionlab_exporter` | `BinksyAPI.exporter()` | Exporte MP4, WebM, PNG ou projet .json (base64). |
| `motionlab_enregistrer_projet` | `BinksyAPI.saveProject()` | Enregistre le projet et ses médias dans le navigateur. |
| `motionlab_lister_projets` | `BinksyAPI.listProjects()` | Liste les projets enregistrés. |
| `motionlab_ouvrir_projet` | `BinksyAPI.openProject()` | Ouvre un projet enregistré (avec ses médias). |
| `motionlab_reset` | `BinksyAPI.reset()` | Remet la session par défaut (confirm: true après accord). |

Les médias importés ne sont pas gardés dans la session automatique (seuls les réglages le sont) : appelle motionlab_enregistrer_projet pour les retrouver au rechargement (motionlab_ouvrir_projet). Les visuels d’exemple sont marqués example: true dans get_state : ne les livre jamais.

Découvrir les outils :

```js
await BinksyAPI.ready();
(await BinksyAPI.describe()).tools.map(t => t.name);
```

Importer les visuels fournis (base64) et choisir un modèle :

```js
await BinksyAPI.importer({ clearFirst: true, files: [
  { name: "visuel-01.jpg", type: "image/jpeg", dataBase64: "<base64 du fichier fourni>" },
  { name: "visuel-02.jpg", type: "image/jpeg", dataBase64: "<base64>" }
] });
await BinksyAPI.setRatio({ ratio: "9:16" });
await BinksyAPI.selectTemplate({ id: "cover-flow" });
await BinksyAPI.setDuration({ seconds: 10 });
```

Exporter la vidéo MP4 1080p et garder le projet :

```js
const { files } = await BinksyAPI.exporter({ format: "mp4", resolution: 1080, fps: 30 });
files[0].name; // sans-titre.mp4, dataBase64 = la vidéo
await BinksyAPI.saveProject({ name: "<nom donné par l’utilisateur>" });
```

* Claude Code / Codex / Hermes / OpenCode : serveur MCP Playwright (`npx @playwright/mcp@latest`), ouvrir `https://motion.binksdesign.com/app.html`, puis appeler `window.BinksyAPI` via `browser_evaluate`.
* Agent du navigateur : Chrome 146+ avec `chrome://flags/#enable-webmcp-testing` (expérimental) ; les outils apparaissent seuls.
* `exporter` renvoie `{ files: [{ name, type, dataBase64 }] }` et ne télécharge rien sauf `download: true`. `reset` exige `confirm: true`, après accord de l’utilisateur.
* Rien n’est envoyé : pas de serveur, pas de compte, pas de clé. L’API ne lit ni n’écrit aucune clé API.

---

## 1. Règle fondamentale

Motion Lab anime les visuels que l’utilisateur fournit. Ce n’est **pas** un générateur d’images, de logos ni de textes de marque.

- **N’invente jamais** un logo, une image, une couleur de marque, une police ou un texte de marque (nom, slogan, titre de projet). Ce que l’utilisateur fournit (images, vidéos, logo, textes, couleurs) **fait foi**.
- Ne redessine pas un SVG fourni, ne le retouche pas : importe-le tel quel comme logo.
- Les **visuels d’exemple** et le **logo d’exemple** (Binks Design) chargés par l’app servent à tester : ne les livre jamais comme travail de l’utilisateur. Vide les emplacements (`#clearAll`) avant d’importer ses fichiers.
- Un titre que tu proposes reste une **suggestion** : fais-le valider avant de l’écrire dans un calque texte. Le texte par défaut d’un nouveau calque (« Nouveau projet ») doit être remplacé ou le calque supprimé.
- **Aucune clé API, aucun compte, aucun service en ligne.** L’app n’embarque aucune IA. Ne demande jamais de clé à l’utilisateur.
- **Tout est local** : lecture des fichiers, rendu WebGL2 et encodage vidéo se font dans le navigateur. Rien n’est envoyé à un serveur (seules les polices de l’interface sont chargées depuis Google Fonts et Fontshare).

Après chaque action, **vérifie l’état visible** (classe `.on` / `.active`, valeur affichée, emplacement rempli, message `#toast`). Un clic n’est pas une preuve de réussite.

## 2. Démarrer

Le dépôt est un site statique : `index.html` (accueil), `app.html` (l’application, un seul fichier), `docs.html`, `agent/index.html`, `AGENTS.md`, `llms.txt`, `robots.txt`, `assets/`.

```bash
python3 -m http.server 8000
# puis ouvrir http://127.0.0.1:8000/app.html?fresh
```

- `?fresh` ignore la session mémorisée et repart du modèle par défaut (Flux vitrine, 16:9). Recommandé pour un travail reproductible.
- L’app a besoin de **WebGL2** (rendu) et, pour l’export vidéo image par image, de **WebCodecs** (`VideoEncoder`). Chromium récent : OK. En headless, lance Chromium avec un rendu WebGL logiciel si besoin (`--use-gl=angle --use-angle=swiftshader --enable-unsafe-swiftshader`) ; l’export est alors lent.
- **Attendre que l’app soit prête** : `window.ML_READY === true` (posé quand les visuels d’exemple sont chargés).
- À l’ouverture, les **12 premiers emplacements** sont remplis par les visuels d’exemple.

### Objet global `window.ML` (interne)

L’app expose `window.ML`, un objet **interne** prévu pour ses propres tests (état `ML.st`, liste `ML.TEMPLATES`, etc.). Ce n’est pas une API publique ni stable. Utilise-le **en lecture seule** pour vérifier : par exemple `ML.st.tpl` (modèle courant), `ML.st.ratio`, `ML.st.dur`, `ML.st.texts`, `ML.TEMPLATES.map(t => [t.id, t.name, t.cat])`. Fais les modifications par l’interface.

## 3. Repères de l’interface

| Zone | Sélecteur | Rôle |
|---|---|---|
| Import | `#fileIn` | `input[type=file]` caché, multiple, `image/*,video/*`. Remplit les emplacements **vides**, puis ajoute à la suite. |
| Import | `#mDrop` | Zone « Importe tes images ou vidéos » (ouvre le sélecteur). |
| Médias | `#clearAll` | « Tout vider » : vide tous les emplacements. |
| Médias | `#demoBtn` | « Exemples » : remet les visuels d’exemple. |
| Médias | `#slots .slot[data-i="N"]` | Emplacement N (0 = premier). `.empty` = vide, `.off` = au-delà du nombre d’images utilisées. |
| Médias | `.slot [data-a="crop"]`, `[data-a="rep"]`, `[data-a="clr"]` | Recadrer, Remplacer (ouvre `#oneIn`), Vider. |
| Médias | `#slotMinus`, `#slotPlus` | Nombre d’images utilisées par le modèle (dans sa plage). |
| Modèles | `#tplSearch` | Recherche par nom de modèle ou de famille. |
| Modèles | `#lib .tpl[data-id="<id>"]` | Bouton d’un modèle ; `.on` = sélectionné. |
| Format | `#ratios [data-r="16:9"]` | `16:9`, `4:3`, `1:1`, `4:5`, `3:4`, `9:16` ; `.active` = courant. |
| Onglets | `.tab[data-tab="media"]`, `"style"`, `"text"` | Médias, Style, Texte. |
| Style | `#pane-style input.rg[data-p="tp.<modèle>.<réglage>"]` | Curseur d’un réglage du modèle. |
| Style | `#pane-style [data-sel][data-v]`, `select[data-selp]`, `[data-tg]` | Choix, listes, interrupteurs du modèle. |
| Style | `#tpReset` | Réinitialiser le modèle. |
| Fond | `#bgType [data-bt="color"]`, `"gradient"`, `"image"` | Type de fond. |
| Fond | `input[type=color][data-p="bg.c1"]`, `[data-p="bg.c2"]`, `input.rg[data-p="bg.angle"]` | Couleurs et angle. |
| Fond | `#bgIn` | `input[type=file]` de l’image de fond. |
| Durée | `#durs [data-d="5"]` … `"30"`, `#durRg` | Durée de la boucle (2 à 40 s). |
| Texte | `#addText` | Ajoute un calque texte (`t1`, `t2`…). |
| Texte | `.layer-card[data-tx="t1"] textarea[data-f="text"]` | Contenu du texte. |
| Texte | `select[data-f="font"]`, `[data-w="400\|500\|600"]`, `[data-al="left\|center\|right"]`, `[data-y]` | Police, graisse, alignement, position Haut/Centre/Bas. |
| Texte | `[data-p="tx.t1.size"]`, `"tx.t1.color"`, `"tx.t1.opacity"` | Taille, couleur, opacité. |
| Texte | `[data-del="t1"]` | Supprimer le calque. |
| Logo | `#logoPick` → `#logoIn` | Importer un logo (PNG ou SVG). |
| Logo | `#logoPos [data-x]`, `[data-p="logo.size"]`, `[data-p="logo.opacity"]`, `#logoClr` | Position, taille, opacité, retirer. |
| Images clés | `.kbtn[data-k="<chemin>"]` | Losange : pose ou retire une clé à l’instant courant. |
| Frise | `#playBtn`, `#scrub`, `#tTime` | Lecture/pause, curseur de temps, temps affiché. |
| Frise | `#tracks .tl-track[data-path] .kf[data-i]` | Clés d’un réglage animé. |
| Frise | `#kfBar [data-e="lin\|io\|in\|out\|hold"]`, `#kfDel` | Courbe de la clé sélectionnée, suppression. |
| Export | `#exportBtn` | Ouvre la boîte d’export. |
| Export | `#dialog [data-o="fmt"] [data-v="mp4\|webm\|png"]` | Format. |
| Export | `#dialog [data-o="res"] [data-v="1080\|2160\|4320"]` | Définition (4320 seulement si proposée). |
| Export | `#dialog [data-o="fps"] [data-v="30\|60"]` | Images/s (vidéo). |
| Export | `#xGo`, `#xCancel` | Lancer, annuler. |
| Projets | `#saveBtn` → `#pName`, `#pOk` | Enregistrer dans le navigateur. |
| Projets | `#projBtn` → `.proj [data-open]`, `[data-delp]`, `#expJson`, `#impJson` (→ `#jsonIn`), `#pClose` | Mes projets, export et import `.json`. |
| Divers | `#undoBtn`, `#redoBtn`, `#modal`, `#toast` | Annuler, rétablir, boîte de dialogue (attribut `hidden`), message d’état. |

Chemins d’images clés : `tp.<modèle>.<réglage>`, `bg.c1`, `bg.c2`, `bg.angle`, `tx.<id>.size|color|opacity|pos`, `logo.size|opacity`.

Les ids de modèles sont visibles dans `#lib .tpl[data-id]` (ex. `showcase-stream` Flux vitrine, `sphere-wall` Mur sphérique, `cover-flow` Cover flow, `rf-box` Cube, `ticker-tilt` Ticker incliné).

## 4. Parcours pas à pas

1. Ouvrir `app.html?fresh`, attendre `window.ML_READY === true`.
2. Cliquer `#clearAll`, puis `set_input_files("#fileIn", [...])` avec les fichiers de l’utilisateur. Vérifier que `#slots .slot:not(.empty)` en compte autant.
3. Choisir le format : `#ratios [data-r="9:16"]` (vérifier `.active`).
4. Choisir le modèle : `#lib .tpl[data-id="..."]` (vérifier `.on`). Si le modèle utilise plus d’images que fournies, il les répète ; ajuste avec `#slotMinus` / `#slotPlus` si l’utilisateur le demande.
5. Régler (onglet Style), le fond et la durée si demandé.
6. Textes et logo : seulement avec ce que l’utilisateur a fourni.
7. Exporter : `#exportBtn`, choisir format / définition / images/s, `#xGo`, attendre le téléchargement.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    b = p.chromium.launch()
    page = b.new_page(viewport={"width": 1440, "height": 900}, accept_downloads=True)
    page.goto("http://127.0.0.1:8000/app.html?fresh")
    page.wait_for_function("window.ML_READY === true", timeout=90_000)

    page.click("#clearAll")
    page.set_input_files("#fileIn", ["visuel-01.jpg", "visuel-02.jpg", "visuel-03.jpg"])
    page.wait_for_function("document.querySelectorAll('#slots .slot:not(.empty)').length >= 3")

    page.click('#ratios [data-r="9:16"]')
    page.click('#lib .tpl[data-id="cover-flow"]')
    page.click('#durs [data-d="10"]')

    page.click("#exportBtn")
    page.click('#dialog [data-o="fmt"] [data-v="mp4"]')
    page.click('#dialog [data-o="res"] [data-v="1080"]')
    with page.expect_download(timeout=900_000) as dl:
        page.click("#xGo")
    dl.value.save_as("cover-flow-9x16.mp4")
```

Régler un curseur : `fill()` ne marche pas sur un `input[type=range]`. Donne la valeur puis déclenche `input` (application) et `change` (mémorisation et historique) :

```js
const r = document.querySelector('input.rg[data-p="tp.sphere-wall.cornerRadius"]');
r.value = 4; r.dispatchEvent(new Event('input', { bubbles: true })); r.dispatchEvent(new Event('change', { bubbles: true }));
```

Une couleur : `page.fill('input[type=color][data-p="bg.c1"]', '#111111')`.

## 5. Images clés

1. Mettre en pause (`#playBtn` ou Espace) et placer le temps : clic sur `#scrub` à la position voulue (0 = début, largeur = fin), ou flèches ← / → (une image, Maj = 10 images).
2. Cliquer le losange `.kbtn[data-k="<chemin>"]` : une clé est posée à l’instant courant avec la valeur actuelle.
3. Placer un autre instant, changer la valeur du réglage : comme il est animé, la clé est créée ou modifiée à cet instant.
4. Vérifier dans `#tracks` : une piste `.tl-track[data-path="<chemin>"]` avec ses `.kf`.
5. Courbe : cliquer une `.kf`, puis `#kfBar [data-e=...]` (Linéaire `lin`, Douce `io`, Entrée `in`, Sortie `out`, Maintien `hold`).

Après la dernière clé, l’interpolation revient vers la première : la boucle reste parfaite.

## 6. Export et fichiers

- Par défaut : **MP4, 1080p, 30 images/s**. Le petit côté vaut la définition : 1080p en 16:9 → 1920 × 1080 ; en 9:16 → 1080 × 1920.
- Nombre d’images = durée de la boucle × images/s ; l’image `i` est rendue à `t = i / total`, donc la dernière enchaîne sur la première.
- MP4 : H.264, sinon VP9 ou AV1 dans le MP4. WebM : VP9, AV1 ou VP8. 8K proposée seulement si le navigateur et la carte graphique le permettent.
- Sans WebCodecs mais avec `MediaRecorder` : enregistrement en temps réel, 1080p 30 images/s. Sans aucun encodeur : vidéo désactivée, PNG seulement.
- PNG : l’image affichée à l’instant courant, en pleine définition.
- **Nom du fichier** : nom du projet simplifié + extension (`Sans titre` → `sans-titre.mp4`, `.webm`, `.png`). Le fichier arrive par téléchargement du navigateur (`expect_download`). En mise en page mobile, si le navigateur sait partager des fichiers, une boîte « C’est prêt » propose Télécharger ou Enregistrer ou partager.
- L’export peut prendre de quelques secondes à plusieurs minutes (bien plus en rendu logiciel). Le message `#toast` annonce « Vidéo exportée », « Image exportée », « Export annulé » ou « L’export a échoué : … ».

## 7. État persistant

| Où | Clé | Contenu |
|---|---|---|
| `localStorage` | `motionlab.session` | Session courante **sans les médias** : modèle, format, réglages, fond, durée, textes, logo (position, taille), images clés. Ignorée avec `?fresh`. |
| IndexedDB | base `motionlab`, magasin `projects` (clé `id`) | Projets enregistrés : `{ id, name, date, state, media: [{ id, type, name, blob }], thumb }`. |
| Fichier | `<nom>.json` (Mes projets → Exporter .json) | `{ app: "motion-lab", v: 1, state, media: [{ id, type, name, data }] }`, médias en data URL. |

Au rechargement, les emplacements reprennent les visuels d’exemple : pour garder les médias de l’utilisateur, enregistre le projet (`#saveBtn`) ou exporte le `.json`. Annuler / rétablir : 120 étapes, non conservées au rechargement.

## 8. Vérifications avant de rendre le travail

- Les emplacements utilisés contiennent **les fichiers de l’utilisateur**, pas les exemples (et pas de logo d’exemple).
- Le modèle (`.tpl.on`), le format (`#ratios .active`) et la durée (`#durVal`) sont ceux demandés.
- Aucun calque texte « Nouveau projet » oublié ; les textes sont ceux validés par l’utilisateur.
- Le fichier exporté existe, a la bonne extension, une taille non nulle, et (si possible) les bonnes dimensions et durée (`ffprobe`).
- Aucun message d’erreur dans `#toast`, aucune erreur dans la console.
- Dis à l’utilisateur où est le fichier et ce que tu as réglé.

## 9. Limites et erreurs

- Pas de son, ni à l’import ni à l’export.
- Une vidéo dans un emplacement se répète sur sa propre durée : pour une boucle parfaite, durée de boucle multiple de celle de la vidéo.
- Images réduites à 2048 px sur leur grand côté (1600 px sur appareil tactile).
- Textes à plat, par-dessus l’animation (ni inclinés ni en 3D) ; 4 polices : Instrument Sans, Clash Display, Instrument Serif, DM Mono.
- Les cartes sont triées carte par carte : avec des réglages extrêmes, deux cartes qui se croisent peuvent mal se superposer.
- Quelques modèles débordent volontairement du cadre (Ticker incliné, Cascade iso, Grande roue).
- Messages : « Ce fichier n’a pas pu être lu. » (média illisible), « Ce fichier n’est pas un projet Motion Lab. » (.json invalide), « L’export a échoué : … » (encodeur indisponible pour ce format ou cette définition : essaie WebM ou une définition plus basse).
- Sans WebGL2, l’app ne peut pas démarrer.

---
---

# Motion Lab — guide for AI agents

This document is for AI agents (Claude Code, Codex, or any agent driving a browser through Playwright / CDP). It explains how to use **Motion Lab** (`app.html`), Binks Design’s motion-template editor: put visuals into slots, pick one of 258 templates, set the frame format, style, background, texts and logo, add keyframes, then export an MP4 or WebM video (or a PNG image).

It lives in two places, always identical: `AGENTS.md` (Markdown, French then English) and the `agent/index.html` page. Short summary: `llms.txt`. Human documentation (French): `docs.html`.

**Important: Motion Lab has a public JavaScript API, `window.BinksyAPI`, and the same tools over WebMCP (section 0, `mcp/` page).** Prefer it; the interface can still be driven with the real selectors listed below. Do not invent functions: the current list is `await BinksyAPI.describe()`.

## 0. Agent API: `window.BinksyAPI`, WebMCP and the `/mcp/` page

**Prefer this API over the interface.** Once the app is ready, `window.BinksyAPI` exposes async methods returning JSON (never DOM nodes) that call the same logic as the UI: the screen updates and auto-save (IndexedDB) follows. Errors are rejected as `{ code, message }`. The same tools are registered through **WebMCP** (`document.modelContext.registerTool`, fallback `navigator.modelContext`) when the browser allows it; one definition table serves both. Setup page: `mcp/` (https://motion.binksdesign.com/mcp/).

| WebMCP tool | Method | Purpose |
|---|---|---|
| `motionlab_describe` | `BinksyAPI.describe()` | Describes the app and its tools. |
| `motionlab_get_state` | `BinksyAPI.getState()` | Reads the state. |
| `motionlab_lister_modeles` | `BinksyAPI.listTemplates()` | Lists / searches the 258 templates. |
| `motionlab_lire_modele` | `BinksyAPI.getTemplate()` | A template’s settings. |
| `motionlab_choisir_modele` | `BinksyAPI.selectTemplate()` | Picks the template. |
| `motionlab_choisir_format` | `BinksyAPI.setRatio()` | Frame format. |
| `motionlab_duree` | `BinksyAPI.setDuration()` | Loop duration (2–40 s). |
| `motionlab_nombre_images` | `BinksyAPI.setImageCount()` | Number of images used. |
| `motionlab_regler` | `BinksyAPI.setValues()` | Sets values by path. |
| `motionlab_fond` | `BinksyAPI.setBackground()` | Colour, gradient or image background. |
| `motionlab_importer` | `BinksyAPI.importer()` | Imports images / videos into slots, or a .json project. |
| `motionlab_vider` | `BinksyAPI.clearSlots()` | Empties slots. |
| `motionlab_exemples` | `BinksyAPI.loadExamples()` | Restores the example visuals (tests only). |
| `motionlab_ajouter_texte` | `BinksyAPI.addText()` | Adds a text layer (provided / approved text). |
| `motionlab_supprimer_texte` | `BinksyAPI.removeText()` | Removes a text layer. |
| `motionlab_logo` | `BinksyAPI.setLogo()` | Sets, places or removes the provided logo. |
| `motionlab_image_cle` | `BinksyAPI.setKeyframe()` | Adds / edits / removes a keyframe (0–1). |
| `motionlab_temps` | `BinksyAPI.setTime()` | Playhead and play / pause. |
| `motionlab_exporter` | `BinksyAPI.exporter()` | Exports MP4, WebM, PNG or .json project (base64). |
| `motionlab_enregistrer_projet` | `BinksyAPI.saveProject()` | Saves the project and its media in the browser. |
| `motionlab_lister_projets` | `BinksyAPI.listProjects()` | Lists saved projects. |
| `motionlab_ouvrir_projet` | `BinksyAPI.openProject()` | Opens a saved project (with media). |
| `motionlab_reset` | `BinksyAPI.reset()` | Resets the session (confirm: true). |

Imported media are not kept in the automatic session (only settings are): call motionlab_enregistrer_projet to get them back after a reload (motionlab_ouvrir_projet). Example visuals are flagged example: true in get_state: never deliver them.

* Claude Code / Codex / Hermes / OpenCode: Playwright MCP server (`npx @playwright/mcp@latest`), open `https://motion.binksdesign.com/app.html`, then call `window.BinksyAPI` through `browser_evaluate`, starting with `await BinksyAPI.describe()`.
* Browser agent: Chrome 146+ with `chrome://flags/#enable-webmcp-testing` (experimental); tools show up by themselves.
* `exporter` returns `{ files: [{ name, type, dataBase64 }] }` and downloads nothing unless `download: true`. `reset` requires `confirm: true`, after the user agrees.
* Nothing is uploaded: no server, no account, no key. The API never reads or writes an API key.

---

## 1. Golden rule

Motion Lab animates the visuals the user provides. It is **not** a generator of images, logos or brand copy.

- **Never invent** a logo, an image, a brand colour, a font or brand copy (name, tagline, project title). What the user provides (images, videos, logo, texts, colours) **is the reference**.
- Do not redraw or retouch a provided SVG: import it as is as the logo.
- The **example visuals** and **example logo** (Binks Design) loaded by the app are for testing: never deliver them as the user’s work. Empty the slots (`#clearAll`) before importing the user’s files.
- A title you propose is only a **suggestion**: get it approved before writing it into a text layer. The default text of a new layer (“Nouveau projet”) must be replaced or the layer deleted.
- **No API key, no account, no online service.** The app embeds no AI. Never ask the user for a key.
- **Everything is local**: file reading, WebGL2 rendering and video encoding happen in the browser. Nothing is uploaded (only the interface fonts are loaded from Google Fonts and Fontshare).

After each action, **check the visible state** (`.on` / `.active` class, displayed value, filled slot, `#toast` message). A click is not proof of success.

## 2. Getting started

The repository is a static site: `index.html` (home), `app.html` (the app, a single file), `docs.html`, `agent/index.html`, `AGENTS.md`, `llms.txt`, `robots.txt`, `assets/`.

```bash
python3 -m http.server 8000
# then open http://127.0.0.1:8000/app.html?fresh
```

- `?fresh` ignores the saved session and starts from the default template (Flux vitrine, 16:9). Recommended for reproducible work.
- The app needs **WebGL2** (rendering) and, for frame-by-frame video export, **WebCodecs** (`VideoEncoder`). Recent Chromium is fine. Headless, launch Chromium with software WebGL if needed (`--use-gl=angle --use-angle=swiftshader --enable-unsafe-swiftshader`); export is then slow.
- **Wait until the app is ready**: `window.ML_READY === true` (set once the example visuals are loaded).
- On load, the **first 12 slots** are filled with example visuals.

### Global object `window.ML` (internal)

The app exposes `window.ML`, an **internal** object meant for its own tests (state `ML.st`, list `ML.TEMPLATES`, etc.). It is neither a public nor a stable API. Use it **read-only** for checks, e.g. `ML.st.tpl` (current template), `ML.st.ratio`, `ML.st.dur`, `ML.st.texts`, `ML.TEMPLATES.map(t => [t.id, t.name, t.cat])`. Make changes through the interface.

## 3. Interface hooks

| Area | Selector | Role |
|---|---|---|
| Import | `#fileIn` | Hidden `input[type=file]`, multiple, `image/*,video/*`. Fills **empty** slots, then appends. |
| Import | `#mDrop` | “Importe tes images ou vidéos” drop zone (opens the picker). |
| Media | `#clearAll` | “Tout vider”: empties every slot. |
| Media | `#demoBtn` | “Exemples”: puts the example visuals back. |
| Media | `#slots .slot[data-i="N"]` | Slot N (0 = first). `.empty` = empty, `.off` = beyond the number of images used. |
| Media | `.slot [data-a="crop"]`, `[data-a="rep"]`, `[data-a="clr"]` | Crop, Replace (opens `#oneIn`), Empty. |
| Media | `#slotMinus`, `#slotPlus` | Number of images used by the template (within its range). |
| Templates | `#tplSearch` | Search by template or family name. |
| Templates | `#lib .tpl[data-id="<id>"]` | A template button; `.on` = selected. |
| Format | `#ratios [data-r="16:9"]` | `16:9`, `4:3`, `1:1`, `4:5`, `3:4`, `9:16`; `.active` = current. |
| Tabs | `.tab[data-tab="media"]`, `"style"`, `"text"` | Media, Style, Text. |
| Style | `#pane-style input.rg[data-p="tp.<template>.<setting>"]` | Slider of a template setting. |
| Style | `#pane-style [data-sel][data-v]`, `select[data-selp]`, `[data-tg]` | Template choices, lists, toggles. |
| Style | `#tpReset` | Reset the template. |
| Background | `#bgType [data-bt="color"]`, `"gradient"`, `"image"` | Background type. |
| Background | `input[type=color][data-p="bg.c1"]`, `[data-p="bg.c2"]`, `input.rg[data-p="bg.angle"]` | Colours and angle. |
| Background | `#bgIn` | `input[type=file]` for the background image. |
| Duration | `#durs [data-d="5"]` … `"30"`, `#durRg` | Loop duration (2 to 40 s). |
| Text | `#addText` | Adds a text layer (`t1`, `t2`…). |
| Text | `.layer-card[data-tx="t1"] textarea[data-f="text"]` | Text content. |
| Text | `select[data-f="font"]`, `[data-w="400\|500\|600"]`, `[data-al="left\|center\|right"]`, `[data-y]` | Font, weight, alignment, Top/Centre/Bottom position. |
| Text | `[data-p="tx.t1.size"]`, `"tx.t1.color"`, `"tx.t1.opacity"` | Size, colour, opacity. |
| Text | `[data-del="t1"]` | Delete the layer. |
| Logo | `#logoPick` → `#logoIn` | Import a logo (PNG or SVG). |
| Logo | `#logoPos [data-x]`, `[data-p="logo.size"]`, `[data-p="logo.opacity"]`, `#logoClr` | Position, size, opacity, remove. |
| Keyframes | `.kbtn[data-k="<path>"]` | Diamond: adds or removes a key at the current time. |
| Timeline | `#playBtn`, `#scrub`, `#tTime` | Play/pause, time scrubber, displayed time. |
| Timeline | `#tracks .tl-track[data-path] .kf[data-i]` | Keys of an animated setting. |
| Timeline | `#kfBar [data-e="lin\|io\|in\|out\|hold"]`, `#kfDel` | Curve of the selected key, delete. |
| Export | `#exportBtn` | Opens the export dialog. |
| Export | `#dialog [data-o="fmt"] [data-v="mp4\|webm\|png"]` | Format. |
| Export | `#dialog [data-o="res"] [data-v="1080\|2160\|4320"]` | Resolution (4320 only when offered). |
| Export | `#dialog [data-o="fps"] [data-v="30\|60"]` | Frames per second (video). |
| Export | `#xGo`, `#xCancel` | Start, cancel. |
| Projects | `#saveBtn` → `#pName`, `#pOk` | Save in the browser. |
| Projects | `#projBtn` → `.proj [data-open]`, `[data-delp]`, `#expJson`, `#impJson` (→ `#jsonIn`), `#pClose` | My projects, `.json` export and import. |
| Misc | `#undoBtn`, `#redoBtn`, `#modal`, `#toast` | Undo, redo, dialog (`hidden` attribute), status message. |

Keyframe paths: `tp.<template>.<setting>`, `bg.c1`, `bg.c2`, `bg.angle`, `tx.<id>.size|color|opacity|pos`, `logo.size|opacity`.

Template ids are visible in `#lib .tpl[data-id]` (e.g. `showcase-stream` Flux vitrine, `sphere-wall` Mur sphérique, `cover-flow` Cover flow, `rf-box` Cube, `ticker-tilt` Ticker incliné).

## 4. Step by step

1. Open `app.html?fresh`, wait for `window.ML_READY === true`.
2. Click `#clearAll`, then `set_input_files("#fileIn", [...])` with the user’s files. Check that `#slots .slot:not(.empty)` counts as many.
3. Pick the format: `#ratios [data-r="9:16"]` (check `.active`).
4. Pick the template: `#lib .tpl[data-id="..."]` (check `.on`). If the template uses more images than provided, it repeats them; adjust with `#slotMinus` / `#slotPlus` if the user asks.
5. Adjust (Style tab), background and duration if asked.
6. Texts and logo: only with what the user provided.
7. Export: `#exportBtn`, pick format / resolution / fps, `#xGo`, wait for the download.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    b = p.chromium.launch()
    page = b.new_page(viewport={"width": 1440, "height": 900}, accept_downloads=True)
    page.goto("http://127.0.0.1:8000/app.html?fresh")
    page.wait_for_function("window.ML_READY === true", timeout=90_000)

    page.click("#clearAll")
    page.set_input_files("#fileIn", ["visual-01.jpg", "visual-02.jpg", "visual-03.jpg"])
    page.wait_for_function("document.querySelectorAll('#slots .slot:not(.empty)').length >= 3")

    page.click('#ratios [data-r="9:16"]')
    page.click('#lib .tpl[data-id="cover-flow"]')
    page.click('#durs [data-d="10"]')

    page.click("#exportBtn")
    page.click('#dialog [data-o="fmt"] [data-v="mp4"]')
    page.click('#dialog [data-o="res"] [data-v="1080"]')
    with page.expect_download(timeout=900_000) as dl:
        page.click("#xGo")
    dl.value.save_as("cover-flow-9x16.mp4")
```

Setting a slider: `fill()` does not work on an `input[type=range]`. Set the value, then dispatch `input` (applies it) and `change` (saves it and records history):

```js
const r = document.querySelector('input.rg[data-p="tp.sphere-wall.cornerRadius"]');
r.value = 4; r.dispatchEvent(new Event('input', { bubbles: true })); r.dispatchEvent(new Event('change', { bubbles: true }));
```

A colour: `page.fill('input[type=color][data-p="bg.c1"]', '#111111')`.

## 5. Keyframes

1. Pause (`#playBtn` or Space) and set the time: click `#scrub` at the wanted position (left = start, right = end), or ← / → keys (one frame, Shift = 10 frames).
2. Click the diamond `.kbtn[data-k="<path>"]`: a key is added at the current time with the current value.
3. Move to another time and change the setting: since it is animated, the key is created or updated at that time.
4. Check `#tracks`: a `.tl-track[data-path="<path>"]` track with its `.kf` keys.
5. Curve: click a `.kf`, then `#kfBar [data-e=...]` (Linear `lin`, Smooth `io`, In `in`, Out `out`, Hold `hold`).

After the last key, interpolation goes back to the first one: the loop stays seamless.

## 6. Export and files

- Default: **MP4, 1080p, 30 fps**. The short side equals the resolution: 1080p in 16:9 → 1920 × 1080; in 9:16 → 1080 × 1920.
- Frame count = loop duration × fps; frame `i` is rendered at `t = i / total`, so the last frame flows into the first.
- MP4: H.264, else VP9 or AV1 in MP4. WebM: VP9, AV1 or VP8. 8K is offered only when the browser and GPU allow it.
- Without WebCodecs but with `MediaRecorder`: real-time recording, 1080p 30 fps. With no encoder at all: video disabled, PNG only.
- PNG: the frame shown at the current time, at full resolution.
- **File name**: simplified project name + extension (`Sans titre` → `sans-titre.mp4`, `.webm`, `.png`). The file arrives as a browser download (`expect_download`). In the mobile layout, when the browser can share files, a “C’est prêt” dialog offers Télécharger (download) or Enregistrer ou partager (share).
- Export takes from a few seconds to several minutes (much longer with software rendering). The `#toast` message reads “Vidéo exportée”, “Image exportée”, “Export annulé” or “L’export a échoué : …”.

## 7. Persistent state

| Where | Key | Content |
|---|---|---|
| `localStorage` | `motionlab.session` | Current session **without media**: template, format, settings, background, duration, texts, logo (position, size), keyframes. Ignored with `?fresh`. |
| IndexedDB | database `motionlab`, store `projects` (key `id`) | Saved projects: `{ id, name, date, state, media: [{ id, type, name, blob }], thumb }`. |
| File | `<name>.json` (Mes projets → Exporter .json) | `{ app: "motion-lab", v: 1, state, media: [{ id, type, name, data }] }`, media as data URLs. |

On reload, slots go back to the example visuals: to keep the user’s media, save the project (`#saveBtn`) or export the `.json`. Undo / redo: 120 steps, not kept across reloads.

## 8. Checks before handing back

- The slots in use hold **the user’s files**, not the examples (and no example logo).
- Template (`.tpl.on`), format (`#ratios .active`) and duration (`#durVal`) are the requested ones.
- No forgotten “Nouveau projet” text layer; texts are the ones the user approved.
- The exported file exists, has the right extension, a non-zero size and, if possible, the right dimensions and duration (`ffprobe`).
- No error message in `#toast`, no error in the console.
- Tell the user where the file is and what you set.

## 9. Limits and errors

- No sound, neither on import nor on export.
- A video in a slot repeats on its own duration: for a seamless loop, make the loop duration a multiple of the video’s.
- Images are downscaled to 2048 px on their long side (1600 px on touch devices).
- Texts stay flat, on top of the animation (not tilted, not 3D); 4 fonts: Instrument Sans, Clash Display, Instrument Serif, DM Mono.
- Cards are sorted card by card: with extreme settings, two crossing cards may overlap in the wrong order.
- A few templates overflow the frame on purpose (Ticker incliné, Cascade iso, Grande roue).
- Messages: “Ce fichier n’a pas pu être lu.” (unreadable media), “Ce fichier n’est pas un projet Motion Lab.” (invalid .json), “L’export a échoué : …” (no encoder for this format or resolution: try WebM or a lower resolution).
- Without WebGL2, the app cannot start.
