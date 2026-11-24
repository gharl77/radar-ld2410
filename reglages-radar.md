# Réglages du radar LD2410C (sous-sol)

Relevés le 2026-11-23, firmware radar 2.04.23022511. Ces valeurs vivent dans
le radar, pas dans le YAML : une remise à zéro usine ou une mise à jour du
firmware du radar peut les effacer.

| Porte | Distance | Seuil mouvement | Seuil immobile |
|:-:|:-:|:-:|:-:|
| g0 | 0 – 0,75 m | 50 | 100 |
| g1 | 0,75 – 1,5 m | 50 | 60 |
| g2 | 1,5 – 2,25 m | 40 | 25 |
| g3 | 2,25 – 3 m | 30 | 25 |
| g4 | 3 – 3,75 m | 20 | 22 |
| g5 | 3,75 – 4,5 m | 15 | 18 |
| g6 | 4,5 – 5,25 m | 15 | 20 |
| g7 | 5,25 – 6 m | 15 | 20 |
| g8 | 6 – 6,75 m | 15 | 20 |

- Délai d'absence : 30 s
- Distance max mouvement : porte 8
- Distance max immobile : porte 8
- Bluetooth du radar : éteint

## Historique

- 2026-11-23 : seuils immobiles abaissés, g2 40 → 25, g3 40 → 25, g5 22 → 18,
  pour mieux garder une personne assise. Bruit de fond mesuré pièce vide
  (moyennes sur 10 s) : maximum 12 sur g2, 11 sur g3, 10 sur g5.
  g4 reste à 22 : son bruit monte à 18. Pour revenir en arrière, remettre
  les anciennes valeurs.
