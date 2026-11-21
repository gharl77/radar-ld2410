# Modèle : radar de présence LD2410C

[English](README.md) · **Français**

`radar-ld2410.yaml` est une copie de `presence-soussol.yaml` où tout ce qui est
propre à l'appareil est remplacé par `NOM`. Le reste (filtres, mode ingénierie
maintenu, proxy Bluetooth, diagnostics) est identique.

## Créer un nouveau radar

1. Copier `radar-ld2410.yaml` dans ESPHome Device Builder sous le nom
   `presence-<piece>.yaml`.
2. Remplacer `NOM` partout (5 endroits) : dans le bloc `substitutions` et dans
   les trois noms de secrets.
3. Créer les trois secrets dans ESPHome (Secrets, en haut à droite) :
   - `presence_<piece>_api_key` : clé de 32 octets en base64
     (`openssl rand -base64 32`)
   - `presence_<piece>_ota_password`
   - `presence_<piece>_ap_password` (8 caractères minimum)
4. Vérifier les broches. `GPIO1/GPIO3` est le port série USB : le premier
   flash par câble doit se faire radar débranché. Sur un montage neuf,
   préférer `GPIO17` (TX) et `GPIO16` (RX), et remettre alors
   `logger: baud_rate` à sa valeur par défaut.
5. Premier flash par USB, ensuite tout se fait par OTA.

## Pourquoi un fichier complet plutôt qu'un « package »

Les noms de secrets (`!secret ...`) ne peuvent pas venir d'une substitution :
un package partagé demanderait quand même des retouches par appareil. Un
fichier autonome reste donc le plus simple à maintenir.

## Garder le modèle à jour

Après un changement dans `presence-soussol.yaml`, régénérer le modèle :
remplacer `presence-soussol` / `Presence-Soussol` / `presence_soussol_` par
leurs équivalents `NOM`.
