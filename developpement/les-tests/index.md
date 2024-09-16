---
title: Introduction
---

# Les tests ? késako ? 

En développement informatique, les **tests** sont des moyens efficaces permettant de vérifier qu'un script ou une application fonctionne comme initialement prévue. L'utilisation de tests permet d'éviter d'introduire des régréssions à la modification du code source et d'éviter les bugs ou comportements innatendus. 

!!! warning "Note importante"

    Les exemples ci-dessous sont écrit en pseudo-code et ont pour objectif de mieux comprendre le déroulé et objectifs des différents types de tests présentés. Le mot clé `assert` induit une vérification entre la valeur attendue et la valeur retournée par le script. 
    
    Dans une véritable application PHP, il est recommandé d'utiliser des frameworks de tests comme [PHPUnit](https://phpunit.de) (**tests unitaire** et **tests d'intégration**) et [Behat](https://docs.behat.org/) (ou [Cypress](https://www.cypress.io/)) (**tests end-to-end**)

## Les différents types de tests

Il existe plusieurs types de tests pour vérifier le bon fonctionnement d'un logiciel. Les sections ci-dessous présentent quelques types de tests courants.

### Tests Unitaires (TU)

Les **tests unitaires** sont les tests les plus basiques. L'objectif est de vérifier le bon fonctionnement d'unités de code (fonctions ou méthodes) de manière individuel et isolée.

#### Caractéristiques

- **Objectif** : Vérifier le bon fonctionnement de chaque unité de code, indépendamment des autres parties du système.
- **Portée** : Très limitée, test de chaque unité de code indépendemment.
- **Simplicité** : Rapides à exécuter et facilement automatisable.


#### Exemple

**Cas de test** : Test d'une fonction qui additionne deux nombres. Le (ou les) test(s) unitaire vérifiera que la fonction retourne le bon résultat pour différentes combinaisons d'entrées.

```python
test_unit "Test add function" :
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0
```

### Tests d'Intégration (TI)

Les **tests d'intégration** permettent de vérifier que plusieurs modules ou composants fonctionnent ensemble comme prévu. Il ne s'agit plus du test d'une unité isolé (TU) mais bien du test de la combinaison des différents modules.

#### Caractéristiques

- **Objectif** : Tester l'interaction entre plusieurs modules/parties du logiciel.
- **Portée** : Plus large que les TU, car ils impliquent plusieurs modules.
- **Niveau de complexité** : Plus complexes à écrire que les TU, car ils nécessitent de configurer plusieurs composants et leurs interactions.

#### Exemple

**Cas de test** : Test de la persistence d'un payload dans une base de donnée, à travers une requête de l'API.

```python
test_integration "Test API payload persistence in database" :

    # Step 1: Setup the initial state
    database = initialize_database()
    api = initialize_api()

    # Step 2: Create the payload to send via the API
    payload = {
        "name": "Pierre DUPOND",
        "email": "pierre.dupond@example.com"
    }

    # Step 3: Send the payload via the API
    response = api.post("/users", payload)

    # Step 4: Assert the API response is successful
    assert response.status_code == 201  # HTTP status code for "Created"

    # Step 5: Retrieve the data directly from the database
    user_db = database.query("SELECT * FROM users WHERE email = 'pierre.dupond@example.com'")

    # Step 6: Assert that the data in the database matches the payload
    assert user_db.name == "Pierre DUPOND"
    assert user_db.email == "pierre.dupond@example.com"
```

### Tests End-to-End (E2E)

Les **tests end-to-end** (ou **tests de bout en bout**) simulent des scénarios d'utilisation complets de l'application, comme un utilisateur final. Ils vérifient que toutes les parties du système (front-end, back-end, base de données, etc.) fonctionnent ensemble correctement, dans l'environnement réel ou proche de la production.

#### Caractéristiques

- **Objectif** : Tester que toutes les fonctionnalités du système fonctionnent comme prévu, de bout en bout, dans un contexte réaliste.
- **Portée** : Très large, couverture complète de l'application.
- **Complexité** : Difficile à mettre en place et à maintenir à cause des nombreuses interactions à gérer (bases de données, API, interfaces utilisateur, etc.).

#### Exemple

**Cas de test** : Test sur un blog pour vérifier qu'un utilisateur peut publier un article.

```python
test_e2e "Test publish article" :
    user = login("pierre.dupond@example.com", "password")

    create_article_page = go_to_create_article()
    create_article_page.fill_title("Les tests")
    create_article_page.fill_content("Tester c'est douter.")
    
    result = create_article_page.publish()

    assert result == "Article published successfully"
    
    article_page = go_to_article("Les tests")
    assert article_page.title == "Les tests"
    assert article_page.content == "Tester c'est douter."
```

## Conclusion

Chaque type de test a son objectif dans le développement logiciel :

- Les **tests unitaires (TU)** vérifient la fiabilité des plus petites unités de code.
- Les **tests d'intégration (TI)** vérifient l'intégration entre les différents modules.
- Les **tests end-to-end (E2E)** vérifient le bon comportement de l'application du point de vue de l'utilisateur final.

Ces tests, lorsqu'ils sont combinés, garantissent la fiabilité et la stabilité d'un logiciel.