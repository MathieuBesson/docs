---
title: Introduction
---

# Les tests avec Symfony

Le framework de test PHPUnit permet de tester des application à l'aide de **tests unitaires**. Cependant, les tests unitaires ne permettent malheureusement pas de tester l'ensemble d'une application. Dans le cas d'une application web complexe, par exemple, il est nécéssaire de vérifier que : 

- Les intéractions entre les différents modules fonctionnent comme attendu (**tests d'intégration**).
- Les fonctionnalités principales répondent toujours aux exigences (**tests fonctionels**).
- Les parcours de l'utilisateur final et ses comportements habituels sur l'application sont bien fonctionnels (**tests end-to-end**).

## Outils de tests de Symfony

Dans cette optique, Symfony propose plusieurs types de tests pour répondre aux différents besoins :

| Type de test           | Classe Symfony ou PHPUnit             | Dossier                      | Test du JS | Rapidité |
| :--------------------- | :------------------------------------ | :--------------------------- | ---------- | -------- |
| **Test unitaire**      | `TestCase (PHPUnit)`                  | `test/Unit`                  | ❌         | 🚀       |
| **Test d'intégration** | `KernelTestCase (Symfony)`            | `test/Integration`           | ❌         | 🚗       |
| **Test fonctionnel**   | `WebTestCase & ApiTestCase (Symfony)` | `test/Functional & test/Api` | ❌         | 🛵       |
| **Test end-to-end**    | `PantherTestCase (Panther)`           | `test/E2E`                   | ✅         | 🛴       |

!!! info "Architecture de dossiers"

    En général dans une application Symfony, l'architecture de dossier des tests au delà de `test/TypeTest` garde l'architecture des classes testés de l'application à partir du dossier `src/`.


Les classes `KernelTestCase`, `WebTestCase`, `ApiTestCase` incluses dans le bundle `symfony/test-pack`,  `PantherTestCase` inclus dans `symfony/panther`, ainsi que `TestCase` de `PHPUnit`, aident à structurer et à simplifier l'écriture de tests en fonction des besoins spécifiques d'une application Symfony.

- **`TestCase`** : Classe de base fournie par PHPUnit, elle est utilisée pour écrire des tests unitaires en permetttant de faire des assertions, des mocks et de vérifier le comportement des unités de code de manière isolée.
- **`KernelTestCase`** : Classe permettant d'accéder au conteneur de services et de tester le comportement de l'application dans un environnement proche de la production, facilitant ainsi les **tests d'intégration**.
- **`WebTestCase`** : Héritée de `KernelTestCase`, elle est destinée aux tests d'applications web. Elle permet d'envoyer des requêtes HTTP et d'analyser les réponses, en vérifiant l'interaction entre le client et le serveur.
- **`ApiTestCase`** : Classe utilisé pour le test d'API, elle permet de gérer les requêtes et les réponses JSON, et de s'assurer que les points de terminaison API fonctionnent comme prévu.
- **`PantherTestCase`** : Classe conçue pour les tests fonctionnels basés sur un navigateur. Elle utilise un navigateur pour exécuter les tests, permettant ainsi de simuler des interactions utilisateur réelles avec l'application web, notamment en testant des fonctionnalités JavaScript.

## Configuration pour les tests

### Configuration et variables d'env

Avant de pouvoir écrire et exécuter des tests d'intégration avec Symfony et PHPUnit, il est nécéssaire de configurer l'environnement de test (isolé de l'environnement de développement et de production).

Le fichier `.env.test` permet de configurer les variables d'environnement dans le context d'éxécution des tests.

!!! info "Surcharge local"

    Il est aussi possible de surcharger la configuration localement pour la machine grâce au fichier `.env.test.local`

Exemple de fichier `.env.test`

```shell
APP_ENV=test
APP_DEBUG=1

# Configuration de la base de données pour les tests
DATABASE_URL="mysql://user:password@127.0.0.1:3306/db_test?serverVersion=5.7"
```

Depuis Symfony `5.3`, il est possible de configurer différemment les bundles en fonction de l'environnement d'éxécution de l'application avec la clé `when@test`.
Le détail des différentes possibilités de configuration de tests est disponible [ici](https://symfony.com/blog/new-in-symfony-5-3-configure-multiple-environments-in-a-single-file)


Exemple avec le bundle `webpack-encore` :

```yaml
webpack_encore:
    output_path: '%kernel.project_dir%/public/build'
    strict_mode: true
    cache: false

when@prod:
    webpack_encore:
        cache: true

when@test:
    webpack_encore:
        strict_mode: false
```

### BDD de test

Pour exécuter des tests d'intégration stables, il est nécéssaire d'avoir un environnement de test stable (jeux de données, BDD...).

Dans cette optique, les fixtures sont un outil très utile apportés avec le package [`orm-fixtures`](https://symfony.com/bundles/DoctrineFixturesBundle/current/index.html).

#### Étapes de mise en place

De manière à obtenir un envrironnement de test totalement identique à chaque lancement des tests, il est recommandé de :

- Détruire la BDD de test déjà présente
- Créer la nouvelle base de donnée de test
- Lancer les migrations
- Lancer les fixtures

Le Makefile suiavnt permet de simplifier ces étapes par l'utilisation d'une unique commande make : `make test`

```makefile
tests:
	symfony console doctrine:database:drop --force --env=test || true
	symfony console doctrine:database:create --env=test
	symfony console doctrine:migrations:migrate -n --env=test
	symfony console doctrine:fixtures:load -n --env=test
	symfony php bin/phpunit $(MAKECMDGOALS)
.PHONY: tests
```

#### Outil de transaction

Au lancement des tests, ceux-ci sont lancés les uns après les autres. Ainsi, l'exécution du premier test peut influer sur les résultats du second en fonction de leur ordre dans la séquence de lancement. 

Le bundle [`dama/doctrine-test-bundle`](https://github.com/dmaicher/doctrine-test-bundle) permet de pallier à ce problème. 
Ainsi, toutes les requêtes faites sur la base de donnée lors des tests est automatiquement effectuée sous forme de transaction, permettant ainsi de ne pas modifier réellement l'état de la base de donnée et donc de ne pas influer sur les autres tests.

## La suite ? 🚀

- [Les tests unitaires en PHP](./../01-tests-unitaire-php.md)
- [Les tests d'intégration avec Symfony](./01-tests-integration-symfony.md)
-  À venir : 
    - Les tests fonctionnels avec Symfony
    - Les tests end-to-end avec [Panther](https://github.com/symfony/panther)