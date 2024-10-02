---
title: Tests d'intégration Symfony
---

# Tests d'intégration avec Symfony

!!! warning "Pré-requis"

    Pour bien comprendre cet article et notamment pour mettre en application les exemples techniques, il peut être nécéssaire d'avoir au préalable consulté l'article sur la configuration et la [mise en place d'un environnement de test avec Symfony](./index.md).

## C'est quoi ?

Au delà des [tests unitaires](./01-tests-unitaire-php.md) testant simplement les entrées, sorties et comportemment de manière isolé des méthodes simples des classes d'une application, il est parfois nécéssaire de vérifier le fonctionnement d'un module ou d'un composant (ex : intéraction avec la BDD, méthodes dites "procédures", intéraction avec des services externes...).

Dans cette optique, les tests d'intégration permettent de vérifier le bon fontionnement d'un ensemble de modules ensemble, de manière isolé du reste de l'application.

!!! quote "Définition"

    Un test d'intégration vérifie le fonctionnement de **plusieurs modules** d'un logiciel **ensemble**.

### Objectifs

- Éviter les régressions.
- Vérifier la communication inter-modulaire.
- Valider les échanges de données entre composants

### Indices de qualité

- **Fiable** : Résultats similaires à chaque exécution, même avec plusieurs composants impliqués.
- **Représentatif** : Reproduction de cas réels pour vérifier les intégrations dans des conditions proches de la production.
- **Automatisé** : Exécutable sans actions manuelles.

## Notions importantes

### La classe `KernelTestCase`

La classe `KernelTestCase` issue du bundle `symfony/test-pack` de Symfony offre de multiples fonctionnalités tel que : 

- La possibilité d'accès au contenur de services de Symfony, 
- L'initialisation du noyau pour configuration de l'environnement (services, routes, bundles, injection de dépendance, variables d'env, )
- L'intéraction avec la base de donnée via Doctrine
- Bien plus encore...

### Accèder à un service dans un test

L'exemple suivant détail l'accès à un service depuis un test d'intégration. 

- Dans un premier temps, il est nécéssaire de démarrer le kernel de Symfony
- Après récupération du contenur d'injection de dépendance, on peut initialiser le service souhaité
- On peut ensuite effectuer des tests sur le service

```php
<?php

class ArticleServiceTest extends KernelTestCase
{
    public function testAddArticle(): void
    {
        // Démarrage du kernel de Symfony
        self::bootKernel();

        // Récupération du contenur de service
        $container = static::getContainer();

        // Initialisation du service 
        $articleService = $container->get(ArticleService::class);
        $article = $articleService->addArticle("...", "...");

        $this->assertEquals("...", $article->getContent());
    }
}
```

### Mocker un service ?

Dans certains cas, on peut avoir besoin de mocker un service utilisé par le service à tester.

Par exemple si `ArticleService` se sert de `SlugGenerator`, il sera nécéssaire de mocker `SlugGenerator` dans le test pour isoler le test des intéraction avec ce service.

Dans ce cas, la création du mock du service peut se faire de la manière suivante : 

```php
<?php

class ArticleServiceTest extends KernelTestCase
{
    public function testAddArticle(): void
    {
        // ...

        // Création du mock de SlugGenerator
        $slugGenerator = $this->createMock(SlugGenerator::class);
        $slugGenerator->expects(self::once())
            ->method('slugify')
            ->with($this->equalTo("How to make pancakes"))
            ->willReturn("how-to-make-pancakes");

        // Injection du mock du service SlugGenerator à la place du vrai service
        $container->set(SlugGenerator::class, $slugGenerator);

        // Récupération du service ArticleService
        $articleService = $container->get(ArticleService::class);

        // ...
    }
}
```

## Exemple final de test d'intégration

En reprenant l'exemple précédant et les principes détaillés plus haut, le test suivant est en charge de vérifier le bon fonctionnement de la méthode `addArticle()`. 

Par étape, il est donc nécéssaire de vérifier que : 

- L'ajout de l'article fait bien appel au service `SlugGenerator` pour associer un slug à l'article.
- Une entité `Article` est correctement hydraté et sauvegardé en base de donnée par la méthode.


```php
<?php

// Service à tester
class ArticleService
{
    public function __construct(
        private EntityManagerInterface $entityManager, 
        private SlugGenerator $slugGenerator
    ) {}

    public function addArticle(string $title, string $content): Article
    {
        $article = new Article();
        $article->setTitle($title);
        $article->setContent($content);
        $article->setSlug($this->slugGenerator->slugify($title));

        $this->entityManager->persist($article);
        $this->entityManager->flush();

        return $article;
    }
}

// Classe de tests du service
class ArticleServiceTest extends KernelTestCase
{
    public function testAddArticle()
    {
        // Démarrage du kernel
        self::bootKernel();

        // Récupération du contenur d'injection de dépendance
        $container = self::$kernel->getContainer();
        $articleExpected = [
            "title" => "How to make pancakes", 
            "content" => "See in marmiton.com please, this is not a cooking site here",
            "slug" => "how-to-make-pancakes"
        ];

        // Création du mock de SlugGenerator
        $slugGenerator = $this->createMock(SlugGenerator::class);
        $slugGenerator->expects(self::once())
            ->method('slugify')
            ->with($this->equalTo($articleExpected["title"]))
            ->willReturn($articleExpected["slug"]);

        // Injection du mock du service SlugGenerator à la place du vrai service
        $container->set(SlugGenerator::class, $slugGenerator);

        // Récupération du service ArticleService
        $articleService = $container->get(ArticleService::class);
        
        // Test de la fonction addArticle()
        $article = $articleService->addArticle($articleExpected["title"], $articleExpected["content"]);

        // Vérification de l'ajout de l'article
        $this->assertInstanceOf(Article::class, $article);
        $this->assertEquals($articleExpected["title"], $article->getTitle());
        $this->assertEquals($articleExpected["content"], $article->getContent());

        // Vérification de la présence de l'article dans la base de donnée
        $entityManager = $container->get('doctrine')->getManager();
        $foundArticle = $entityManager->getRepository(Article::class)->find($article->getId());

        $this->assertNotNull($foundArticle);
        $this->assertEquals($title, $foundArticle->getTitle());
    }
}
```

Le test précédant test donc l'intégration de la méthode `addArticle()` avec la base de donnée et le service `SlugGenerator`. 
L'objectif établi est ici atteint, ce test permettant de valider le bon fontionnement de cette méthode dans son environnement proche, de manière isolé du reste de l'application.

## La suite ? 🚀

**Comment valider une fonctionnalité entière : Les tests fonctionnelles**

À plus haut niveau, il est possible de tester et vérifier que des comportements ou fonctionnalités entières répondent correctement au besoin métier. Une doc est prévu à ce sujet sur l'implémentation des tests fonctionnelles dans une application Symfony. 🚀