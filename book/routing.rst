.. index::
   single: Routing

Routing
=======

Beautiful URLs are an absolute must for any serious web application. This
means leaving behind ugly URLs like ``index.php?article_id=57`` in favor
of something like ``/read/intro-to-symfony``.

URLs bonitas são uma necessidade absoluta para qualquer aplicação web séria. Isto
significa deixar para trás URLs feias como ``index.php?article_id=57`` em favor
de algo como ``/read/intro-to-symfony``.

Having flexibility is even more important. What if you need to change the
URL of a page from ``/blog`` to ``/news``? How many links should you need to
hunt down and update to make the change? If you're using Symfony's router,
the change is simple.

Ter flexibilidade é ainda mais importante. E se você precisar alterar a URL de 
uma página de ``/blog`` para ``/news``? Quantos links você vai precisar caçar e 
atualizar para fazer a alteração? Se você estiver usando o router do Symfony, a 
alteração é simples.

The Symfony2 router lets you define creative URLs that you map to different
areas of your application. By the end of this chapter, you'll be able to:

O router do Symfony2 deixa você definir URLs criativas que você mapeia para 
diferentes áreas da sua aplicação. Ao fim deste capítulo, você será capaz de:

* Create complex routes that map to controllers
* Generate URLs inside templates and controllers
* Load routing resources from bundles (or anywhere else) 
* Debug your routes

* Criar rotas complexas que apontam aos controllers
* Gerar URLs dentro de templates e controllers
* **Carregar recursos de roteamento de bundles (ou qualquer outro lugar)** **VERIFICAR**
* Debugar suas rotas

.. index::
   single: Routing; Basics

Routing em Ação
---------------

A *route* is a map from a URL pattern to a controller. For example, suppose
you want to match any URL like ``/blog/my-post`` or ``/blog/all-about-symfony``
and send it to a controller that can look up and render that blog entry.
The route is simple:

Um *route* é um apontamento de um padrão de URL para um controller. Por exemplo, 
suponha que você quer combinar qualquer URL como ``/blog/my-post`` ou ``/blog/all-about-symfony`` 
e enviar isso ao controller que pode procurar e mostrar esta entrada do blog. A rota é simples:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        blog_show:
            pattern:   /blog/{slug}
            defaults:  { _controller: AcmeBlogBundle:Blog:show }

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog_show" pattern="/blog/{slug}">
                <default key="_controller">AcmeBlogBundle:Blog:show</default>
            </route>
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog_show', new Route('/blog/{slug}', array(
            '_controller' => 'AcmeBlogBundle:Blog:show',
        )));

        return $collection;

