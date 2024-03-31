
# TP1



url:

```https://start.spring.io/starter.zip?name=webapp&groupId=com.boot&artifactId=webapp&version=0.0.1-SNAPSHOT&description=Demo+project+for+Spring+Boot&packageName=com.example.demo&type=maven-project&packaging=jar&javaVersion=1.8&language=java&bootVersion=2.3.3.RELEASE&dependencies=devtools&dependencies=h2&dependencies=data-jpa&dependencies=security&dependencies=thymeleaf&dependencies=web```




## Partie 1 BDD

L'utilisation de JPA (Java Persistence API) permet de s'abstraire du langage de requête de la base de données.

H2 est une base de données embarquée, en mémoire, écrite en Java qui permet de garder le build portable.

La configuration de H2 est très simple. Il suffit de l'ajouter dans le POM pour que Spring Boot créer automatiquement la database, créer les objets JDBC et configure Hibernate. Hibernate va à son tour scanner toutes les annotations JPA et va créer automatiquement les tables dans la BDD.

Connexion console :
	JDBC URL=url=jdbc:h2:mem:testdb



## Partie 2 MVC

### CRUD

L'acronyme CRUD (pour create, read, update, delete) désigne les quatre opérations de base pour la persistance des données, en particulier le stockage d'informations en base de données.


Operation |SQL    |HTTP
----------|-------|--------
Create    |INSERT |POST
Read      |SELECT |GET
Update    |UPDATE |PUT
Delete    |DELETE |DELETE

L'annotation `@Autowired` placée sur un setter permet de demander à Spring de d'injecter une dépendance.


L'annotation `@Service` placée sur une classe permet de declarer à Spring que la classe est un service.

* Créer une classe annotée composant

```java
public class ProductLoader implements ApplicationListener<ContextRefreshedEvent>{}
```

Y injecter le ProductRepository.
Dans le onApplication event injecter plusieurs instances de produit dans le productRepository.

* Créer l'interface ProductService permettant au controller d'interragir avec le modèle:

```java
public interface ProductService {
    Iterable<Product> listAllProducts();

    Product getProductById(Integer id);

    Product saveProduct(Product product);

    void deleteProduct(Integer id);
}
```

* Implementer le @Service ProductServiceImpl en mappant toutes les méthodes sur le ProductRepository (à injecter)

* Completer le controller suivant. Il doit utiliser le ProductService.

```java
@Controller
public class ProductController {
	 private ProductService productService;
	 //GET sur /products

	    public String list(Model model){
	        model.addAttribute("products", ....
	        return "products";
	    }

	    //GET sur "product/show/{id}"
	    public String showProduct(@PathVariable Integer id, Model model){
	        model.addAttribute("product", ...
	        return "productshow";
	    }

	    //GET sur "product/edit/{id}"
	    public String edit(@PathVariable Integer id, Model model){
	        model.addAttribute("product",...
	        return "productform";
	    }

	    //GET sur"product/new"
	    public String newProduct(Model model){
	        model.addAttribute("product", new Product());
	        return "productform";
	    }

	    //POST sur "product"
	    public String saveProduct(Product product){
	    	...
	        return "redirect:/product/show/" + product.getId();
	    }

	    //GET sur "product/delete/{id}"
	    public String delete(@PathVariable Integer id){
	    	...
	        return "redirect:/products";
	    }
}
```


### Création / mise à jour d'un product

La même page HTML peut être utilisée pour créer et mettre à jour les produits.

L'astuce consiste à faire en sorte que le controller retourne un objet vide pour une création et l'objet existant pour une mise à jour.
De cette façon il n'y aura pas de problèmes d'objets null. Et toutes les propriétés non nulle peuvent se transformer en formulaire.

La ligne suivante crée le formulaire dans Thymeleaf.

```xml
<form class="form-horizontal" th:object="${product}" th:action="@{/product}" method="post">
```

Le tag `th:object` relie l'objet product au formulaire.

Le tag `th:action` définit quelle url doit être appelée lors de la soumission du formulaire (/product) url. La méthode `post` est aussi renseignée.

C'est donc le controller suivant qui va être appelé:

