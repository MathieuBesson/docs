---
title: "03 - YAGNI, You Ain't Gonna Need It"
author :
    name : Mathieu BESSON
    linkedin : mathieubesson
publication_date: 5/09/2024
---

# YAGNI : You Ain't Gonna Need It

## Principes clés

!!! quote "Principe"

    Ce principe encourage à ne pas implémenter des fonctionnalités ou écrire du code qui ne sont pas nécessaires au moment actuel, et donc ne pas anticiper des besoins futures sans qu'ils ne soient strictement requis.

!!! note

    Attention, cependant il reste important d'architecturer son code de manière à faciliter l'évolution de celui-ci, notamment à travers la mise en place des [principes SOLID](SOLID/index.md).

**Objectifs :**

- Réduire la complexité inutile.
- Améliorer la maintenabilité et la flexibilité.

## Exemple

**⚠️ Code sans le principe YAGNI**

Dans la class ci-dessous `Task`, deux propriétés sont inutiles et une fonction vide avec un `TODO` (probablement là depuis la nuit des temps...), ce qui peut perturber la lisibilité du code.

```php
<?php

class Task
{
    private string $name;
    private bool $isComplete = false;
    // ⚠️ Mauvaise pratique : Ces deux champs sont ici mais pas encore nécéssaire, donc inutiles pour le moment
    private bool $isRecurring;
    private ?int $recurrenceInterval;

    public function __construct(string $name)
    {
        $this->name = $name;
        $this->isRecurring = false;
        $this->recurrenceInterval = null;
    }

    public function completeTask(): void
    {
        $this->isComplete = true;
    }

    public function giveNextInterval(): void
    {
        // TODO : To implement
    }
}

// Utilisation
$task = new Task("Find a job");
$task->completeTask();
```

**✅ Code qui suit le principe YAGNI**

Après suppression du code dit "mort" (les propriétés et la méthode), le code source ne contient que du code utile au programme.

```php
<?php

class Task
{
    private string $name;
    private bool $isComplete = false;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    public function completeTask(): void
    {
        $this->isComplete = true;
    }
}

// Utilisation
$task = new Task("Find a job");
$task->completeTask();
```
