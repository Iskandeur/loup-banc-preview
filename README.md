# Jeu du loup : banc d'ambiance, moteur v0

Une plaine de hautes herbes au coucher du soleil, une lisière de jungle, des ruines, une mare, et un loup qu'on pilote.
On ne joue pas encore : on règle l'ambiance et la dose de voxels.

Fichier unique `index.html` (three.js 0.170 en CDN, MIT). Tout le reste est procédural : terrain, arbres, rochers, ruines,
loup, texture de feuilles, son (WebAudio). Aucun asset externe, aucune licence tierce hors three.js.

## Variantes (touches 1 / 2 / 3)
- **A, tout microvoxel** : terrain (cases de 0,5 m), arbres, rochers, ruines et loup voxelisés ; AO par sommet ; herbes en tiges cubiques.
- **B, hybride** : relief (cases de 1 m), rochers et ruines en voxels ; herbes, feuillage, loup, lumière et ciel lisses.
- **C, quasi réaliste** : tout lisse sauf les ruines.

Paramètres d'URL : `?v=A|B|C`, `?t=0..1` (heure), `?g=<nb d'herbes>`, `?q=low` (mode léger), `?raw=1` (sans post-traitement).

## Commandes
ZQSD/WASD/flèches pour marcher, Maj pour courir, glisser la souris pour tourner la caméra, molette pour la distance,
T/R pour faire avancer ou reculer l'heure, M pour couper le son, H pour masquer l'interface. Sur mobile : le pouce gauche marche, le droit tourne la caméra.

## Technique
- Rendu WebGL2. WebGPU est détecté mais pas encore utilisé (v0) : il faudra réécrire les shaders en TSL.
- Herbes : 200 000 brins instanciés (45 000 en mode léger) sur une tuile qui suit le loup. Vent par rafales, écartement autour du loup, translucidité à contre-jour.
- Voxels : un voxeliseur maison qui ne garde que les faces exposées, avec occlusion ambiante par sommet (le look « Minecraft avec shaders »).
- Rayons volumétriques en espace écran (masque de ciel tiré du tampon de profondeur, flou radial vers le soleil), brume exponentielle, ACES.
- Ombres PCF douces sur PC, désactivées en mode léger. La résolution baisse toute seule sous 48 i/s.

## L'écart avec John Lin
Son moteur est natif (C++/Vulkan) : le monde **est** une grille de voxels sur le GPU, éclairée par du lancer de rayons (GI réelle, ombres de chaque voxel, lumière qui rebondit).
Ici on fabrique des maillages à partir des voxels et on simule la GI (lumière d'hémisphère, AO par sommet, brume). Ce qui comblerait l'écart, par ordre de gain :
1. WebGPU + compute : stocker le monde en grille et raymarcher la GI (cone tracing ou radiance cascades).
2. Ombres de contact et AO en espace écran.
3. Des voxels plus fins sur le loup et la végétation, avec du LOD.