```java
@RequestMapping(value = "product", method = RequestMethod.POST)
public String saveProduct(Product product){
    productService.saveProduct(product);
    return "redirect:/product/" + product.getId();
}
```
Il retourne vers l'objet modifié.

L'étape suivante est cruciale pour que les mise à jour fonctionnent.

Toutes les entités présentes en BDD ont un ID. L'utilisateur ne peut pas l'éditer mais il doit être présent dans la requête de retour vers le serveur.
Si l'ID est manquant, le système ne peut pas savoir si c'est une MàJ ou une création. En cas d'oubli de l'ID une nouvelle entité sera crée en BDD.

La manière de gérer ce problème est d'inclure des champs cachés. (Pour hibernate le champ version permet d'éviter les conflits de mise à jour)

```xml
<input type="hidden" th:field="*{id}"/>
<input type="hidden" th:field="*{version}"/>
```

* Dans la prochaine étape nous allons utiliser dans fragments dans nos pages HTML pour éviter de répéter du code.

index.html

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <title>Spring boot Web app</title>
<!--     Inclusion d'un fragment, uniquement visible quand executé par spring -->
<!--     La mise en commentaire permet de faire afficher la page dans le browser sans erreur -->

    <!--/*/ <th:block th:include="fragments/headerinc :: head"></th:block> /*/-->
</head>
<body>
<div class="container">
    <!--/*/ <th:block th:include="fragments/menu :: header"></th:block> /*/-->
</div>
</body>
</html>
```

fragments/headerinc.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en" th:fragment="head">
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
<link
	href="http://cdn.jsdelivr.net/webjars/bootstrap/3.3.4/css/bootstrap.min.css"
	th:href="@{/webjars/bootstrap/3.3.4/css/bootstrap.min.css}"
	rel="stylesheet" media="screen" />
<script src="http://cdn.jsdelivr.net/webjars/jquery/2.1.4/jquery.min.js"
	th:src="@{/webjars/jquery/2.1.4/jquery.min.js}"></script>
<link href="../../static/css/boot.css" th:href="@{../css/boot.css}"
	rel="stylesheet" media="screen" />
</head>
<body>
</body>
</html>
```

fragments/menu.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <link href="http://cdn.jsdelivr.net/webjars/bootstrap/3.3.4/css/bootstrap.min.css"
          th:href="@{/webjars/bootstrap/3.3.4/css/bootstrap.min.css}"
          rel="stylesheet" media="screen"/>

    <script src="http://cdn.jsdelivr.net/webjars/jquery/2.1.4/jquery.min.js"
            th:src="@{/webjars/jquery/2.1.4/jquery.min.js}"></script>

   <link href="../../static/css/boot.css" th:href="@{../css/boot.css}"
	rel="stylesheet" media="screen" />
</head>
<body>

<div class="container">
    <div th:fragment="header">
        <nav class="navbar navbar-default">
            <div class="container-fluid">
                <div class="navbar-header">
                    <a class="navbar-brand" href="#" th:href="@{/}">Home</a>
                    <ul class="nav navbar-nav">
                        <li><a href="#" th:href="@{/products}">Products</a></li>
                        <li><a href="#" th:href="@{/product/new}">Create Product</a></li>
                    </ul>

                </div>
            </div>
        </nav>

        <div class="jumbotron">
            <div class="row text-center">
                <div class="">
                    <h2>Spring Framework</h2>

                    <h3>Spring Boot Web App</h3>
                </div>
            </div>
            <div class="row text-center">
                <img alt="boots" src="images/boots.jpg" width="300"
		th:src="@{../../images/boots.jpg}"/>
            </div>
        </div>
    </div>
