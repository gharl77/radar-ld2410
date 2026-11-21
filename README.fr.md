# radar-ld2410

[English](README.md) · **Français**

Configuration ESPHome d'un capteur de présence fait d'un ESP32 et d'un radar
à ondes millimétriques HLK-LD2410C, réglée pour Home Assistant. La même carte
sert aussi de proxy Bluetooth.

La config de ce dépôt fait tourner un vrai appareil (`presence-soussol`, dans
un sous-sol). Les noms d'entités et les commentaires sont en français.

## Ce que ça fait

- Capteurs binaires de **présence, mouvement et présence immobile**, publiés
  instantanément.
- **Distances et énergies** moyennées et dédoublonnées avant l'envoi, pour ne
  pas noyer la base de Home Assistant (voir [Pourquoi les filtres](#pourquoi-les-filtres)).
- **Énergies par porte (g0 à g8)** à côté de leurs seuils, pour la
  calibration, avec une carte de tableau de bord prête à l'emploi.
- **Mode ingénierie maintenu** automatiquement, y compris après un redémarrage
  du radar.
- **Proxy Bluetooth** pour Home Assistant (framework ESP-IDF).
- **Diagnostics** : signal Wi-Fi, durée de fonctionnement, adresse IP, mémoire
  libre, cause du dernier redémarrage.
- **Aucun secret dans le YAML** : la clé API et les mots de passe OTA et du
  point d'accès de secours sont des références `!secret`.

## Matériel

| Pièce | Notes |
|---|---|
| Carte ESP32 (`esp32dev`, WROOM) | Un module WROVER utilise GPIO16/17 pour sa PSRAM. |
| HLK-LD2410C | Firmware radar 2.04 ici. UART à 256000 bauds, 8N1. |

Câblage de cet appareil :

| LD2410C | ESP32 |
|---|---|
| VCC | 5V |
| GND | GND |
| TX | GPIO3 (RX) |
| RX | GPIO1 (TX) |

GPIO1/GPIO3 est l'UART0, le port série USB. C'est pourquoi la config met
`logger: baud_rate: 0` (pas de logs série), et pourquoi un flash par câble
demande en général de débrancher le radar. **Sur un montage neuf, préférer
GPIO16 (RX) / GPIO17 (TX)** et retirer la ligne `baud_rate: 0`.

## Fichiers

| Fichier | Rôle |
|---|---|
| [`presence-soussol.yaml`](presence-soussol.yaml) | La config de l'appareil en service. |
| [`modeles/radar-ld2410.yaml`](modeles/radar-ld2410.yaml) | Modèle : la même config avec des `NOM` à remplacer. |
| [`modeles/LISEZ-MOI.md`](modeles/LISEZ-MOI.md) | Comment créer un nouveau capteur à partir du modèle. |
| [`home-assistant/carte-seuils.yaml`](home-assistant/carte-seuils.yaml) | Carte de tableau de bord : énergie par porte et seuil. |
| [`reglages-radar.md`](reglages-radar.md) | Relevé des seuils du radar (ils vivent dans le radar, pas dans le YAML). |

## Démarrage rapide

1. Copier [`modeles/radar-ld2410.yaml`](modeles/radar-ld2410.yaml) dans
   ESPHome Device Builder sous le nom `presence-<piece>.yaml`.
2. Remplacer `NOM` partout (5 endroits).
3. Créer les trois secrets, à côté de vos `wifi_ssid` / `wifi_password` :
   `presence_<piece>_api_key` (`openssl rand -base64 32`),
   `presence_<piece>_ota_password`, `presence_<piece>_ap_password`.
4. Vérifier les broches UART dans le bloc `substitutions`.
5. Premier flash par USB ; les mises à jour suivantes se font par OTA.

Détails : [`modeles/LISEZ-MOI.md`](modeles/LISEZ-MOI.md).

## Entités

Dans Home Assistant, les noms sont préfixés par le nom de l'appareil.

| Entité | Signification |
|---|---|
| Présence détectée | Toute cible. `delayed_off` de 5 s contre le clignotement. |
| Mouvement détecté / Présence immobile | Cible en mouvement / cible immobile. |
| Distance de détection, Distance (mouvement), Distance (immobile) | cm, moyenne sur 5 s, envoyée si elle change de 10 cm. |
| Énergie du mouvement / Énergie immobile | %, moyenne sur 5 s, envoyée si elle change de 3 points. |
| Luminosité | Capteur de lumière du radar, valeur brute de 0 à 255 (pas des lux). |
| gN énergie mouvement / immobile | Énergie par porte, moyenne sur 10 s, envoyée si elle change de 2 points. |
| gN seuil mouvement / immobile | Seuils par porte (stockés dans le radar). |
| Radar - Délai, Distance max mouvement / immobile | Délai d'absence (s) et dernière porte active. |
| Radar - Mode ingénierie, Radar - Bluetooth | Interrupteurs du radar. |
| Radar - Redémarrer, Relire les paramètres, Remise à zéro usine | Boutons du radar. La remise à zéro est désactivée par défaut. |
| Statut, Signal Wi-Fi, Durée de fonctionnement, Adresse IP | Santé de l'appareil. |
| Mémoire libre, Cause du dernier redémarrage | Diagnostic des redémarrages inexpliqués. |

Chaque porte couvre 0,75 m : g0 va de 0 à 0,75 m, g8 de 6 à 6,75 m.

## Calibrer les seuils

1. Ajouter [`home-assistant/carte-seuils.yaml`](home-assistant/carte-seuils.yaml)
   à un tableau de bord (Ajouter une carte → Manuel), en remplaçant
   `presence_soussol` par le nom de votre appareil.
2. Laisser la pièce vide un moment et lire l'énergie de chaque porte : c'est
   le bruit de fond.
3. Régler chaque seuil un peu au-dessus du bruit. Un 🔴 marque une porte dont
   l'énergie dépasse le seuil, donc qui déclenche la détection.
4. S'asseoir sans bouger là où les gens s'installent d'habitude et vérifier
   que la porte correspondante passe au rouge dans la colonne « Immobile ».
   Sinon, baisser ce seuil immobile.

## Leçons apprises

### Le mode ingénierie ne tient pas

Le capteur de lumière et les énergies par porte ne publient qu'en mode
ingénierie, et le radar quitte ce mode à chaque redémarrage : à la mise sous
tension, après son bouton de redémarrage et après un changement de son
Bluetooth. `restore_mode: ALWAYS_ON` sur l'interrupteur n'a eu aucun effet
ici. Ce qui fonctionne : l'activer 10 s après le démarrage, puis revérifier
toutes les 60 s avec un `interval`.

### Pourquoi les filtres

Le radar envoie environ 10 mesures par seconde. Sans filtre, un seul capteur
d'énergie produisait environ 32 000 changements d'état par jour dans une pièce
vide. Avec `throttle_average`, `round` et `delta`, le même capteur en produit
environ 1 400. Les capteurs binaires ne sont pas filtrés : la détection reste
instantanée.

### Avertissements d'unités dans Home Assistant

Le composant LD2410 donne à ses seuils la classe `signal_strength` avec
l'unité `%`, aux distances max la classe `distance` sans unité, et au capteur
de lumière la classe `illuminance` sans unité. Home Assistant journalise un
avertissement pour chacun, et annonce que la première combinaison cessera de
fonctionner en 2027.5. La config les neutralise avec `device_class: ""`.

### Bluetooth du radar

Le Bluetooth propre au radar ne sert qu'à l'app du fabricant (HLKRadarTool),
par exemple pour mettre à jour le firmware du radar. Le laisser éteint le
reste du temps. Une mise à jour du firmware du radar ou une remise à zéro
usine peut effacer les seuils, d'où [`reglages-radar.md`](reglages-radar.md).

## Testé avec

ESPHome 2026.10.2 (ESP-IDF) et firmware LD2410C
2.04.23022511.
