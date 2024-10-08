---
title: 01 - Single responsibility principle
author :
    name : Mathieu BESSON
    linkedin : mathieubesson
publication_date: 5/09/2024
---

# Single responsibility principle (SRP), Principe de responsabilité unique

!!! quote "Principe"

    Une classe doit avoir une et une seule raison de changer, une classe ne doit avoir qu'un seul emploi.

**Détail :**

- Une classe ne doit assumer qu'une seule et unique responsabilité.
- Cela évite le couplage fort et rend le code moins fragile.

## ⚠️ Sans le principe de SRP

Une classe `Book` fourre-tout : 

```php
<?php

// Class d'entité livre
class Book
{
    private string $title;
    private string $author;
    private string $isbn;

    public function __construct(string $title, string $author, string $isbn)
    {
        $this->title = $title;
        $this->author = $author;
        $this->isbn = $isbn;
    }

    public function getTitle(): string
    {
        return $this->title;
    }

    public function getAuthor(): string
    {
        return $this->author;
    }

    public function getIsbn(): string
    {
        return $this->isbn;
    }

    // ⚠️ Mauvaise pratique : responsabilité d'afficher les détails du livre
    public function printDetails(Book $book): void
    {
        echo 
            "Titre : " . $book->getTitle() . PHP_EOL .
            "Auteur : " . $book->getAuthor() . PHP_EOL .
            "ISBN : " . $book->getIsbn() . PHP_EOL;
    }

    // ⚠️ Mauvaise pratique : responsabilité de sauvegarde dans la base de donnée
    public function saveToDatabase(): void
    {
        // Connexion à la BDD et sauvegarde du livre
        // ...
    }
}

// Création du livre
$book = new Book('Clean Code', 'Robert C. Martin (Uncle Bob)', '789-364-142');
$book->printDetails();
$book->saveToDatabase();
```

### Les problèmes

La classe Book a 3 responsabilités :

- Gérer les données du livre (titre, auteur, ISBN).
- Afficher les détails du livre.
- Sauvegarder le livre dans la base de donnée.

Ce code ne respect donc pas le principe de responsabilité unique, ce qui créera à terme une classe fourre-tout et incompréhensible.

## ✅ Avec le principe de SRP

Voici les trois classes séparées qui respectent le principe de responsabilité unique (SRP) :

- Une classe `Book` pour gérer les données du livre.
- Une classe `BookPrinter` pour gérer l'affichage des détails du livre.
- Une classe `BookRepository` pour gérer la persistance du livre (sauvegarde dans la base de données).

```php
<?php

class Book
{
    private string $title;
    private string $author;
    private string $isbn;

    public function __construct(string $title, string $author, string $isbn)
    {
        $this->title = $title;
        $this->author = $author;
        $this->isbn = $isbn;
    }

    public function getTitle(): string
    {
        return $this->title;
    }

    public function getAuthor(): string
    {
        return $this->author;
    }

    public function getIsbn(): string
    {
        return $this->isbn;
    }
}

class BookPrinter
{
    public function printDetails(Book $book): void
    {
        echo 
            "Titre : " . $book->getTitle() . PHP_EOL .
            "Auteur : " . $book->getAuthor() . PHP_EOL .
            "ISBN : " . $book->getIsbn() . PHP_EOL;
    }
}

class BookRepository
{
    public function saveToDatabase(): void
    {
        // Connexion à la BDD et sauvegarde du livre
        // ...
    }
}

// Création du livre
$book = new Book('Clean Code', 'Robert C. Martin (Uncle Bob)', '789-364-142');

// Affichage des détails du livre
$printer = new BookPrinter();
$printer->printDetails($book);


// Sauvegarde du livre dans la base de données
$repository = new BookRepository();
$repository->saveToDatabase($book);
```

### Explications

Dans cette nouvelle version du code les responsabilités sont correctement répartient dans des clases correctement nommées, et ayant chacune un rôle bien précis.  

- Classe `Book` : Gestion des données du livre (titre, auteur, ISBN).
- Classe `BookPrinter` : Affichage des informations du livre dans différents formats (Ouvert à l'ajout de nouveaux formats : JSON, HTML...).
- Classe `BookRepository` : Persistance des données du livre, comme la sauvegarde dans une base de données.

## Solution générique

- Découper une classe qui détient trop de rôles en de multiples classes avec un rôle unique.

## Avantages du SRP

- Code plus clair, plus lisible et plus maintenable.
- Chaque classe à son rôle.
- Nommage clair et explicite.