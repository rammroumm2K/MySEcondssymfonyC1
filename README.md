# MySecondSymfonyC1

Installation de la version lts (Long Term Support) avec la majorité des bibliothèques pour un site web (`--webapp`)

    symfony new MySecondSymfonyC1 --webapp --version=lts

en cas d'oubli de --webapp

    cd MySecondSymfonyC1
    composer require webapp

## Mise à jour des versions de sécurités

    composer update

## Lancement du serveur

    symfony serve -d
ou

    symfony server:start -d

Pour le fermer 

    symfony server:stop

L'adresse est généralement de type https://127.0.0.1:8000

### Création d'un contrôleur

    symfony console make:controller

ou

    php bin/console make:controller

Le nom doit être en PascalCase terminé par Controller, mais Symfony se charge de le corriger en cas d'oubli.

    php bin/console make:controller HomeController
    created: src/Controller/HomeController.php
    created: templates/home/index.html.twig

On va vérifier la route par défaut

    php bin/console debug:route

#### Modification de la route

```php
// src/Controller/HomeController.php

# ...

    #[Route('/', name: 'homepage')]
    public function index(): Response
    {
        return $this->render('home/index.html.twig', [
            'title' => 'Homepage',
        ]);
    }
# ...
```

On peut accéder à l'accueil depuis la racine de notre

https://127.0.0.1:8000

#### Modifications de `HomeController`

Pour obtenir 2 pages, homepage et about

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class HomeController extends AbstractController
{
    #[Route('/', name: 'homepage')]
    public function index(): Response
    {
        return $this->render('home/index.html.twig', [
            'title' => 'Homepage',
        ]);
    }
    #[Route('/about', name: 'about_me')]
    public function aboutMe(): Response
    {
        return $this->render('home/about.html.twig', [
            'title' => 'About me',
        ]);
    }
}

```

#### Modification de base.html.twig

```twig
{# templates/base.html.twig #}
{# ... #}
<title>{% block title %}MySecondSymfonyC1{% endblock %}</title>
{# ... #}
```

#### Création de menu.html.twig

Nous utilisons path() pôur les liens vers les noms de routes pour pouvoir les changer à un seul endroit : `src/controller`

templates/home/menu.html.twig

```twig
<nav>
    {# on utilise path('nom_du_chemin') lorsqu'on veut un lien vers une page #}
    <a href="{{ path('homepage') }}">Homepage</a>
    <a href="{{ path('about_me') }}">About me</a>
</nav>
```

#### Modification de index.html.twig


```twig
{% extends 'base.html.twig' %}

{# on surcharge le block title avec {{ parent }}
    et le titre passé par le contrôleur
#}
{% block title %}{{ parent() }} | {{ title }}{% endblock %}


{% block body %}
    <div class="container">
        <h1>{{ title }}</h1>
{# inclusion depuis la racine du projet ! (templates) #}
{% include 'home/menu.html.twig' %}
    </div>

{% endblock %}
```

About est similaire.