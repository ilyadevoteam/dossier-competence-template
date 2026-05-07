# Template Dossier de Compétence Interactif

Un template prêt à l'emploi pour transformer ton dossier de compétence en site interactif :
- 💬 **Chat IA** à gauche — répond aux questions des recruteurs / clients sur ton parcours.
- 📜 **Page scrolly-telling** à droite — présente expériences, compétences, formations.

Stack : **HTML/CSS/JS vanilla** + **Google Gemini** (proxy serverless) — déployé sur **Vercel**.

---

## 🚀 Démarrage rapide

### 1. Récupérer une clé API Gemini

Va sur [Google AI Studio](https://aistudio.google.com/apikey), crée une clé (gratuite jusqu'à un quota confortable).

### 2. Configurer la clé en local

```bash
cp .env.example .env.local
```

Ouvre `.env.local` et remplace le placeholder par ta vraie clé. Vercel lit `.env.local` en dev (jamais commité, voir `.gitignore`).

### 3. Remplacer le contenu par le tien

- **`api/corpus.md`** → ton dossier de compétence en markdown. Servi uniquement en contexte au chatbot (jamais exposé publiquement).
- **`index.html`** → édite le hero, les cartes d'expérience, compétences, formations.
- **`assets/avatar-portrait-*.png`** → ton portrait. Le hero utilise `homme` par défaut, change le `src` dans le code pour utiliser `femme` ou ta propre image.
- **`assets/super-cat-devoteam.png`** → mascotte du chat. Optionnelle.

### 4. Lancer en local

```bash
npm install
npm run dev    # vercel dev — sert le statique + la fonction serverless
```

Ouvre l'URL affichée (par défaut `http://localhost:3000`).

---

## 📂 Structure

```
template_dossier_compétence/
├── README.md
├── .env.example                 # template à copier en .env.local
├── .gitignore                   # exclut .env*, node_modules, .vercel
├── .vercelignore                # exclut README.md du déploiement
├── package.json                 # devDeps Vercel uniquement
├── tsconfig.json                # types pour la fonction serverless
├── vercel.json                  # runtime + includeFiles + headers
├── index.html                   # site complet (chat + scrolly)
├── api/
│   ├── chat.ts                  # fonction serverless — proxy Gemini
│   └── corpus.md                # ⭐ dossier de compétence (à remplacer)
├── vendor/                      # police Inter auto-hébergée
└── assets/                      # avatars + mascotte
```

---

## 🌐 Déploiement sur Vercel

Premier déploiement :

```bash
npx vercel link            # lie le repo au projet Vercel
npx vercel env add GEMINI_API_KEY production
npx vercel --prod
```

Après ça, **chaque `git push` sur `main` redéploie automatiquement** si tu as connecté le repo GitHub via l'UI Vercel (Project Settings → Git).

### Variables d'environnement Vercel

| Var | Prod | Dev (`.env.local`) | Défaut |
|---|---|---|---|
| `GEMINI_API_KEY` | ✅ obligatoire | ✅ obligatoire | — |
| `GEMINI_MODEL` | optionnel | optionnel | `gemini-2.0-flash` |
| `GEMINI_CACHE` | optionnel | optionnel | `true` (Gemini context caching) |

---

## 🔒 Sécurité

- `GEMINI_API_KEY` reste **côté serveur uniquement** (lue via `process.env` dans `api/chat.ts`). Jamais dans le bundle frontend.
- `api/corpus.md` est bundlé avec la fonction (`vercel.json` → `includeFiles`) mais **non servi en statique** — Vercel ignore les fichiers non-handlers de `api/`.
- `.env`, `.env.local`, `.vercel/` sont dans `.gitignore`.
- Le chatbot ne répond qu'à partir du corpus (pas d'hallucination libre, prompt système verrouillé).

---

## 🎨 Personnalisation

### Couleurs

Variables CSS au début de la balise `<style>` dans `index.html` :

```css
--red-poppy: #F8485E;       /* Couleur principale (Devoteam) */
--fresh-mint: #22D69B;      /* Accent secondaire */
--cream: #FBF5EC;           /* Fond général */
```

### Prompts suggérés du chat

Cherche `prompt-button` dans `index.html` — adapte texte + `data-prompt` des 4 boutons.

### Système prompt du chatbot

Édite la fonction `buildSystemPrompt(gender)` dans `api/chat.ts` (ou la constante `SYSTEM_PROMPT` qui en dérive pour le mode `homme`).

### Supprimer le toggle homme/femme (recommandé pour un usage perso)

