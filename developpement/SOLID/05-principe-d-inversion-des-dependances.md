---
title: 05 - Dependency Inversion Principle
---

# Dependency Inversion Principle (DIP), Inversion des dépendances

Une classe doit dépendre de son abstraction, pas de son implémentation.

**Détail :**

- Éssayer au maximum de typer des interfaces en paramètre plutôt que des objets directement. 
- Les dépendances entre classes doivent se faire à travers des abstractions.

## ⚠️ Code qui ne suit pas le principe de DIP

Dans l'exemple de code suivant, la classe `ReportGenerator` est directement dépendante de la classe `PDFExporter`, donc toute modification de la classe concrète `PDFExporter` peut affecter la classe `ReportGenerator`.

```php
<?php

// Class concrète d'export d'un PDF
class PDFExporter
{
    public function export(string $content): void 
    {
        // Export du contenu en PDF
    }
}

// Class concrète de génération d'un rapport
class ReportGenerator
{
    private $exporter;

    // ⚠️ Mauvaise pratique : Couplage fort entre class concrètes ReportGenerator et PDFExporter
    public function __construct(PDFExporter $exporter)
    {
        $this->exporter = $exporter;
    }

    public function generateReport(): void
    {
        $content = "Report Content";
        $this->exporter->export($content);
    }
}
```

### Pourquoi ce code ne suit pas le principe de DIP ?

la classe `ReportGenerator` apporte une dépendance direct avec `PDFExporter` introduisant un couplage fort entre ces deux classes concrètes. 

L'objectif est de diminuer ce couplage en replaçant la dépendance concrète par une dépendance d'abstraction. 

## 🔝 Code qui suit le principe de DIP

```php
<?php

// Interface des différents exporteurs
interface Exporter
{
    public function export(string $content): void;
}

// Class concrète d'export d'un PDF
class PDFExporter implements Exporter
{
    public function export(string $content): void
    {
        // Export du contenu en PDF
    }
}

// Class concrète de génération d'un rapport
class ReportGenerator
{
    private $exporter;

    // 🔝 Bonne pratique : Couplage faible avec une interface (= abstraction)
    public function __construct(Exporter $exporter)
    {
        $this->exporter = $exporter;
    }

    public function generateReport(): void
    {
        $content = "Report Content";
        $this->exporter->export($content);
    }
}
```

### Comment ce code repecte le principe de DIP ?

- Cette modification réduit le couplage entre les différentes classes.
- `ReportGenerator` dépend maintenant de l'interface `Exporter`, et plus de la classe concrète `PDFExporter`. 
- Il est maintenant possible d'ajouter d'autres types d'exporter sans modifier `ReportGenerator`.

## Solution générique

- Utiliser des interfaces/abstractions pour éviter le couplage fort entre classes concrètes (contrat > implémentation concrète).
- Utiliser l'injection de dépendance pour fournir les implémentations concrètes des abstractions.
- Utiliser la la composition plutôt que l'héritage, et veiller à la séparation des responsabilités.

## Avantages de l'utilisation du principe de Dependency Inversion Principle (DIP)

- Réduction du couplage fort entre les implémentation et les abstractions.
- Le code est plus flexible, plus lisible et moins interdépendant.