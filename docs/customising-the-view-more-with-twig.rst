[Parte 5] – Customizando a View: Extensões do Twig, a barra lateral e a Biblioteca Assetic
==========================================================================================

Visão geral
-----------

Este capítulo, continua trabalhando com o frontend do Symblog. Vamos ajustar a página inicial para exibir informações 
sobre os comentários de um post do blog, juntamente com endereçamento SEO, adicionando o título do blog para a URL. 

Também iremos trabalhar na barra lateral para adicionar 2 componentes comuns de blogs, a Nuvem de Tags e os Últimos 
Comentários. 

Vamos explorar os vários ambientes do Symfony 2 e aprender a executar o Symblog no ambiente de produção. 

Iremos estender os templates com Twig para utilizar um novo filtro e trabalharemos com Assetic para gerenciar os assets 
do site. 

No final deste capítulo, você terá integrado comentários na página inicial, um componente de nuvem de Tags e Últimos 
Comentários na barra lateral e terá usado Assetic para gerenciar os arquivos assets. Você conseguirá executar o Symblog 
no ambiente de produção.

A Homepage - Blogs e Comentários
---------------------------------

Até agora, a página inicial lista as entradas mais recentes do blog, mas não dá qualquer informação sobre os comentários 
para os posts dos blogs. Agora que temos a entidade ``Comment`` construída, podemos voltar à página inicial para inserir 
essas informações. 

Como configuramos as ligações entre as entidades ``Blog`` e ``Comment``, sabemos que o Doctrine 2 irá recuperar os 
comentários relativos a um blog (lembre-se, nós adicionamos um membro ``$comments`` à entidade ``Blog``). 

