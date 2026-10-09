# Hand Logger

Logger de mains de poker live (cash et tournoi), pensé pour le téléphone et pour aller vite à la table.
Tout tient dans `index.html` ; les autres fichiers servent à l'installer comme une app et à le faire marcher hors ligne.

## Fichiers

- `index.html` — l'app complète
- `manifest.webmanifest`, `sw.js`, `icon-*.png` — installation sur l'écran d'accueil + hors ligne
- `SCHEMA.md` — le format JSON des logs

## Installer sur le téléphone

Le mode hors ligne et l'installation ne marchent que si la page est servie en **https**.
Le plus simple, gratuit : GitHub Pages.

1. Crée un repo (ex. `hand-logger`) et pousse ces fichiers à la racine.
2. Sur GitHub : Settings → Pages → Source : `main` / root.
3. Ouvre `https://<ton-user>.github.io/hand-logger/` sur ton téléphone :
   - iPhone (Safari) : Partager → « Sur l'écran d'accueil »
   - Android (Chrome) : menu → « Installer l'application »
4. Ouvre-la une fois avec du réseau ; ensuite elle marche sans.

Tester en local sur l'ordi : `python3 -m http.server 8000` dans le dossier, puis `http://localhost:8000`.

## Sauvegardes

Les mains sont stockées dans le navigateur du téléphone (IndexedDB). C'est fiable pour une app installée,
mais ce n'est **pas** une sauvegarde : désinstaller l'app ou vider les données du navigateur efface tout.
**Exporte en JSON après chaque session** (accueil → Exporter JSON, ou bouton Exporter dans la session).
L'accueil affiche un rappel quand des mains n'ont pas été exportées depuis 2 jours.

## Mise à jour

Remplace les fichiers sur GitHub. L'app récupère la nouvelle version au prochain lancement avec réseau.
Si une modif ne s'affiche pas, ferme complètement l'app et rouvre-la.
