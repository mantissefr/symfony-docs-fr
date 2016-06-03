La vue
======

Après avoir lu la première partie de ce tutoriel, vous avez décidé que Symfony2
méritait 10 minutes de plus. Excellent choix ! Dans cette seconde partie, vous 
en apprendrez plus sur le moteur de template de Symfony2, `Twig`_. Twig est un
moteur de template PHP flexible, rapide et sécurisé. Il rend vos templates plus
concis et lisibles; il les rend également plus accessibles aux web designers.

.. note::

    Plutôt que Twig, vous pouvez tout aussi bien utiliser :doc:`PHP </cookbook/templating/PHP>`
    pour vos templates. Les deux moteurs sont supportés par Symfony2.

Découvrir Twig
--------------

.. tip::

   La `documentation`_ officielle de Twig est le meilleur endroit pour approfondir ses connaissances de ce moteur de rendu. Cette section est juste un rapide aperçu des
    concepts de base.

Un gabarit Twig (template) est un fichier texte qui peut générer n'importe quel type de
contenu (HTML, CSS, Javascript, XML, CSV, LaTeX, ...). Les éléments Twig sont séparés du reste du contenu du gabarit par ces délimiteurs :

* ``{{ ... }}``: Affiche une variable ou le résultat de l'évaluation d'une expression.

* ``{% ... %}``: Contrôle la logique du gabarit; il est utilisé pour éxécuter des
  boucles ``for`` et des déclarations ``if``, entre autres.
  
