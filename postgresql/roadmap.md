# 🐘 PostgreSQL + DBeaver sur Ubuntu — Guide Complet

> Guide de référence pour créer et gérer une base de données PostgreSQL locale avec DBeaver sur Ubuntu.

---

## 📋 Prérequis

- Ubuntu 24.x
- DBeaver installé
- Terminal avec accès `sudo`

---

## 1️⃣ Installation de PostgreSQL

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```

### Démarrer et activer le service

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql   # démarrage automatique au boot
```

### Vérifier que PostgreSQL tourne

```bash
sudo systemctl status postgresql
```

> ✅ Tu dois voir `active (running)`

---

## 2️⃣ Vérifier le port utilisé

> ⚠️ Important : si tu as Docker, le port par défaut `5432` peut être occupé.

```bash
sudo ss -tlnp | grep postgres
```

Note le port affiché (ex: `5432` ou `5434`). Tu en auras besoin dans DBeaver.

---

## 3️⃣ Créer un utilisateur et une base de données

### Ouvrir le shell PostgreSQL

```bash
sudo -u postgres psql
```

### Dans le shell `postgres=#`

```sql
-- Créer un utilisateur
CREATE USER mon_user WITH PASSWORD 'mon_mot_de_passe';

-- Créer une base de données
CREATE DATABASE ma_base OWNER mon_user;

-- Donner tous les privilèges
GRANT ALL PRIVILEGES ON DATABASE ma_base TO mon_user;

-- Quitter
\q
```

---

## 4️⃣ Configurer l'authentification (pg_hba.conf)

```bash
sudo nano /etc/postgresql/17/main/pg_hba.conf
```

### Vérifier que ces lignes utilisent `scram-sha-256`

```
local   all             all                                     scram-sha-256
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
```

### Sauvegarder et redémarrer

```
Ctrl+O  →  Enter  →  Ctrl+X
```

```bash
sudo systemctl restart postgresql
```

---

## 5️⃣ Connecter DBeaver

### Ouvrir DBeaver → New Connection

1. Cliquer sur l'icône **+** (New Database Connection)
2. Choisir **PostgreSQL**
3. Remplir les champs :

| Champ       | Valeur                   |
|-------------|--------------------------|
| Host        | `localhost`              |
| Port        | `5434` *(ou ton port)*   |
| Database    | `ma_base`                |
| Username    | `mon_user`               |
| Password    | `mon_mot_de_passe`       |

4. Cliquer **Test Connection** → télécharge le driver si demandé
5. Cliquer **Finish**

---

## 6️⃣ Créer des tables via l'éditeur SQL

### Ouvrir un éditeur SQL

- Clic droit sur la base → **SQL Editor** → **New SQL Script**
- Ou raccourci : `Ctrl+]`

### Exemple de script SQL complet

```sql
-- Créer une table
CREATE TABLE etudiants (
    id          SERIAL PRIMARY KEY,
    nom         VARCHAR(100) NOT NULL,
    prenom      VARCHAR(100) NOT NULL,
    email       VARCHAR(150) UNIQUE NOT NULL,
    date_naissance DATE,
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insérer des données
INSERT INTO etudiants (nom, prenom, email, date_naissance) VALUES
('Mohameden', 'Abdy', 'abdy@supnum.mr', '2005-01-01'),
('Ould Ahmed', 'Mohamed', 'mohamed@supnum.mr', '2004-05-15');

-- Vérifier
SELECT * FROM etudiants;
```

### Exécuter

| Action                  | Raccourci      |
|-------------------------|----------------|
| Exécuter tout le script | `Ctrl+A` puis `Ctrl+Enter` |
| Exécuter une ligne      | Placer le curseur + `Ctrl+Enter` |
| Rafraîchir le sidebar   | `F5`           |

---

## 7️⃣ Commandes utiles en psql

```bash
# Ouvrir psql
sudo -u postgres psql

# Lister les bases
\l

# Se connecter à une base
\c ma_base

# Lister les tables
\dt

# Décrire une table
\d etudiants

# Quitter
\q
```

---

## ⚠️ Problèmes fréquents

### Port occupé par Docker
```bash
sudo ss -tlnp | grep postgres
# Si port = 5434, utiliser 5434 dans DBeaver
```

### Erreur d'authentification dans DBeaver
```bash
# Réinitialiser le mot de passe
sudo -u postgres psql -c "ALTER USER mon_user WITH PASSWORD 'nouveau_mdp';"
sudo -u postgres psql -c "SELECT pg_reload_conf();"
```

### Table non visible dans le sidebar
> Clic droit sur **Tables** → **Refresh** ou appuyer sur `F5`

---

## 📁 Fichiers de configuration importants

| Fichier | Rôle |
|--------|------|
| `/etc/postgresql/17/main/pg_hba.conf` | Méthodes d'authentification |
| `/etc/postgresql/17/main/postgresql.conf` | Configuration générale (port, mémoire...) |

---

*Guide rédigé pour SUPNUM — Nouakchott 🇲🇷*
