# spring-boot-rest-jpa-mysql
![1](https://user-images.githubusercontent.com/28655112/40723172-2a40d920-6416-11e8-9139-6a47c2ce7ed5.png)

## Objectif :

Construction d'une API REST Spring Boot avec intégration de la base de données MySQL et du JPA

Dans cette application, la base de données MySQL sera intégrée et Java Persistence API (JPA) pour interroger la base de données. JPA est une bibliothèque de mapping relationnel d'objets (ORM) . 

## Configuration de la base de données MySQL
-	Installer MySQL et créer une table.

-	Ouvrir le terminal MySQL et exécuter les commandes suivantes pour créer la base de données et la table des blogs :
```bash	
CREATE DATABASE restapi;
USE restapi;
CREATE TABLE blog (
  id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(500) NOT NULL,
  content VARCHAR(5000) NOT NULL);
```
## Configuration de l'application

Pour connecter l'application Spring Boot à la base de données MySQL:

•	Ajouter les dépendances requises (par exemple, la bibliothèque MySQL) au fichier "pom.xml". Ces dépendances sont les bibliothèques Java nécessaires pour se connecter à la base de données

•	Fournir les propriétés de connexion de base de données à l'application. Ces propriétés incluent la chaîne de connexion à la base de données (url), le port, le nom d'utilisateur, le mot de passe, etc.

•	Créer une classe qui parle à la base de données. Cette classe est communément appelée "classe Repository "

### Ajout de dépendances MySQL

Pour connecter l'application Spring Boot à la base de données MySQL, la bibliothèque "mysql-connector-java" est requise. De même, Spring JPA nécessite une bibliothèque "spring-boot-starter-data-jpa".
```bash
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```
### Définition des propriétés de connexion:

Pour connecter l'application Spring Boot à la base de données, nous devons fournir l'URL, le nom d'utilisateur et le mot de passe de la base de données à l'application.

Pour ce faire, on va créer un fichier "application.properties" dans le dossier "ressources"  contenant le code suivant :
```bash
spring.datasource.url=jdbc:mysql://localhost:3306/restapi
spring.datasource.username=root
spring.datasource.password=
```
•	spring.datasource.url: contient la chaîne de connexion MySQL 
•	spring.datasource.username: est le nom d'utilisateur MySQL 
•	spring.datasource.password: est le mot de passe MySQL

### Communiquer à la base de données: création d'une classe Repository

Maintenant que la base de données est prête, les dépendances installées, les propriétés de connexion fournies, la prochaine chose à faire est de créer une classe qui parle à la base de données. Cette classe est communément appelée Repository.

•	Créer une interface qui étend JpaRepository. Le JpaRepository nous donne quelques fonctionnalités hors de la boîte telles que la récupération de tous les enregistrements, enregistrement unique, sauvegarde, mise à jour, suppression, etc

•	Ajoutez l'annotation @Repository à l'interface pour indiquer au printemps qu'il s'agit d'un référentiel
 ```bash
 @Repository
public interface BlogRespository extends JpaRepository<Blog, Integer> {
    // custom query to search to blog post by title or content
    List<Blog> findByTitleContainingOrContentContaining(String text, String textAgain);
}
```

### Conversion de blog en une entité (@Entity)

•	On va créer une classe de blog "Blog.java" avec les champs de notre table. Chaque instance de Blog.java est une entrée dans notre table (c'est-à-dire une ligne).

•	Pour indiquer au Spring que Blog.java est une entité, nous devons ajouter une annotation @Entity à la classe.

•	La colonne "id" est la clé primaire et le champ généré automatiquement. Nous mettons l'annotation @Id sur le champs pour indiquer à Spring que "id" est une clé primaire,

•	@GeneratedValue (strategy = GenerationType.AUTO) indique à Spring que le champ est généré automatiquement et ne sera pas fourni par l'utilisateur, mais sera généré par la base de données.

•	On peut ajouter d’autre annotation comme @Table(name = “Blog”) lorsque le nom de la table est différent à celle du classe et l’annotation @Column(name="title") lorsque le nom du champ est différent à celle du colonne de la table de base de données

### Autowired, l'alternative Singleton

Une solution fournie par Spring est l'annotation @Autowired au lieu de créer un singleton (une méthode statique qui renvoie une instance de cette classe afin que les modifications puissent être conservées dans les différentes requêtes http). Si nous annotons une classe avec @Autowired, Spring résout automatiquement l'instance et l'injecte dans la classe qui l'a déclarée.

Par exemple, nous avons besoin d'une instance de BlogRepository dans la classe BlogController. 
```bash
@Autowired
BlogRespository blogRespository;
```
Et par conséquent, nous pouvons utiliser blogRepository n'importe où dans le contrôleur sans avoir à l'instancier.

## Le Controleur :

•	nous utilisons BlogRepository et l'annotation @Autowired

•	blogRepository a une méthode comme findAll, findOne, save, delete que nous n'avons pas définie dans "BlogRepository.java". Ces méthodes sont fournies par le JpaRepository que nous avons étendu.

•	findAll (): Renvoie toutes les lignes de la table. Ceci est équivalent à: ``` SELECT * FROM blog ```

•	findOne (param): renvoie un élément correspondant au champ de clé primaire avec param . Ceci est équivalent à:
```bash
SELECT * FROM blog WHERE id=param LIMIT 1
```
•	save (blog): enregistre l'entrée dans la base de données. Cette fonction créera un nouvel enregistrement si un nouvel article de blog est fourni et mettra à jour un article existant si un article de blog existant est fourni. Ceci est équivalent à:
```bash
INSERT INTO blog(title, content) VALUES (blog.title, blog.content)
```
Ou 
```bash
UPDATE blog SET title=blog.title, content=blog.content WHERE id=blog.id
```
•	delete (param): supprime une entrée dans la table en fonction de l'identifiant fourni équivalent à:
```bash
DELETE FROM blog WHERE id=param
```
## Requêtes JPA personnalisées

Comme JPA nous fournit les opérations CRUD par défaut (c'est-à-dire findAll, findOne, save, delete), il nous reste à définir nos requêtes personnalisées exemples : 
 
 ![2](https://user-images.githubusercontent.com/28655112/40724961-32ae4986-641a-11e8-9cd0-8d659a383451.png)
 
La clause "findBy" est le mot clé de la requête principale. Ce qui suit est le "Nom de colonne" puis la requête "Contraindre" comme Contient, Contenant, GreatherThan, LessThan etc
```bash
JPA:       findByIdGreatherThan(20)   
SQL:       WHERE id > 20
-----------------------------------------------------------------
JPA:       findByIdGreatherThanAndTitleContaining(20, "google")
SQL:       WHERE id > 20 AND title LIKE "%google%"
-----------------------------------------------------------------
JPA:       findByIdGreaterThanEqualOrTitle(5, "google api")
SQL:       WHERE id >= 5 OR title = "google api"
```
## Résultat: Output REST Controller 

Ajouter un blog 2 : http://localhost:8080/blog
 
 ![4](https://user-images.githubusercontent.com/28655112/40725033-52fce60c-641a-11e8-9b6c-528822fbefd3.PNG)
 
Nous obtiendrons à la base de données le résultat suivant :
 
![5](https://user-images.githubusercontent.com/28655112/40725077-68f47db2-641a-11e8-9184-54c33987f4cf.PNG)

Récupérer tous les blogs : http://localhost:8080/blog

![3](https://user-images.githubusercontent.com/28655112/40725104-7c4a9df6-641a-11e8-89a4-f228f71bf31e.PNG)