Vamos atualizar o template da visualização da página inicial, localizado em 
``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig``, com o seguinte código:

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}

    {# .. #}
    
    <footer class="meta">
        <p>Comentários: {{ blog.comments|length }}</p>
        <p>Postado por <span class="highlight">{{ blog.author }}</span> em {{ blog.created|date('h:iA') }}</p>
        <p>Tags: <span class="highlight">{{ blog.tags }}</span></p>
    </footer>
    
    {# .. #}

Temos utilizado os métodos getter de ``comments`` para recuperar os comentários do blog e, depois, passamos a coleção 
através do filtro ``length`` do Twig. Acesse a página inicial agora ``http://symblog.dev/app_dev.php/`` e você verá o 
número de comentários para cada blog sendo exibido.

Como explicado acima, já informamos ao Doctrine 2 que o membro ``$comments``, da entidade ``Blog``, é mapeado para a 
entidade ``Comment``. Vimos isso no capítulo anterior, com os seguintes metadados da entidade ``Blog``, localizada em 
``src/Blogger/BlogBundle/Entity/blog.php``.

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    /**
     * @ORM\OneToMany(targetEntity="Comment", mappedBy="blog")
     */
    protected $comments;

Então, nós sabemos que Doctrine 2 é consciente da relação entre blogs e comentários, mas como é que vamos preencher o 
membro ``$comments`` com a entidade ``Comment`` relacionada? Se você se lembra do método ``BlogRepository`` que criamos 
(mostrado abaixo) para obter os blogs para a página inicial, não fizemos nenhuma seleção para recuperar a entidade 
``Comment`` relacionada.

.. code-block:: php

    // src/Blogger/BlogBundle/Repository/BlogRepository.php
    
    public function getLatestBlogs($limit = null)
    {
        $qb = $this->createQueryBuilder('b')
                   ->select('b')
                   ->addOrderBy('b.created', 'DESC');

        if (false === is_null($limit))
            $qb->setMaxResults($limit);

        return $qb->getQuery()
                  ->getResult();
    }
    
No entanto, o Doctrine 2 usa um processo chamado de ``carregamento parcial (lazy loading)`` onde a entidade ``Comment`` é 
recuperada do banco de dados, como e quando necessário, no nosso caso, quando a chamada para ``{{Blog.comments|length}}`` 
é feita. Podemos demonstrar este processo utilizando a barra de ferramentas do desenvolvedor. 

Começamos a explorar os conceitos básicos da barra de ferramentas do desenvolvedor e agora é hora de introduzir uma de 
suas características mais úteis, o Doctrine 2 profiler. 

O Doctrine 2 profiler pode ser acessado clicando no último ícone a barra de ferramentas do desenvolvedor. O número ao 
lado deste ícone mostra o número de consultas executadas no banco de dados para a requisição HTTP atual.

.. image:: /_static/images/part_5/doctrine_2_toolbar_icon.jpg
    :align: center
    :alt: Ícone Developer toolbar - Doctrine 2

Após clicar no ícone, você visualizará informações sobre as consultas que foram executadas pelo Doctrine 2 no banco de 
dados para o requisição HTTP atual.

.. image:: /_static/images/part_5/doctrine_2_toolbar_queries.jpg
    :align: center
    :alt: Consultas Developer toolbar - Doctrine 2

Como você pôde ver na captura de tela acima, há uma série de consultas realizadas para um pedido para a página inicial. 

A segunda consulta executada, recupera as entidades do blog do banco de dados e é executado como um resultado do método 
``getLatestBlogs()`` na classe ``BlogRepository``. Após esta consulta, você irá notar uma série de outras consultas que 
recebem os comentários do banco de dados, um blog de cada vez. Isso é possível por causa do ``WHERE t0.blog_id =?`` em 
cada uma das consultas, onde o ``?`` é substituído pelo valor do parâmetro (o blog Id) na linha seguinte. 

Cada uma destas consultas é proveniente das chamadas para ``{{Blog.comments}}`` no template da página inicial. Cada vez 
que esta função é executada, o Doctrine 2 carrega gradativamente a entidade ``Comment``, que se relaciona com a entidade 
``Blog``. 

Embora o ``Lazy loading`` seja muito eficaz na recuperação de entidades relacionadas do banco de dados, nem sempre é a 
maneira mais eficiente. Doctrine 2 consegue ``juntar`` entidades relacionadas quando consultamos o banco de dados. 

Dessa forma, podemos resgatar o ``Blog`` e as entidades ``Comments``  relacionadas, em uma consulta, fora do banco de 
dados .

Atualize o código do ``QueryBuilder`` no ``BlogRepository``, localizado em 
``src/Blogger/BlogBundle/Repository/BlogRepository.php``, para juntarmos os comentários.

.. code-block:: php

    // src/Blogger/BlogBundle/Repository/BlogRepository.php

    public function getLatestBlogs($limit = null)
    {
        $qb = $this->createQueryBuilder('b')
                   ->select('b, c')
                   ->leftJoin('b.comments', 'c')
                   ->addOrderBy('b.created', 'DESC');

        if (false === is_null($limit))
            $qb->setMaxResults($limit);

        return $qb->getQuery()
                  ->getResult();
    }

Se você atualizar a página e examinar as saídas do Doctrine 2 na barra de ferramentas do desenvolvedor, você vai notar 
que o número de consultas caiu. Você também pode ver que a tabela de comentário foi unificada à tabela de blog.

``Lazy loading`` e ``join`` entre entidades relacionadas, são dois conceitos muito poderosos, mas eles precisam ser 
usados corretamente. O equilíbrio correto entre os 2 deve ser encontrado para garantir que sua aplicação esteja 
funcionando tão eficientemente quanto possível. 

A princípio, pode parecer bem interessante juntar isso tudo em cada entidade relacionada, para que você nunca precise 
usar o ``Lazy loading`` e a contagem de consultas a banco de dados fique sempre baixa. No entanto, é importante lembrar 
que, quanto mais informações você recuperar do banco de dados, o Doctrine 2 precisará de mais processamento para 
preencher os presentes objetos da entidade. Mais dados, também, significa mais memória usada pelo servidor para 
armazenar os objetos da entidade.

Antes de prosseguirmos, vamos fazer uma pequena adição ao template da página inicial para o número de comentários que 
acabamo de adicionar. 

Atualizar o template da página inicial, localizado em ``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig``, 
para adicionar um link para mostrar os comentários do blog.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}

    {# .. #}
    
    <footer class="meta">
        <p>Comentários: <a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id }) }}#comments">{{ blog.comments|length }}</a></p>
        <p>Postado por <span class="highlight">{{ blog.author }}</span> em {{ blog.created|date('h:iA') }}</p>
        <p>Tags: <span class="highlight">{{ blog.tags }}</span></p>
    </footer>
    
    {# .. #}
            
A barra lateral
---------------

Atualmente, a barra lateral do symblog está parecendo um pouco vazia. Atualizaremos a barra lateral com 2 componentes 
comuns de blog, a Nuvem de Tags e uma lista dos Últimos Comentários.

A Nuvem de Tag
~~~~~~~~~~~~~~ 

A Nuvem de Tag, mostra tags para cada post do blog enfatizado, de forma a mostrar as tags mais comuns. Para isso, 
precisamos de uma maneira de recuperar todas as tags de todos os blogs. 

Vamos criar alguns novos métodos na classe ``BlogRepository``, do arquivo localizado em 
``src/Blogger/BlogBundle/Repository/BlogRepository.php``. Copie e cole o seguinte código:

.. code-block:: php

    // src/Blogger/BlogBundle/Repository/BlogRepository.php

    public function getTags()
    {
        $blogTags = $this->createQueryBuilder('b')
                         ->select('b.tags')
                         ->getQuery()
                         ->getResult();

        $tags = array();
        foreach ($blogTags as $blogTag)
        {
            $tags = array_merge(explode(",", $blogTag['tags']), $tags);
        }

        foreach ($tags as &$tag)
        {
            $tag = trim($tag);
        }

        return $tags;
    }

    public function getTagWeights($tags)
    {
        $tagWeights = array();
        if (empty($tags))
            return $tagWeights;
        
        foreach ($tags as $tag)
        {
            $tagWeights[$tag] = (isset($tagWeights[$tag])) ? $tagWeights[$tag] + 1 : 1;
        }
        // Embaralhar as tags
        uksort($tagWeights, function() {
            return rand() > rand();
        });
        
        $max = max($tagWeights);
        
        // Peso Máximo de 5
        $multiplier = ($max > 5) ? 5 / $max : 1;
        foreach ($tagWeights as &$tag)
        {
            $tag = ceil($tag * $multiplier);
        }
    
        return $tagWeights;
    }

Como as tags são armazenadas no banco de dados como valores separados por vírgula (CSV), precisamos de uma maneira de 
dividi-los e devolvê-los como um array. Isto é realizado pelo método ``getTags()``. 

O método ``getTagWeights()``, também consegue usar um array de tags para calcular ``o peso`` de cada tag com base na sua 
popularidade, dentro do array. As tags também são embaralhadas para exibi-las na página de forma aleatória.

Agora, temos a Nuvem de Tags, precisamos exibi-la. Criar uma nova ação no ``PageController`` em
``src/Blogger/BlogBundle/Controller/PageController.php`` para trabalhar com a barra lateral.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    
    public function sidebarAction()
    {
        $em = $this->getDoctrine()
                   ->getEntityManager();

        $tags = $em->getRepository('BloggerBlogBundle:Blog')
                   ->getTags();

        $tagWeights = $em->getRepository('BloggerBlogBundle:Blog')
                         ->getTagWeights($tags);

        return $this->render('BloggerBlogBundle:Page:sidebar.html.twig', array(
            'tags' => $tagWeights
        ));
    }

A ação é muito simples, ele usa os 2 novos métodos do ``BlogRepository`` para gerar a Nuvem de Tag e passar esta nuvem 
para a visão (View). 

Agora, vamos criar esta View em ``src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig``.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig #}
    
    <section class="section">
        <header>
            <h3>Nuvem de Tags</h3>
        </header>
        <p class="tags">
            {% for tag, weight in tags %}
                <span class="weight-{{ weight }}">{{ tag }}</span>
            {% else %}
                <p>Não há tags</p>
            {% endfor %}
        </p>
    </section>

O template também é muito simples. Ele só interage com as várias tags definindo uma classe para o peso da tag. O loop 
``for``, nos mostra como acessar o par ``chave`` e ``valor`` do array, com ``tag`` sendo a chave e ``peso`` sendo o 
valor. Há uma série de variações de como utilizar o loop ``for`` na 
`Documentação do Twig <http://twig.sensiolabs.org/doc/templates.html#for>`_.

Se você voltar ao layout do tamplate principal ``BloggerBlogBundle``, localizado em 
``src/Blogger/BlogBundle/Resources/views/layout.html.twig``, você vai perceber que colocamos um espaço reservado para o 
bloco da barra lateral. 

Vamos substituir este bloco agora, renderizando a nova ação da barra lateral. 

Lembre-se do capítulo anterior, o método ``render`` do Twig irá processar o conteúdo a partir de uma ação do controlador, 
neste caso, a ação ``sidebar`` do controlador ``Page``.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}

    {# .. #}

    {% block sidebar %}
        {% render "BloggerBlogBundle:Page:sidebar" %}
    {% endblock %}

Finalmente, vamos adicionar o CSS para a Nuvem de Tags. Adicione uma folha de estilo nova em 
``src/Blogger/BlogBundle/Resources/public/css/sidebar.css``.

.. code-block:: css

    .sidebar .section { margin-bottom: 20px; }
    .sidebar h3 { line-height: 1.2em; font-size: 20px; margin-bottom: 10px; font-weight: normal; background: #eee; padding: 5px;  }
    .sidebar p { line-height: 1.5em; margin-bottom: 20px; }
    .sidebar ul { list-style: none }
    .sidebar ul li { line-height: 1.5em }
    .sidebar .small { font-size: 12px; }
    .sidebar .comment p { margin-bottom: 5px; }
    .sidebar .comment { margin-bottom: 10px; padding-bottom: 10px; }
    .sidebar .tags { font-weight: bold; }
    .sidebar .tags span { color: #000; font-size: 12px; }
    .sidebar .tags .weight-1 { font-size: 12px; }
    .sidebar .tags .weight-2 { font-size: 15px; }
    .sidebar .tags .weight-3 { font-size: 18px; }
    .sidebar .tags .weight-4 { font-size: 21px; }
    .sidebar .tags .weight-5 { font-size: 24px; }

Como criamos uma nova folha de estilo, precisamos incluí-la. Atualize o layout do template principal 
``BloggerBlogBundle``, localizado em ``src/Blogger/BlogBundle/Recursos/views/layout.html.twig``, com o seguinte código:

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}

    {# .. #}
    
    {% block stylesheets %}
        {{ parent() }}
        <link href="{{ asset('bundles/bloggerblog/css/blog.css') }}" type="text/css" rel="stylesheet" />
        <link href="{{ asset('bundles/bloggerblog/css/sidebar.css') }}" type="text/css" rel="stylesheet" />
    {% endblock %}
    
    {# .. #}

.. note::

    Se você não estiver usando o método de link simbólico para referenciar o pacote assets para a pasta ``web``, você 
    deve re-executar o comando para instalar os assets, para copiar a novo arquivo CSS.

    .. code-block:: bash

        $ php app/console assets:install web
        
Se você atualizar o site Symblog, você vai ver a Nuvem de Tags renderizada na barra lateral. A fim de obter as tags com 
peso diferente para renderizar, você pode precisar atualizar as fixtures do blog para que algumas tags fiquem mais 
usadas, mais do que outras.

Comentários Recentes
~~~~~~~~~~~~~~~~~~~~ 

Agora que a Nuvem de Tags está no seu devido lugar, vamos adicionar o componente dos Comentários mais Recentes à barra 
lateral.

Primeiro, precisamos de uma forma para recuperar os últimos comentários dos blogs. Para isso, vamos adicionar um novo 
método para ``CommentRepository``, localizado em ``src/Blogger/BlogBundle/Repository/CommentRepository.php``.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Repository/CommentRepository.php

    public function getLatestComments($limit = 10)
    {
        $qb = $this->createQueryBuilder('c')
                    ->select('c')
                    ->addOrderBy('c.id', 'DESC');

        if (false === is_null($limit))
            $qb->setMaxResults($limit);

        return $qb->getQuery()
                  ->getResult();
    }

Agora,  atualize a ação ``sidebar`` em ``src/Blogger/BlogBundle/controller/PageController.php`` para recuperar os 
últimos comentários e passá-los para a View.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    
    public function sidebarAction()
    {
        // ..

        $commentLimit   = $this->container
                               ->getParameter('blogger_blog.comments.latest_comment_limit');
        $latestComments = $em->getRepository('BloggerBlogBundle:Comment')
                             ->getLatestComments($commentLimit);
    
        return $this->render('BloggerBlogBundle:Page:sidebar.html.twig', array(
            'latestComments'    => $latestComments,
            'tags'              => $tagWeights
        ));
    }

Perceba, usamos um novo parâmetro, chamado ``Blogger_blog.comments.latest_comment_limit``, para limitar o número de 
comentários recuperados. 

Para criar este parâmetro, atualize o arquivo de configuração em ``src/Blogger/BlogBundle/Resources/config/config.yml`` 
com o seguinte código:

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/config.yml
    
    parameters:
        # ..

        # Máximo de Últimos Comentários do Blog
        blogger_blog.comments.latest_comment_limit: 10

Finalmente, precisamos renderizar os últimos comentários na barra lateral do template. 

Atualize o templete localizado em ``src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig`` com o seguinte 
código:

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig #}

    {# .. #}

    <section class="section">
        <header>
            <h3>Últimos Comentários</h3>
        </header>
        {% for comment in latestComments %}
            <article class="comment">
                <header>
                    <p class="small"><span class="highlight">{{ comment.user }}</span> comentou no
                        <a href="{{ path('BloggerBlogBundle_blog_show', { 'id': comment.blog.id }) }}#comment-{{ comment.id }}">
                            {{ comment.blog.title }}
                        </a>
                        [<em><time datetime="{{ comment.created|date('c') }}">{{ comment.created|date('Y-m-d h:iA') }}</time></em>]
                    </p>
                </header>
                <p>{{ comment.comment }}</p>
                </p>
            </article>
        {% else %}
            <p>Não há comentários recentes</p>
        {% endfor %}
    </section>

Se você atualizar o site Symblog, você verá os Últimas Comentários sendo exibidos na barra lateral abaixo da Nuvem de 
Tags.

.. image:: /_static/images/part_5/sidebar.jpg
    :align: center
    :alt: Barra lateral - Nuvem de Tags e Últimos Comentários

Extensões Twig
---------------

Até agora, estamos apresentando as datas dos comentários do posts publicados no blog em um formato padrão, como 
`2011-04-21`. Uma abordagem interessante, seria exibir as datas dos comentários em termos de há quanto tempo o 
comentário foi publicado, como `postado 3 horas atrás`. 

Poderíamos adicionar um método para a entidade ``Comment`` e alterar os templates para usar este método ao invés de 
``{{comment.created | date ('Ymd h: iA')}}``.

Como podemos usar essa funcionalidade em outros lugares, faria mais sentido movê-lo para fora da entidade ``Comment``. 
Como transformar a data é especificamente uma tarefa da camada de visão, devemos implementar isso usando o gerador de 
templates do Twig. O Twig disponibiliza uma Interface de Extensão.

Podemos usar a `Interface de Extensão <http://www.twig-project.org/doc/extensions.html>`_ no Twig para estender a 
funcionalidade padrão que ele proporciona. 

Vamos criar um novo filtro de extensão do Twig, que pode ser usado como se segue:

.. code-block:: html
    
    {{ comment.created|created_ago }}
    
Isto iria retornar o comentário criado com a data em um formato como `postado 2 dias atrás`.
    
A Extensão
~~~~~~~~~~

Crie um arquivo para a extensão do Twig em ``src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogExtension.php`` e 
atualize-o com o seguinte conteúdo:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogExtension.php

    namespace Blogger\BlogBundle\Twig\Extensions;

    class BloggerBlogExtension extends \Twig_Extension
    {
        public function getFilters()
        {
            return array(
                'created_ago' => new \Twig_Filter_Method($this, 'createdAgo'),
            );
        }

        public function createdAgo(\DateTime $dateTime)
        {
            $delta = time() - $dateTime->getTimestamp();
            if ($delta < 0)
                throw new \InvalidArgumentException("createdAgo não está habilitado para trabalhar com datas no futuro");

            $duration = "";
            if ($delta < 60)
            {
                // Segundos
                $time = $delta;
                $duration = $time . " segundos" . (($time > 1) ? "s" : "") . " atrás";
            }
            else if ($delta <= 3600)
            {
                // Minutos
                $time = floor($delta / 60);
                $duration = $time . " minutos" . (($time > 1) ? "s" : "") . " atrás";
            }
            else if ($delta <= 86400)
            {
                // Horas
                $time = floor($delta / 3600);
                $duration = $time . " horas" . (($time > 1) ? "s" : "") . " atrás";
            }
            else
            {
                // Dias
                $time = floor($delta / 86400);
                $duration = $time . " dias" . (($time > 1) ? "s" : "") . " atrás";
            }

            return $duration;
        }

        public function getName()
        {
            return 'blogger_blog_extension';
        }
    }

Criar uma extensão é bastante simples. Nós substituímos o método ``getFilters()`` para retornar qualquer número de 
filtros que queremos estar disponibilizando. Neste caso, estamos criando o filtro ``created_ago``. 

Este filtro, é, então, registado para usar o método ``createdAgo``, que simplesmente, transforma um objeto ``DateTime`` 
em uma string representando a duração passada desde quando o valor foi armazenado no objeto ``DateTime``.

Registrando a Extensão
~~~~~~~~~~~~~~~~~~~~~~

Para fazer a extensão do Twig ficar disponível, precisamos atualizar o arquivo de serviços, localizado em 
``src/Blogger/BlogBundle/Resources/config/services.yml``, com o seguinte código:

.. code-block:: yaml

    services:
        blogger_blog.twig.extension:
            class: Blogger\BlogBundle\Twig\Extensions\BloggerBlogExtension
            tags:
                - { name: twig.extension }

Você pôde ver que estamos registrando um novo serviço usando a classe de extensão do Twig ``BloggerBlogExtension`` que 
acabamos de criar.

Atualizando a View
~~~~~~~~~~~~~~~~~~
    
O novo filtro do Twig está pronto para ser usado. Vamos atualizar a lista Comentários mais Recentes da barra lateral, 
para usar o filtro ``created_ago``. 

Atualize o template da barra lateral, localizado em ``src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig``, 
com o seguinte código:


.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig #}

    {# .. #}
    
    <section class="section">
        <header>
            <h3>Últimos Comentários</h3>
        </header>
        {% for comment in latestComments %}
            {# .. #}
            <em><time datetime="{{ comment.created|date('c') }}">{{ comment.created|created_ago }}</time></em>
            {# .. #}
        {% endfor %}
    </section>

Se você acessar  ``http://symblog.dev/app_dev.php/``, você vai ver que as datas dos últimos comentários estão usando o 
filtro Twig para renderizar a duração, desde quando o comentário foi postado.

Vamos atualizar os comentários listados na página de exibição do blog para usar o novo filtro. Substitua o conteúdo do 
templete localizado em ``src/Blogger/BlogBundle/Resources/views/Comment/index.html.twig`` com o seguinte código:

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Comment/index.html.twig #}

    {% for comment in comments %}
        <article class="comment {{ cycle(['odd', 'even'], loop.index0) }}" id="comment-{{ comment.id }}">
            <header>
                <p><span class="highlight">{{ comment.user }}</span> comentou <time datetime="{{ comment.created|date('c') }}">{{ comment.created|created_ago }}</time></p>
            </header>
            <p>{{ comment.comment }}</p>
        </article>
    {% else %}
        <p>Não há comentários para este post. Seja o primeiro a comentar...</p>
    {% endfor %}

.. tip::

    Há várias extensões do Twig úteis, disponíveis na biblioteca 
    `Extensões do Twig <https://github.com/fabpot/Twig-extensions>`_ no GitHub. Se você criar uma extensão útil, envie 
    uma solicitação de recebimento para este repositório e ele pode ser incluído para que outras pessoas o usem.

Fazendo o Slug da URL
---------------------

Atualmente, a URL para cada post do blog, só mostra o id do blog. Enquanto essa abordagem é perfeitamente aceitável, do 
ponto de vista funcional, não é grande coisa para trabalhos com SEO.

Por exemplo,a URL ``http://symblog.dev/1`` não dá qualquer informação sobre o conteúdo do blog. Algo como 
``http://symblog.dev/1/a-day-with-symfony2`` seria muito melhor. 

Assim, vamos fazer um slug do título do blog e usá-lo como parte desta URL. Esse Slug do título irá remover todos os 
caracteres, não ASCII, e irão substituí-los com um ``-``.

Atualizando a rota
~~~~~~~~~~~~~~~~~~

Para começar, vamos modificar a regra de roteamento para a página de exibição do blog para adicionar o componente slug. 

Atualize a regra de roteamento no arquivo localizado em ``src/Blogger/BlogBundle/Resources/config/routing.yml``.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    
    BloggerBlogBundle_blog_show:
        pattern:  /{id}/{slug}
        defaults: { _controller: BloggerBlogBundle:Blog:show }
        requirements:
            _method:  GET
            id: \d+

O controlador
~~~~~~~~~~~~~

Tal como acontece com o componente ``id`` existente, o novo componente ``slug`` será passado para a ação do controlador 
como um argumento, então vamos atualizar o controlador localizado em 
``src/Blogger/BlogBundle/Controller/BlogController.php``.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/BlogController.php

    public function showAction($id, $slug)
    {
        // ..
    }

.. tip::

    A ordem, na qual os argumentos são passados para a ação do controlador, não importa. Somente os nomes dos argumentos. 
    O Symfony2, é capaz de combinar os argumentos da rota com a lista de parâmetros passados. 

    Embora, ainda, não tenhamos utilizado os valores padrão dos componentes, vale a pena mencioná-los aqui. Se 
    adicionamos     outro componente para a regra de rota, podemos especificar um valor padrão para o componente usando 
    as opções ``Default``.

    .. code-block:: yaml

        BloggerBlogBundle_blog_show:
            pattern:  /{id}/{slug}/{comments}
            defaults: { _controller: BloggerBlogBundle:Blog:show, comments: true }
            requirements:
                _method:  GET
                id: \d+

    .. code-block:: php

        public function showAction($id, $slug, $comments)
        {
            // ..
        }

    Usando este método, uma requisição para ``http://symblog.dev/1/symfony2-blog``, resultaria em ``$comments`` sendo 
    definido como true na ``showAction``.

Fazendo o Slug do título
~~~~~~~~~~~~~~~~~~~~~~~~

Como queremos gerar o slug do título do blog, vamos gerar o valor do slug automaticamente. Poderíamos simplesmente 
executar esta operação em tempo de execução no campo de título, mas vamos guardar o slug da entidade ``Blog`` e mantê-lo 
no banco de dados.

Atualizando a entidade Blog
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Vamos adicionar um novo membro para a entidade ``Blog`` para armazenar o slug. Atualize a entidade ``Blog``, localizada 
em ``src/Blogger/BlogBundle/Entity/blog.php``, com o seguinte código:

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    class Blog
    {
        // ..

        /**
         * @ORM\Column(type="string")
         */
        protected $slug;

        // ..
    }

Agora gere os assessores para o novo membro ``$slug``. Como antes, execute o comando abaixo:

.. code-block:: bash

    $ php app/console doctrine:generate:entities Blogger

Em seguida, vamos atualizar o esquema do banco de dados.

.. code-block:: bash

    $ php app/console doctrine:migrations:diff
    $ php app/console doctrine:migrations:migrate

Para gerar o valor do slug, vamos utilizar o método ``slugify`` do tutorial do Symfony 1 
`Jobeet <http://www.symfony-project.org/jobeet/1_4/Propel/en/08>`_ . 

Adicione o método ``slugify`` para a entidade do ``Blog`` localizada em ``src/Blogger/BlogBundle/Entity/blog.php``.

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    public function slugify($text)
    {
        // trocar caracteres não letras e não dígitos por '-'
        $text = preg_replace('#[^\\pL\d]+#u', '-', $text);

        // trim
        $text = trim($text, '-');

        // tradução literal
        if (function_exists('iconv'))
        {
            $text = iconv('utf-8', 'us-ascii//TRANSLIT', $text);
        }

        // lowercase
        $text = strtolower($text);

        // remove caracteres indesejados
        $text = preg_replace('#[^-\w]+#', '', $text);

        if (empty($text))
        {
            return 'n-a';
        }

        return $text;
    }

Como queremos gerar automaticamente o slug do título, podemos gerar o slug quando o valor do título é definido. Para 
isso, podemos atualizar o acessor ``setTitle`` para definir, também, o valor do slug. 

Atualize a entidade ``Blog``, localizada em ``src/Blogger/BlogBundle/setTitle/blog.php``, com o seguinte código:

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    public function setTitle($title)
    {
        $this->title = $title;

        $this->setSlug($this->title);
    }

Agora, atualize o método ``setSlug`` para fazer o slug da variável ``Slug``, antes de ser definido.

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    public function setSlug($slug)
    {
        $this->slug = $this->slugify($slug);
    }

Agora, recarregue o Data Fixtures para criar os slugs do blog.

.. code-block:: bash

    $ php app/console doctrine:fixtures:load

Atualizando as rotas geradas
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Finalmente, precisamos atualizar as chamadas existentes para a geração de rotas para a página de exibição do blog. Há 
uma variedade de lugares onde este item tem de ser atualizado.

Abra o template da página inicial, localizada em ``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig``, e 
substitua o seu conteúdo com o seguinte código. 

Houve três modificações para a geração da rota ``BloggerBlogBundle_blog_show`` neste template. As edições simplesmente 
passam o slug do blog para a função ``path`` do Twig.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}

    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block body %}
        {% for blog in blogs %}
            <article class="blog">
                <div class="date"><time datetime="{{ blog.created|date('c') }}">{{ blog.created|date('l, F j, Y') }}</time></div>
                <header>
                    <h2><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id, 'slug': blog.slug }) }}">{{ blog.title }}</a></h2>
                </header>
    
                <img src="{{ asset(['images/', blog.image]|join) }}" />
                <div class="snippet">
                    <p>{{ blog.blog(500) }}</p>
                    <p class="continue"><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id, 'slug': blog.slug }) }}">Continue lendo...</a></p>
                </div>
    
                <footer class="meta">
                    <p>Comentários: <a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id, 'slug': blog.slug }) }}#comments">{{ blog.comments|length }}</a></p>
                    <p>Postado por <span class="highlight">{{ blog.author }}</span> em {{ blog.created|date('h:iA') }}</p>
                    <p>Tags: <span class="highlight">{{ blog.tags }}</span></p>
                </footer>
            </article>
        {% else %}
            <p>Não há entradas de blog para Symblog</p>
        {% endfor %}
    {% endblock %}

