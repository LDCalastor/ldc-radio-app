# LDC RADIO — Desktop App (Electron)

Webradio LDC RADIO packagée en application desktop Windows/macOS/Linux avec :
- **Discord Rich Presence** (ce que tu écoutes s'affiche sur ton profil)
- **Auto-update depuis GitHub Releases** (popup au démarrage si nouvelle version)
- **System tray** (minimise dans la barre des tâches)
- **Always-on-top** (fenêtre toujours au-dessus)

![Version](https://img.shields.io/badge/version-2.4.0-ff002b?style=flat-square)
![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-00f0ff?style=flat-square)
![Electron](https://img.shields.io/badge/Electron-34-9d00ff?style=flat-square)

---

## 🚀 Setup rapide (10 minutes)

### 1. Prérequis
- **Node.js 20+** → https://nodejs.org/ (prendre la LTS)
- **Compte GitHub** avec un repo `LDCalastor/ldc-radio-app` créé (pour les releases + auto-update)

### 2. Configuration

Ouvre `config.js` et remplis :

```js
DISCORD_CLIENT_ID: 'TON_ID_DISCORD_ICI',  // https://discord.com/developers/applications
WEB_URL: 'https://ldc-radio.netlify.app/',
RELEASES_URL: 'https://github.com/LDCalastor/ldc-radio-app/releases/',
```

**Pour le Discord RPC** :
1. Va sur https://discord.com/developers/applications → **New Application**
2. Nom : `LDC RADIO`
3. Copie l'**Application ID** → colle-le dans `config.js`
4. Onglet **Rich Presence → Art Assets** → upload ces 4 images :
   - `ldc_logo` (logo principal)
   - `ldc_phonk`
   - `ldc_funk`
   - `ldc_mix`

### 3. Installer les dépendances

```bash
npm install
```

Ça télécharge : Electron (~80 Mo), electron-builder (~40 Mo), discord-rpc, **electron-updater**, electron-log.

### 4. Tester

```bash
npm run dev
```

DevTools s'ouvrent. Joue une piste → vérifie Discord.

### 5. Builder

```bash
npm run build:win
```

→ Crée dans `dist/` :
- `LDC-RADIO-Setup-3.2.0.exe` (installateur NSIS)
- `LDC-RADIO-Portable-3.2.0.exe` (portable)
- `latest.yml` ← **fichier critique pour l'auto-update**

---

## 📦 Publier une nouvelle version (workflow auto-update)

L'auto-update pointe vers **GitHub Releases**. Pour qu'il détecte une nouvelle version, tu dois publier sur GitHub.

### Méthode 1 : Publication manuelle

1. **Bump la version** dans `package.json` :
   ```json
   "version": "3.3.0"
   ```

2. **Build** :
   ```bash
   npm run build:win
   ```

3. **Aller sur GitHub** → repo `ldc-radio-app` → **Releases** → **Draft a new release**

4. **Tag version** : `v3.3.0` (avec le `v` devant)

5. **Upload ces fichiers depuis `dist/`** :
   - `LDC-RADIO-Setup-3.3.0.exe`
   - `LDC-RADIO-Portable-3.3.0.exe`
   - `latest.yml` ← **OBLIGATOIRE** (c'est lui qui est lu par electron-updater)
   - `LDC-RADIO-Setup-3.3.0.exe.blockmap` (optionnel, pour les diff-updates)

6. **Publish release**

7. ✅ Les utilisateurs existants verront la popup de mise à jour au prochain lancement (ou dans l'heure qui suit).

### Méthode 2 : Publication automatique (recommandé)

Crée un **Personal Access Token** GitHub :
- https://github.com/settings/tokens → **Generate new token (classic)** → scope `repo`
- Copie le token

Puis lance :

**Windows PowerShell :**
```powershell
$env:GH_TOKEN = "ghp_xxxxxxxxxxxx"
npm run build:win -- --publish always
```

**Windows CMD :**
```cmd
set GH_TOKEN=ghp_xxxxxxxxxxxx
npm run build:win -- --publish always
```

**macOS/Linux :**
```bash
export GH_TOKEN="ghp_xxxxxxxxxxxx"
npm run build:win -- --publish always
```

→ electron-builder crée automatiquement la release sur GitHub, upload les binaires et `latest.yml`. Tout en 1 commande.

---

## 🔄 Comment fonctionne l'auto-update

### Côté utilisateur

1. **Au démarrage** (5s après le lancement), l'app vérifie `https://github.com/LDCalastor/ldc-radio-app/releases/latest`
2. **Si une version plus récente existe** → **POPUP** :
   ```
   ┌────────────────────────────────────────┐
   │  Nouvelle version disponible           │
   │                                        │
   │  LDC RADIO 3.3.0 est disponible        │
   │  Tu utilises actuellement la v3.2.0    │
   │                                        │
   │  Notes : ...                           │
   │                                        │
   │  [Télécharger maintenant] [Plus tard]  │
   │  [Voir les détails]                    │
   └────────────────────────────────────────┘
   ```
3. Si l'utilisateur clique **Télécharger** → progression visible (dans la barre des tâches Windows + dans la page Settings)
4. Téléchargement fini → **2e POPUP** :
   ```
   ┌────────────────────────────────────────┐
   │  Mise à jour prête                     │
   │                                        │
   │  LDC RADIO 3.3.0 est prête             │
   │                                        │
   │  L'app va redémarrer pour appliquer.   │
   │                                        │
   │  [Redémarrer maintenant] [Plus tard]   │
   └────────────────────────────────────────┘
   ```
5. Si l'utilisateur clique **Redémarrer** → `autoUpdater.quitAndInstall()` → l'installateur NSIS se lance, l'app se relance en 3.3.0
6. Sinon, la mise à jour s'applique au prochain lancement

### Vérification manuelle (à tout moment)

- **Settings → section APP DESKTOP → bouton "VÉRIFIER MISE À JOUR"**
- **Tray (clic droit sur l'icône) → "Vérifier les mises à jour"**
- **Menu Aide → "Vérifier les mises à jour"**

### Paramètres dans `config.js`

```js
UPDATE_CHECK_ON_STARTUP: true,       // Vérifier au lancement
UPDATE_CHECK_INTERVAL_MIN: 60,       // Re-vérifier toutes les 60 min
UPDATE_AUTO_DOWNLOAD: false,         // false = demander avant de télécharger
UPDATE_ALLOW_PRERELEASE: false,      // true = installer aussi les pré-releases
```

---

## 🎮 Discord Rich Presence — rendu attendu

```
┌──────────────────────────────────────────────┐
│ [artwork PHONK]   ♫ TRACK VOL.42             │
│ [mini LDC logo]   sur LDC PHONK              │
│                    ↑ 03:17 écoulées           │
│                                              │
│ [🎧 Écouter en ligne]                        │
│ [⬇️ Télécharger l'app]                        │
└──────────────────────────────────────────────┘
```

Les 2 boutons pointent vers tes URLs configurées dans `config.js` (site web + GitHub releases).

**Important** : tu dois avoir uploadé les 4 images (`ldc_logo`, `ldc_phonk`, `ldc_funk`, `ldc_mix`) sur le Discord Developer Portal avec ces noms EXACTS.

---

## 🗂 Structure du projet

```
ldc-radio-app/
├── main.js                 # Process principal (Electron + RPC + Auto-Update)
├── preload.js              # Bridge sécurisé IPC
├── config.js               # Configuration centralisée
├── package.json            # Dépendances + config electron-builder
├── index.html              # Application webradio avec hooks Electron
├── build/
│   ├── icon.ico            # Icône Windows (à fournir)
│   ├── icon.icns           # Icône macOS (à fournir)
│   └── icon.png            # Icône Linux (à fournir)
├── assets/                 # Vidéos de fond (optionnel)
│   ├── 1.mp4
│   └── ...
└── dist/                   # Builds (généré automatiquement)
    ├── LDC-RADIO-Setup-3.2.0.exe
    ├── LDC-RADIO-Portable-3.2.0.exe
    └── latest.yml          # ← Fichier lu par electron-updater
```

---

## 🖼 Icônes

Place dans `build/` :

| Plateforme | Fichier | Résolution |
|------------|---------|------------|
| Windows    | `icon.ico` | Multi-res (16, 32, 48, 64, 128, 256) |
| macOS      | `icon.icns` | Multi-res (16 → 1024) |
| Linux      | `icon.png` | 512×512 minimum |

Conversion rapide avec ImageMagick :
```bash
magick icon-1024.png -define icon:auto-resize=256,128,64,48,32,16 build/icon.ico
```

Ou en ligne : https://icoconvert.com/

---

## 🛠 Scripts npm

| Commande | Action |
|----------|--------|
| `npm start` | Lance l'app |
| `npm run dev` | Lance avec DevTools (mode dev, auto-update désactivé) |
| `npm run build:win` | Build Windows (NSIS + portable) |
| `npm run build:mac` | Build macOS (DMG) |
| `npm run build:linux` | Build Linux (AppImage + deb) |
| `npm run build` | Build pour la plateforme courante |

---

## 🐛 Troubleshooting

### "Discord RPC ne s'affiche pas"
- Discord doit être **lancé** sur le PC AVANT d'ouvrir l'app
- Vérifie le `DISCORD_CLIENT_ID` dans `config.js`
- Dans Discord : Settings → Activity Privacy → "Display current activity" = ON
- Ouvre DevTools (F12 en dev) → Console → cherche `[RPC] ✓ Connecté à Discord`

### "Les images du RPC ne s'affichent pas"
- Les clés (`ldc_logo`, `ldc_phonk`, etc.) doivent correspondre **exactement** aux noms uploadés sur Discord Developer Portal
- Discord met 5-10 min à mettre à jour le cache après upload d'asset

### "L'auto-update ne détecte pas la nouvelle version"
1. Vérifie que `latest.yml` est bien présent dans la release GitHub
2. Le repo doit être **public** (ou tu configures un token d'accès)
3. Vérifie la version : la v3.3.0 ne sera détectée que si l'app actuelle est en < 3.3.0
4. Attends 5-10s après le lancement, la vérif n'est pas instantanée
5. Log file : `%APPDATA%\LDC RADIO\logs\main.log` (Windows)

### "Erreur Cannot create symbolic link" (Windows)
→ Active le **mode développeur** Windows (Paramètres → Confidentialité et sécurité → Pour les développeurs) et relance le terminal.

### "SmartScreen bloque le .exe"
Normal pour les apps non signées. Clic droit → Propriétés → **Débloquer**, ou utiliser la version portable.

### "Build échoue avec 'cannot find module'"
```bash
rm -rf node_modules package-lock.json
npm install
```

---

## 📄 Liens

- **Site web** : https://ldc-radio.netlify.app/
- **Releases** : https://github.com/LDCalastor/ldc-radio-app/releases/
- **Discord Developer Portal** : https://discord.com/developers/applications

---

© 2025 ALASTOR — LDC RADIO
