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

## Création de notre `.env.local`

Le fichier `.env` est le fichier de configuration qui est mis sur `git` et donc `github`

C'est pour celà que nous allons le copier sous le nom de `.env.local`

    cp .env .env.local

Ouvrez `.env.local`

Changez cette ligne

    APP_ENV=dev
    APP_SECRET=c6f06c078199d1f00879e1b9c146cddf
en

    APP_ENV=prod
    APP_SECRET=une_autre_clef_secrete_sécurité

si vous retapez  `php bin/console debug:route`

Vous ne trouverez plus que les routes de production

Dans le fichier `.env.local`

Trouvez la ligne de base de données :

```bash
# ne pas oublier de remettre en dev
APP_ENV=dev

# DATABASE_URL="mysql://app:!ChangeMe!@127.0.0.1:3306/app?serverVersion=8.0.32&charset=utf8mb4"
# DATABASE_URL="mysql://app:!ChangeMe!@127.0.0.1:3306/app?serverVersion=10.11.2-MariaDB&charset=utf8mb4"
DATABASE_URL="postgresql://app:!ChangeMe!@127.0.0.1:5432/app?serverVersion=16&charset=utf8"
```

Commentez la ligne postgresql et décommentez la ligne mysql

Passez vos paramètres de connexion dans l'ordre

utilisateur:mot_de_passe@ip_serveur:port/nomdelaDB?options

```bash
DATABASE_URL="mysql://root:@127.0.0.1:3306/mysecondesymfonyc1?serverVersion=8.0.31&charset=utf8mb4"
# DATABASE_URL="mysql://app:!ChangeMe!@127.0.0.1:3306/app?serverVersion=10.11.2-MariaDB&charset=utf8mb4"
# DATABASE_URL="postgresql://app:!ChangeMe!@127.0.0.1:5432/app?serverVersion=16&charset=utf8"
```

## Création de la DB

    php bin/console doctrine:database:create

La base de donnée devrait être créée si mysql.exe est activé ou Wamp démarré 

## Création d'une entité

Une entité est la représentation objet d'un élément de sauvegarde de données, dans notre cas, en choisissant mysql, il s'agira d'une table

    php bin/console make:entity

```bash
 php bin/console make:entity

 Class name of the entity to create or update (e.g. FierceGnome):
 > Article
Article

 Add the ability to broadcast entity updates using Symfony UX Turbo? (yes/no) [no]:
 > no

 created: src/Entity/Article.php
 created: src/Repository/ArticleRepository.php

 New property name (press <return> to stop adding fields):
 >
  Success!
  

 php bin/console make:entity

 Class name of the entity to create or update (e.g. GrumpyChef):
 > Article
Article

 Your entity already exists! So let's add some new fields!

 New property name (press <return> to stop adding fields):
 > title

 Field type (enter ? to see all types) [string]:
 >


 Field length [255]:
 > 160

 Can this field be null in the database (nullable) (yes/no) [no]:
 >

 updated: src/Entity/Article.php

text

```

## Première migration

    php bin/console make:migration

    success :  created: migrations/Version20240903142811.php

puis

    php bin/console doctrine:migrations:migrate

## Ajout de champs à l'entité `Article`

On utilise maker pour ça

    php bin/console make:entity Article

```php
// src/Entity/Article.php


#...

#[ORM\Entity(repositoryClass: ArticleRepository::class)]
class Article
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 160)]
    private ?string $title = null;

    #[ORM\Column(type: Types::TEXT)]
    private ?string $text = null;

    #[ORM\Column(type: Types::DATETIME_MUTABLE, nullable: true)]
    private ?\DateTimeInterface $date_created = null;

    #[ORM\Column(type: Types::DATETIME_MUTABLE, nullable: true)]
    private ?\DateTimeInterface $date_published = null;

    #[ORM\Column(nullable: true)]
    private ?bool $published = null;

    # ... getters and setters sauf pour les booleans

    // ! pour boolean, is est rajouté pour le getter
    // bug si on avait choisi isPublished -> isPublished
    public function isPublished(): ?bool
    {
        return $this->published;
    }
    
    // Le setter peut boguer si on avait choisi comme nom
    // isPublished -> setPublished
    public function setPublished(?bool $published): static
    {
        $this->published = $published;

        return $this;
    }
}

```