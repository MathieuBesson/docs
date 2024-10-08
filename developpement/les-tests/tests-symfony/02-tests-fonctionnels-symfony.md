---
title: Tests fonctionnels Symfony
---

# Tests fonctionnels avec Symfony

!!! warning "Pré-requis"

    Pour bien comprendre cet article et notamment pour mettre en application les exemples techniques, il peut être nécéssaire d'avoir au préalable consulté l'article sur la configuration et la [mise en place d'un environnement de test avec Symfony](./index.md).

## C'est quoi ?

Dans un cadre de test plus large que les [tests unitaires](./../01-tests-unitaire-php.md) ou les [tests d'intégration](./01-tests-integration-symfony.md), les tests fonctionnels vérifient le fonctionnement correcte d'une application en se basant plus particulièrement sur les specs fonctionnelles.

Ce type de test n'est pas isolé (à contrario des TU et TI) et est en charge de s'assurer du bon fonctionnnement d'une fonctionnalité/composant dans son entiereté : intéraction utilisateur, API, BDD... 

!!! quote "Définition"

    Un test fonctionnel est un processus (en mode boite grise) vérifiant qu'une application répond correctement à ses spécifications fonctionnels.

### Objectifs

- Vérifier que chaque fonctionnalité clé répond aux exigences métiers.
- Valider les interactions entre le système (interface, API, BDD...) et l’utilisateur (UI, API, etc.).

### Exemples de cas tests fonctionnels

- L'utilisateur peut se connecter sur la plateforme.
- Le processus de commande d'un produit guide l'utilisateur à travers le tunnel de paiement jusqu'à l'achat.
- Un utilisateur peut prendre un rendez-vous avec un prestataire en selectionnant le créneau souhaité.
- Un utilisateur administrateur peut créer un produit sur la plateforme (*Exemple final*)

### Indices de qualité

- ✅ **Fidélité à l'expérience utilisateur** : Le test doit simuler des actions et scénarios réels du point de vue de l'utilisateur final.
- ✅ **Interaction complète avec le système** : Il doit couvrir toutes les couches de l'application (UI, API, base de données) et tester leur intégration.
- ✅ **Vérifie besoin métier** : Le test doit vérifier que les fonctionnalités répondent aux besoins métiers définis.
- ✅ **Exhaustif** : Il doit tester tous les cas d'une fonctionnalités, y compris les cas limites et les erreurs potentielles.

## Notions importantes

!!! info "Exemples avec Symfony et PHPUnit"

    Les exemples suivants permettent de mettre en place des tests fonctionnels dans une application Symfony à l'aide du framework de test PHPUnit. Les paragraphes suivants présentent donc les notions de base de ce framework de test (dans le cadre des tests fonctionnels).

### La classe `WebTestCase`

La classe `WebTestCase` issue du bundle `symfony/test-pack` de Symfony offre de multiples fonctionnalités tel que : 

- Simulation d'un client HTTP classique
- Accès au conteneur de service
- Simulation d'intéractions utilisateur
- Inspection du DOM et manipulation de contenu HTML
- Vérification des redirections
- Gestion des sessions et cookies

### Client HTTP et crawler

Dans le cadre de la mise en place de tests fonctionnels, il est souvent utile d'avoir accès à un client HTTP capable de faire des requêtes et de traiter les réponses.

Dans cette optique la classe de test `WebTestCase` offre de nombreuses fonctionnalités. Liste des possibilités avec le `client` et le `crawler` :

Requêtes et traitement de la réponse :  

```php
<?php

// Création du client HTTP
$client = static::createClient();

// Requêtes HTTP avec des différents verbes
$client->request('GET', '/products');
$client->request('POST', '/admin/products/create', ['name' => 'Toy']);


// Assertions sur la réponse de la requête

// Vérification du code de status
$this->assertEquals(200, $client->getResponse()->getStatusCode());

// Assertion sur le contenu de la réponse
$this->assertSelectorTextContains('h1', 'Toy');

// Assertion sur le contenu du DOM
$this->assertCount(1, $crawler->filter('h1'));
```

Gestion des redirections : 

```php
<?php

// Vérification redirection
$this->assertResponseRedirects('/products/1234');
// Suivi de redirection
$client->followRedirect();
```

Accès au conteneur d'injection de dépendance : 

```php
<?php

// Accès au conteneur de dépendance 
$container = self::$container;
$doctrine = $container->get('doctrine');
```

Gestion de la session et des cookies : 

```php
<?php

// Modification de la session
$session = $client->getContainer()->get('session');
$session->set('user_id', 123); // Simuler une session utilisateur
$client->request('GET', '/products');

// Modification des cookies
$client->getCookieJar()->set(new Cookie('test_cookie', 'cookie_value'));
$client->request('GET', '/products');
```

### Authentifier un utilisateur

Dans le cas de test d'une partie protégé par authentification d'une application, il est parfois nécéssaire d'authentifier un utilisateur.

Dans cette optique, la classe `WebTestCase` fournit une fonction permettant d'authentifier n'importe quel utilisateur : `loginUser`

```php
<?php

$client = static::createClient();

// Récupérer l'utilisateur (par exemple depuis Doctrine)
$user = $client->getContainer()->get('doctrine')->getRepository(User::class)->findOneByEmail('contact@doc.mathieu-besson.fr');

// Authentifier l'utilisateur
$client->loginUser($user);
```

### Traitement d'un formulaire

Dans le cadre du test d'un formulaire il est parfois nécéssaire de tester la soumission des formulaires.

```php
<?php

// Soumission d'un formulaire
$crawler = $client->request('GET', '/form-page');

// Récuparation du formulaire
$form = $crawler->filter("form[name=create_product]")

// Remplissage et soumission du formulaire
$form->form([
    'product_name' => 'Toy'
])
$client->submit($form);
```

### Les assertions

Ci-dessous une liste des assertions les plus souvent utilisés dans les tests d'intégration, permettant de vérifier que le cas testé répond bien aux attentes définis initialement.

Les assertions sur les **réponses** :

| Type                     | Méthode                                                                         |
|--------------------------|---------------------------------------------------------------------------------|
| **Statut HTTP**          | `assertResponseStatusCodeSame(200)`                                             |
| **Redirection**          | `assertResponseRedirects('/products'); `                                        |
| **Contenu réponse**      | `assertStringContainsString('Produits', $client->getResponse()->getContent());` |
| **Contenu réponse JSON** | `assertJson($client->getResponse()->getContent());`                             |

Les assertions sur le **crawler** :

| Type                     | Méthode                                                |
|--------------------------|--------------------------------------------------------|
| **Existance balise**     | `assertSelectorExists('h1')`                           |
| **Contenu balise**       | `assertSelectorTextContains('h1', 'Bienvenue')`        |
| **Non existence classe** | `assertSelectorNotExists('.error-messsage')`           |
| **Nombre d'éléments**    | `assertCount(3, $crawler->filter('div.product-item'))` |

## Exemple final de test fonctionnel

Le test suivant utilise toutes les notions précédantes et est en charge de vérifier qu'un utilisateur administrateur est capable de créer un produit et que celui-ci est correctement créé et disponible sur la plateforme.

Dans l'exemple de test ci-dessous, les étapes sont les suivantes :

- Récupération du client HTTP
- Récupération et connexion de l'utilisateur administrateur
- Vérification d'accès à la page de création du produit
- Création du nouveau produit avec le formulaire
- Vérification d'insertion du produit dans la BDD
- Vérification de la disponibilité du produit sur la plateforme 

```php
<?php

class ProductControllerTest extends WebTestCase
{
    public function testAdminCanCreateProduct()
    {
        // 1. Création d'un client HTTP
        $client = static::createClient();

        // 2. Récupération du repository des utilisateurs et trouver un admin
        $userRepository = static::getContainer()->get(UserRepository::class);
        $adminUser = $userRepository->findOneByEmail('admin@example.com');

        // 3. Simulation de la connexion de l'admin
        $client->loginUser($adminUser);

        // 4. Récupération de l'URL de la page de création de produit
        $router = static::getContainer()->get(RouterInterface::class);
        $productCreateUrl = $router->generate('admin.product_new'); 

        // 5. Vérification d'accès à la page de création de produit
        $crawler = $client->request('GET', $productCreateUrl);
        $this->assertResponseIsSuccessful();
        $this->assertSelectorTextContains('h1', 'Créer un nouveau produit');

        // 6. Soumission du formulaire de création de produit
        $form = $crawler->selectButton('Créer')->form([
            'product[name]' => 'Test Product',
            'product[price]' => 19.99,
            'product[inStock]' => true,
        ]);
        $client->submit($form);

        // 7. Vérification des données de produits insérés dans la base de donnée
        $productRepository = static::getContainer()->get(ProductRepository::class);
        $product = $productRepository->findOneBy(['name' => 'Test Product']);

        // Le produit existe avec les données correctes
        $this->assertNotNull($product);
        $this->assertEquals(19.99, $product->getPrice());
        $this->assertTrue($product->isInStock());

        // 8. Vérification de la redirection vers la page de la liste des produits
        $productListUrl = $router->generate('admin.product_list');
        $this->assertResponseRedirects($productListUrl);
        $client->followRedirect();

        // 9. Vérification de l'affichage du produit créé
        $this->assertSelectorTextContains('.product-name', 'Test Product');
        $this->assertSelectorTextContains('.product-price', '19.99');
        $this->assertSelectorTextContains('.product-stock-status', 'En stock');
    }
}
```

Ce test permet de vérifier que la création d'un produit fonctionne correctement. Pour être totalement sûr du fonctionnement de la création d'un produit, il serait nécéssaire d'implémenter d'autres tests pour vérifier le comportement avec : 

- des données érronés (mauvais type de donnée, incohérence, données vides, doublon produit, longueur chaine et entiers)
- des droits d'accès insuffisant
- des données facultatives non remplis...

## La suite ? 🚀

**Valider une fonctionnalité comme un utilisateur final : Les tests end-to-end**

Dans le cadre d'une application ayant une partie front-end dynamique avec des framework JS (React.js, Vue.js, Angular...), les tests fonctionnels comme présentés précedemment ne seront pas efficaces. 

Dans ce type de cas il est nécéssaire de simuler un navigateur lors du test pour valider une fonctionnalité de l'application, ce type de tests est appelé **tests end-to-end**. Il est possible de mettre en place ce type de tests à l'aide d'outil comme [Panther](https://github.com/symfony/panther) permettant de scrapper directement le contenu d'une page, en utilisant un navigateur, comme le ferait l'utilisateur final 🚀.