</div>
</body>
</html>
```


* Puis nous allons ajouter les vues produit

products.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
    xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
<head lang="en">

<title>Spring Framework</title>

<!--/*/ <th:block th:include="fragments/headerinc :: head"></th:block> /*/-->
</head>
<body>
    <div class="container">
        <!--/*/ <th:block th:include="fragments/menu :: header"></th:block> /*/-->

        <div th:if="${not #lists.isEmpty(products)}">
            <form th:action="@{/logout}" method="post">
                <div class="col-sm-10">
                    <h2>Product Listing</h2>
                </div>
                <div class="col-sm-2" style="padding-top: 30px;">
                    <span sec:authorize="isAuthenticated()"> <input
                        type="submit" value="Sign Out" />
                    </span>
                </div>
            </form>
            <table class="table table-striped">
                <tr>
                    <th>Id</th>
                    <th>Product Id</th>
                    <th>Description</th>
                    <th>Price</th>
                <th sec:authorize="hasAnyRole('ROLE_USER','ROLE_ADMIN')">View</th>
                <th sec:authorize="hasRole('ROLE_ADMIN')">Edit</th>
                <th sec:authorize="hasRole('ROLE_ADMIN')">Delete</th>
                </tr>
                <tr th:each="product : ${products}">
                    <td th:text="${product.id}"><a href="/product/${product.id}">Id</a></td>
<!--    la notation th est equivalente au java     Product Id permet d'avoir un affichage avec un browser seul-->
                    <td th:text="${product.productId}">Product Id</td>
                    <td th:text="${product.description}">descirption</td>
                    <td th:text="${product.price}">price</td>
                <td sec:authorize="hasAnyRole('ROLE_USER','ROLE_ADMIN')"><a th:href="${'/product/show/' + product.id}">View</a></td>
                <td sec:authorize="hasRole('ROLE_ADMIN')"><a th:href="${'/product/edit/' + product.id}">Edit</a></td>
                <td sec:authorize="hasRole('ROLE_ADMIN')"><a th:href="${'/product/delete/' + product.id}">Delete</a></td>
                </tr>
            </table>
        </div>
    </div>
</body>
</html>
```

productshow.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
<head lang="en">
<title>Spring Framework</title>
<!--/*/ <th:block th:include="fragments/headerinc :: head"></th:block> /*/-->
</head>
<body>
    <div class="container">
    <!--/*/ <th:block th:include="fragments/menu :: header"></th:block> /*/-->

     <form class="form-horizontal" th:action="@{/logout}" method="post">
        <div class="form-group">
            <div class="col-sm-10"><h2>Product Details</h2></div>
            <div class="col-sm-2" style="padding-top: 25px;">
         <span sec:authorize="isAuthenticated()">
              <input type="submit" value="Sign Out"/>
           </span>

            </div>
        </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Product Id:</label>
                    <div class="col-sm-10">
                        <p class="form-control-static" th:text="${product.id}">Product Id</p></div>
                </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Product Bar Code:</label>
                    <div class="col-sm-10">
                        <p class="form-control-static" th:text="${product.productId}">Product Bar code</p></div>
                </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Description:</label>
                    <div class="col-sm-10">
                        <p class="form-control-static" th:text="${product.description}">description</p>
                    </div>
                </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Price:</label>
                    <div class="col-sm-10">
                        <p class="form-control-static" th:text="${product.price}">Priceaddd</p>
                    </div>
                </div>
                <div class="form-group">
                    <label class="col-sm-2 control-label">Image Url:</label>
                    <div class="col-sm-10">
                        <p class="form-control-static" th:text="${product.imageUrl}">url....</p>
                    </div>
                </div>
            </form>
        </div>

</body>
</html>
```

productform.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">

    <title>Spring Framework</title>

    <!--/*/ <th:block th:include="fragments/headerinc :: head"></th:block> /*/-->
</head>
<body>
<div class="container">
    <!--/*/ <th:block th:include="fragments/menu :: header"></th:block> /*/-->

    <form class="form-horizontal" th:action="@{/logout}" method="post">
        <div class="form-group">
            <div class="col-sm-10"> <h2>Product Create/Update</h2></div>
            <div class="col-sm-2" style="padding-top: 30px;">
                <input  type="submit" value="Sign Out"/>
            </div>
        </div>
    </form>

    <div>
        <form class="form-horizontal" th:object="${product}" th:action="@{/product}" method="post">
            <input type="hidden" th:field="*{id}"/>
            <input type="hidden" th:field="*{version}"/>
            <div class="form-group">
                <label class="col-sm-2 control-label">Bar Code:</label>
                <div class="col-sm-10">
                    <input type="text" class="form-control" th:field="*{productId}"/>
                </div>
            </div>
            <div class="form-group">
                <label class="col-sm-2 control-label">Description:</label>
                <div class="col-sm-10">
                    <input type="text" class="form-control" th:field="*{description}"/>
                </div>
            </div>
            <div class="form-group">
                <label class="col-sm-2 control-label">Price:</label>
                <div class="col-sm-10">
                    <input type="text" class="form-control" th:field="*{price}"/>
                </div>
            </div>
            <div class="form-group">
                <label class="col-sm-2 control-label">Image Url:</label>
                <div class="col-sm-10">
                    <input type="text" class="form-control" th:field="*{imageUrl}"/>
                </div>
            </div>
            <div class="row">
                <button type="submit" class="btn btn-default">Submit</button>
            </div>
        </form>
    </div>
</div>

</body>
</html>
```



