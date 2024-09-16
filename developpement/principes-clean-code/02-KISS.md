---
title: "02 - KISS, Keep It Simple, Stupid" 
---

# KISS : Keep It Simple, Stupid

## Principes clés

!!! quote "Principe"

    Principe qui favorise la simplicité de conception et l'évitement de toute complexité inutile pour aller à l'essentiel.

**Objectifs :**

- Lisibilité du code : le code et plus simple, plus claire et plus intuitif.
- Dimunition des erreurs par les membres de l'équipe.
- Maintenabilité/évolutivité du code accrue.

## Exemple

**⚠️ Code sans le principe KISS**

Dans le code ci-dessous, on recréer une fonction native du langage php pour éffectuer la somme de tous les éléments d'un tableau.

```php
<?php

function calculateSum(array $numbers): int
{
    $sum = 0;
    $i = 0;
    
    while ($i < count($numbers)) {
        $sum += $numbers[$i];
        $i++;
    }
    
    return $sum;
}

$numbers = [1, 2, 3, 4, 5];
echo calculateSum($numbers); // Résultat : 15
```

**✅ Code qui suit le principe KISS**

Il suffit en fait simplement de vérifier dans la [documentation](https://www.php.net/manual/en/function.array-sum.php) qu'une fonction de ce type existe.

```php
<?php

echo array_sum($[1, 2, 3, 4, 5]); // Résultat : 15
```