* ``{# ... #}``: Permet d'inclure des commentaires dans les gabarits. Contrairement au langage HTML, les commentaires ne sont pas inclus dans le rendu du gabarit.

Ci-dessous un template minimal qui illustre quelques bases; en utilisant deux
variables ``page_title`` et ``navigation``, qui ont été passées au gabarit :

.. code-block:: html+jinja

    <!DOCTYPE html>
    <html>
        <head>
            <title>My Webpage</title>
        </head>
        <body>
            <h1>{{ page_title }}</h1>

            <ul id="navigation">
                {% for item in navigation %}
                    <li><a href="{{ item.href }}">{{ item.label }}</a></li>
                {% endfor %}
            </ul>
        </body>
    </html>


Pour interpréter un gabarit dans Symfony, utilisez la méthode ``render`` depuis un 
contrôleur et passez toutes les variables requises par le gabarit :

.. code-block:: php

    $this->render('AcmeDemoBundle:Demo:hello.html.twig', array(
        'name' => $name,
    ));

Les variables passées à un gabarit peuvent être des chaînes de caractères, des
tableaux ou même des objets. Twig les gère de la même manière et vous permet
d'accéder aux « attributs » d'une variable grâce à la notation « point » (``.``) :

.. code-block:: jinja

    {# array('name' => 'Fabien') #}
    {{ name }}

    {# array('user' => array('name' => 'Fabien')) #}
    {{ user.name }}

    {# force la consultation du tableau #}
    {{ user['name'] }}

    {# array('user' => new User('Fabien')) #}
    {{ user.name }}
    {{ user.getName }}

    {# force la consultation de méthode #}
    {{ user.name() }}
    {{ user.getName() }}

    {# transmettre des paramètres à une méthode #}
    {{ user.date('Y-m-d') }}

.. note::

    Il est important de savoir que les accolades ne font pas partie de la variable
    mais de son affichage. Si vous accéder à une variable dans un tag, ne mettez
    pas d'accolades autour.

Architecture de gabarits
------------------------

Bien souvent, les gabarits d'un projet partagent des éléments communs, comme les
célèbres en-têtes et pieds de page. Twig résoud ce problème élégamment avec le concept d' « héritage de gabarit ». Cela fonctionne exactement
comme les classes PHP : l'héritage de gabarit vous permet de définir un gabarit
de base qui contiendra tous les éléments communs de votre site puis d'inclure
des « blocs » de contenu que les gabarits enfants pourront surcharger.

Le template ``index.html.twig`` hérite du template ``base.html.twig``, grâce à la balise ``extends`` :

.. code-block:: html+jinja

    {# app/Resources/views/index.html.twig #}
    {% extends "base.html.twig" %}

    {% block content %}
        <h1>Bienvenu sur Symfony!</h1>
    {% endblock %}

Maintenant, jettons à un oeil à un exemple simple du template ``base.html.twig`` :

.. code-block:: jinja

    {# app/Resources/views/base.html.twig #}
    <!DOCTYPE html>
    <html>
        <head>
            <meta charset="UTF-8" />
            <title>{% block title %}Welcome!{% endblock %}</title>
            {% block stylesheets %}{% endblock %}
            <link rel="icon" type="image/x-icon" href="{{ asset('favicon.ico') }}" />
        </head>
        <body>
            {% block body %}{% endblock %}
            {% block javascripts %}{% endblock %}
        </body>
    </html>

La balise ``{% block %}`` définit des blocs que les gabarits enfants vont pouvoir surcharger.
Dans cet exemple, le gabarit index.html.twig surcharge le block ``body``, mais pas le bloc ``title``, qui affichera le contenu par défaut défini dans le gabarit ``base.html.twig``.

Utiliser les tags, les filtres et les fonctions
-----------------------------------------------

L'une des meilleurs fonctionnalités de Twig est son extensibilité via les tags,
les filtres et les fonctions. Symfony2 est fourni avec beaucoup de fonctions
préconstruites pour faciliter le travail des designers.

Inclure d'autres templates
~~~~~~~~~~~~~~~~~~~~~~~~~~

La meilleure manière de partager un morceau de code entre plusieurs templates
distincts est de créer un nouveau template qui sera inclus dans les autres.

Créez un template ``embedded.html.twig`` :

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/embedded.html.twig #}
    Hello {{ name }}

Et changez le template ``index.html.twig`` pour l'inclure:

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig #}
    {% extends "AcmeDemoBundle::layout.html.twig" %}

    {# override the body block from embedded.html.twig #}
    {% block content %}
        {% include "AcmeDemoBundle:Demo:embedded.html.twig" %}
    {% endblock %}

Imbriquer d'autres contrôleurs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Et si vous vouliez inclure le résultat d'un autre contrôleur dans votre template ?
C'est très utile en travaillant avec Ajax, ou quand les templates inclus
ont besoin de variables qui ne sont pas disponibles dans le template principal.

Supposez que vous avez créé une action ``fancy`` et que vous voulez l'inclure
à l'interieur du template ``index``. Pour faire cela, utilisez le tag ``render`` :

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/index.html.twig #}
    {% render "AcmeDemoBundle:Demo:fancy" with {'name': name, 'color': 'green'} %}

Ici, la chaîne de caractères ``AcmeDemoBundle:Demo:fancy`` fait référence à l'action
``fancy`` du contrôleur ``Demo``. Les arguments (``name`` et ``color``) agissent
comme des variables de requête simulée (comme si l'action ``fancyAction`` 
était gérée comme une toute nouvelle requête) et sont mis à disposition du contrôleur :

.. code-block:: php

    // src/Acme/DemoBundle/Controller/DemoController.php

    class DemoController extends Controller
    {
        public function fancyAction($name, $color)
        {
            // create some object, based on the $color variable
            $object = ...;

            return $this->render('AcmeDemoBundle:Demo:fancy.html.twig', array('name' => $name, 'object' => $object));
        }

        // ...
    }

Créer des liens entre les pages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Créer des liens entre les pages d'une application web est incontournable. Au 
lieu de coder en dur les URLs dans les templates, la fonction ``path`` peut 
générer des URLs en se basant sur la configuration du routing. De cette manière, 
toutes vos URLs peuvent être facilement mise à jour en changeant juste le fichier 
de configuration :

.. code-block:: html+jinja

    <a href="{{ path('_demo_hello', { 'name': 'Fabien' }) }}">Hello Fabien!</a>

La fonction ``path`` prend le nom de la route et un tableau de paramètres comme
arguments. Le nom de la route est la clé principale sous laquelle les
routes sont référencées et les paramètres sont les valeurs définies dans le
masque (pattern) de chaque route :

.. code-block:: php

    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    // ...

    /**
     * @Route("/hello/{name}", name="_demo_hello")
     * @Template()
     */
    public function helloAction($name)
    {
        return array('name' => $name);
    }

.. tip::

    La fonction ``url`` génère des URLs *absolues* : ``{{ url('_demo_hello', {
    'name': 'Thomas'}) }}``.

Inclure les assets: images, javascripts, et feuilles de style
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Que serait Internet sans images, javascripts, et feuilles de style ?
Symfony2 fournit la fonction ``asset`` pour les gérer très facilement :

.. code-block:: jinja

    <link href="{{ asset('css/blog.css') }}" rel="stylesheet" type="text/css" />

    <img src="{{ asset('images/logo.png') }}" />

Le but principal de la fonction ``asset`` est de rendre votre application plus
portable. Grâce à cette fonction, vous pouvez déplacer le répertoire racine
de votre application n'importe où sous le répertoire racine web sans changer le
moindre code dans vos templates.

Echapper les variables
----------------------

Twig est configuré par défaut pour échapper automatiquement le flux de sortie.
Lisez la `documentation`_  de Twig pour en apprendre plus sur l'échappement et
l'extension Escaper.

Le mot de la fin
----------------

Twig est simple mais puissant. Grâce aux layouts, aux blocks, aux templates et
aux inclusions d'actions, il est très facile d'organiser vos templates de façon
logique et extensible. Pourtant, si vous n'êtes pas à l'aise avec
Twig, vous pouvez toujours utiliser PHP dans les templates de Symfony sans aucun
soucis.


Vous avez travaillé à peine 20 minutes avec Symfony2, mais vous pouvez déjà faire
des choses incroyables avec. C'est la puissance de Symfony2. Apprendre les concepts de base
est très simple, et vous apprendrez bientôt que cette simplicité est cachée derrière
une architecture flexible.

Mais il ne faut pas aller trop vite. D'abord, vous devez en apprendre plus sur le 
contrôleur et c'est justement le sujet de la :doc:`prochaine partie de ce tutoriel<the_controller>`.
Prêt pour 10 nouvelles minutes avec Symfony2 ?

.. _Twig:          http://twig.sensiolabs.org/	
.. _documentation: http://twig.sensiolabs.org/documentation