Le template a un **bouton de bascule de genre** dans le hero (à côté du portrait). C'est une petite feature de démo (Thomas ↔ Sarah avec swap des accords + avatar + chatbot). Pour ton usage perso tu n'en as pas besoin — voici comment le retirer proprement.

**1. Dans `index.html` — supprime ces blocs :**

| Quoi | Où chercher | Action |
|---|---|---|
| Bouton toggle | `<button class="gender-toggle" id="gender-toggle"` | Supprimer le `<button>` (4-5 lignes) |
| Script inline pre-paint | `<script>` juste après le bouton, IIFE qui swap avatar+text | Supprimer le bloc `<script>...</script>` |
| Pre-resolve dans `<head>` | Le `<script>` en haut du `<head>` qui set `window.__avatarGender` | Supprimer le bloc |
| Bloc JS toggle | `// ============ AVATAR GENDER TOGGLE ============` | Supprimer jusqu'à la ligne `genderToggleEl.addEventListener('click', ...)` incluse (~50 lignes) |
| Fonction `applyAccord` | `function applyAccord(text)` (dans le modal renderer) | Supprimer la fonction. Dans `openExpModal`, remplace `applyAccord(data.role)` → `data.role`, `applyAccord(data.context)` → `data.context`, `applyAccord(a)` → `a` |
| Champ `gender` dans fetch | `gender: window.__currentGender || 'homme'` (dans le `JSON.stringify` du fetch `/api/chat`) | Supprimer ce champ |

**2. Dans `index.html` — convertis les balises gender-aware en texte simple, avec ton vrai prénom :**

```html
<!-- Avant -->
<span data-text-m="Thomas" data-text-f="Sarah">Thomas</span>
<!-- Après -->
TonPrénom
```

5 occurrences pour `Thomas`/`Sarah` (panel, agent-tag, hero h1, footer, et 1 dans le span accent du hero). 2 occurrences pour `Consultant`/`Consultante` (Devoteam exp role + modal role) — garde la forme qui correspond à ton genre.

Pour les **4 prompt buttons** :
```html
<!-- Avant -->
<button class="prompt-button" data-prompt-template="Résume-moi l'expérience de {name} chez TotalEnergies.">...</button>
<!-- Après -->
<button class="prompt-button" data-prompt="Résume-moi l'expérience de TonPrénom chez TotalEnergies.">...</button>
```

**3. Dans `api/chat.ts` — simplifie :**

- Supprime le type `Gender`
- Supprime la fonction `buildSystemPrompt(gender)` — récupère son contenu pour reconstruire un `SYSTEM_PROMPT` constant unique (avec ton prénom + tes accords)
- Supprime la lecture de `body.gender` du body de la requête
- Le chemin `promptForGender = gender === "homme" ? SYSTEM_PROMPT : buildSystemPrompt("femme")` devient simplement `SYSTEM_PROMPT`
- Simplifie : `activeCache = await ensureCache()` toujours (plus de branche par genre)

**4. Dans `api/corpus.md`** : remplace `Thomas PASTÈQUE` par ton nom complet (et adapte les accords si tu es au féminin — le corpus est rédigé au masculin par défaut).

**5. Avatar — fixe le src :**

Une fois le JS toggle retiré, l'`<img id="avatar-img">` n'a plus de `src` initial. Ajoute-le en dur :
```html
<img id="avatar-img" src="./assets/avatar-portrait-homme.png" alt="Portrait illustré du candidat"/>
```

(Tu peux supprimer le PNG du genre non utilisé pour alléger le déploiement.)

#### Récap fichiers impactés

| Fichier | Travail |
|---|---|
| `index.html` | ~70 lignes à éditer (button, scripts, JS toggle, attributs `data-text-*`, prompts, fetch) |
| `api/chat.ts` | Refactor `buildSystemPrompt` → `SYSTEM_PROMPT` constant, retirer `body.gender` et la branche par genre (~25 lignes) |
| `api/corpus.md` | Find & replace nom complet + accords si féminin |
| `assets/` | Optionnel : supprimer le PNG inutilisé |

---

## 🧪 Vérifications avant de partager

Local (`npm run dev`) :
1. La page se charge (chat à gauche, scrolly à droite).
2. Une question dans le chat → réponse contextuelle correcte.
3. DevTools Network → seul `/api/chat` est appelé (jamais `googleapis.com` directement depuis le browser).

Production :
1. Visiter ton URL Vercel — chat fonctionne.
2. `https://ton-app.vercel.app/api/corpus.md` doit retourner **404** (corpus protégé).
3. `https://ton-app.vercel.app/.env` doit retourner **404**.