## Partie 3 Sécurité / Rôles et permissions


Ce que nous voulons réaliser:

*	Pour un utilisateur pas authentifié: Vue page d'accueil et liste de produits.
* Pour un utilisateur authentifié (Rôle USER) vue en plus des détails du produit.
* Pour un  utilisateur authentifié (Rôle ADMIN) CRUD sur les produits.


A faire:

* Ajout dépendance `thymeleaf-extras-springsecurity5`
* Ajout de la page login.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<title>Login Form</title>
<!--/*/ <th:block th:include="fragments/headerinc :: head"></th:block> /*/-->
</head>
<body>
    <div class="container">
        <!--/*/ <th:block th:include="fragments/menu :: header"></th:block> /*/-->
        <div th:if="${param.error}">
            <label style="color: red">Invalid username and password.</label>
        </div>
        <div th:if="${param.logout}">
            <label> You have been logged out. </label>
        </div>

        <form th:action="@{/login}" method="post">

            <table class="table table-striped">
                <tr>
                    <td><label> User Name : <input type="text" name="username" />
                    </label></td>
                </tr>
                <tr>
                    <td><label> Password : <input type="password" name="password" />
                    </label></td>
                </tr>
                <tr>
                    <td>
                        <button type="submit" class="btn btn-default">Sign In</button>
                    </td>
                </tr>
            </table>

        </form>
    </div>
</body>
</html>
```

* Ajout dans le ProductController de la gestion de la page login



* Modification du controller de sécurité:
* Spécifier les url nécessitant de sécurité.
* Pour toutes les autres rajouter le code suivant:

```java
	.anyRequest().permitAll()
			.and().formLogin().loginPage("/login").permitAll()
			.and().logout().permitAll();
```

* Réutiliser le bean de création d'utilisateurs du TP Thymleaf-security

*  Ajout dans le menu d'une condition de création de product: `sec:authorize="hasRole('ROLE_ADMIN')"`


## Partie 4 Sécurisation de l'application avec la BDD


### Création des entitées JPA.

Bien lire les commentaires sur les classes. Les classes Roles et User ont une relation multivaleur (many-to-many) qui détermine que pour chaque enregistrement d'une table, il peut y avoir aucun, un ou plusieurs enregistrements d'une autre table qui lui soit liés.


```java
package com.boot.entities;

/**
 * Cette interface permet d avoir une base commune entre les classes User et Role
 */
public interface DomainObject {

    Integer getId();

    void setId(Integer id);
}
```


```java
package com.boot.entities;

import javax.persistence.*;
import java.util.Date;

//Permer meme avec heritage de ne pas generer une table pour cet objet
@MappedSuperclass
public class AbstractDomainClass implements DomainObject {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    Integer id;

    @Version
    private Integer version;

    private Date dateCreated;
    private Date lastUpdated;

    @Override
    public Integer getId() {
        return this.id;
    }

    @Override
    public void setId(Integer id) {
        this.id = id;
    }

    public Integer getVersion() {
        return version;
    }

    public void setVersion(Integer version) {
        this.version = version;
    }

    public Date getDateCreated() {
        return dateCreated;
    }

    public Date getLastUpdated() {
        return lastUpdated;
    }

    //Permet de s assurer que la date est bien mise a jour a chaque update en BDD
    @PreUpdate
    @PrePersist
    public void updateTimeStamps() {
        lastUpdated = new Date();
        if (dateCreated==null) {
            dateCreated = new Date();
        }
    }
}
```


```java
package com.boot.entities;

