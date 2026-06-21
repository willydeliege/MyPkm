---
areas:
  - "[[linux]]"
aliases: []
id: Power management
tags: []
---

# Power management

- **udev** → réagit immédiatement au branchement/débranchement du chargeur.
- **systemd timer** → vérifie toutes les minutes si la batterie est tombée sous
  15%, même sans changement d’AC.

---

## 1. Script unique

Toujours `/usr/local/bin/power-profile-switch.sh` :

```bash
#!/bin/bash

# Récupérer l'état du chargeur
AC_STATE=$(cat /sys/class/power_supply/AC/online 2>/dev/null)
if [ -z "$AC_STATE" ]; then
  AC_STATE=$(cat /sys/class/power_supply/ACAD/online 2>/dev/null)
fi

# Récupérer le pourcentage de batterie
BATTERY=$(cat /sys/class/power_supply/BAT0/capacity 2>/dev/null)

PROFILE=""

# Appliquer profil power-profiles-daemon
if [ "$AC_STATE" = "1" ]; then
    PROFILE="performance"
    powerprofilesctl set performance
else
    if [ "$BATTERY" -lt 15 ]; then
        PROFILE="power-saver"
        powerprofilesctl set power-saver
    else
        PROFILE="balanced"
        powerprofilesctl set balanced
    fi
fi

# Notification profil
notify-send -u low -i battery "Power Profile" "Profil activé : $PROFILE"

# Si on vient d'un évènement AC (udev), gérer la luminosité
if [ "$POWER_EVENT" = "AC_CHANGE" ]; then
    if [ "$AC_STATE" = "1" ]; then
        brightnessctl set 80%
        notify-send -u low -i display-brightness "Luminosité" "Réglée à 80% (chargeur branché)"
    else
        brightnessctl set 30%
        notify-send -u low -i display-brightness "Luminosité" "Réglée à 30% (sur batterie)"
    fi
fi

```

➡️ Ce script sera utilisé **par udev ET par systemd**.

---

## 2. Règle **udev**

Fichier `/etc/udev/rules.d/99-power-profile.rules` :

```udev
SUBSYSTEM=="power_supply", ACTION=="change", RUN+="/usr/local/bin/power-profile-switch.sh"
```

Recharge :

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger -s power_supply
```

---

## 3. Service systemd (pour le timer)

Fichier `/etc/systemd/system/power-profile.service` :

```ini
[Unit]
Description=Auto switch power profile based on AC and battery state

[Service]
Type=oneshot
ExecStart=/usr/local/bin/power-profile-switch.sh
```

---

## 4. Timer systemd

Fichier `/etc/systemd/system/power-profile.timer` :

```ini
[Unit]
Description=Run power profile switch periodically

[Timer]
OnBootSec=1min
OnUnitActiveSec=1min

[Install]
WantedBy=timers.target
```

Active le timer :

```bash
sudo systemctl daemon-reexec
sudo systemctl enable --now power-profile.timer
```

---

## 5. Résultat ✅

- **Branchement/débranchement du chargeur** → changement instantané grâce à
  **udev**.
- **Niveau de batterie qui passe sous 15%** → vérification toutes les minutes
  grâce à **systemd timer**.

---

Veux-tu que je t’ajoute une **notification de bureau** (par `notify-send`) pour
voir quel profil est appliqué, à chaque changement ? i
