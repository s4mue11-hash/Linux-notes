# Diagnostic et correction audio — Lenovo Yoga Slim 7 14AGP11

**Catégorie :** Linux / Dépannage matériel  
**Machine :** Lenovo Yoga Slim 7 14AGP11  
**Système :** Ubuntu 26.04  
**Noyau testé :** `7.0.0-43-generic`  
**Contrôleur audio :** AMD Ryzen HD Audio Controller  
**Codec :** Realtek ALC287  
**Statut :** Résolu

---

## 1. Problème rencontré

Après l'installation d'Ubuntu en dual boot avec Windows 11, le son des haut-parleurs intégrés était nettement moins bon sous Ubuntu que sous Windows.

Symptômes constatés :

- Son étouffé et métallique.
- Manque de profondeur et de qualité sonore.
- Restitution moins bonne que sous Windows.
- Fonctionnement audio présent, mais qualité anormale.

L'objectif était de trouver une correction directement au niveau de la configuration audio Linux, sans installer de logiciel audio superflu.

## 2. Diagnostic

### Étape 1 — Vérification du serveur audio

Commande :

```bash
wpctl status
```

Résultat : PipeWire et WirePlumber étaient actifs. Le périphérique de sortie correspondant aux haut-parleurs était détecté.

Vérification complémentaire :

```bash
pactl info
```

Constats :
- Serveur audio : PipeWire.
- Compatibilité PulseAudio fonctionnelle.
- Fréquence : 48 kHz.
- Sortie stéréo.

### Étape 2 — Identification du matériel

Commande :

```bash
lspci | grep -i audio
```

Le contrôleur audio AMD était détecté.

Le codec utilisé par la machine a ensuite été identifié comme étant un **Realtek ALC287**.

Commande :

```bash
grep -iE 'Codec:|Pin Default|Speaker|EAPD|Amp-Out' /proc/asound/card1/codec#0
```

Le codec ALC287 et la sortie haut-parleur interne étaient bien présents.

### Étape 3 — Vérification des niveaux audio

Commandes :

```bash
pactl list sinks
```

```bash
alsamixer
```

Constats :
- Sortie audio non muette.
- Volume proche de 100 %.
- Configuration stéréo.
- Aucun niveau anormalement bas identifié dans les contrôles accessibles.

Le problème ne semblait donc pas provenir d'un simple réglage de volume.

### Étape 4 — Analyse des messages du noyau

Commande :

```bash
sudo dmesg | grep -i alc287
```

Extrait observé :

```text
snd_hda_codec_alc269 hdaudioC1D0: ALC287: picked fixup for PCI SSID 17aa:0000
autoconfig for ALC287: line_outs=1 (0x14/0x0/0x0/0x0/0x0) type:speaker
```

Ces informations ont orienté le diagnostic vers la configuration spécifique du codec et du routage des haut-parleurs.

## 3. Solution appliquée

Une correction spécifique au codec ALC287 a été utilisée :

`alc287-yoga9-bass-spk-pin`

Cette configuration permet d'appliquer un correctif de routage des broches du haut-parleur adapté à certains modèles Lenovo Yoga.

### Étape 1 — Création du fichier de configuration

Commande :

```bash
sudo nano /etc/modprobe.d/alsa-model.conf
```

Contenu ajouté :

```bash
options snd-hda-intel model=,alc287-yoga9-bass-spk-pin
```

**Attention :** la virgule initiale est volontaire.

Elle permet de laisser le premier emplacement de modèle vide et d'appliquer le modèle au contrôleur audio suivant, correspondant ici au codec Realtek.

### Étape 2 — Redémarrage

La machine a été redémarrée afin de charger la nouvelle configuration du module audio.

```bash
sudo reboot
```

## 4. Résultat obtenu

Après redémarrage :

- Les haut-parleurs intégrés fonctionnaient correctement.
- Le son était nettement moins étouffé.
- La restitution sonore était beaucoup plus satisfaisante.
- L'amélioration était immédiatement perceptible à l'écoute.

**Résultat : problème résolu.**

La correction a donc permis d'améliorer significativement la restitution audio sous Ubuntu.

## 5. Compétences mobilisées

Cette intervention a permis de travailler plusieurs notions :

| Domaine | Compétence |
|---|---|
| Linux | Gestion des fichiers système |
| Terminal | Utilisation des commandes de diagnostic |
| Audio Linux | Compréhension de PipeWire et ALSA |
| Matériel | Identification d'un codec audio |
| Kernel | Configuration d'un module avec modprobe |
| Troubleshooting | Diagnostic progressif et résolution d'un problème matériel |

## 6. Points à surveiller

- Vérifier le fonctionnement audio après une mise à jour importante du noyau.
- Conserver une copie du fichier `/etc/modprobe.d/alsa-model.conf`.
- Si un problème réapparaît, vérifier que le paramètre est toujours appliqué.
- Ne pas supposer que ce correctif est compatible avec tous les ordinateurs équipés d'un ALC287.

Pour vérifier la configuration :

```bash
cat /etc/modprobe.d/alsa-model.conf
```

Pour consulter les messages audio du noyau :

```bash
sudo dmesg | grep -iE 'snd|hda|alc287'
```

---

**Conclusion :** ce dépannage illustre une démarche de diagnostic Linux : identifier le matériel, examiner les services et les pilotes, analyser les journaux système, appliquer une configuration ciblée et vérifier le résultat.

