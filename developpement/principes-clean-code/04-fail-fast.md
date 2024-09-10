---
title: "04 - Fail Fast"
---

# Principe Fail-Fast

### Principes clés

!!! quote "Principe"

    Principe qui conseil de signaler le plus rapidement possible une erreur, en cas d'échec, autant le faire le plus vite possible.
    
**Objectifs :**

- Détection précoce des erreurs et évitement des problèmes en aval.
- Réduction des comportements inattendus.
- Facililte le debugage.
- Augmente la robustesse du programme. 

### Exemple :

**⚠️ Code sans le principe Fail-Fast**

Ici, on suppose

```php
<?php

function readFileContent(string $filePath): string
{
    $fileContent = file_get_contents($filePath);
    // ⚠️ Mauvaise pratique : Supposition d'existence et de lisibilité du fichier sans vérification
    return $fileContent;
}


// Génerera une erreur si le fichier n'existe pas
echo readFileContent("file.txt"); 
```

**✅ Code qui suit le principe Fail-Fast**

```php
<?php

function readFileContent(string $filePath): string
{
    if (!file_exists($filePath)) {
        throw new RuntimeException("Le fichier spécifié n'existe pas : $filePath");
    }

    $fileContent = file_get_contents($filePath);
    return $fileContent;
}


try {
    // Lance une exception si le fichier n'existe pas
    echo readFileContent("nonexistent_file.txt"); 
} catch (RuntimeException $e) {
    // Gestion spécifique de l'erreur
    echo $e->getMessage();
}
```
