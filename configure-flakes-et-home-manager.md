---
id: configure-flakes-et-home-manager
aliases:
  - configure flakes et home-manager
tags:
  - nixos
---

# Configure flakes et home-manager

Parfait, nous allons injecter votre utilisateur willefi et brancher le système
sur le canal de développement unstable. Comme vous êtes sur la version unstable,
un fichier flake.lock va être généré automatiquement. Il verrouille les versions
exactes de vos paquets. Pensez à l'ajouter à Git (git add flake.lock) pour
garantir la reproductibilité absolue de votre système. Voici les fichiers finaux
prêts à l'emploi.

## 1. Votre flake.nix final

Remplacez le contenu de votre fichier ~/dotfiles/flake.nix par celui-ci :

```nix
{ description = "Configuration NixOS de willefi avec Flakes et Home Manager
(Unstable)";

inputs = { # Branchement sur le canal unstable nixpkgs.url =
"github:nixos/nixpkgs/nixos-unstable";

    home-manager = {
      url = "github:nix-community/home-manager/master"; # master suit la branche unstable
      inputs.nixpkgs.follows = "nixpkgs";
    };

};

outputs = { self, nixpkgs, home-manager, ... }@inputs: {
# nixosConfigurations.${hostname}
nixosConfigurations.nixos = nixpkgs.lib.nixosSystem { system = "x86_64-linux";
modules = [ ./hardware-configuration.nix ./configuration.nix

        home-manager.nixosModules.home-manager
        {
          home-manager.useGlobalPkgs = true;
          home-manager.useUserPackages = true;
          home-manager.users.willefi = import ./home.nix;
        }
      ];
    };

};
}
```

## 2. Votre home.nix final

Créez ou remplacez le fichier ~/dotfiles/home.nix par celui-ci :

```nix
{ config, pkgs, ... }:

{ home.username = "willefi"; home.homeDirectory = "/home/willefi";

# On utilise l'état "unstable" pour Home Manager

home.stateVersion = "26.05";

# Paquets de l'utilisateur willefi (exemples à modifier selon vos besoins)

home.packages = with pkgs; [ firefox htop fastfetch ];

# Votre configuration Git est maintenant gérée directement ici !

programs.git = { enable = true; userName = "willefi"; userEmail =
"votre.email@exemple.com"; # À personnaliser };

programs.home-manager.enable = true; }

```

## 3. Activer et pousser sur Git

Exécutez cette suite de commandes dans votre terminal pour appliquer et
sauvegarder le tout :

```bash
cd ~/dotfiles

# 1. On force Git à suivre TOUS les fichiers (indispensable pour les Flakes)

git add flake.nix home.nix configuration.nix hardware-configuration.nix

# 2. On reconstruit le système

sudo nixos-rebuild switch --flake .#nixos

# 3. Le lockfile a été créé par la commande précédente, on l'ajoute à Git

git add flake.lock

# 4. On commit et on envoie sur votre dépôt distant

git commit -m "Migration réussie vers Flakes + Home Manager pour willefi en
unstable"
git push
```
