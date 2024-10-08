---
title: 02 - Open/Closed Principle
author :
    name : Mathieu BESSON
    linkedin : mathieubesson
publication_date: 5/09/2024
---

# Open/Closed Principle (OCP), Ouvert/fermé

!!! quote "Principe"

    Une entité (classe ou fonction) doit être ouverte à l'extension mais fermée à la modification.

**Détail :**

- On doit pouvoir ajouter des fonctionnalités à une entité sans modifier le code déjà existant.
- Favoriser l'extension du code à sa modification.

## ⚠️ Sans le principe de l'OCP

Dans l'exemple suivant, nous avons une classe `Book` et une classe `BookDiscountCalculator` qui calcule une réduction sur un livre. 

Si nous voulons ajouter une nouvelle stratégie de réduction (par exemple, un type spécial de réduction pour les e-book), nous devons modifier directement la méthode `calculateDiscount`, ce qui respecte pas le principe OCP.

```php
<?php

// Class d'entité livre
class Book
{
    const BOOK_TYPE_EBOOK = 'ebook';
    const BOOK_TYPE_LIMITED_EDITION = 'limited_edition';

    const BOOK_TYPE_DISCOUNT_EBOOK = 0.15;
    const BOOK_TYPE_DISCOUNT_LIMITED_EDITION = 0.20;

    private string $title;
    private float $price;

    public function __construct(string $title, float $price)
    {
        $this->title = $title;
        $this->price = $price;
    }

    public function getTitle(): string
    {
        return $this->title;
    }

    public function getPrice(): float
    {
        return $this->price;
    }
}

// Class de calcul d'une réduction sur un livre
class BookDiscountCalculator
{
    // ⚠️ Mauvaise pratique : cette fonction doit obligatoirement être modifier en cas d'ajout d'un nouveau type de réduction 
    public function calculateDiscount(Book $book, string $bookType): float
    {
        if ($bookType === Book::BOOK_TYPE_EBOOK) {
            // Ebooks : 15% de réduction
            return $book->getPrice() * Book::BOOK_TYPE_DISCOUNT_EBOOK; 
        } elseif ($bookType === Book::BOOK_TYPE_LIMITED_EDITION) {
            // Éditions limitées : 20% de réduction
            return $book->getPrice() * Book::BOOK_TYPE_DISCOUNT_LIMITED_EDITION; 
        }
        return 0;
    }
}

// Création du livre
$book = new Book('Clean Code', 30.0);

// Calcul de la réduction pour un ebook
$calculator = new BookDiscountCalculator();
$discount = $calculator->calculateDiscount($book, Book::BOOK_TYPE_EBOOK);
echo "Réduction pour un ebook : $discount" . PHP_EOL;

// Calcul de la réduction pour un livre en édition limitée
$discount = $calculator->calculateDiscount($book, Book::BOOK_TYPE_LIMITED_EDITION);
echo "Réduction pour un livre en édition limitée : $discount" . PHP_EOL;
```

### Les problèmes

La classe `BookDiscountCalculator` doit être modifiée à chaque fois qu'un nouveau type de livre (ou une nouvelle logique de réduction) est introduit. Cela ne respecte pas le principe OCP, car nous devons modifier du code existant pour ajouter une nouvelle fonctionnalité.

## ✅ Avec le principe de l'OCP

```php
<?php

// Class d'entité livre
class Book
{
    private string $title;
    private float $price;

    public function __construct(string $title, float $price)
    {
        $this->title = $title;
        $this->price = $price;
    }

    public function getTitle(): string
    {
        return $this->title;
    }

    public function getPrice(): float
    {
        return $this->price;
    }
}

// Interface de stratégie de réduction
interface DiscountStrategy
{
    public function calculateDiscount(Book $book): float;
}

// Class concrète de réduction pour un ebook 
class EbookDiscount implements DiscountStrategy
{
    const DISCOUNT = 0.15;

    public function calculateDiscount(Book $book): float
    {
        // Ebooks : 15% de réduction
        return $book->getPrice() * self::DISCOUNT;
    }
}

// Class concrète de réduction pour un livre en édition limitée
class LimitedEditionDiscount implements DiscountStrategy
{
    const DISCOUNT = 0.20;

    public function calculateDiscount(Book $book): float
    {
        // Éditions limitées : 20% de réduction
        return $book->getPrice() * self::DISCOUNT;
    }
}

// Création du livre
$book = new Book('Clean Code', 30.0);

// Calcul de la réduction pour un ebook
$ebookDiscount = new EbookDiscount();
$discount = $ebookDiscount->calculateDiscount($book);
echo "Réduction pour un ebook : $discount" . PHP_EOL;

// Calcul de la réduction pour un livre en édition limitée
$limitedEditionDiscount = new LimitedEditionDiscount();
$discount = $limitedEditionDiscount->calculateDiscount($book);
echo "Réduction pour un livre en édition limitée : $discount" . PHP_EOL;
```

!!! note

    Au vue des similarités dans le code des classes `EbookDiscount` et `LimitedEditionDiscount`, celles-ci pourraient être refactorisées de manière à mutualiser la fonction de calcul de discount, à condition que celle-ci respecte les mêmes règles pour toutes les méthodes de calcul.

### Explications

Dans cette version, les différentes stratégies de calcul de réduction sont séparées dans des classes distinctes qui implémentent l'interface `DiscountStrategy` : `EbookDiscount` et `LimitedEditionDiscount`.

En cas d'ajout d'un nouveau type de réduction, il suffira d'ajouter une classe implémentant l'interface `DiscountStrategy` sans modifier les classes existantes.

Le code est donc : 

- Ouvert à l'extension : Ajout de nouveaux types de réduction, mais...
- Fermé à la modification : Pas besoin de modifier le code existant pour ajouter de nouvelles fonctionnalités.

## Solution générique

- Créer une interface pour définir une/des méthodes communes par typologie.
- Créer des classes spécifiques pour chaque typologie, implémentant cette interface.
- Possibilité d'ajout de nouvelles typologies en créant de nouvelles classes, sans modification du code existant.

## Avantages de l'OCP

- Code plus clair et plus lisible.
- Beaucoup plus maintenable grâce à l'implémentation de l'interface.