import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.JoinTable;
import javax.persistence.ManyToMany;
import java.util.ArrayList;
import java.util.List;


@Entity
public class Role extends AbstractDomainClass {

    private String role;

    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable
    // ~ defaults to @JoinTable(name = "USER_ROLE", joinColumns = @JoinColumn(name = "role_id"),
    //     inverseJoinColumns = @joinColumn(name = "user_id"))
    private List<User> users = new ArrayList<>();

    public String getRole() {
        return role;
    }

    public void setRole(String role) {
        this.role = role;
    }

    public List<User> getUsers() {
        return users;
    }

    public void setUsers(List<User> users) {
        this.users = users;
    }

    public void addUser(User user){
        if(!this.users.contains(user)){
            this.users.add(user);
        }

        if(!user.getRoles().contains(this)){
            user.getRoles().add(this);
        }
    }

    public void removeUser(User user){
        this.users.remove(user);
        user.getRoles().remove(this);
    }

}
```


```java
 package com.boot.entities;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;


@Entity
public class User extends AbstractDomainClass  {

    private String username;

    //Le mot de passe est annote Transient afin de ne pas stoquer le mot de passe en clair
    //mais d utiliser celui encrypte
    @Transient
    private String password;

    private String encryptedPassword;
    private Boolean enabled = true;

    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable
    // ~ defaults to @JoinTable(name = "USER_ROLE", joinColumns = @JoinColumn(name = "user_id"),
    //     inverseJoinColumns = @joinColumn(name = "role_id"))
    private List<Role> roles = new ArrayList<>();
    private Integer failedLoginAttempts = 0;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getEncryptedPassword() {
        return encryptedPassword;
    }

    public void setEncryptedPassword(String encryptedPassword) {
        this.encryptedPassword = encryptedPassword;
    }

    public Boolean getEnabled() {
        return enabled;
    }

    public void setEnabled(Boolean enabled) {
        this.enabled = enabled;
    }


    public List<Role> getRoles() {
        return roles;
    }

    public void setRoles(List<Role> roles) {
        this.roles = roles;
    }

    public void addRole(Role role){
        if(!this.roles.contains(role)){
            this.roles.add(role);
        }

        if(!role.getUsers().contains(this)){
            role.getUsers().add(this);
        }
    }

    public void removeRole(Role role){
        this.roles.remove(role);
        role.getUsers().remove(this);
    }

    public Integer getFailedLoginAttempts() {
        return failedLoginAttempts;
    }

    public void setFailedLoginAttempts(Integer failedLoginAttempts) {
        this.failedLoginAttempts = failedLoginAttempts;
    }
}
```






### Repositories JPA

* Nous allons créer un UserRepository et un RoleRepository sur le même exemple que le ProductRepository.

* Dans UserRepository ajouter la méthode

`User findByUsername(String username);`

* Les classes d'implémentation seront générées automatiquement par Spring JPA.



### Services JPA

* Ajouter :

```java
package com.boot.services;

import java.util.List;

public interface CRUDService<T> {
    List<?> listAll();

    T getById(Integer id);

    T saveOrUpdate(T domainObject);

    void delete(Integer id);
}

```

`RoleService` et `UserService` étendent `CRUDService` qui définit les opérations CRUD de base sur les entités.
`UserService`, avec la méthode supplémentaire     `User findByUsername(String username);` , est une interface de service plus spécialisée pour les opérations CRUD sur `User`.

Nous avons rendu les interfaces de service génériques pour masquer nos implémentations de service en utilisant le design pattern Façade. Les implémentations peuvent être Spring Data JPA avec référentiel, DAO, ou même JDBC simple, ou un service Web externe. Le code client n'a pas besoin de connaître l'implémentation. En utilisant des interfaces, nous sommes en mesure de tirer parti de plusieurs implémentations concrètes des services.

* Créer les trois classes (idem que ProductService)

* Completer UserServiceImpl

```java
package com.boot.services;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Profile;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.boot.entities.User;
import com.boot.repositories.UserRepository;
import com.boot.services.security.EncryptionService;


