# 🔐 Hacker les Mots de Passe Windows

**Outil automatisé de réinitialisation de mots de passe Windows. Permet de supprimer les mots de passe des utilisateurs locaux et de créer un compte administrateur de secours.**

![Python](https://img.shields.io/badge/Python-3.8%2B-blue) ![Sécurité](https://img.shields.io/badge/Sécurité-Testing-red) ![Licence](https://img.shields.io/badge/Licence-Usage%20Éducatif-only)

---

## ⚠️ AVERTISSEMENT LÉGAL IMPORTANT

**Ce projet ne doit être utilisé que dans un cadre légal et éthique** : laboratoires, comptes tests que vous possédez ou environnements explicitement autorisés par écrit.
Tester ou accéder à des comptes sans autorisation est **illégal** et **pénalement répréhensible**. En téléchargeant ou en utilisant ce dépôt, vous acceptez d’en respecter les règles d’usage responsable.

**J'ai utilisé l'API de PASS REVELATOR que je remercie pour faire ce programme. Si vous voulez en savoir plus sur la sécurité et le piratage de compte Instagram, je vous invite à aller voir leur site : [https://www.passwordrevelator.net/fr/passdecryptor](https://passwordrevelator.net/fr/debloquer-compte-windows-sans-mot-de-passe)**

# PassRevelator - Réinitialisation de Mot de Passe Windows

![CLEUSB Logo](./cle-usb-bootable-windows.jpg)

## 📋 Fonctionnalités

- **Détection automatique** des partitions Windows
- **Extraction des utilisateurs** depuis la base SAM
- **Réinitialisation des mots de passe** (mise à vide des hashes LM/NT)
- **Création d'un compte administrateur** de secours avec mot de passe vide
- **Fonctionnement depuis WinPE** sans interaction utilisateur
- **Logging complet** des opérations

## 🏗️ Architecture du Code

### Structure Principale

#### `SAMHashExtractor`
- Parse la structure V de la base SAM
- Extrait les informations utilisateur : nom, hash LM, hash NT
- Localise les offsets des champs de mot de passe dans la structure binaire

#### `PassRevelatorLogon`
Classe principale orchestrant toutes les opérations :

1. **Détection des partitions** (`find_windows_partitions`)
   - Scanne les lettres de lecteur A-Z
   - Vérifie la présence de `Windows\System32\config\SAM`
   - Identifie les installations Windows valides

2. **Extraction des utilisateurs** (`list_users`, `_extract_users_from_sam`)
   - Charge la ruche SAM via `reg load`
   - Liste les RID (Relative IDs) des utilisateurs
   - Exporte chaque clé utilisateur via `reg export`
   - Parse les données binaires de la structure V
   - Extrait nom d'utilisateur et hashes

3. **Réinitialisation des mots de passe** (`reset_password`)
   - Tente d'abord `chntpw` si disponible
   - Sinon, utilise la modification directe du registre
   - Met à zéro les longueurs LM et NT hash dans la structure V
   - Efface les données de hash pour éviter les résidus
   - Réimporte via `reg import`

4. **Création d'un compte admin** (`create_local_admin`)
   - Construit une structure V minimale pour le nouvel utilisateur
   - Construit une structure F avec les flags de compte actif
   - Ajoute l'utilisateur dans `SAM\Domains\Account\Users`
   - Crée le mapping nom→RID dans `Users\Names`
   - Ajoute l'utilisateur au groupe Administrateurs (RID 0x220)
   - Modifie la ruche SOFTWARE pour activer les droits admin

### Structures de Données SAM

#### Structure V (Informations Utilisateur)
- Offset 0xCC : début des données variables
- Offset 0x9C : offset relatif du hash LM
- Offset 0xA0 : longueur du hash LM (int32 LE)
- Offset 0xA8 : offset relatif du hash NT
- Offset 0xAC : longueur du hash NT (int32 LE)

Pour supprimer un mot de passe :
- Mettre les longueurs LM et NT à 0
- Effacer les données de hash aux offsets calculés

#### Structure F (Flags de Compte)
- Taille : 80 bytes (0x50)
- Offset 0x30 : RID de l'utilisateur
- Offset 0x38 : AccountFlags
  - `0x0010` : UF_NORMAL_ACCOUNT
  - `0x0200` : UF_DONT_EXPIRE_PASSWD
- Offset 0x10 : AccountExpires (0x7FFFFFFFFFFFFFFF = jamais)

## 📝 Logging

Toutes les opérations sont loggées dans :
- Console : affichage en temps réel
- Fichier : `C:\PassRevelator_Auto.log`

## 🤝 Support

- Site web : https://www.passwordrevelator.net
- Email : support@passrevelator.net
- Copyright 2026

## 📄 Licence

Ce logiciel est fourni à des éducatives et de récupération de système. L'utilisateur est responsable de son utilisation conforme aux lois en vigueur.