The pattern defined by the ``blog_show`` route acts like ``/blog/*`` where
the wildcard is given the name ``slug``. For the URL ``/blog/my-blog-post``,
the ``slug`` variable gets a value of ``my-blog-post``, which is available
for you to use in your controller (keep reading).

O padrão definido pelo route ``blog_show`` funciona como ``/blog/*`` onde o 
coringa é dado o nome ``slug``. Para a URL ``/blog/my-blog-post``, a variável 
``slug`` pega o valor de ``my-blog-post``, que está disponível para você usar 
no seu controller (continue lendo).

The ``_controller`` parameter is a special key that tells Symfony which controller
should be executed when a URL matches this route. The ``_controller`` string
is called the :ref:`logical name<controller-string-syntax>`. It follows a
pattern that points to a specific PHP class and method:

O parâmetro ``_controller`` é uma chave especial que diz ao Symfony qual 
controller deve ser executado quando uma URL combina com esta rota. A string 
``_controller`` é chamada de :ref:`logical name<controller-string-syntax>`. 
Ela segue um padrão que aponta para uma classe e método PHP específicos: 

.. code-block:: php

    // src/Acme/BlogBundle/Controller/BlogController.php
    
    namespace Acme\BlogBundle\Controller;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class BlogController extends Controller
    {
        public function showAction($slug)
        {
            $blog = // use the $slug varible to query the database
            
            return $this->render('AcmeBlogBundle:Blog:show.html.twig', array(
                'blog' => $blog,
            ));
        }
    }

Congratulations! You've just created your first route and connected it to
a controller. Now, when you visit ``/blog/my-post``, the ``showAction`` controller
will be executed and the ``$slug`` variable will be equal to ``my-post``.

Parabéns! Você acabou de criar seu primeiro route e conectou ele a um controller. 
Agora, quando você visitar ``/blog/my-post``, o controller ``showAction`` será 
executado e a variável ``$slug`` será igual a ``my-post``.

This is the goal of the Symfony2 router: to map the URL of a request to a
controller. Along the way, you'll learn all sorts of tricks that make mapping
even the most complex URLs easy.

Isso é o objetivo do router do Symfony2: mapear a URL de uma requisição a um 
controller. Ao longo do caminho, você vai aprender todos os tipos de truques 
para tornar o mapeamento mesmo das mais complexas URLs fácil.

.. index::
   single: Routing; Under the hood

Routing: Under the Hood
-----------------------
Routing: Debaixo o capô
-----------------------

When a request is made to your application, it contains an address to the
exact "resource" that the client is requesting. This address is called the
URL, (or URI), and could be ``/contact``, ``/blog/read-me``, or anything
else. Take the following HTTP request for example:

Quando uma requisição é feita à sua aplicação, ela contém um endereço para o 
exato "recurso" que o cliente está requisitando. O endereço é chamado de URL, 
(ou URI), e pode ser ``/contact``, ``/blog/read-me``, ou qualquer outra coisa. 
Tome a seguinte requisição HTTP por exemplo:

.. code-block:: text

    GET /blog/my-blog-post

The goal of the Symfony2 routing system is to parse this URL and determine
which controller should be executed. The whole process looks like this:

O objetivo do sistema de routing do Symfony2 é de analisar essa URL e determinar 
qual controller deve ser executado. O processo inteiro se parece como isso:

#. The request is handled by the Symfony2 front controller (e.g. ``app.php``);

#. The Symfony2 core (i.e. Kernel) asks the router to inspect the request;

#. The router matches the incoming URL to a specific route and returns information
   about the route, including the controller that should be executed;

#. The Symfony2 Kernel executes the controller, which ultimately returns
   a ``Response`` object.


#. A requisição é manipulada pelo front controller do Symfony2 (ex. ``app.php``);

#. O núcleo do Symfony2 (ex. Kernel) pede ao router para inspecionar a requisição;

#. O router combina a URL recebida com um route específico e retorna informação 
sobre  o route, incluindo o controller que deve ser executado;

#. O Kernel do Symfony2 executa o controller, que finalmente retorna um objeto ``Response``.

.. figure:: /images/request-flow.png
   :align: center
   :alt: Symfony2 request flow

   The routing layer is a tool that translates the incoming URL into a specific
   controller to execute.

   A camada de routing é uma ferramenta que traduz a URL recebida em um controller 
   específico a ser executado.

.. index::
   single: Routing; Creating routes

Creating Routes
---------------
Criando Routes
--------------

Symfony loads all the routes for your application from a single routing configuration
file. The file is usually ``app/config/routing.yml``, but can be configured
to be anything (including an XML or PHP file) via the application configuration
file:

O Symfony carrega todas os routes para sua aplicação de um simples arquivo de 
configuração de routing. O arquivo geralmente é ``app/config/routing.yml`` mas 
pode ser configurado para ser qualquer coisa (incluindo um arquivo XML ou PHP) 
através do arquivo de configuração da aplicação:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            # ...
            router:        { resource: "%kernel.root_dir%/config/routing.yml" }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config ...>
            <!-- ... -->
            <framework:router resource="%kernel.root_dir%/config/routing.xml" />
        </framework:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array(
            // ...
            'router'        => array('resource' => '%kernel.root_dir%/config/routing.php'),
        ));

.. tip::

    Even though all routes are loaded from a single file, it's common practice
    to include additional routing resources from inside the file. See the
    :ref:`routing-include-external-resources` section for more information.

    Apesar de todas as rotas serem carregadas de um único arquivo, é uma 
    prática comum incluir recursos de routing adicionais de dentro do 
    arquivo. Veja a seção :ref:`routing-include-external-resources` para mais
    informações.

Basic Route Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~
Configuração básica de Route
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Defining a route is easy, and a typical application will have lots of routes.
A basic route consists of just two parts: the ``pattern`` to match and a
``defaults`` array:

Definir um route é facil, e uma aplicação típica vai ter um monte de routes. Um 
route básico consiste de somente duas partes: o ``padrão`` para combinar e um array 
de ``defaults``:

.. configuration-block::

    .. code-block:: yaml

        _welcome:
            pattern:   /
            defaults:  { _controller: AcmeDemoBundle:Main:homepage }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="_welcome" pattern="/">
                <default key="_controller">AcmeDemoBundle:Main:homepage</default>
            </route>

        </routes>

    ..  code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('_welcome', new Route('/', array(
            '_controller' => 'AcmeDemoBundle:Main:homepage',
        )));

        return $collection;

This route matches the homepage (``/``) and maps it to the ``AcmeDemoBundle:Main:homepage``
controller. The ``_controller`` string is translated by Symfony2 into an
actual PHP function and executed. That process will be explained shortly
in the :ref:`controller-string-syntax` section.

Este route combina com a homepage (``/``) e aponta para o controller 
``AcmeDemoBundle:Main:homepage``. A string ``_controler`` é transformada pelo 
Symfony2 em uma função de verdade do PHP e executado. O processo será explicado 
brevemente na seção :ref:`controller-string-syntax`.

.. index::
   single: Routing; Placeholders

Routing with Placeholders
~~~~~~~~~~~~~~~~~~~~~~~~~
Routing com Placeholders
~~~~~~~~~~~~~~~~~~~~~~~~~

Of course the routing system supports much more interesting routes. Many
routes will contain one or more named "wildcard" placeholders:

É claro que o sistema de routing suporta routes muito mais interessantes. 
Muitos routes vão conter um ou mais dos chamados placeholders "coringa":

.. configuration-block::

    .. code-block:: yaml

        blog_show:
            pattern:   /blog/{slug}
            defaults:  { _controller: AcmeBlogBundle:Blog:show }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog_show" pattern="/blog/{slug}">
                <default key="_controller">AcmeBlogBundle:Blog:show</default>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog_show', new Route('/blog/{slug}', array(
            '_controller' => 'AcmeBlogBundle:Blog:show',
        )));

        return $collection;

The pattern will match anything that looks like ``/blog/*``. Even better,
the value matching the ``{slug}`` placeholder will be available inside your
controller. In other words, if the URL is ``/blog/hello-world``, a ``$slug``
variable, with a value of ``hello-world``, will be available in the controller.
This can be used, for example, to load the blog post matching that string.

O padrão vai combinar qualquer coisa que pareça com ``/blog/*``. Ainda melhor, 
o valor combinando com o placeholder ``{slug}`` vai estar disponível dentro do 
controller. Em outras palavras, se a URL é ``/blog/hello-word``, uma variável 
``$slug``, com o valor de ``hello-world``, estará disponível no controller. Isso 
pode ser usado, por exemplo, para carregar o post do blog que combina com esta string.

The pattern will *not*, however, match simply ``/blog``. That's because,
by default, all placeholders are required. This can be changed by adding
a placeholder value to the ``defaults`` array.

O padrão *não* vai combinar com um simples ``/blog``. Isso porque, por padrão, 
todos os placeholders são obrigatórios. Isto pode ser alterado adicionando um 
valor ao placeholder no array ``defaults``.

Required and Optional Placeholders
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Placeholders Obrigatórios e Opcionais
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To make things more exciting, add a new route that displays a list of all
the available blog posts for this imaginary blog application:

Para tornar as coisas mais empolgantes, adicione um novo route que mostra 
uma lista de todos os posts disponíveis do blog para essa aplicação de blog 
imaginária:

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog
            defaults:  { _controller: AcmeBlogBundle:Blog:index }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
        )));

        return $collection;

So far, this route is as simple as possible - it contains no placeholders
and will only match the exact URL ``/blog``. But what if you need this route
to support pagination, where ``/blog/2`` displays the second page of blog
entries? Update the route to have a new ``{page}`` placeholder:

Até agora, esse route está tão simples quanto é possível - ele não contém 
placeholders e somente vai combinar com a exata URL ``/blog``. Mas e se você 
precisar que essa rota suporte paginação, onde ``/blog/2`` mostra a segunda página 
de entradas do blog? Atualize o route para ter um placeholder ``{page}`` novo:

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog/{page}
            defaults:  { _controller: AcmeBlogBundle:Blog:index }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog/{page}">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog/{page}', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
        )));

        return $collection;

Like the ``{slug}`` placeholder before, the value matching ``{page}`` will
be available inside your controller. Its value can be used to determine which
set of blog posts to display for the given page.

Assim como o placeholder ``{slug}`` anterior, o valor combinando com ``{page}`` 
estará disponível dentro do controller. Seu valor poderá ser utilizado para 
determinar qual conjunto de posts do blog será exibido para a página passada.

But hold on! Since placeholders are required by default, this route will
no longer match on simply ``/blog``. Instead, to see page 1 of the blog,
you'd need to use the URL ``/blog/1``! Since that's no way for a rich web
app to behave, modify the route to make the ``{page}`` parameter optional.
This is done by including it in the ``defaults`` collection:

Mas espere aí! Uma vez que os placeholders são obrigatórios por padrão, este 
route não vai mais combinar com um simples ``/blog``. Em vez disso, para ver 
a página 1 do blog, você precisou usar a URL ``/blog/1``! Como essa não é a 
maneira que um aplicativo web rico comporta-se, modifique seu route para fazer 
do parâmetro ``{page}`` opcional. Isso é feito incluindo ele na coleção ``defaults``:

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog/{page}
            defaults:  { _controller: AcmeBlogBundle:Blog:index, page: 1 }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog/{page}">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
                <default key="page">1</default>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog/{page}', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
            'page' => 1,
        )));

        return $collection;

By adding ``page`` to the ``defaults`` key, the ``{page}`` placeholder is no
longer required. The URL ``/blog`` will match this route and the value of
the ``page`` parameter will be set to ``1``. The URL ``/blog/2`` will also
match, giving the ``page`` parameter a value of ``2``. Perfect.

Ao adicionar ``page`` à chave ``defaults``, o placeholder ``{page}`` não é mais 
obrigatório. A URL ``/blog`` vai combinar com esse route e o valor do parâmetro 
``page`` será definido para ``1``. A URL ``/blog/2`` também vai combinar, passando 
ao parâmetro o valor ``2``. Perfeito.

+---------+------------+
| /blog   | {page} = 1 |
+---------+------------+
| /blog/1 | {page} = 1 |
+---------+------------+
| /blog/2 | {page} = 2 |
+---------+------------+

.. index::
   single: Routing; Requirements

Adding Requirements
~~~~~~~~~~~~~~~~~~~
Adicionando Requerimentos
~~~~~~~~~~~~~~~~~~~~~~~~~

Take a quick look at the routes that have been created so far:

Dê uma olhada rápida nas rotas que foram criadas até agora:

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog/{page}
            defaults:  { _controller: AcmeBlogBundle:Blog:index, page: 1 }

        blog_show:
            pattern:   /blog/{slug}
            defaults:  { _controller: AcmeBlogBundle:Blog:show }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog/{page}">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
                <default key="page">1</default>
            </route>

            <route id="blog_show" pattern="/blog/{slug}">
                <default key="_controller">AcmeBlogBundle:Blog:show</default>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog/{page}', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
            'page' => 1,
        )));

        $collection->add('blog_show', new Route('/blog/{show}', array(
            '_controller' => 'AcmeBlogBundle:Blog:show',
        )));

        return $collection;

Can you spot the problem? Notice that both routes have patterns that match
URL's that look like ``/blog/*``. The Symfony router will always choose the
**first** matching route it finds. In other words, the ``blog_show`` route
will *never* be matched. Instead, a URL like ``/blog/my-blog-post`` will match
the first route (``blog``) and return a nonsense value of ``my-blog-post``
to the ``{page}`` parameter.

Você poderia apontar o problema? Perceba que ambas rotas possuem padrões que
combinam URL's que pareçam com ``/blog/*``. O router do Symfony vai sempre
escolher o **primeiro** route que combine com o que procura. Em outras palavras,
o route ``blog_show`` **nunca** combinará. Em vez disso, uma URL como 
``/blog/my-blog-post`` vai combinar com o primeiro route (``blog``) e retornar
um valor absurdo de ``my-blog-post`` ao parâmetro ``{page}``.  

+--------------------+-------+-----------------------+
| URL                | route | parameters            |
+====================+=======+=======================+
| /blog/2            | blog  | {page} = 2            |
+--------------------+-------+-----------------------+
| /blog/my-blog-post | blog  | {page} = my-blog-post |
+--------------------+-------+-----------------------+

The answer to the problem is to add route *requirements*. The routes in this
example would work perfectly if the ``/blog/{page}`` pattern *only* matched
URLs where the ``{page}`` portion is an integer. Fortunately, regular expression
requirements can easily be added for each parameter. For example:

A solução do problema é adicionar *requisitos* de route. Os routes nesse exemplo 
podem funcionar perfeitamente se o padrão ``/blog/{page}`` combinar *apenas* URLs
onde a porção ``{page}`` é um inteiro. Felizmente, requisitos em forma de expressões
regulares podem ser facilmente adicionados para cada parâmetro. Por exemplo:

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog/{page}
            defaults:  { _controller: AcmeBlogBundle:Blog:index, page: 1 }
            requirements:
                page:  \d+

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog/{page}">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
                <default key="page">1</default>
                <requirement key="page">\d+</requirement>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog/{page}', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
            'page' => 1,
        ), array(
            'page' => '\d+',
        )));

        return $collection;

The ``\d+`` requirement is a regular expression that says that the value of
the ``{page}`` parameter must be a digit (i.e. a number). The ``blog`` route
will still match on a URL like ``/blog/2`` (because 2 is a number), but it
will no longer match a URL like ``/blog/my-blog-post`` (because ``my-blog-post``
is *not* a number).

As a result, a URL like ``/blog/my-blog-post`` will now properly match the
``blog_show`` route.

+--------------------+-----------+-----------------------+
| URL                | route     | parameters            |
+====================+===========+=======================+
| /blog/2            | blog      | {page} = 2            |
+--------------------+-----------+-----------------------+
| /blog/my-blog-post | blog_show | {slug} = my-blog-post |
+--------------------+-----------+-----------------------+

.. sidebar:: Earlier Routes always Win

    What this all means is that the order of the routes is very important.
    If the ``blog_show`` route were placed above the ``blog`` route, the
    URL ``/blog/2`` would match ``blog_show`` instead of ``blog`` since the
    ``{slug}`` parameter of ``blog_show`` has no requirements. By using proper
    ordering and clever requirements, you can accomplish just about anything.

Since the parameter requirements are regular expressions, the complexity
and flexibility of each requirement is entirely up to you. Suppose the homepage
of your application is available in two different languages, based on the
URL:

.. configuration-block::

    .. code-block:: yaml

        homepage:
            pattern:   /{culture}
            defaults:  { _controller: AcmeDemoBundle:Main:homepage, culture: en }
            requirements:
                culture:  en|fr

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="homepage" pattern="/{culture}">
                <default key="_controller">AcmeDemoBundle:Main:homepage</default>
                <default key="culture">en</default>
                <requirement key="culture">en|fr</requirement>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('homepage', new Route('/{culture}', array(
            '_controller' => 'AcmeDemoBundle:Main:homepage',
            'culture' => 'en',
        ), array(
            'culture' => 'en|fr',
        )));

        return $collection;

For incoming requests, the ``{culture}`` portion of the URL is matched against
the regular expression ``(en|fr)``.

+-----+--------------------------+
| /   | {culture} = en           |
+-----+--------------------------+
| /en | {culture} = en           |
+-----+--------------------------+
| /fr | {culture} = fr           |
+-----+--------------------------+
| /es | *won't match this route* |
+-----+--------------------------+

.. index::
   single: Routing; Method requirement

Adding HTTP Method Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In addition to the URL, you can also match on the *method* of the incoming
request (i.e. GET, HEAD, POST, PUT, DELETE). Suppose you have a contact form
with two controllers - one for displaying the form (on a GET request) and one
for processing the form when it's submitted (on a POST request). This can
be accomplished with the following route configuration:

.. configuration-block::

    .. code-block:: yaml

        contact:
            pattern:  /contact
            defaults: { _controller: AcmeDemoBundle:Main:contact }
            requirements:
                _method:  GET

        contact_process:
            pattern:  /contact
            defaults: { _controller: AcmeDemoBundle:Main:contactProcess }
            requirements:
                _method:  POST

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="contact" pattern="/contact">
                <default key="_controller">AcmeDemoBundle:Main:contact</default>
                <requirement key="_method">GET</requirement>
            </route>

            <route id="contact_process" pattern="/contact">
                <default key="_controller">AcmeDemoBundle:Main:contactProcess</default>
                <requirement key="_method">POST</requirement>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('contact', new Route('/contact', array(
            '_controller' => 'AcmeDemoBundle:Main:contact',
        ), array(
            '_method' => 'GET',
        )));

        $collection->add('contact_process', new Route('/contact', array(
            '_controller' => 'AcmeDemoBundle:Main:contactProcess',
        ), array(
            '_method' => 'POST',
        )));

        return $collection;

Despite the fact that these two routes have identical patterns (``/contact``),
the first route will match only GET requests and the second route will match
only POST requests. This means that you can display the form and submit the
form via the same URL, while using distinct controllers for the two actions.

.. note::
    If no ``_method`` requirement is specified, the route will match on
    *all* methods.

Like the other requirements, the ``_method`` requirement is parsed as a regular
expression. To match ``GET`` *or* ``POST`` requests, you can use ``GET|POST``.

.. index::
   single: Routing; Advanced example
   single: Routing; _format parameter

.. _advanced-routing-example:

Advanced Routing Example
~~~~~~~~~~~~~~~~~~~~~~~~

At this point, you have everything you need to create a powerful routing
structure in Symfony. The following is an example of just how flexible the
routing system can be:

.. configuration-block::

    .. code-block:: yaml

        article_show:
          pattern:  /articles/{culture}/{year}/{title}.{_format}
          defaults: { _controller: AcmeDemoBundle:Article:show, _format: html }
          requirements:
              culture:  en|fr
              _format:  html|rss
              year:     \d+

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="article_show" pattern="/articles/{culture}/{year}/{title}.{_format}">
                <default key="_controller">AcmeDemoBundle:Article:show</default>
                <default key="_format">html</default>
                <requirement key="culture">en|fr</requirement>
                <requirement key="_format">html|rss</requirement>
                <requirement key="year">\d+</requirement>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('homepage', new Route('/articles/{culture}/{year}/{title}.{_format}', array(
            '_controller' => 'AcmeDemoBundle:Article:show',
            '_format' => 'html',
        ), array(
            'culture' => 'en|fr',
            '_format' => 'html|rss',
            'year' => '\d+',
        )));

        return $collection;

As you've seen, this route will only match if the ``{culture}`` portion of
the URL is either ``en`` or ``fr`` and if the ``{year}`` is a number. This
route also shows how you can use a period between placeholders instead of
a slash. URLs matching this route might look like:

 * ``/articles/en/2010/my-post``
 * ``/articles/fr/2010/my-post.rss``

.. _book-routing-format-param:

.. sidebar:: The Special ``_format`` Routing Parameter

    This example also highlights the special ``_format`` routing parameter.
    When using this parameter, the matched value becomes the "request format"
    of the ``Request`` object. Ultimately, the request format is used for such
    things such as setting the ``Content-Type`` of the response (e.g. a ``json``
    request format translates into a ``Content-Type`` of ``application/json``).
    It can also be used in the controller to render a different template for
    each value of ``_format``. The ``_format`` parameter is a very powerful way
    to render the same content in different formats.

Special Routing Parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~

As you've seen, each routing parameter or default value is eventually available
as an argument in the controller method. Additionally, there are three parameters
that are special: each adds a unique piece of functionality inside your application:

* ``_controller``: As you've seen, this parameter is used to determine which
  controller is executed when the route is matched;

* ``_format``: Used to set the request format (:ref:`read more<book-routing-format-param>`);

* ``_locale``: Used to set the locale on the session (:ref:`read more<book-translation-locale-url>`);

.. index::
   single: Routing; Controllers
   single: Controller; String naming format

.. _controller-string-syntax:

Controller Naming Pattern
-------------------------

Every route must have a ``_controller`` parameter, which dictates which
controller should be executed when that route is matched. This parameter
uses a simple string pattern called the *logical controller name*, which
Symfony maps to a specific PHP method and class. The pattern has three parts,
each separated by a colon:

    **bundle**:**controller**:**action**

For example, a ``_controller`` value of ``AcmeBlogBundle:Blog:show`` means:

+----------------+------------------+-------------+
| Bundle         | Controller Class | Method Name |
+================+==================+=============+
| AcmeBlogBundle | BlogController   | showAction  |
+----------------+------------------+-------------+

The controller might look like this:

.. code-block:: php

    // src/Acme/BlogBundle/Controller/BlogController.php
    
    namespace Acme\BlogBundle\Controller;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    
    class BlogController extends Controller
    {
        public function showAction($slug)
        {
            // ...
        }
    }

Notice that Symfony adds the string ``Controller`` to the class name (``Blog``
=> ``BlogController``) and ``Action`` to the method name (``show`` => ``showAction``).

You could also refer to this controller using its fully-qualified class name
and method: ``Acme\BlogBundle\Controller\BlogController::showAction``.
But if you follow some simple conventions, the logical name is more concise
and allows more flexibility.

.. note::

   In addition to using the logical name or the fully-qualified class name,
   Symfony supports a third way of referring to a controller. This method
   uses just one colon separator (e.g. ``service_name:indexAction``) and
   refers to the controller as a service (see :doc:`/cookbook/controller/service`).

Route Parameters and Controller Arguments
-----------------------------------------

The route parameters (e.g. ``{slug}``) are especially important because
each is made available as an argument to the controller method:

.. code-block:: php

    public function showAction($slug)
    {
      // ...
    }

In reality, the entire ``defaults`` collection is merged with the parameter
values to form a single array. Each key of that array is available as an
argument on the controller.

In other words, for each argument of your controller method, Symfony looks
for a route parameter of that name and assigns its value to that argument.
In the advanced example above, any combination (in any order) of the following
variables could be used as arguments to the ``showAction()`` method:

* ``$culture``
* ``$year``
* ``$title``
* ``$_format``
* ``$_controller``

Since the placeholders and ``defaults`` collection are merged together, even
the ``$_controller`` variable is available. For a more detailed discussion,
see :ref:`route-parameters-controller-arguments`.

.. tip::

    You can also use a special ``$_route`` variable, which is set to the
    name of the route that was matched.

.. index::
   single: Routing; Importing routing resources

.. _routing-include-external-resources:

Including External Routing Resources
------------------------------------

All routes are loaded via a single configuration file - usually ``app/config/routing.yml``
(see `Creating Routes`_ above). Commonly, however, you'll want to load routes
from other places, like a routing file that lives inside a bundle. This can
be done by "importing" that file:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        acme_hello:
            resource: "@AcmeHelloBundle/Resources/config/routing.yml"

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <import resource="@AcmeHelloBundle/Resources/config/routing.xml" />
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;

        $collection = new RouteCollection();
        $collection->addCollection($loader->import("@AcmeHelloBundle/Resources/config/routing.php"));

        return $collection;

.. note::

   When importing resources from YAML, the key (e.g. ``acme_hello``) is meaningless.
   Just be sure that it's unique so no other lines override it.

The ``resource`` key loads the given routing resource. In this example the
resource is the full path to a file, where the ``@AcmeHelloBundle`` shortcut
syntax resolves to the path of that bundle. The imported file might look
like this:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/routing.yml
       acme_hello:
            pattern:  /hello/{name}
            defaults: { _controller: AcmeHelloBundle:Hello:index }

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="acme_hello" pattern="/hello/{name}">
                <default key="_controller">AcmeHelloBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        // src/Acme/HelloBundle/Resources/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('acme_hello', new Route('/hello/{name}', array(
            '_controller' => 'AcmeHelloBundle:Hello:index',
        )));

        return $collection;

The routes from this file are parsed and loaded in the same way as the main
routing file.

Prefixing Imported Routes
~~~~~~~~~~~~~~~~~~~~~~~~~

You can also choose to provide a "prefix" for the imported routes. For example,
suppose you want the ``acme_hello`` route to have a final pattern of ``/admin/hello/{name}``
instead of simply ``/hello/{name}``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        acme_hello:
            resource: "@AcmeHelloBundle/Resources/config/routing.yml"
            prefix:   /admin

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <import resource="@AcmeHelloBundle/Resources/config/routing.xml" prefix="/admin" />
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;

        $collection = new RouteCollection();
        $collection->addCollection($loader->import("@AcmeHelloBundle/Resources/config/routing.php"), '/admin');

        return $collection;

The string ``/admin`` will now be prepended to the pattern of each route
loaded from the new routing resource.

.. index::
   single: Routing; Debugging

Visualizing & Debugging Routes
------------------------------

While adding and customizing routes, it's helpful to be able to visualize
and get detailed information about your routes. A great way to see every route
in your application is via the ``router:debug`` console command. Execute
the command by running the following from the root of your project.

.. code-block:: bash

    php app/console router:debug

The command will print a helpful list of *all* the configured routes in
your application:

.. code-block:: text

    homepage              ANY       /
    contact               GET       /contact
    contact_process       POST      /contact
    article_show          ANY       /articles/{culture}/{year}/{title}.{_format}
    blog                  ANY       /blog/{page}
    blog_show             ANY       /blog/{slug}

You can also get very specific information on a single route by including
the route name after the command:

.. code-block:: bash

    php app/console router:debug article_show

.. index::
   single: Routing; Generating URLs

Generating URLs
---------------

The routing system should also be used to generate URLs. In reality, routing
is a bi-directional system: mapping the URL to a controller+parameters and
a route+parameters back to a URL. The
:method:`Symfony\\Component\\Routing\\Router::match` and
:method:`Symfony\\Component\\Routing\\Router::generate` methods form this bi-directional
system. Take the ``blog_show`` example route from earlier::

    $params = $router->match('/blog/my-blog-post');
    // array('slug' => 'my-blog-post', '_controller' => 'AcmeBlogBundle:Blog:show')

    $uri = $router->generate('blog_show', array('slug' => 'my-blog-post'));
    // /blog/my-blog-post

To generate a URL, you need to specify the name of the route (e.g. ``blog_show``)
and any wildcards (e.g. ``slug = my-blog-post``) used in the pattern for
that route. With this information, any URL can easily be generated:

.. code-block:: php

    class MainController extends Controller
    {
        public function showAction($slug)
        {
          // ...

          $url = $this->get('router')->generate('blog_show', array('slug' => 'my-blog-post'));
        }
    }

In an upcoming section, you'll learn how to generate URLs from inside templates.

.. tip::

    If the frontend of your application uses AJAX requests, you might want
    to be able to generate URLs in JavaScript based on your routing configuration.
    By using the `FOSJsRoutingBundle`_, you can do exactly that:
    
    .. code-block:: javascript
    
        var url = Routing.generate('blog_show', { "slug": 'my-blog-post});

    For more information, see the documentation for that bundle.

.. index::
   single: Routing; Absolute URLs

Generating Absolute URLs
~~~~~~~~~~~~~~~~~~~~~~~~

By default, the router will generate relative URLs (e.g. ``/blog``). To generate
an absolute URL, simply pass ``true`` to the third argument of the ``generate()``
method:

.. code-block:: php

    $router->generate('blog_show', array('slug' => 'my-blog-post'), true);
    // http://www.example.com/blog/my-blog-post

.. note::

    The host that's used when generating an absolute URL is the host of
    the current ``Request`` object. This is detected automatically based
    on server information supplied by PHP. When generating absolute URLs for
    scripts run from the command line, you'll need to manually set the desired
    host on the ``Request`` object:
    
    .. code-block:: php
    
        $request->headers->set('HOST', 'www.example.com');

.. index::
   single: Routing; Generating URLs in a template

Generating URLs with Query Strings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``generate`` method takes an array of wildcard values to generate the URI.
But if you pass extra ones, they will be added to the URI as a query string::

    $router->generate('blog', array('page' => 2, 'category' => 'Symfony'));
    // /blog/2?category=Symfony

Generating URLs from a template
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The most common place to generate a URL is from within a template when linking
between pages in your application. This is done just as before, but using
a template helper function:

.. configuration-block::

    .. code-block:: html+jinja

        <a href="{{ path('blog_show', { 'slug': 'my-blog-post' }) }}">
          Read this blog post.
        </a>

    .. code-block:: php

        <a href="<?php echo $view['router']->generate('blog_show', array('slug' => 'my-blog-post')) ?>">
            Read this blog post.
        </a>

Absolute URLs can also be generated.

.. configuration-block::

    .. code-block:: html+jinja

        <a href="{{ url('blog_show', { 'slug': 'my-blog-post' }) }}">
          Read this blog post.
        </a>

    .. code-block:: php

        <a href="<?php echo $view['router']->generate('blog_show', array('slug' => 'my-blog-post'), true) ?>">
            Read this blog post.
        </a>

Summary
-------

Routing is a system for mapping the URL of incoming requests to the controller
function that should be called to process the request. It both allows you
to specify beautiful URLs and keeps the functionality of your application
decoupled from those URLs. Routing is a two-way mechanism, meaning that it
should also be used to generate URLs.

Learn more from the Cookbook
----------------------------

* :doc:`/cookbook/routing/scheme`

.. _`FOSJsRoutingBundle`: https://github.com/FriendsOfSymfony/FOSJsRoutingBundle