...
public class UserServiceImpl implements UserService {
    // a injecter
    private UserRepository userRepository;

    ...
    // a injecter
    private EncryptionService encryptionService;

    ....

    @Override
    public List<?> listAll() {
        ...
        return users;
    }

    @Override
    public User getById(Integer id) {
        return ...
    }

    @Override
    public User saveOrUpdate(User domainObject) {
        if(domainObject.getPassword() != null){
            domainObject.setEncryptedPassword(encryptionService.encryptString(domainObject.getPassword()));
        }
        return ...
    }
    @Override
      @Transactional
       public void delete(Integer id) {
        ...
    }

    @Override
    public User findByUsername(String username) {
        return ...
    }
}


```

* Completer RoleServiceImpl

```java
package com.boot.services;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Profile;
import org.springframework.stereotype.Service;

import com.boot.entities.Role;
import com.boot.repositories.RoleRepository;

...
public class RoleServiceImpl implements RoleService {
    //inject
    private RoleRepository roleRepository;

    ...

    @Override
    public List<?> listAll() {
       ...
        return roles;
    }

    @Override
    public Role getById(Integer id) {
        return ...
    }

    @Override
    public Role saveOrUpdate(Role domainObject) {
        return ...
    }

    @Override
    public void delete(Integer id) {
        ...
    }
}

```


### Cryptage du mot de passe

* Ajout des dépendances suivantes

```xml
        <dependency>
            <groupId>org.jasypt</groupId>
            <artifactId>jasypt-springsecurity4</artifactId>
            <version>1.9.3</version>
        </dependency>
        <dependency>
            <groupId>com.github.ulisesbocchio</groupId>
            <artifactId>jasypt-spring-boot-starter</artifactId>
            <version>3.0.3</version>
        </dependency>
```

* Dans la classe placer les bonnes annotations


```java
package com.boot.configuration;

import org.jasypt.util.password.StrongPasswordEncryptor;

...
public class CommonBeanConfig {
    ...
    public StrongPasswordEncryptor strongEncryptor() {
        StrongPasswordEncryptor encryptor = new StrongPasswordEncryptor();
        return encryptor;
    }
}


```

* Ajout de EncryptionService

```java
package com.boot.services.security;

public interface EncryptionService {
    String encryptString(String input);
    boolean checkPassword(String plainPassword, String encryptedPassword);
}

```

* Implémenter EncryptionServiceImpl en utilisant org.jasypt.util.password.StrongPasswordEncryptor


* Spring Security fournit une interface UserDetailsService pour rechercher le nom d'utilisateur, le mot de passe et GrantedAuthorities pour tout utilisateur donné. Cette interface ne fournit qu'une seule méthode, loadUserByUsername (). Cette méthode renvoie une implémentation de l'interface UserDetails de Spring Security qui fournit des informations utilisateur.

* Compléter et ajouter tous les setters nécessaires.


```java
package com.boot.services.security;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import java.util.Collection;


public class UserDetailsImpl implements UserDetails {

    private Collection<SimpleGrantedAuthority> authorities;
    private String username;
    private String password;
    private Boolean enabled = true;
...
		public void setAuthorities(Collection<SimpleGrantedAuthority> authorities) {
				this.authorities = authorities;
		}
}

```

* Dans cette classe, nous avons défini les champs de notre modèle de données et leurs méthodes setter correspondantes. Le `SimpleGrantedAuthority` que nous avons définit est une implémentation Spring Security d'une autorité que nous convertirons de notre rôle. Pensez à une autorité comme étant une «permission» ou un «droit» généralement exprimé sous forme de chaîne de caractère.

Nous devons fournir une implémentation de la méthode `loadUserByUsername ()` de `UserDetailsService`. Mais le défi est que la méthode `findByUsername ()` de notre `UserService` renvoie une entité `User`, tandis que Spring Security attend un objet `UserDetails` de la méthode `loadUserByUsername ()`.

Nous allons créer un convertisseur pour cela afin de convertir l'implémentation `User` en `UserDetails`.

* Compléter:

```java
package com.boot.services.security;

import java.util.ArrayList;
import java.util.Collection;

import org.springframework.core.convert.converter.Converter;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import com.boot.entities.User;

