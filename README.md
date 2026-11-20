postmarketOS (Openbox) for Samsung Galaxy Tab E 9.6 (SM-T561 / gtel3g)
 by Jimmy
# Samsung Galaxy Tab E (SM-T561 / `gtel3g`) — Guide Interface / Wi-Fi sous postmarketOS

reprise du dépôt qui etait déjà sur postmarket os .mais c'était en travaux .jeu essayé de reprendre le projet .

Ce dépôt et cette archive contiennent tout le nécessaire pour transformer une **Samsung Galaxy Tab E 9.6 3G (SM-T561)** sous **postmarketOS** (noyau Linux `3.10.17`) en une tablette Linux légère, tactile, (~150 Mo de RAM utilisés au repos) et 100 % autonome en Wi-Fi.

---encore du boulot pour le son , bluetooh ect.... mais le wif iest stable pour l'instant 


La Samsung Galaxy Tab E (SM-T561) utilise un processeur Spreadtrum SC7730SE verrouillé sur un noyau Linux **3.10.17**. Sur ce noyau ancien, une installation vierge de postmarketOS souffre de plusieurs blocages majeurs :
1. **Fermeture instantanée des applications (erreur `ENOSYS`) :** Les bibliothèques récentes tentent de charger des icônes vectorielles `.svg` ou des fonctions GTK4 non supportées par le noyau `3.10.17`.
2. **Wi-Fi instable ou refusant le mot de passe :** `NetworkManager` envoie des options modernes (PMF / 802.11w, randomisation MAC) rejetées par l'ancien pilote Broadcom/Spreadtrum, et le DNS est régulièrement effacé.
3. **Interface peu adaptée au tactile :** Les bureaux classiques consomment trop de mémoire vive sur les 1,5 Go de RAM de la tablette.

Ce pack résout automatiquement tous ces points.

---

## Contenu de l'archive (`SM-T561-Pack-Complet.zip`)

* **`install.sh`** : Script d'installation automatique qui configure l'ensemble du système.
* **`README.md`** : Ce guide complet d'installation et d'utilisation.
* **`config/`** : Fichiers de configuration optimisés pour **Openbox** (fenêtres), **tint2** (barre des tâches tactile en bas avec thème sombre et accents vert menthe), **jgmenu** (menu tactile à grandes icônes) et **GTK-3.0**.
* **`apps/`** : Raccourcis `.desktop` adaptés au tactile (Menu, Navigateur, Gestionnaire de fichiers, Terminal, Clavier virtuel, Extinction propre).
* **`bin/bat-status.sh`** : Script léger affichant le pourcentage réel de la batterie et l'état de charge directement dans la barre du bas.
* **`Pictures/`** : Fond d'écran léger intégré.
* **`wifi/wpa_supplicant.conf.exemple`** : Fichier modèle prêt à remplir pour une connexion Wi-Fi directe sans `NetworkManager`.

---

## Partie 1 : Flasher postmarketOS sur la mémoire interne (Mode Download)

Si postmarketOS n'est pas encore installé sur la mémoire interne de votre tablette (`mmcblk0p26`) :

### 1. Installer les outils sur le PC (Linux)
* **Sur Arch Linux / CachyOS :**
  ```bash
  sudo pacman -S heimdall android-tools
  ```
* **Sur Debian / Ubuntu / Linux Mint :**
  ```bash
  sudo apt install heimdall-flash android-sdk-libsparse-utils
  ```

### 2. Convertir l'image système en format sparse (si nécessaire)
Le mode Download de Samsung exige une image au format **Android sparse** (qui compresse l'espace vide pour un transfert rapide). Si votre image `samsung-gtel3g.img` est une image brute :
```bash
img2simg samsung-gtel3g.img samsung-gtel3g-sparse.img
```

### 3. Démarrer la tablette en mode Download
1. Éteignez complètement la tablette (ou maintenez `Volume Bas + Power` pendant 10 secondes).
2. Maintenez simultanément **`Volume Bas + Home + Power`** jusqu'à l'écran d'avertissement bleu/vert.
3. Appuyez une fois sur **`Volume Haut`** pour entrer en mode **Downloading...**.
4. Branchez la tablette en USB au PC.

### 4. Flasher le noyau (`BOOT`) et le système (`USERDATA`)
```bash
heimdall detect
sudo heimdall flash --BOOT boot.img --USERDATA samsung-gtel3g-sparse.img
```
La tablette redémarre automatiquement sur postmarketOS en mémoire interne.  
*(Important : retirez toute carte microSD avant le démarrage, car le noyau 3.10.17 cherche sa partition système en priorité sur la carte SD.)*

---

## Partie 2 : Installer ce pack (Interface Tactile + Icônes + Wi-Fi)

Une fois la tablette démarrée sous postmarketOS et reliée en USB au PC (IP par défaut : `172.16.42.1`) :

### 1. Envoyer et extraire l'archive sur la tablette
Depuis votre PC (remplacez `user` par votre nom d'utilisateur sur la tablette) :
```bash
scp SM-T561-Pack-Complet.zip user@172.16.42.1:~/
ssh user@172.16.42.1 "mkdir -p ~/pack-sm-t561 && unzip ~/SM-T561-Pack-Complet.zip -d ~/pack-sm-t561"
```

### 2. Configurer votre réseau Wi-Fi (2,4 GHz uniquement)
La puce Wi-Fi interne de la SM-T561 ne supporte que la bande **2,4 GHz** en sécurité **WPA2** (le 5 GHz et le WPA3 ne sont pas détectés).

Éditez le fichier modèle sur la tablette :
```bash
ssh -t user@172.16.42.1 "nano ~/pack-sm-t561/wifi/wpa_supplicant.conf.exemple"
```
Remplacez `NOM_DE_LA_BOX_2.4GHZ` et `MOT_DE_PASSE_WIFI` par les identifiants de votre box, puis sauvegardez (`Ctrl + O`, `Entrée`, `Ctrl + X`).

### 3. Lancer l'installation automatique
```bash
ssh -t user@172.16.42.1 "cd ~/pack-sm-t561 && sh install.sh"
```

Le script va automatiquement :
* Installer les applications légères compatibles (`tint2`, `pcmanfm`, `lxterminal`, `jgmenu`, `matchbox-keyboard`, `dillo`, `feh`, `dropbear`, `wpa_supplicant`).
* Désactiver le chargeur SVG défectueux et activer le thème d'icônes PNG **Faenza**.
* Appliquer le bureau tactile, les raccourcis de la barre du bas et la jauge de batterie.
* Isoler `NetworkManager`, activer la connexion Wi-Fi automatique via `/etc/local.d/wifi-direct.start` et verrouiller les DNS rapides (`1.1.1.1` / `8.8.8.8`).
* Redémarrer l'interface graphique.

--- 

## Conseils d'utilisation et limites matérielles
* **Débranchez le câble USB :** Une fois `install.sh` terminé, débranchez le câble USB. La tablette se connecte désormais toute seule à votre Wi-Fi 2,4 GHz à chaque démarrage.
* **Applications recommandées :** Privilégiez les applications légères déjà installées (**Dillo**, **PCManFM**, **LXTerminal**, **Mousepad**, **Viewnior**, **mpv**). Évitez les logiciels GTK4 ou Electron qui dépassent la capacité du noyau `3.10.17` et des 1,5 Go de RAM.
* **Connexion USB directe depuis un PC Linux :** Si vous rebranchez la tablette en USB plus tard, son adresse IP USB reste `172.16.42.1` (avec le serveur SSH léger **Dropbear** actif).




j'ai mis deux roms compatibles avec sm t561 en zip et twrp compatible sm t561 
