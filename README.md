# prestashop-skimmer-forensics

> Rapport d'investigation technique, scripts de détection et procédures d'assainissement suite à la neutralisation d'un web skimmer JS sur PrestaShop.

---

## 📌 Présentation de l'incident

Ce dépôt documente la gestion forensique et technique d'une cyberattaque par **web skimmer JavaScript** subie le 18 septembre. 

L'attaque visait à intercepter furtivement les coordonnées bancaires saisies dans le navigateur des clients via un formulaire falsifié superposé à la page de paiement. Le code agissant au vol, aucune donnée bancaire n'a été conservée ni modifiée au niveau de la base de données SQL.

### 🔍 Éléments clés de l'attaque
* **Type de menace :** Web Skimmer / JS Injection / Exfiltration WebRTC (`RTCPeerConnection`).
* **Technique de camouflage :** *Timestomping* (falsification de la date de modification du fichier à 2023 pour tromper les scanners basés sur les dates récentes).
* **Vecteur d'infection :** Fichier PHP/JS altéré au cœur du thème PrestaShop.
* **Exfiltration :** Envoi direct vers 4 adresses IP distantes (2 situées en France, 2 en Allemagne).

---

## 🛠️ Scripts & Outils d'Investigation

### 1. `scan.php` — Script de détection comportementale
Permet de balayer l'ensemble du répertoire PrestaShop (`/themes/`, `/modules/`, `/controllers/`) sans se fier aux horodatages système, en recherchant les signatures spécifiques du pirate :
* `__prceCk`
* `RTCPeerConnection`
* `createDataChannel`
* `eval(function(...)`

### 2. `flush.php` — Réinitialisation d'OPcache
Permet d'effacer la mémoire RAM du serveur PHP afin d'expulser les versions compilées des fichiers malveillants immédiatement après leur suppression sur le disque.

### 3. Commandes d'investigation SSH
```bash
# Recherche de la signature du pirate dans l'arborescence
grep -rnw "__prceCk" ~/www/

# Détection des appels WebRTC suspects dans les thèmes et modules
grep -rnw "RTCPeerConnection" ~/www/themes/ ~/www/modules/

# Identification des fichiers modifiés récemment (hors cache)
find ~/www/ \( -name "*.php" -o -name "*.js" -o -name "*.tpl" \) -not -path "*/var/cache/*" -mtime -5



## 🛡️ Procédure d'Assainissement & Sécurisation

* **Nettoyage du code source :** Extraction du bloc malveillant dans le fichier de thème identifié.
* **Purge de la mémoire vive & des caches :** Exécution de `flush.php` et vidage manuel du dossier `var/cache/`.
* **Rotation complète des secrets :**
  * Renouvellement des accès FTP, SSH et MySQL.
  * Régénération des clés de sécurité (`cookie_key` dans `config/parameters.php`).
  * Modification de tous les mots de passe administrateurs PrestaShop.
* **Audits des accès :** Vérification de la table des employés (`ps_employee`) et analyse des logs de connexion.

---

## ⚖️ Conformité, CNIL & Volet Légal

* **CNIL (RGPD Article 33) :** Notification de fuite de données transmise dans le délai légal de 72 heures.
* **Dépôt de plainte :** Transmission aux services de gendarmerie/police d'un dossier technique complet (horodatages falsifiés, chemins de fichiers et les 4 adresses IP des serveurs de destination pour réquisitions judiciaires).
* **Prévention clients :** Information ciblée auprès des clients ayant initié une commande durant la fenêtre de compromission du 18 septembre.

---

## 🛒 À propos du projet

Ce travail de sécurisation et de documentation a été réalisé pour maintenir le plus haut niveau d'exigence et de protection sur notre boutique en ligne.

Retrouvez nos produits d'exception et notre sélection d'épices directement sur **[Le Comptoir de Toamasina](https://lecomptoirdetoamasina.fr/fr/)**, spécialiste de la vanille, des baies rares et du **[poivre de Madagascar](https://lecomptoirdetoamasina.fr/fr/33-poivre-de-madagascar)**.