...
public class UserToUserDetails implements Converter<User, UserDetails> {
    @Override
    public UserDetails convert(User user) {
        UserDetailsImpl userDetails = new UserDetailsImpl();

        if (user != null) {
            userDetails.set...// tout setter
            Collection<SimpleGrantedAuthority> authorities = new ArrayList<>();
            user.getRoles().forEach(role -> {
                authorities.add(new SimpleGrantedAuthority(role.getRole()));
            });
            userDetails.setAuthorities(authorities);
        }

        return userDetails;
    }
}

```

* Cette classe implémente l'interface Spring Core Coverter et surcharge la méthode `convert ()` qui accepte un objet User à convertir.

Le convertisseur étant prêt, il est désormais facile d’implémenter l’interface `UserDetailsService`. La classe d'implémentation est la suivante.

```java
package com.boot.services.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.core.convert.converter.Converter;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import com.boot.entities.User;
import com.boot.services.UserService;

@Service("userDetailsService")
public class UserDetailsServiceImpl implements UserDetailsService {

    private UserService userService; // to set
    private Converter<User, UserDetails> userUserDetailsConverter;

    ...

    ...
    @Qualifier(value = "userToUserDetails")
    public void setUserUserDetailsConverter(Converter<User, UserDetails> userUserDetailsConverter) {
        this.userUserDetailsConverter = userUserDetailsConverter;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userUserDetailsConverter.convert(...//utiliser userService
    }
}

```
### Configuration de la Sécurité

* La classe de configuration de sécurité actuelle, `SecurityConfiguration`, étend `WebSecurityConfigurerAdapter` pour configurer deux choses. Un fournisseur d'authentification et les URL de l'application à protéger.Il reste à enregistrer le fournisseur d'authentification DAO pour une utilisation avec Spring Security.

Nous allons commencer par mettre en place un encodeur de mot de passe pour encoder les mots de passe présents dans l'objet `UserDetails` renvoyé par le `configuredUserDetailsService`. Nous allons définir un nouveau bean pour `PasswordEncoder` de Spring Security qui prend en charge le bean `StrongPassordEncryptor`.

Rappelez-vous que nous avons créé `StrongPassordEncryptor` plus tôt dans la classe de configuration `CommonBeanConfig`?

* Compléter la classe

```java
package com.boot.configuration;

import org.jasypt.springsecurity4.crypto.password.PasswordEncoder;
import org.jasypt.util.password.StrongPasswordEncryptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

...
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.authorizeRequests()
			.antMatchers("/product/edit/*","/product/new","/product/delete/*","/product/show/*").authenticated()
			.anyRequest().permitAll()
			.and().formLogin().loginPage("/login").permitAll()
			.and().logout().permitAll();
		http.csrf().disable();
		http.headers().frameOptions().disable();
	}

	...
	public PasswordEncoder passwordEncoder(StrongPasswordEncryptor passwordEncryptor){
	    PasswordEncoder passwordEncoder = new PasswordEncoder();
	    passwordEncoder.setPasswordEncryptor(passwordEncryptor);
	    return passwordEncoder;
	}
	...
	public DaoAuthenticationProvider daoAuthenticationProvider(PasswordEncoder passwordEncoder, UserDetailsService userDetailsService){
	    DaoAuthenticationProvider daoAuthenticationProvider = new DaoAuthenticationProvider();
	    daoAuthenticationProvider.setPasswordEncoder(passwordEncoder);
	    daoAuthenticationProvider.setUserDetailsService(userDetailsService);
	    return daoAuthenticationProvider;
	}
	...
    public void configureAuthManager(AuthenticationManagerBuilder authenticationManagerBuilder, @Qualifier("daoAuthenticationProvider") AuthenticationProvider authenticationProvider){
	    authenticationManagerBuilder.authenticationProvider(authenticationProvider);
	}
}

