# 📧 Microsoft Exchange Server & Sécurité de la Messagerie

## 🎯 Objectifs
- Déployer et configurer un serveur de messagerie d'entreprise **Microsoft Exchange Server 2019**.
- Administrer les différents types de boîtes aux lettres, ressources et groupes de distribution.
- Sécuriser les flux SMTP et implémenter des règles de transport (bandeau `[EXTERNE]`, filtrage des pièces jointes malveillantes).
- Maîtriser l'intégration avec **Active Directory** et la gestion des certificats/DSRM.

---

## 📋 Concepts Clés

### 1. Architecture & Intégration AD
- **Extension de Schéma AD** : L'installation d'Exchange modifie de manière permanente le schéma Active Directory (ajout d'attributs pour la messagerie et organisation unique).
- **Role unique (Mailbox Role)** : Regroupe le stockage des bases de données (`.edb`), le service de transport et l'accès client (Webmail, MAPI/EWS).
- **Autodiscover** : Permet aux clients Outlook de se configurer automatiquement en interrogeant l'annuaire ou le DNS interne.

### 2. Protocoles & Flux SMTP
- **SMTP (25 / 587)** : Utilisé exclusivement pour *pousser* et acheminer le courrier entre serveurs ou depuis un client.
- **MAPI / EWS / HTTPS (443)** : Dialogue riche entre le client (Outlook/Webmail) et le serveur pour le courrier, les agendas, les contacts et les disponibilités.
- **Connecteurs de Réception & D'Envoi** : Contrôlent quelles adresses et quelles machines peuvent déposer ou sortir du courrier (prévention du *relais ouvert*).

---

## 🛠️ Étapes de Configuration (Challenge & Lab)

### 1️⃣ Création des Boîtes & Ressources
1. **Boîtes utilisateurs** : Création de boîtes nominatives (`Camille Roux`, `Nolan Petit`).
2. **Boîte partagée (`Accueil`)** :
   - Compte AD associé **désactivé** pour la sécurité.
   - Attribution des droits *Accès complet* et *Envoyer en tant que* aux utilisateurs autorisés.
3. **Boîte de ressource (`Salle projet`)** : Configuration de la boîte de salle avec acceptation automatique des réservations selon la disponibilité.
4. **Groupe de distribution (`Support`)** : Création d'un groupe universel dans l'AD pour l'envoi de messages groupés.

---

### 2️⃣ Implémentation des Règles de Transport

#### A. Injection du bandeau [EXTERNE]
- **Condition** : `L'expéditeur est situé à l'extérieur de l'organisation`.
- **Action** : Insérer un bandeau d'avertissement HTML/texte **au début du message** (Option *Prépendre* / *Plus d'options*).
- **Secours** : Encapsuler le message si l'insertion directe échoue.

#### B. Blocage des pièces jointes à risque
- **Condition** : Le message contient une pièce jointe avec l'extension `.vbs`, `.js`, `.ps1` ou `.jar`.
- **Action** : Rejeter le message et envoyer un rapport de non-remise (NDR) avec une explication claire.

---

## 🧪 Validation & Tests

1. **Test d'envoi interne & groupe** : Validation de l'acheminement des mails vers la boîte partagée `Accueil` et le groupe `Support`.
2. **Test de réservation de salle** : Envoi d'une invitation à `Salle projet` -> Vérification de la réponse automatique d'acceptation/refus selon les créneaux.
3. **Test de la règle [EXTERNE]** : Simulation d'un mail entrant via un script PowerShell/SMTP externe -> Vérification de la présence du bandeau d'avertissement dans le Webmail (OWA).
4. **Test de blocage de script** : Envoi d'un fichier `test.vbs` -> Réception immédiate du rapport de non-remise (NDR).

---

## ⚠️ Point d'Attention & Exploitation

- **Purgement des journaux de transaction (`.log`)** : Les journaux d'Exchange ne sont supprimés qu'après la réussite d'une **sauvegarde complète compatible VSS**. Une sauvegarde en échec entraînera le remplissage complet du disque et l'arrêt du service de messagerie (`Back Pressure`).
- **Certificats SSL/TLS** : Nécessaires pour sécuriser HTTPS (OWA/EWS) et le chiffrement des flux inter-serveurs.
