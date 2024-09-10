---
title: Interface Segregation Principle (ISP)
---

# I pour ISP : Interface Segregation Principle (Ségrégation des interfaces)

Un client ne doit être forcé d'implémenter des méthodes qu'il n'utilise pas.

**Détail :**

- Il faut préférer faire plusieurs interfaces (avec des rôles spécifiques) qu’une unique fourre-tout.
- Une interface ne devrait contenir que les méthodes nécessaires aux classes qui l'implémentent.
- Une classe implémentant une interface ne devrait être obliger d'implémenter une méthode dont elle n'a pas besoin.

## ⚠️ Code qui ne respecte pas le principe d'ISP :

Le code ci-dessous contient une interface large `Document` qui nécéssite l'implémentation de nombreuses méthodes pour les classes `TextDocument` et `PDFDocument`.

```php
<?php

// Interface d'un Document
interface Document
{
    public function open(): void;
    public function save(): void;
    public function print(): void;
    public function convertToPDF(): void;
}

// Class concrète d'un document au format text
class TextDocument implements Document
{
    // Méthodes cohérentes avec l'interface
    public function open(): void {}
    public function save(): void {}
    public function print(): void {}

    // ⚠️ Mauvaise pratique : Méthode incohérente avec l'interface
    public function convertToPDF(): void
    {
        throw new Exception("Text documents do not support PDF conversion.");
    }
}

// Class concrète d'un document au format PDF
class PDFDocument implements Document
{
    // Méthodes cohérentes avec l'interface
    public function open(): void {}
    public function print(): void {}

    // ⚠️ Mauvaise pratique : Méthodes incohérentes avec l'interface
    public function save(): void
    {
        throw new Exception("PDF documents cannot be saved.");
    }

    public function convertToPDF(): void
    {
        throw new Exception("PDF documents are already in PDF format.");
    }
}
```

### Pourquoi ce code ne suit pas le principe d'ISP ?

Les classes `TextDocument` et `PDFDocument` doivent implémenter des méthodes qui n'ont pas de sens avec leur rôle, seulement dans le but de respecter le contrat avec l'interface `Document`: 

- `TextDocument` : n'a pas besoin de la méthode `convertToPDF()`.
- `PDFDocument` : n'a pas besoin des méthodes `save()` et `convertToPDF()`.

## 🔝 Code qui suit le principe d'ISP :

```php
<?php

// Interface d'un Document ouvrable
interface Openable
{
    public function open(): void;
}

// Interface d'un Document suavegardable
interface Savable
{
    public function save(): void;
}

// Interface d'un Document imprimable
interface Printable
{
    public function print(): void;
}

// Interface d'un Document PDF convertible
interface PDFConvertible
{
    public function convertToPDF(): void;
}

// Class concrète d'un document au format text (avec uniquement les méthodes nécéssaire)
class TextDocument implements Openable, Savable, Printable
{
    public function open(): void {}
    public function save(): void {}
    public function print(): void {}
}

// Class concrète d'un document au format PDF (avec uniquement les méthodes nécéssaire)
class PDFDocument implements Openable, Printable, PDFConvertible
{
    public function open(): void {}
    public function print(): void {}
    public function convertToPDF(): void {}
}
```

### Comment ce code repecte le principe d'ISP ?

- Les interfaces ne traite qu'un seul rôle.
- Les classes sont donc libre d'implémenter ou non ces interfaces en fonction du besoin et de la pertinence.

## Solution générique

- Diviser les interfaces importantes et globales en interfaces plus petites, chacune ayant à une responsabilité unique. Les classes n'implémentent que les interfaces pertinentes pour elles.
- Créer des interfaces en fonction des besoins des classes qui les implémenteront. Les interfaces ne doivent inclure que les méthodes nécessaires à ces clients (classes).
- Éviter l'implémentation d'interface inutiles : implémentations vides ou non pertinentes pour les classes 

## Avantages de l'utilisation du principe de Ségrégation des interfaces (ISP)

- Code plus modulable et plus lisible