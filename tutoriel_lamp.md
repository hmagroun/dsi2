# Atelier Pratique : Déploiement d'une Application Web LAMP

**Environnement :** Machine locale Ubuntu (Desktop ou Server)

**Accès réseau :** `localhost` (`127.0.0.1`)

## 1. Introduction & Objectifs

### L'architecture LAMP

La pile **LAMP** est l'un des ensembles logiciels open-source les plus populaires pour l'hébergement d'applications web dynamiques. Elle se compose de :

* **L**inux : Le système d'exploitation hôte.
* **A**pache : Le serveur HTTP qui traite les requêtes web.
* **M**ySQL : Le système de gestion de base de données relationnelle (SGBDR).
* **P**HP : Le langage de script côté serveur.

```
       +-----------------------------------------------+
       |             Navigateur Web (Client)           |
       +-----------------------------------------------+
                               |
                   Requête HTTP (Port 80)
                               v
       +-----------------------------------------------+
       |             Serveur Web Apache                |
       +-----------------------------------------------+
                               |
                     Exécution de script
                               v
       +-----------------------------------------------+
       |               Moteur PHP                      |
       +-----------------------------------------------+
                               |
                    Requêtes SQL (localhost)
                               v
       +-----------------------------------------------+
       |            Base de données MySQL              |
       +-----------------------------------------------+
```

## 2. Module 1 : Installation et configuration d'Apache

### Étape 1.1 : Mise à jour de l'index des paquets

Avant toute installation, mettez à jour la liste des paquets disponibles dans les dépôts d'Ubuntu :

```bash
sudo apt update
```

### Étape 1.2 : Installation du serveur web Apache

Installez le paquet `apache2` via le gestionnaire `apt` :

```bash
sudo apt install apache2 -y
```

### Étape 1.3 : Vérification du service Apache

Vérifiez que le service est en cours d'exécution et configuré pour démarrer au boot :

```bash
systemctl status apache2
```

*Signal de succès :* Le terminal affiche `active (running)`.

### Étape 1.4 : Validation dans le navigateur

Ouvrez votre navigateur web ou utilisez `curl` depuis le terminal pour vérifier la réponse du serveur local sur le port 80 :

```bash
curl -I http://localhost
```

Vous pouvez également ouvrir l'adresse `http://localhost` dans votre navigateur web. La page d'accueil par défaut d'Ubuntu Apache (Ubuntu Default Page) s'affiche.

## 3. Module 2 : Installation et sécurisation de MySQL

### Étape 2.1 : Installation du serveur MySQL

Installez le système de gestion de base de données MySQL :

```bash
sudo apt install mysql-server -y
```

### Étape 2.2 : Sécurisation de l'installation MySQL

Exécutez le script d'installation sécurisée fourni par MySQL. Ce script permet d'éliminer les configurations par défaut non sécurisées :

```bash
sudo mysql_secure_installation
```

Répondez aux invites comme suit :

1. **VALIDATE PASSWORD COMPONENT** : Répondez `Y` (Oui) ou `N` (Non) selon vos besoins de complexité de mot de passe en environnement local.
2. **Set Root Password** : Définissez et confirmez un mot de passe robuste pour l'utilisateur `root` de MySQL.
3. **Remove anonymous users** : Répondez `Y` pour supprimer les comptes anonymes.
4. **Disallow root login remotely** : Répondez `Y` pour interdire toute connexion distante avec le compte `root`.
5. **Remove test database** : Répondez `Y` pour supprimer la base de données de test.
6. **Reload privilege tables** : Répondez `Y` pour appliquer immédiatement les modifications.

### Étape 2.3 : Création d'une base de données et d'un utilisateur local

Par mesure de sécurité, évitez d'utiliser le compte `root` pour vos applications. Connectez-vous à la console MySQL :

```bash
sudo mysql
```

Créez une base de données nommée `demo_db`, un utilisateur local `app_user` (limité à `localhost`), et attribuez-lui les privilèges nécessaires :

```sql
CREATE DATABASE demo_db;
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'MotDePasseSecurise123!';
GRANT ALL PRIVILEGES ON demo_db.* TO 'app_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 4. Module 3 : Installation et intégration de PHP

### Étape 3.1 : Installation de PHP et des modules requis

PHP nécessite le paquet de base ainsi que des modules permettant de communiquer avec le serveur Apache et la base de données MySQL :

```bash
sudo apt install php libapache2-mod-php php-mysql -y
```

### Étape 3.2 : Ajustement des priorités des fichiers d'index d'Apache

Par défaut, Apache recherche d'abord le fichier `index.html`. Pour donner la priorité aux fichiers PHP (site par défaut), éditez le fichier de configuration `dir.conf` :

```bash
sudo nano /etc/apache2/mods-enabled/dir.conf
```

Modifiez le contenu pour placer `index.php` en première position :

```apache
<IfModule mod_dir.c>
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

Enregistrez et fermez le fichier (`Ctrl + O`, `Entrée`, puis `Ctrl + X`).

### Étape 3.3 : Redémarrage d'Apache

Appliquez les changements en rechargeant la configuration d'Apache :

```bash
sudo systemctl reload apache2
```

## 5. Module 4 : Test de l'application et validation finale

### Étape 5.1 : Validation de l'interprétation PHP (`info.php`)

Créez un script PHP d'information à la racine du site web par défaut (`/var/www/html/`) :

```bash
sudo nano /var/www/html/info.php
```

Ajoutez le code suivant :

```php
<?php
phpinfo();
?>
```

Enregistrez le fichier, puis ouvrez l'URL suivante dans votre navigateur :

`http://localhost/info.php`

*Signal de succès :* Une page structurée détaillant la version de PHP et la configuration des modules chargés s'affiche.

### Étape 5.2 : Test de connexion à la base de données MySQL

Créez un script d'application complet afin de vérifier la communication entre Apache, PHP et MySQL sur `localhost` :

```bash
sudo nano /var/www/html/test_db.php
```

Ajoutez le script suivant :

```php
<?php
$servername = "localhost";
$username   = "app_user";
$password   = "MotDePasseSecurise123!";
$dbname     = "demo_db";

// Connexion à la base de données
$conn = new mysqli($servername, $username, $password, $dbname);

// Vérification de la connexion
if ($conn->connect_error) {
    die("<p style='color:red;'>Échec de la connexion : " . $conn->connect_error . "</p>");
}
echo "<p style='color:green;'>Connexion réussie à la base de données MySQL sur localhost !</p>";

// Création d'une table de test
$sql_create = "CREATE TABLE IF NOT EXISTS todos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tache VARCHAR(255) NOT NULL,
    cree_le TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)";
$conn->query($sql_create);

// Insertion d'un enregistrement
$sql_insert = "INSERT INTO todos (tache) VALUES ('Configurer la pile LAMP sur Ubuntu')";
$conn->query($sql_insert);

// Lecture et affichage des données
$result = $conn->query("SELECT * FROM todos");

echo "<h3>Liste des tâches (Table 'todos') :</h3><ul>";
while($row = $result->fetch_assoc()) {
    echo "<li>" . htmlspecialchars($row["tache"]) . " <i>(" . $row["cree_le"] . ")</i></li>";
}
echo "</ul>";

$conn->close();
?>
```

Accédez à la page dans votre navigateur :

`http://localhost/test_db.php`