Além disso, uma atualização precisa ser feita para a seção Comentários mais Recentes da barra lateral, template 
localizado em ``src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig``.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig #}

    {# .. #}

    <a href="{{ path('BloggerBlogBundle_blog_show', { 'id': comment.blog.id, 'slug': comment.blog.slug }) }}#comment-{{ comment.id }}">
        {{ comment.blog.title }}
    </a>

    {# .. #}

Finalmente, a função ``createAction`` do ``CommentController`` precisa ser atualizado ao redirecionar para a página de 
exibição do blog, em uma postagem de comentário bem-sucedido. 

Atualize o ``CommentController``, localizado em ``src/Blogger/BlogBundle/Controller/CommentController.php``, com o 
seguinte código:

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/CommentController.php
    
    public function createAction($blog_id)
    {
        // ..

        if ($form->isValid()) {
            // ..
                
            return $this->redirect($this->generateUrl('BloggerBlogBundle_blog_show', array(
                'id'    => $comment->getBlog()->getId(),
                'slug'  => $comment->getBlog()->getSlug())) .
                '#comment-' . $comment->getId()
            );
        }

        // ..
    }

Agora, se você navegar para a página inicial do Symblog ``http://symblog.dev/app_dev.php/`` e clicar em um dos títulos 
dos posts do blog, você vai ver que o slug do blog foi acrescentado ao final da URL.

Ambientes
---------

Os ambientes são um poderoso recurso, ainda que simples, mantido no Symfony 2. Você pode não estar ciente, mas estamos 
utilizando os ambientes desde a parte 1 deste tutorial. 

Com ambientes, podemos configurar vários aspectos do Symfony 2 e da aplicação, para executar de forma diferente, 
dependendo das necessidades específicas durante o ciclo de vida da aplicação. 

Por padrão, Symfony 2 vem configurado com 3 ambientes:

    1. ``Dev``  - Desenvolvimento
    2. ``Test`` - Teste
    3. ``Prod`` - Produção

O objetivo desses ambientes, é auto-explicativo, mas, estes ambientes podem ser configurados de forma diferente para 
suas necessidades individuais. 

Ao desenvolver a aplicação, é útil ter a barra de ferramentas do desenvolvedor na tela, exibindo exceções e/ou erros que 
estão acontecendo, enquanto na produção, você não quer exibir qualquer erros e/ou exceções pois, exibir essas 
informações, seria um risco de segurança, pois um monte de detalhes internos da aplicação e do servidor, estariam 
expostos. 

Na produção, seria melhor exibir páginas personalizadas de erro com mensagens simplificadas, enquanto registra-se essas 
exceções e/ou erros em arquivos de texto. 

Também é útil ter o cache ativado, para assegurar que o aplicativo está sendo executado da melhor forma possível. O cache 
estando habilitado no ambiente de ``Desenvolvimento``, seria trabalhoso, pois é preciso esvaziar o cache cada vez que 
você faz alterações em arquivos de configuração, etc.

O outro ambiente é o ``test``. Este é usado quando estamos executando testes sobre a aplicação, tais como testes de 
unidade ou testes funcionais. Não cobrimos testes ainda, mas será abordado em profundidade no próximo capítulo.

Controladores de Frente (Front Controllers)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Até agora, temos utilizado somente o ambiente de ``desenvolvimento``. Especificamos isso executando o controlador de 
frente ``app_dev.php`` ao fazermos requisições ao Symblog, por exemplo, ``http://symblog.dev/app_dev.php/about``. 

Se verificarmos o controlador de frente, localizado em ``web/app_dev.php``, você verá a seguinte linha:

.. code-block:: php

    $kernel = new AppKernel('dev', true);

Esta linha é que inicia o Symfony 2. Ela cria uma nova instância de ``AppKernel`` do Symfony 2  e define o ambiente como 
``dev``.

Entretanto, se verificarmos o controlador de frente para o ambiente de ``produção`` localizado em ``web/ app.php`` 
veremos a seguinte linha:

.. code-block:: php

    $kernel = new AppKernel('prod', false);

Note que, neste caso, o ambiente ``prod`` é passado para o ``AppKernel``.

O ambiente de teste, supostamente, não será executado através do browser web pois não há o controlador de frente 
``app_test.php``.

Configurações
~~~~~~~~~~~~~

Vimos acima, como os front controllers são utilizados para mudar o ambiente do aplicativo que é executado. Agora, vamos 
explorar como as diversas definições são modificado durante a execução do aplicativo, em cada ambiente. 

Se você verificar os arquivos em ``app/config``, você verá vários arquivos ``config.yml``. Especificamente, há um 
principal, chamado ``config.yml`` e outros 3 sufixados com o nome de um ambiente; ``config_dev.yml``, 
``config_test.yml`` e ``config_prod.yml``. 

Cada um desses arquivos é carregado, dependendo do ambiente atual. Se explorarmos o arquivo ``config_dev.yml``, você 
verá as seguintes linhas do topo do arquivo:

.. code-block:: yaml

    imports:
        - { resource: config.yml }

As diretivas de ``importação``, farão com que o arquivo ``config.yml`` seja incluído nestes arquivos. As mesmas diretivas 
de ``importação``, podem ser encontradas na parte superior do 2 outros arquivos de configuração de ambiente, 
``config_test.yml`` e ``config_prod.yml``. 

Por um conjunto comum de definições de configuração estar sendo incluindo em ``config.yml``, podemos substituir as 
configurações específicas para cada ambiente. 

Podemos ver, no arquivo de configuração do ambiente de ``desenvolvimento``, localizado em ``app/config/config_dev.yml``, 
as seguintes linhas configurando o uso da barra de ferramentas do desenvolvedor.

.. code-block:: yaml

    # app/config/config_dev.yml
    
    web_profiler:
        toolbar: true

Esta configuração está ausente no arquivo de configuração de ``produção``, pois não queremos que a Developer Toolbar seja 
exibida.

Executando em Ambiente de Produção
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Para os ansiosos por ver o site funcionando no ambiente de ``produção``, agora é a hora.

Primeiro, precisamos limpar o cache usando um dos comandos do Symfony 2.

.. code-block:: bash

    $ php app/console cache:clear --env=prod

Agora, acesse ``http://symblog.dev/``. Observe que o controlador de frente ``app_dev.php`` está ausente.

.. note::
    
    Para aqueles de vocês que estão usando a configuração dinâmica de hosts virtuais, como feito na parte 1, vocês 
    precisam adicionar o seguinte trecho de código no arquivo ``.htaccess`` localizado em ``web/.htaccess``.
    
    .. code-block:: text
    
        <IfModule mod_rewrite.c>
            RewriteBase /
            # ..
        </IfModule>
        
Você perceberá que o site parece praticamente o mesmo, mas algumas poucas, mas, importantes características, estão 
diferentes. 

A barra de ferramentas do desenvolvedor não está mais presente e as detalhadas mensagens de exceção não são mais 
exibidas, tente acessar ``http://symblog.dev/999``.

.. image:: /_static/images/part_5/production_error.jpg
    :align: center
    :alt: Produção - Erro 404
    
A mensagem de exceção detalhada foi substituída por uma mensagem simplificada, informando o utilizador, do problema. 
Essas telas de exceção, podem ser personalizadas para se parecer com sua aplicação. Vamos explorar isso no próximo 
capítulo.

Além disso, você notará que o arquivo ``/logs/prod.log`` do aplicativo, está se enchendo de registros sobre a execução da 
aplicação. Isso é útil quando se tem problemas com a aplicação, em ambiente de ``produção``, com erros e exceções não 
sendo mais exibidos na tela.

.. tip::

    Como a requisição para ``http://symblog.dev/`` foi encaminhado pelo arquivo de rota para ``app.php``? Tenho certeza 
    de que criamos arquivos, como ``index.html`` ou ``index.php`` que agem como índice do site, mas como ``app.php`` 
    faz isso? Isso acontece graças a um ``RewriteRule`` no arquivo ``web/.htaccess``

    .. code-block:: text

        RewriteRule ^(.*)$ app.php [QSA,L]

    Podemos ver que, esta linha, tem uma expressão regular que combina com qualquer texto, mostrado por ``^ (. *) $`` e 
    passa para ``app.php``.

    Você pode estar em um servidor Apache que não tem o ``mod_rewrite.c`` habilitado. Se este for o caso, você pode, 
    simplesmente, adicionar ``app.php`` na URL, como em ``http://symblog.dev/app.php/``.

Apesar de cobrirmos o básico do ambiente de ``produção``, não cobrimos muitas outras atividades relacionadas com o 
ambiente de ``produção``, como a personalização das páginas de erro e de implantação para o servidor de produção, usando 
ferramentas como `Capifony <http://capifony.org/>`_. Estes tópicos serão abordados no próximo capítulo.

Criando Novos Ambientes
~~~~~~~~~~~~~~~~~~~~~~~

Finalmente, vale a pena lembrar que, você pode configurar seus próprios ambientes, facilmente, em Symfony 2. Por exemplo, 
você pode querer que um ambiente de teste serja executado no servidor de produção, mas exibindo algumas informações de 
depuração, como exceções. 

Isso permitiria que a plataforma fosse testada manualmente no servidor de produção com configurações de produção e 
desenvolvimento, que o servidor pudesse diferenciar.

Como criar um novo ambiente é uma tarefa simples, ele está fora do escopo deste tutorial. Existe um excelente 
`Artigo <http://symfony.com/doc/current/cookbook/configuration/environments.html>`_ no livro do Symfony 2 que cobre 
este assunto.

Assetic
-------

A distribuição Standard do Symfony 2, vem com uma biblioteca para tratar assets, chamada 
`Assetic <https://github.com/kriswallsmith/assetic>`_. A biblioteca foi desenvolvida por 
`Kris Wallsmith <https://twitter.com/#!/kriswallsmith>`_ e foi inspirado na biblioteca do Python 
`Webassets <http://elsdoerfer.name/files/docs/webassets/>`_.

Assetic, lida com 2 partes de gerenciamento de assets, como imagens, folhas de estilo e JavaScript e os filtros que podem 
ser aplicadas a esses assets. 

Estes filtros são capazes de realizar tarefas úteis como ``minifying`` do seu CSS e JavaScript, passando arquivos 
`CoffeeScript <http://jashkenas.github.com/coffee-script/>`_ para o compilador CoffeeScript, combinando-os em conjunto, 
para reduzir o número de requisições HTTP, feitas para o servidor.

Atualmente, temos utilizado a função ``asset`` do Twig para incluir assets no template, como se segue abaixo:

.. code-block:: html
    
    <link href="{{ asset('bundles/bloggerblog/css/blog.css') }}" type="text/css" rel="stylesheet" />

As chamadas para a função ``asset`` serão substituídas por Assetic.

Assets
~~~~~~

A biblioteca Assetic descreve um asset como:

    `Um Assetic asset, é algo com conteúdo filtrável que pode ser carregado e despejado. Um asset também inclui 
    metadados, alguns dos quais podem ser manipulados e alguns dos quais são imutáveis.`

Simplificando, os assets são os recursos que o aplicativo usa, tais como folhas de estilo e imagens.

Folhas de Estilo
................

Vamos começar pela substituição das chamadas atuais para os ``assets``, folhas de estilo, no layout do template principal 
``BloggerBlogBundle``. 

Atualize o conteúdo do template, localizado em ``src/Blogger/BlogBundle/Resources/views/layout.html.twig``, com o 
seguinte código:

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    
    {# .. #}

    {% block stylesheets %}
        {{ parent () }}
        
        {% stylesheets 
            '@BloggerBlogBundle/Resources/public/css/*'
        %}
            <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
        {% endstylesheets %}
    {% endblock %}
    
    {# .. #}

Substituímos os 2 links anteriores para arquivos CSS com algumas funcionalidades Assetic. Usando ``folhas de estilo`` 
Assetic, especificamos que, todos os arquivos CSS localizados em ``src/Blogger/BlogBundle/Resources/public/css``, devem 
ser combinados em um único arquivo e, depois, exibí-lo. 

Combinar arquivos é muito simples, mas ganhamos uma efetiva otimização do frontend do site, reduzindo o número de 
arquivos necessários. Menos arquivos, significa menos requisições HTTP para o servidor. 

Foi utilizado o ``*`` para especificar todos os arquivos no diretório ``css``. Poderíamos ter, simplesmente, listado 
cada arquivo individualmente, como se segue:

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    
    {# .. #}

    {% block stylesheets %}
        {{ parent () }}
        
        {% stylesheets 
            '@BloggerBlogBundle/Resources/public/css/blog.css'
            '@BloggerBlogBundle/Resources/public/css/sidebar.css'
        %}
            <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
        {% endstylesheets %}
    {% endblock %}

    {# .. #}
    
O resultado final, em ambos os casos, é o mesmo. A primeira opção, usando ``*``, garante que quando novos arquivos CSS 
são adicionados ao diretório, eles serão sempre incluídos no arquivo CSS combinado pelo Assetic. 

Isto, pode não ser a funcionalidade desejada para o seu site, mas, use o método acima para atender às suas necessidades 
de momento ou futuras.
    
Se você observar a saída HTML de ``http://symblog.dev/app_dev.php/``, você vai ver o CSS incluído (Note que nós estamos 
executando o ambiente de ``desenvolvimento`` novamente).

.. code-block:: html
    
    <link href="/app_dev.php/css/d8f44a4_part_1_blog_1.css" rel="stylesheet" media="screen" />
    <link href="/app_dev.php/css/d8f44a4_part_1_sidebar_2.css" rel="stylesheet" media="screen" />
    
Em primeiro lugar, você deve estar se perguntando, por que há 2 arquivos. Acima foi dito que Assetic combinaria os 
arquivos em um único arquivo CSS. Isto é porque estamos executando o Symblog no ambiente de ``desenvolvimento``. 

Podemos pedir ao Assetic para ser executado em modo não-debug, definindo o sinalizador de depuração para false, como se 
segue:

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    
    {# .. #}
    
    {% stylesheets 
        '@BloggerBlogBundle/Resources/public/css/*'
        debug=false
    %}
        <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
    {% endstylesheets %}

    {# .. #}
    
Agora, se você atualizar o HTML renderizado, você verá algo como:

.. code-block:: html

    <link href="/app_dev.php/css/3c7da45.css" rel="stylesheet" media="screen" />
    
Se você visualizar o conteúdo deste arquivo, você vai ver que os 2 arquivos CSS, ``blog.css`` e ``sidebar.css``, foram 
combinados em um único arquivo. O nome dado ao arquivo CSS gerado, é criado aleatoriamente pelo Assetic. 

Se você quiser controlar o nome dado para o arquivo gerado, use a opção de ``saída`` como se segue:

.. code-block:: html

    {% stylesheets 
        '@BloggerBlogBundle/Resources/public/css/*'
        output='css/blogger.css'
    %}
        <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
    {% endstylesheets %}

Antes de continuar, remova o sinalizador de depuração do trecho anterior, pois queremos retomar o comportamento padrão 
dos assets.

Precisamos, também, atualizar o template básico das aplicações, localizado em ``app/Resources/views/base.html.twig``.

.. code-block:: html

    {# app/Resources/views/base.html.twig #}
    
    {# .. #}
    
    {% block stylesheets %}
        <link href='http://fonts.googleapis.com/css?family=Irish+Grover' rel='stylesheet' type='text/css'>
        <link href='http://fonts.googleapis.com/css?family=La+Belle+Aurore' rel='stylesheet' type='text/css'>
        {% stylesheets 
            'css/*'
        %}
            <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
        {% endstylesheets %}
    {% endblock %}
    
    {# .. #}
    
JavaScripts
...........

Embora atualmente não possuimos arquivos Javascript em nossa aplicação, a sua utilização em Assetic é a mesma coisa das 
folhas de estilo.

.. code-block:: html

    {% javascripts 
        '@BloggerBlogBundle/Resources/public/js/*'
    %}
        <script type="text/javascript" src="{{ asset_url }}"></script>
    {% endjavascripts %}

Filtros
~~~~~~~

O verdadeiro poder do Assetic, vem dos filtros. Os filtros podem ser aplicados à assets ou coleção de assets. Existe uma 
série de filtros disponibilizados na biblioteca, incluindo os seguintes filtros:

    1. ``CssMinFilter``:            coloca o conteúdo do CSS em uma única linha
    2. ``JpegoptimFilter``:         otimiza seus JPEGs
    3. ``Yui\CssCompressorFilter``: comprime CSS usando o compressor YUI
    4. ``Yui\JsCompressorFilter``:  comprime o JavaScript usando o compressor YUI
    5. ``CoffeeScriptFilter``:      compila CoffeeScript em JavaScript

Há uma lista completa de filtros disponíveis no
`Leia-me do Assetic <https://github.com/kriswallsmith/assetic/blob/master/README.md>`_.

Muitos destes filtros, passam a tarefa real para outro programa ou biblioteca, tal como YUI Compressor, assim você pode 
precisar instalar/configurar as bibliotecas apropriadas para usar alguns dos filtros.

Baixe o `YUI Compressor <http://yuilibrary.com/download/yuicompressor/>`_, descompacte o arquivo e copie o arquivo do 
``diretório criado`` para ``app/Resources/java/yuicompressor-2.4.6.jar``. Assumimos que você baixou a versão ``2.4.6`` 
do YUI Compressor. Se não, mude o número da versão ilustrada para a que você baixou.

Em seguida, vamos configurar um filtro Assetic para comprimir o CSS usando o YUI Compressor.

Atualize o arquivo de configuração dos aplicativos, localizado em ``app/config/config.yml``, com o seguinte código:

.. code-block:: yaml
    
    # app/config/config.yml
    
    # ..

    assetic:
        filters:
            yui_css:
                jar: %kernel.root_dir%/Resources/java/yuicompressor-2.4.6.jar
    
    # ..
    
Temos configurado um filtro chamado ``yui_css``, que irá utilizar o YUI Compressor, Executável Java, que colocamos no 
diretório de recursos da aplicação (ilustrado acima). 

Para usar o filtro, você precisa especificar quais os assets que você deseja que o filtro seja aplicado. 

Atualize o template localizado em ``src/Blogger/BlogBundle/Recursos/views/layout.html.twig`` para aplicar o filtro 
``yui_css``.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}

    {# .. #}
    
    {% stylesheets 
        '@BloggerBlogBundle/Resources/public/css/*'
        output='css/blogger.css'
        filter='yui_css'
    %}
        <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
    {% endstylesheets %}

    {# .. #}

Agora, se você atualizar o site Symblog e ver a saída dos arquivos Assetic, você vai notar que eles foram comprimidos. 

Enquanto a minimização/compressão, é uma boa para os servidores de produção, ele pode tornar a depuração difícil, 
especialmente, quando o JavaScript é minimizado. Podemos desativar a minimização, quando o ambiente de 
``desenvolvimento`` for executado, pela junção do filtro com um ``?`` como se segue:

.. code-block:: html
    
    {% stylesheets 
        '@BloggerBlogBundle/Resources/public/css/*'
        output='css/blogger.css'
        filter='?yui_css'
    %}
        <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
    {% endstylesheets %}

Inserindo os assets para a o ambiente de produção
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Na produção, podemos inserir os arquivos assets usando a biblioteca Assetic. Assim, eles se tornam recursos atuais no 
disco, prontos para serem disponibilizados pelo servidor web. 

O processo de criação dos assets através do Assetic, com cada requisição de página, pode ser bastante lento, 
especialmente quando os filtros estão sendo aplicadas aos assets. 

Inserção dos assets para o ambiente de ``produção``, garante que Assetic não é usada para servir os assets e, em vez 
disso, os arquivos pré-processados de assets são servidos diretamente pelo servidor web. 

Execute o seguinte comando, para criar a inserção dos arquivos assets:

.. code-block:: bash

    $ app/console --env=prod assetic:dump

Você vai perceber que vários arquivos CSS foram criados em ``web/css``, incluindo o arquivo combinado ``blogger.css``. 
Agora, se você executar o site Symblog no ambiente de ``produção`` acessando ``http://symblog.dev/``, os arquivos serão 
disponibilizados diretamente da pasta ``web/css``.

.. note::

    Se você inserir os arquivos assets para o disco e quer voltar para o ``Ambiente de desenvolvimento``, você terá que 
    limpar os arquivos assets criados em ``web/`` para permitir que Assetic recrie os arquivos.

Leitura adicional
~~~~~~~~~~~~~~~~~

Nós apenas ilustramos o que Assetic pode executar. Há mais recursos disponíveis on-line, especialmente no livro do 
Symfony 2, incluindo:

`Como usar Assetic para Manter Assets <http://symfony.com/doc/current/cookbook/assetic/asset_management.html>`_

`Como Minimizar (Minify) JavaScripts e folhas de estilo com YUI Compressor <http://symfony.com/doc/current/cookbook/assetic/yuicompressor.html>`_

`Como usar Assetic Para Otimização de Imagem com Funções Twig <http://symfony.com/doc/current/cookbook/assetic/jpeg_optimize.html>`_

`Como Aplicar um filtro Assetic a uma extensão de arquivo específica <http://symfony.com/doc/current/cookbook/assetic/apply_to_option.html>`_

Há, também, uma série de grandes artigos escritos por `Richard Miller <https://twitter.com/#!/mr_r_miller>`_, incluindo:

`Symfony 2: Usando CoffeeScript com Assetic <http://miller.limethinking.co.uk/2011/05/16/symfony2-using-coffeescript-with-assetic/>`_

`Symfony 2: Algumas notas sobre Assetic <http://miller.limethinking.co.uk/2011/06/02/symfony2-a-few-assetic-notes/>`_

`Symfony 2: Funções Assetic Twig <http://miller.limethinking.co.uk/2011/06/23/symfony2-assetic-twig-functions/>`_

.. tip::

    Vale a pena mencionar aqui que, Richard Miller, tem uma coleção de excelentes artigos a respeito de várias áreas do 
    Symfony 2 em seu site, incluindo injeção de dependência, Serviços e os guias acima mencionados sobre Assetic. Basta 
    pesquisar por posts com a tag `symfony2 <http://miller.limethinking.co.uk/tag/symfony2/>`_

Conclusão
---------

Cobrimos uma série de novas áreas com relação ao Symfony 2, incluindo os ambientes do Symfony 2 e como usar a biblioteca 
Assetic. Fizemos melhorias para a página inicial e acrescentamos alguns componentes para a barra lateral.

No próximo capítulo, vamos abordar os testes. Vamos explorar tanto o teste de unidade quanto o teste funcional usando 
PHPUnit. Veremos como Symfony 2 vem completo com uma variedade de classes para auxiliar na escrita de testes funcionais 
que simulam solicitações da Web, nos permitindo preencher formulários e clicar em links e então inspecionar a resposta 
retornada.