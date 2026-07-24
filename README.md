# 🛡️ Lightweight Linux System Health & Process Monitor (English Version)

A lightweight, automated Bash-based system monitoring tool designed to track system resource usage, issue real-time threshold alerts, and manage log rotation automatically using the Linux `cron` daemon.

---

## 🎯 Overview

In cloud environments and Linux servers, unmonitored memory leaks or unexpected process spikes can lead to server crashes and downtime , also if a hacker hijacks your linux machine/server running on cloud he might run scripts on your machine maybe to mine crypto in that case your CPU usage will shoot up . This project provides an automated background monitoring solution that periodically tracks memory metrics, appends timestamped records, triggers warning alerts when usage exceeds defined safety limits, and automatically prevents disk space exhaustion via log rotation.

---

## ✨ Features

* ⏱️ **Automated Execution:** Runs background checks at custom intervals using `cron`.
* 📊 **Resource Tracking:** Captures memory usage percentages alongside precise timestamps.
* ⚠️ **Threshold Alerting:** Triggers explicit warning flags when memory usage passes **80%**.
* 🧹 **Automated Log Rotation:** Automatically truncates logs once they exceed 100 lines to protect server disk space.

---

## 📐 Execution Flow

```text
┌──────────────┐     Executes every 2 min     ┌────────────────────────┐
│  Cron Daemon │ ───────────────────────────> │  log_processes.sh      │
└──────────────┘                              └───────────┬────────────┘
                                                          │
                                     ┌────────────────────┴────────────────────┐
                                     ▼                                         ▼
                         ┌───────────────────────┐                 ┌───────────────────────┐
                         │ 1. Check Log Size     │                 │ 2. Check Memory %     │
                         │    (Rotate if >100)   │                 │    (via `free` + `awk`)│
                         └───────────┬───────────┘                 └───────────┬───────────┘
                                     │                                         │
                                     └────────────────────┬────────────────────┘
                                                          ▼
                                            ┌───────────────────────────┐
                                            │ Write to `process_log.txt`│
                                            └───────────────────────────┘`


```
Create the file directory :
```
mkdir security_monitor
```
then :- touch log_processes.sh  

then :- nano log_processes.sh  

paste the script 

🛠️ Script Overview (log_processes.sh)
```
#!/bin/bash

LOG_FILE="process_log.txt"
MAX_LINES=100
TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

# Create the log file if it doesn't exist
touch "$LOG_FILE"

# 1. Check Log Size (Log Rotation)
if [ $(wc -l < "$LOG_FILE") -gt $MAX_LINES ]; then
    echo "[$TIMESTAMP] 🧹 Log limit reached ($MAX_LINES lines). Truncating log." > "$LOG_FILE"
fi

# 2. Capture Memory Metrics
MEM_USAGE=$(free | awk '/Mem:/ {printf("%.0f"), $3/$2 * 100}')

# 3. Log Metrics
echo "[$TIMESTAMP] Memory Usage: ${MEM_USAGE}%" >> "$LOG_FILE"

# 4. Threshold Alert
if [ "$MEM_USAGE" -gt 80 ]; then
    echo "[$TIMESTAMP] ⚠️ WARNING: High memory usage at ${MEM_USAGE}%!" >> "$LOG_FILE"
fi
```

Configure Cron Automation
``` 
crontab -e
```
Add the following entry to execute the script every 2 minutes:
```
*/2 * * * * /home/YOUR_USERNAME/security_monitor/log_processes.sh
```
<img width="860" height="664" alt="Screenshot from 2026-07-24 16-24-14" src="https://github.com/user-attachments/assets/fe91073a-ed87-4595-980b-80d104194a60" />

📜 Sample Log Output (process_log.txt)
```
[2026-07-24 15:59:49] Memory Usage: 66%
[2026-07-24 16:00:36] Memory Usage: 58%
[2026-07-24 16:02:00] Memory Usage: 84%
[2026-07-24 16:02:00] ⚠️ WARNING: High memory usage at 84%!
[2026-07-24 16:04:00] 🧹 Log limit reached (100 lines). Truncating log.

