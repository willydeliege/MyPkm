---
id: configure-nixos-gitable-config
aliases:
  - Configure nixos gitable config
tags:
  - nixos
---

# Configure nixos gitable config

Félicitations pour votre installation de NixOS ! Pour placer votre configuration
sur Git tout en conservant un système fonctionnel et facile à mettre à jour, le
plus simple est de créer un dépôt Git dans votre dossier personnel (par exemple
~/dotfiles) et d'utiliser des liens symboliques. Cela vous évite d'avoir à
manipuler les fichiers en sudo à chaque modification. [1, 2, 3] · 1970 M01 1

## 1. Préparer votre dépôt local

Pour commencer, déplacez vos fichiers de configuration actuels vers un dossier
de votre choix dans votre espace utilisateur.

- Ouvrez votre terminal.
- Créez un dossier pour vos configurations, par exemple mkdir ~/dotfiles.
- Copiez le fichier principal actuel : sudo cp /etc/nixos/configuration.nix
  ~/dotfiles/.
- Copiez également le fichier matériel : sudo cp
  /etc/nixos/hardware-configuration.nix ~/dotfiles/.
- Changez les droits de ces fichiers pour que votre utilisateur puisse les
  modifier sans les privilèges administrateur : sudo chown $USER:users
  ~/dotfiles/\*.nix. [2, 4, 5, 6, 7]

## 2. Lier les fichiers au système

Maintenant que les fichiers sont dans votre dossier Git, il faut dire à NixOS où
aller les chercher. [8]

- Supprimez les fichiers originaux : `sudo rm /etc/nixos/\*.nix`.
- Créez un lien symbolique vers le fichier principal :
  `sudo ln -s ~/dotfiles/configuration.nix /etc/nixos/configuration.nix`.
  - Créez un lien symbolique vers le fichier matériel :
    `sudo ln -s ~/dotfiles/hardware-configuration.nix /etc/nixos/hardware-configuration.nix`.
    - Testez que tout fonctionne bien en effectuant une reconstruction :
      `sudo nixos-rebuild switch`. [1, 2, 9, 10]

## 3. Activer Git et publier sur le dépôt

Il ne vous reste plus qu'à versionner ces fichiers.

- Rendez-vous dans votre dossier : cd ~/dotfiles.
- Initialisez le dépôt : git init.
- Ajoutez les fichiers : git add configuration.nix hardware-configuration.nix.
- Validez la configuration : git commit -m "Première configuration NixOS".
- Liez et poussez le tout vers un dépôt distant (GitHub, GitLab, etc.) avec la
  commande git remote add origin <URL_DE_VOTRE_DEPOT> et git push -u origin
  main. [6, 11, 12, 13, 14]

Si vous avez déjà installé [Home Manager](https://nixos.wiki/wiki/Home_Manager)
ou si vous utilisez la fonctionnalité des Flakes (expérimentale mais devenue un
standard pour les nouvelles installations), la structure des fichiers sera
légèrement différente.
