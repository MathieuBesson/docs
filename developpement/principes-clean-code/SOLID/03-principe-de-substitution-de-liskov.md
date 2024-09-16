---
title: 03 - Liskov’s Substitution Principle
---

# Liskov’s Substitution Principle (LSP), Substitution de Liskov

!!! quote "Principe"

    Les objets doivent être remplaçables par des instances de leurs sous-types sans altérer le fonctionnement correct du système.

**Détail :**

- Les objets d'une classe dérivée (par héritage ou interface) doivent pouvoir remplacer la classe de base sans alterer le fonctionnement du programme. 
- Les signatures des fonctions doivent être identiques (en types et en nombre) entre classe parente et enfant.
- Les exceptions levées doivent être identiques. 

## ⚠️ Sans le principe du LSP

L'exemple suivant contient quelques mauvaises pratiques correspondants au non repect du principe de LSP.

Dans ce code, la classe `Car` détient beaucoup de spécifictés, notamment lié à un véhicule fonctionnant avec du carburant (`fuel` et `refuel()`). 

La classe `ElectricCar` étend la classe `Car` pour avoir accès à la fonction `drive()`, cependant du à la contrainte d'un véhicule à carburant induit dans la classe parente, la classe `ElectricCar` ne peut logiquement pas implémenter la fonction `refuel()` et doit donc lancer une Exception.

```php
<?php

// Class générale pour une voiture classique
class Car
{
    protected int $fuel;

    public function __construct(int $fuel)
    {
        $this->fuel = $fuel;
    }

    public function drive(): string
    {
        if ($this->fuel <= 0) {
            return "Can't drive, out of fuel!";
        }
        $this->fuel--;
        return "Driving...";
    }

    public function refuel(int $amount): void
    {
        $this->fuel += $amount;
    }
}

// Class spécifique pour une voiture électrique
class ElectricCar extends Car
{
    public function __construct()
    {
        // Les voitures électriques n'utilisent pas de carburant, donc le constructeur est vide
        // ⚠️ Mauvaise pratique : Mais cela n'a pas de sens pour une voiture électrique
        $this->fuel = 0;  
    }

    // ⚠️ Mauvaise pratique : Override de la méthode refuel pour une voiture électrique
    public function refuel(int $amount): void
    {
        throw new Exception("Electric cars don't need fuel!");
    }
}

// Création d'une voiture standard à carburant
$classicCar = new Car(10);
echo $classicCar->drive() . PHP_EOL; // Fonctionne : "Driving..."

// Création d'une voiture électrique
$electricCar = new ElectricCar();
echo $electricCar->drive() . PHP_EOL; // Erreur : "Can't drive, out of fuel!" (logique cassée)
```

### Les problèmes

La classe `ElectricCar` est dérivée de `Car`, mais elle ne respecte pas le contrat de la classe `Car` dont elle hérite. Elle ne peut pas utiliser le carburant ni se comporter comme une voiture classique.

L'appel à `refuel()` dans `ElectricCar` lance une exception, ce qui ne respect pas le principe LSP. Une voiture électrique ne devrait pas hériter d'une classe qui s'attend à un comportement basé sur du carburant.

## ✅ Avec le principe du LSP

```php
<?php

// Interface pour des voitures qui peuvent conduire
interface Drivable
{
    public function drive(): string;
}

// Classe de base pour une voiture à carburant
class FuelCar implements Drivable
{
    protected int $fuel;

    public function __construct(int $fuel)
    {
        $this->fuel = $fuel;
    }

    public function drive(): string
    {
        if ($this->fuel <= 0) {
            return "Can't drive, out of fuel!";
        }
        $this->fuel--;
        return "Driving a fuel car...";
    }

    public function refuel(int $amount): void
    {
        $this->fuel += $amount;
    }
}

// Classe pour les voitures électriques
class ElectricCar implements Drivable
{
    protected int $battery;

    public function __construct(int $batteryLevel)
    {
        $this->battery = $batteryLevel;
    }

    public function drive(): string
    {
        if ($this->battery <= 0) {
            return "Can't drive, battery empty!";
        }
        $this->battery--;
        return "Driving an electric car...";
    }

    public function recharge(int $amount): void
    {
        $this->battery += $amount;
    }
}

// Création d'une voiture standard à carburant
$fuelCar = new FuelCar(10);
echo $fuelCar->drive() . PHP_EOL;  // Fonctionne : "Driving a fuel car..."

// Création d'une voiture électrique
$electricCar = new ElectricCar(5);
echo $electricCar->drive() . PHP_EOL;  // Fonctionne : "Driving an electric car..."
```

### Explications

Séparation des comportements : 

- `FuelCar` et `ElectricCar` implémentent l'interface `Drivable`, 
- Cependant chacun leurs propres mécanismes de conduite et d'alimentation (`fuel` pour `FuelCar`, `battery` pour `ElectricCar`).

Maintenant, que ce soit une voiture à essence ou une voiture électrique, on peut utiliser la fonction `drive()` en étant garanti du comportement de la fonction (signature commune, exceptions communes).

## Solution générique

- Regrouper les comportements communs dans des interfaces adaptées, plutôt que de forcer des sous-classes à hériter de méthodes non pertinentes.
- Utiliser la [composition](https://openclassrooms.com/fr/courses/1665806-programmez-en-oriente-objet-en-php/7307115-evoluez-vers-la-composition) pour combiner des fonctionnalités spécifiques au lieu d'hériter d'une classe de base qui impose des comportements inadaptés.
- Les sous-classes doivent garantir au minimum les mêmes résultats ou mieux que la classe de base, sans élimnier les comportements attendus à la base.


## Avantages du LSP

- Moins de bug liés à la mauvaise utilisation de l'héritage.
- Facilite la lisibilité et la maintenance du code.