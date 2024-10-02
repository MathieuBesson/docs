---
title: Tests unitaires en PHP
---

# Les tests unitaires en PHP

## C'est quoi ? 

!!! quote "Définition"

    Un test unitaire est un outil permettant de valider le fonctionnement d'une unité de code (fonctions ou méthodes), indépendamment des autres parties du système (isolation).

### Objectifs

- Éviter les régressions.
- Vérifier le fonctionnement indépendant de chaque fonction.
- Isoler les anomalies.

### Indices de qualité

- **Deterministe** : À chaque exécution le test effectue les strictement les mêmes vérifications et retourne le même résultat.
- **Unitaire** : Isolé et indépendant des composants extérieurs, en utilisant des mocks pour simuler les interactions externes.
- **Normalisé** : Le nom d'un test doit respecter la norme : `test + fonctionNom + comportementATester` en [camelCase](https://fr.wikipedia.org/wiki/Camel_case) (ex : `testregisterWithAdminUser()`).
- **Automatisé** : Éxécutable Sans actions manuelles.

!!! info "Outil"

    Pour mettre en place des tests unitaires avec [PHPUnit](https://phpunit.de) sur un projet PHP, il est d'abord nécéssaire de suivre la doc d'installation officielle de [PHPUnit](https://docs.phpunit.de/en/11.3/installation.html#composer) avec composer.

## Notions importantes

### Étapes d'un test : AAA

**AAA** : `Arrange`, `Act`, `Assert`

- **Arrange** : Définition et initialisation des variables et objets nécessaires au test (paramètres de la fonction à tester, mocks, contexte, attendu...).
- **Act** : Exécution de la méthode à tester.
- **Assert** : Comparaison du résultat (retour et modification du contexte) de la méthode avec les attentes du test.

!!! info "TestCase"

    La classe `TestCase` de PHPUnit intègre un grand nombre de fonctionnalités pour effectuer des tests unitaires : assertions, mocks, annotations... Elle est à la base de toutes les fonctionnalités liées aux tests unitaires avec PHPunit.


Avec PHPUnit, ces trois étapes donnent : 

```php
<?php

// Class à tester
class Calculator{
    public static function add(float $x, float $y): float
    {
        return $x + $y;
    }
}

// Class de test
class CalculatorTest extends TestCase {

    public function testAdd() {
        // Step 1 : Arrange
        $x = 2;
        $y = 1;
        $resultExpected = 3; 

        // Step 2 : Act
        $resultTest = CalculatorTest::add($x + $y);

        // Step 3 : Assert
        $this->assertEquals($resultExpected, $resultTest);
    }
}
```

### Assertions

Une assertion est une méthode permettant de valider qu'une chose est vraie dans un test.
Dans le contexte de tests unitaires, les assertions comparent la valeur retournée par la méthode testé et la valeur attendue.

Avec PHPUnit, il existe plusieurs types d'assertions basées sur différents types de vérification, en voici quelques-unes :

- Boolean : `assertTrue()` et  `assertFalse()`
- Identity : `assertSame()` (type et valeur)
- Equality  : `assertEquals()`, `assertObjectEquals()` (objets), `assertFileEquals()`...
- Iterable : `assertArrayHasKey()`, `assertContains()`...
- Objects : `assertObjectHasProperty()`
- Types : `assertIsArray()`, `assertIsBool()`, `assertIsFloat()`, `assertNull()`...
- Strings : `assertStringStartsWith()`, `assertStringContainsString()`, `assertFileMatchesFormat()`...


!!! info 

    La liste complète avec des exemples est disponible [ici](https://docs.phpunit.de/en/10.5/assertions.html).

### Isoler un test

Un test unitaire par définition doit isoler la méthode testée de ses interactions avec les autres modules du code.

Il est donc nécessaire d'isoler le test des interactions externes (services, classes externes, bases de données, fichiers...).

#### La solution : stubs et mocks

#### Stub

Un stub est un objet simulé qui retourne des réponses prédéfinies lorsqu'il est appelé. 
Il ne contient pas de logique complexe et sert à fournir des valeurs spécifiques en réponse à certaines interactions.

**Caractéristiques :**

- Retourne une réponse fixe et pré-configuré.
- Ne vérifie pas un comportement interne à l'objet simulé.

**Exemple avec PHPUnit :**

La classe `BookService` fait appel à la classe `ExternalBookApi` qui utilise une API externe pour récupérer les informations d'un livre.

Les tests unitaires devant être isolés les appels à l'API de la classe `ExternalBookApi` doivent être simulés grâce à un **stub**.

```php
<?php

class ExternalBookApi {
    public function getBookByISBN($isbn) {
        // Appel à une API externe pour obtenir les infos du livre (simulé ici)
        // Cela pourrait retourner des informations complexes venant d'une API externe
        return [
            'title' => 'Clean Code',
            'author' => 'Robert C. Martin'
        ];
    }
}

class BookService {
    private $externalBookApi;

    public function __construct(ExternalBookApi $externalBookApi) {
        $this->externalBookApi = $externalBookApi;
    }

    public function getBookTitleByISBN($isbn) {
        // Utilise l'API externe pour obtenir les infos du livre
        $book = $this->externalBookApi->getBookByISBN($isbn);
        return $book['title']; // Retourner le titre du livre
    }
}

// Classe de test
class BookServiceTest extends TestCase {

    public function testGetBookTitleByISBNWithStub() {
        // Création du stub pour ExternalBookApi
        $apiStub = $this->createStub(ExternalBookApi::class);

        // Configuration du retour de la fonction getBookByISBN() : retour fixe identique d'un livre fictif
        $apiStub->method('getBookByISBN')
                ->willReturn([
                    'title' => 'Clean Code',
                    'author' => 'Robert C. Martin'
                ]);

        // Création d'une instance de BookService avec le stub
        $bookService = new BookService($apiStub);

        // Appel de la méthode à tester
        $title = $bookService->getBookTitleByISBN('123-456-789');

        // Vérification du titre retournée
        $this->assertEquals('Clean Code', $title);
    }
}
```

#### Mock

Un mock est un stub avancé, il permet en plus de définir des attentes spécifiques auprès de l'objet simulé (méthode appellée plusieurs fois, arguments spécifiques...).

**Caractéristiques :**

- Permet de faire des vérifications du comportement interne à l'objet simulé.

**Exemple avec PHPUnit :**

La classe `BookService` fait appel à la classe `LoggerService` pour logger les accès au livre sur le serveur.

Pour isoler le test de la fonction `accessBook()`, il est nécessaire de simuler l'utilisation de la méthode `loggerService->log()`. D'une part pour isoler l'utilisation de la fonction de log, mais aussi d'autre part pour vérifier que celle-ci est bien appelée (`$this->once()`) avec les bons paramètres (`$this->equalTo()`).

```php
<?php

class LoggerService {
    public function log($message) {
        // Enregistrement d'un log dans un fichier de log...
    }
}

class BookService {
    private $loggerService;

    public function __construct(LoggerService $loggerService) {
        $this->loggerService = $loggerService;
    }

    public function accessBook($bookTitle) {
        // Journaliser l'accès au livre
        $this->loggerService->log("Accessed book: $bookTitle");

        // ... Récupération du livre (non testé, ce n'est pas le but de l'exemple)

        return $bookTitle;
    }
}

class BookServiceTest extends TestCase {

    public function testAccessBookWithMock() {
        // Création d'un mock pour LoggerService
        $loggerMock = $this->createMock(LoggerService::class);

        // Configuration du mock pour vérifier que la méthode log() est appelée avec le bon message
        $loggerMock->expects($this->once()) // On s'attend à ce que log() soit appelée une fois
                   ->method('log')
                   ->with($this->equalTo('Accessed book: Clean Code')); // On s'attend à ce que la fonction soit appeler avec ce paramètre

        // Création d'une instance de BookService avec le mock
        $bookService = new BookService($loggerMock);

        // Appel de la méthode à tester
        $title = $bookService->accessBook('Clean Code');

        // Vérification du titre retournée
        $this->assertEquals('Clean Code', $title);
    }
}
```

#### Limites et inconvénients

Les éléments suivants ne peuvent pas être mockés et doivent donc être véritablement instancié dans les tests en cas de besoin : 

- Les classes et méthodes finales.
- Les méthodes statiques et privées.
- Les énums et constantes.
- les accesseurs et mutateurs (get et set).

**Inconvénients :**

- La création des mocks peut être longue.
- En cas de changement du composant mocké impactant le mock, les tests continueront à être valide tout en étant incohérent avec le code source.
- Plus il y a de mocks, moins les tests sont proches de la réalité.

### Les fixtures 

Pour réaliser plusieurs tests sur une même classe, il est parfois nécéssaire de mutualiser la partie **Arrange** des trois étapes d'un test. 

Dans cet objectif, PHPUnit fournit deux méthodes lancées avant et après chaque test de la classe : `setUp()` et `tearDownn()`.

**Exemple** : 

```php
<?php

class FooTest extends TestCase {

    private Foo $foo;

    public function setUp()
    {
        // Arrange
        $this->foo = new Foo();
    }

    public function testFoo() 
    {
        // Act et Assert avec $this->foo
    }

    public function testBar() 
    {
        // Act et Assert avec $this->foo
    }
}
```

### Les annotations

Une annotation est une indication en haut d'un bloc de code (méthode ou classe) permettant d'jouter des éléments de configuration.

**Format :**

```php
/**
 * @annotation 
 */
```

#### Annotations avec PHPUnit

- `@before` et `@after`: Indique les méthodes devant être appelées avant et après chaque test.
- `@expectedException Exception` : Remplace le try, catch d'une `Exception`.
- `@dataProvider` : Fournit un jeu de test à une fonction de test.
```php
<?php

class FooTest extends TestCase
{
    /**
     * @dataProvider multiplyProvider
     */
    public function testMultiply($a, $b, $expected)
    {
        $this->assertSame($expected, $a + $b);
    }

    public function multiplyProvider()
    {
        return [
            [0, 3, 0],
            [1, 5, 5],
            [2, 9, 18],
        ];
    }
}
```

## La suite ? 🚀

Les tests unitaires sont un outil puissant pour valider au fil du temps le fonctionnement d'une méthode de manière isolé. Cependant, de part leur isolation du contexte de l'application et leur aspect unitaire, les tests unitaires ne permettent pas de valider le fonctionnement total d'une fonctionnalité ou de l'application en général, de bout en bout. Il est donc parfois recommandé d'utiliser d'autres types de tests comme les tests d'intégration, les tests fonctionnels ou encore les tests end-to-end pour valider différemment le comportement d'une l'application.

L'article suivant explore le mise en place de plusieurs types de [tests dans l'environnement d'une application Symfony](./tests-symfony/index.md).