```

* `PasswordEncoder` va utiliser la bibliothèque `Jasypt` pour encoder le mot de passe et vérifier que les mots de passe correspondent. Le `UserDetailsService` récupérera l'objet User de la base de données et le transférera à Spring Security en tant qu'objet `UserDetails`.


### Initialisation de l'Application


Pour les données de départ de l'application, nous avons une classe d'implémentation `ApplicationListener` qui est appelée sur le `ContextRefresedEvent` au démarrage. Dans cette classe, nous utiliserons Spring pour injecter les référentiels JPA `UserRepository` et `RoleRepository`. Nous allons créer deux entités `User` et deux `Role` et les enregistrer dans la base de données au démarrage de l'application.

* Supprimer la classe `ProductLoader`

* Compléter la classe suivante

```java
package com.boot.mocks;

import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.stereotype.Component;

import com.boot.entities.Product;
import com.boot.entities.Role;
import com.boot.entities.User;
import com.boot.repositories.ProductRepository;
import com.boot.services.RoleService;
import com.boot.services.UserService;

...
public class SpringJpaBootstrap implements ApplicationListener<ContextRefreshedEvent> {

    private ProductRepository productRepository; //To inject
    private UserService userService; //To inject
    private RoleService roleService; //To inject

    private Logger log = LoggerFactory.getLogger(SpringJpaBootstrap.class);

...


    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        loadProducts();
        loadUsers();
        loadRoles();
        assignUsersToUserRole();
        assignUsersToAdminRole();
    }

    private void loadProducts() {
        Product shirt = new Product();
        shirt.setDescription(" Chemise Lacouste");
        shirt.setPrice(18.95f);
        shirt.setImageUrl("https://www.vision-naire.com/wp-content/uploads/2016/11/T-SHIRT-V-150x150.jpg");
        shirt.setProductId("235268845711068308");
        productRepository.save(shirt);

        log.info("Saved Shirt - id: " + shirt.getId());

        Product pant = new Product();
        pant.setDescription("Pantalon Hugo Boot");
        pant.setPrice(120.9f);
        pant.setImageUrl("https://www.vision-naire.com/wp-content/uploads/2020/06/jogging-visionnaire-2-100x100.jpg");
        pant.setProductId("168639393495335947");
        productRepository.save(pant);

        log.info("Saved Mug - id:" + pant.getId());
    }

    private void loadUsers() {
        User user1 = new User();
        user1.setUsername("user");
        user1.setPassword("user");
        userService.saveOrUpdate(user1);

        User user2 = new User();
        user2.setUsername("admin");
        user2.setPassword("admin");
        userService.saveOrUpdate(user2);

    }

    private void loadRoles() {
        Role role = ...
				// Créer et enregistrer les rôles "ADMIN" et "USER"
    }
    private void assignUsersToUserRole() {
        List<Role> roles = ...
        List<User> users = ...

        roles.forEach(role -> {
            if (role.getRole().equalsIgnoreCase("USER")) {
							...
							//assigner le rôle à l'utilisateur "user"

            }
        });
    }
    private void assignUsersToAdminRole() {
        List<Role> roles = ...
        List<User> users = ...

        roles.forEach(role -> {
            if (role.getRole().equalsIgnoreCase("ADMIN")) {
							...
							//assigner le rôle à l'utilisateur "admin"
            }
        });
    }
}
```

### Thymleaf Security

* Actuellement, USER et ROLE sont référencés à partir du code de couche de présentation comme ROLE_USER et ROLE_ADMIN. Cela était nécessaire car nous nous appuyions sur le fournisseur d'authentification en mémoire de Spring Security pour gérer nos utilisateurs et nos rôles, et la fonctionnalité interne de Spring Security mappe un rôle configuré au nom de rôle précédé de ROLE_.

* Avec le fournisseur d'authentification DAO, nos rôles sont mappés aux autorités telles quelles (nous l'avons fait dans le convertisseur `UserToUserDetails`), et nous pouvons les référer directement à partir du code en tant que USER et ADMIN.

* Le deuxième changement est apporté par GrantedAuthority utilisé par l'interface Spring Security `UserDetails`. Si vous vous en souvenez, nous avons mappé notre implémentation de rôle à `SimpleGrantedAuthority` dans le convertisseur `UserToUserDetails`.

* Par conséquent, dans les modèles Thymeleaf, nous devons changer les expressions d'autorisation hasRole () et hasAnyRole () en hasAuthority () et hasAnyAuthority ()

* Changer les templates menu et produit