```
# 🛡️ Moniteur Léger de Santé Système et Processus Linux (version francaise)

Un outil léger et automatisé de surveillance système basé sur Bash, conçu pour suivre l'utilisation des ressources système, émettre des alertes de seuil en temps réel et gérer automatiquement la rotation des journaux à l'aide du démon `cron` de Linux.

---

## 🎯 Vue d'ensemble

Dans les environnements cloud et les serveurs Linux, les fuites de mémoire non surveillées ou les pics inattendus de processus peuvent entraîner des pannes de serveur et des interruptions de service. De plus, si un pirate informatique prend le contrôle de votre machine/serveur Linux s'exécutant sur le cloud, il peut exécuter des scripts sur votre machine, par exemple pour miner de la cryptomonnaie ; dans ce cas, votre utilisation du processeur (CPU) va monter en flèche. Ce projet fournit une solution de surveillance automatisée en arrière-plan qui suit périodiquement les métriques de mémoire, ajoute des enregistrements horodatés, déclenche des alertes d'avertissement lorsque l'utilisation dépasse les limites de sécurité définies, et prévient automatiquement l'épuisement de l'espace disque grâce à la rotation des journaux.

---

## ✨ Fonctionnalités

* ⏱️ **Exécution automatisée :** Exécute des vérifications en arrière-plan à des intervalles personnalisés à l'aide de `cron`.
* 📊 **Suivi des ressources :** Capture les pourcentages d'utilisation de la mémoire avec des horodatages précis.
* ⚠️ **Alerte de seuil :** Déclenche des signaux d'avertissement explicites lorsque l'utilisation de la mémoire dépasse **80 %**.
* 🧹 **Rotation automatisée des journaux :** Tronque automatiquement les journaux dès qu'ils dépassent 100 lignes pour protéger l'espace disque du serveur.

---

## 📐 Flux d'exécution

```text
┌──────────────┐     Exécute toutes les 2 min     ┌────────────────────────┐
│  Démon Cron  │ ───────────────────────────────> │  log_processes.sh      │
└──────────────┘                                  └───────────┬────────────┘
                                                              │
                                         ┌────────────────────┴────────────────────┐
                                         ▼                                         ▼
                             ┌───────────────────────┐                 ┌───────────────────────┐
                             │ 1. Vérifier taille    │                 │ 2. Vérifier % mémoire │
                             │    (Rotation si >100) │                 │    (via free + awk)   │
                             └───────────┬───────────┘                 └───────────┬───────────┘
                                         │                                         │
                                         └────────────────────┬────────────────────┘
                                                              ▼
                                                ┌───────────────────────────┐
                                                │ Écrire dans              │
                                                │ `process_log.txt`         │
                                                └───────────────────────────┘
```
Créer le répertoire du projet :

Bash
```
mkdir security_monitor
```
Puis :

Bash
touch log_processes.sh  
Puis :

Bash
nano log_processes.sh  

Coller le script.

🛠️ Aperçu du script (log_processes.sh)
```
#!/bin/bash

LOG_FILE="process_log.txt"
MAX_LINES=100
TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

# Créer le fichier journal s'il n'existe pas
touch "$LOG_FILE"

# 1. Vérifier la taille du journal (Rotation des journaux)
if [ $(wc -l < "$LOG_FILE") -gt $MAX_LINES ]; then
    echo "[$TIMESTAMP] 🧹 Log limit reached ($MAX_LINES lines). Truncating log." > "$LOG_FILE"
fi

# 2. Capturer les métriques de mémoire
MEM_USAGE=$(free | awk '/Mem:/ {printf("%.0f"), $3/$2 * 100}')

# 3. Enregistrer les métriques
echo "[$TIMESTAMP] Memory Usage: ${MEM_USAGE}%" >> "$LOG_FILE"

# 4. Alerte de seuil
if [ "$MEM_USAGE" -gt 80 ]; then
    echo "[$TIMESTAMP] ⚠️ WARNING: High memory usage at ${MEM_USAGE}%!" >> "$LOG_FILE"
fi
```
Configurer l'automatisation Cron

```
crontab -e
```
Ajoutez l'entrée suivante pour exécuter le script toutes les 2 minutes :

```
*/2 * * * * /home/VOTRE_NOM_D_UTILISATEUR/security_monitor/log_processes.sh
```
📜 Exemple de sortie du journal (process_log.txt)
```
[2026-07-24 15:59:49] Memory Usage: 66%
[2026-07-24 16:00:36] Memory Usage: 58%
[2026-07-24 16:02:00] Memory Usage: 84%
[2026-07-24 16:02:00] ⚠️ WARNING: High memory usage at 84%!
[2026-07-24 16:04:00] 🧹 Log limit reached (100 lines). Truncating log.
```
