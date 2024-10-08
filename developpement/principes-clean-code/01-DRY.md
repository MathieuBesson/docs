---
title: "01 - DRY, Don't Repeat Yourself"
author :
    name : Mathieu BESSON
    linkedin : mathieubesson
publication_date: 5/09/2024
---

# DRY : Don't Repeat Yourself

## Principes clés

!!! quote "Principe"

    Principe qui consiste à éviter la redondance de code de manière à faciliter la maintenance, le test, le débogage et les évolutions du code source.

**Objectifs :**

- Éviter les répétitions de code dans un code source
- Optimisation du code
- Meilleur lisibilité
- Éviter les oublis
- Meilleur productivité

**Moyens :**

- Abstraction : Diminuer le nombre d'opérations logiques et les regrouper au maximum
- Normalisation des données : Proposer une standardisation pour regrouper les éléments par corrélation

## Exemple

**⚠️ Code sans le principe DRY :**

```php
<?php

class Book {
    protected int $id;
    protected string $reference;
    protected string $title;
    protected string $author;
    protected array $realisator;
}

// ⚠️ Mauvaise pratique : Duplication de propriétés
class Movie {
    protected int $id;
    protected string $reference;
    protected string $title;
    protected string $author;
    protected array $mainActors;
}
```

**✅ Code qui suit le principe DRY :**

```php
<?php

class Work {
    protected int $id;
    protected string $reference;
    protected string $title;
    protected string $author;
}

class Book extends Work {
    protected string $realisator;
}

class Movie extends Work {
    protected array $mainActors;
}
```

Ici nous avons pu regrouper les propriétés de `Book` et `Movie` vers la class `Work`, ce qui permet de limiter la duplication de code.