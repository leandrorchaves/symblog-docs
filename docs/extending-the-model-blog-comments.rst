[Parte 4] - O Model de Comentários: Adicionando comentários, Repositórios e Migrações do Doctrine 2
===================================================================================================

Visão geral
-----------

Este capítulo, terá como base, o Model do blog definida no capítulo anterior. Criaremos o Model de comentário, que vai 
trabalhar com os comentários dos posts do blog. 

Nós ilustraremos a criação de relações entre Models, já que um blog pode conter muitos comentários. Usaremos o 
Doctrine 2 QueryBuilder e suas classes de repositório para recuperar as entidades do banco de dados. 

O conceito de Migrações do Doctrine 2 ``Doctrine 2 Migrations``, também serão exploradas para fornecer uma maneira mais 
programática de implantar as alterações no banco de dados. 

No final deste capítulo, você terá criado o Model de comentários e irá relacioná-lo com o Model do blog. Criaremos uma 
página inicial, que permita os usuários enviar comentários para um post do blog. 


A Homepage
----------

Começaremos este capítulo com a construção da homepage. 

Em um blog de verdade, é exibido trechos de cada post do blog, ordenado do mais novo para o mais antigo. O post completo
do blog, estará disponível, através de links, em uma página de exibição do blog. 

Como já temos a rota, o controlador e a View da página inicial, podemos simplesmente atualizá-la.

Recuperando os blogs: Consultando o Model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Para exibir os blogs, precisamos recuperá-los do banco de dados. Doctrine 2 trabalha com 
`Doctrine Query Language <http://www.doctrine-project.org/docs/orm/2.1/en/reference/dql-doctrine-query-language.html>`_ 
(DQL) e um `QueryBuilder <http://www.doctrine-project.org/docs/orm/2.1/en/reference/query-builder.html>`_ para conseguir 
isso (Você também pode executar SQL pelo Doctrine 2, mas este método é desencorajado, pois ele tira a abstração de banco 
de dados que o Doctrine 2 nos dá). 

Nós usaremos o ``QueryBuilder``, pois ele fornece uma maneira amigável de trabalhar com orientação a objetos gerados pelo 
DQL, para nos permitir consultar o banco de dados. 

Vamos atualizar a ação ``index`` do controlador ``Page``, localizado em 
``src/Blogger/BlogBundle/Controller/PageController.php``, para trazer os blogs a partir do banco de dados.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    class PageController extends Controller
    {
        public function indexAction()
        {
            $em = $this->getDoctrine()
                       ->getEntityManager();
    
            $blogs = $em->createQueryBuilder()
                        ->select('b')
                        ->from('BloggerBlogBundle:Blog',  'b')
                        ->addOrderBy('b.created', 'DESC')
                        ->getQuery()
                        ->getResult();
    
            return $this->render('BloggerBlogBundle:Page:index.html.twig', array(
                'blogs' => $blogs
            ));
        }
        
        // ..
    }

Começaremos obtendo uma instância do ``QueryBuilder`` do ``EntityManager``. O ``EntityManager`` nos permite construir a 
consulta usando os vários métodos que o ``QueryBuilder`` disponibiliza. 

A lista completa de métodos disponíveis, está disponível na documentação do ``QueryBuilder``. Leia também sobre 
`Métodos auxiliares <http://www.doctrine-project.org/docs/orm/2.1/en/reference/query-builder.html#helper-methods>`_. 

Os métodos que usaremos serão ``select()``, ``from()`` e ``addOrderBy()``. 

Tal como acontece com as interações anteriores do Doctrine 2, podemos usar a notação de escrita curta para referenciar a 
entidade ``Blog`` através do ``BloggerBlogBundle:Blog`` (lembre-se que isto é o mesmo que fazer 
``Blogger\BlogBundle\Entity\Blog``). 

Quando tivermos terminado de especificar os critérios para a consulta, chamaremos o método ``getQuery()`` que retornará 
uma instância do ``DQL``. 

Não conseguimos obter resultados a partir do objeto ``QueryBuilder``, temos sempre que converter esse objeto 
para uma instância ``DQL`` primeiro. A instância ``DQL``, fornece um método ``getResult()`` que retorna uma coleção de 
entidades ``Blog``. 

Veremos, mais tarde, que a instância ``DQL`` tem vários 
`Métodos para retornar resultados <http://www.doctrine-project.org/docs/orm/2.1/en/reference/dql-doctrine-query-language.html#query-result-formats>`_ 
incluindo ``getSingleResult()`` e ``getArrayResult()``.

A Visão (View)
..............

Agora que temos uma coleção de entidades ``blog``, precisamos exibi-las. 

Substitua o conteúdo do template inicial, localizado em ``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig``, 
pelo código abaixo:

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block body %}
        {% for blog in blogs %}
            <article class="blog">
                <div class="date"><time datetime="{{ blog.created|date('c') }}">{{ blog.created|date('l, F j, Y') }}</time></div>
                <header>
                    <h2><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id }) }}">{{ blog.title }}</a></h2>
                </header>
        
                <img src="{{ asset(['images/', blog.image]|join) }}" />
                <div class="snippet">
                    <p>{{ blog.blog(500) }}</p>
                    <p class="continue"><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id }) }}">Continue lendo...</a></p>
                </div>
        
                <footer class="meta">
                    <p>Comentários: -</p>
                    <p>Postado por <span class="highlight">{{blog.author}}</span> em {{ blog.created|date('h:iA') }}</p>
                    <p>Tags: <span class="highlight">{{ blog.tags }}</span></p>
                </footer>
            </article>
        {% else %}
            <p>Não existem entradas de blog para Symblog</p>
        {% endfor %}
    {% endblock %}

Usamos algumas estruturas de controle do Twig , por exemplo, a estrutura ``for..else..endfor``. 

Se você nunca usou um gerador de templates antes, você, provavelmente, está familiarizado com o trecho de código PHP 
abaixo:

.. code-block:: php

    <?php if (count($blogs)): ?>
        <?php foreach ($blogs as $blog): ?>
            <h1><?php echo $blog->getTitle() ?><?h1>
            <!-- resto do conteúdo -->
        <?php endforeach ?>
    <?php else: ?>
        <p>Não existem entradas de blog</p>
    <?php endif ?>

A estrutura de controle do Twig ``for..else..endfor``, é uma forma mais limpa de realizar esta tarefa. A maioria do 
código, dentro do template da página inicial, está concentrado em mostrar a informação do blog em HTML. No entanto, 
existem algumas coisas que precisamos perceber. 

Em primeiro lugar, fizemos uso da função do caminho ``path`` do Twig para gerar a rota para a página de apresentação do 
blog. Como a página de apresentação do blog exige um ``id`` do blog via URL, precisamos passar este ``id`` como um 
argumento para a função ``path``. Faça o seguinte:

.. code-block:: html
    
    <h2><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id }) }}">{{ blog.title }}</a></h2>
    
Em segundo lugar, imprimimos o conteúdo do blog usando ``<p>{{blog.blog (500)}}</ p>``. O argumento ``500`` que passamos, 
é o comprimento máximo do post do blog que queremos receber de retorno da função. Para que isso funcione é preciso 
atualizar o método ``getBlog`` que o Doctrine 2 gerou anteriormente para nós. 

Atualize o método ``getBlog`` da entidade ``Blog`` localizada em ``src/Blogger/BlogBundle/Entity/ blog.php``.

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php
    public function getBlog($length = null)
    {
        if (false === is_null($length) && $length > 0)
            return substr($this->blog, 0, $length);
        else
            return $this->blog;
    }

Como o comportamento usual do método ``getBlog`` deve ser o de devolver uma entrada do post do blog, definimos o 
parâmetro ``$length`` para ter um padrão ``null``. Se ``NULL`` é passado, então, a entrada do post do blog é retornado.

Agora, se você acessar ``http://symblog.dev/app_dev.php/``, você deve ver a página mostrando as entradas dos posts do 
blog mais recentes. Você deve ser capaz de navegar, indo para a página do post do blog, clicando no título do 
blog ou clicando no link 'Continuar lendo ... '.

.. image:: /_static/images/part_4/homepage.jpg
    :align: center
    :alt: symblog homepage

Embora possamos criar consultas para entidades no controlador, aqui não é o melhor lugar para se fazer isso. Seria 
melhor colocar a consulta fora do controlador, por algumas razões:

    1. Não poderiamos reutilizar a consulta em qualquer outra parte da aplicação, sem ter que duplicar o código 
       ``QueryBuilder``.
    2. Se duplicássemos o código ``QueryBuilder``, teríamos de fazer múltiplas modificações no futuro, se fosse preciso 
       mudar a consulta.
    3. Separar a consulta e o controlador, nos permite testar a consulta, independentemente do controlador.

Doctrine 2 possui classes de repositório para facilitar este processo.

Repositórios Doctrine 2 
-----------------------

Nós já vimos algo sobre as classes de repositórios do Doctrine 2 no capítulo anterior, quando criamos a página de 
apresentação do blog. 

Utilizamos a implementação padrão da classe ``Doctrine\ORM\EntityRepository`` para recuperar uma entidade blog do banco 
de dados, através do método ``find()``. Como queremos criar uma consulta personalizada, precisamos criar um repositório 
personalizado. Doctrine 2 pode ajudar nessa tarefa. 

Atualize os metadados das entidades do ``Blog``, no arquivo ``src/Blogger/BlogBundle/Entity/blog.php``.

.. code-block:: php
    
    // src/Blogger/BlogBundle/Entity/Blog.php
    /**
     * @ORM\Entity(repositoryClass="Blogger\BlogBundle\Repository\BlogRepository")
     * @ORM\Table(name="blog")
     * @ORM\HasLifecycleCallbacks()
     */
    class Blog
    {
        // ..
    }

Você pôde ver que especificamos o local do namespace para a classe ``BlogRepository`` que esta entidade está relacionada. 

Como já atualizamos os metadados do Doctrine 2 para a entidade ``Blog``, precisamos re-executar o comando 
``doctrine:generate:entities``, como é ilustrado abaixo.

.. code-block:: bash

    $ php app/console doctrine:generate:entities Blogger
    
Doctrine 2 criará a classe shell para o ``BlogRepository``, localizado em 
``src/Blogger/BlogBundle/Repository/BlogRepository.php``.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Repository/BlogRepository.php
    
    namespace Blogger\BlogBundle\Repository;

    use Doctrine\ORM\EntityRepository;

    /**
     * BlogRepository
     *
     * This class was generated by the Doctrine ORM. Add your own custom
     * repository methods below.
     */
    class BlogRepository extends EntityRepository
    {

    }

A classe ``BlogRepository`` estende a classe ``EntityRepository`` que fornece o método ``find()`` que usamos 
anteriormente. 

Vamos atualizar a classe ``BlogRepository`` , movendo o código ``QueryBuilder`` do controlador ``Page``, para 
``BlogRepository``.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Repository/BlogRepository.php

    namespace Blogger\BlogBundle\Repository;

    use Doctrine\ORM\EntityRepository;

    /**
     * BlogRepository
     *
     * This class was generated by the Doctrine ORM. Add your own custom
     * repository methods below.
     */
    class BlogRepository extends EntityRepository
    {
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
    }

Criamos o método ``getLatestBlogs`` que irá retornar as entradas mais recentes do blog, da mesma forma que o código do 
controlador ``QueryBuilder`` fez. 

Na classe repositório, temos acesso direto ao ``QueryBuilder`` através do método ``createQueryBuilder()``. Nós também 
adicionamos um parâmetro padrão ``$limit`` para que possamos limitar o número de resultados retornados. 

O resultado da consulta, é muito semelhante ao que temos no controlador. Você deve ter notado que não especificamos qual 
entidade usar, no método ``from()``. Isso é porque nós estamos dentro do ``BlogRepository`` que está associado com a 
entidade ``Blog``. 

Se prestarmos atenção na implementação do método ``createQueryBuilder``, na classe ``EntityRepository``, poderemos ver o 
método ``from()`` sendo invocado.

.. code-block:: php
    
    // Doctrine\ORM\EntityRepository
    public function createQueryBuilder($alias)
    {
        return $this->_em->createQueryBuilder()
            ->select($alias)
            ->from($this->_entityName, $alias);
    }

Finalmente, vamos atualizar a ação ``index`` do controlador ``Page`` para usar o ``BlogRepository``.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    class PageController extends Controller
    {
        public function indexAction()
        {
            $em = $this->getDoctrine()
                       ->getEntityManager();
                       
            $blogs = $em->getRepository('BloggerBlogBundle:Blog')
                        ->getLatestBlogs();
                       
            return $this->render('BloggerBlogBundle:Page:index.html.twig', array(
                'blogs' => $blogs
            ));
        }
        
        // ..
    }

Agora, quando você atualizar a página inicial, deve ser exibido exatamente o mesmo de antes. Tudo o que nós fizemos, foi 
colocar nosso código nas classes corretas para que possam realizar as tarefas corretas.

Mais sobre o Model: Criando a Entidade Comentário
-------------------------------------------------

Os blogs são apenas metade da história. Precisamos permitir que os leitores comentem os posts do blog. Estes comentários 
também precisam ser persistentes e ligados à entidade ``Blog``, pois um blog pode conter muitos comentários.

Vamos começar definindo os conceitos básicos da classe de entidade ``Comment``. 

Crie um novo arquivo localizado em ``src/Blogger/BlogBundle/Entity/Comment.php`` e cole o seguinte código:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Comment.php

    namespace Blogger\BlogBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity(repositoryClass="Blogger\BlogBundle\Repository\CommentRepository")
     * @ORM\Table(name="comment")
     * @ORM\HasLifecycleCallbacks()
     */
    class Comment
    {
        /**
         * @ORM\Id
         * @ORM\Column(type="integer")
         * @ORM\GeneratedValue(strategy="AUTO")
         */
        protected $id;

        /**
         * @ORM\Column(type="string")
         */
        protected $user;

        /**
         * @ORM\Column(type="text")
         */
        protected $comment;

        /**
         * @ORM\Column(type="boolean")
         */
        protected $approved;
        
        /**
         * @ORM\ManyToOne(targetEntity="Blog", inversedBy="comments")
         * @ORM\JoinColumn(name="blog_id", referencedColumnName="id")
         */
        protected $blog;

        /**
         * @ORM\Column(type="datetime")
         */
        protected $created;

        /**
         * @ORM\Column(type="datetime")
         */
        protected $updated;

        public function __construct()
        {
            $this->setCreated(new \DateTime());
            $this->setUpdated(new \DateTime());
            
            $this->setApproved(true);
        }

        /**
         * @ORM\preUpdate
         */
        public function setUpdatedValue()
        {
           $this->setUpdated(new \DateTime());
        }
    }

O conteúdo do código acima, já foi abordado no capítulo anterior, porém, usamos metadados para criar um link para a 
entidade ``Blog``. Como comentário é para um post de um blog, temos que configurar um link na entidade ``Comment`` para
pertencer à entidade ``Blog``. 

Especificamos um link ``ManyToOne`` visando a entidade ``Blog``. Também especificamos que o inverso estará disponível em 
``comments``. Para isso, precisamos atualizar a entidade ``Blog`` para que o Doctrine 2 saiba que um blog pode conter 
muitos comentários. 

Atualize a entidade ``Blog``, localizada em ``src/Blogger/BlogBundle/Entity/blog.php``, adicionando este mapeamento:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    namespace Blogger\BlogBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;
    use Doctrine\Common\Collections\ArrayCollection;

    /**
     * @ORM\Entity(repositoryClass="Blogger\BlogBundle\Repository\BlogRepository")
     * @ORM\Table(name="blog")
     * @ORM\HasLifecycleCallbacks()
     */
    class Blog
    {
        // ..
        
        /**
         * @ORM\OneToMany(targetEntity="Comment", mappedBy="blog")
         */
        protected $comments;
        
        // ..
        
        public function __construct()
        {
            $this->comments = new ArrayCollection();
            
            $this->setCreated(new \DateTime());
            $this->setUpdated(new \DateTime());
        }
        
        // ..
    }

Existem algumas considerações aqui. 

    Primeiro, adicionamos metadados aos membros ``$comments``. Lembre-se, no capítulo anterior, não adicionamos qualquer 
    metadado para este membro porque nós não queriamos que o Doctrine 2 os manipulasse. Isso ainda é verdade, mas, 
    queremos que o Doctrine 2 possa preencher esse membro com a entidade ``Comment`` relativa. Isso é o que ativa os 
    metadados. 

    Segundo, Doctrine 2 pede que transformemos os membros ``$comments`` em um objeto ``ArrayCollection``. Isso deve 
    ser feito no ``construtor``. Além disso, observe a declaração de ``use`` importar a classe ``ArrayCollection``.

Como criamos a entidade ``Comment``, e atualizamos a entidade ``Blog``, vamos deixar que o Doctrine 2 gere os assessores. 

Execute o seguinte comando Doctrine 2:

.. code-block:: bash

    $ php app/console doctrine:generate:entities Blogger
    
Ambas as entidades devem estar atualizadas com os métodos de acesso corretos. Você irá notar que a classe 
``CommentRepository`` foi criada em ``src/Blogger/BlogBundle/Repository/CommentRepository.php`` como especificado 
nos metadados.

Finalmente, precisamos atualizar o banco de dados para refletir as mudanças de nossas entidades. Nós podemos usar a 
funcionalidade ``doctrine:schema:update`` da seguinte forma, mas em vez disso, vamos introduzir as migrações do 
Doctrine 2.

.. code-block:: bash

    $ php app/console doctrine:schema:update --force

Migrações Doctrine 2 (Doctrine 2 Migrations)
--------------------------------------------

As extensões e pacotes das Migrações do Doctrine 2, não vem com a distribuição Standard do Symfony 2, é preciso 
instalá-las manualmente, como fizemos com as extensões e pacotes dos Data Fixtures. 

Abra o arquivo ``deps``, localizado na raiz do projeto, e adicione a extensão e pacotes das Migrações do Doctrine 2, como 
se segue abaixo:

.. code-block:: text
    
    [doctrine-migrations]
        git=http://github.com/doctrine/migrations.git

    [DoctrineMigrationsBundle]
        git=http://github.com/symfony/DoctrineMigrationsBundle.git
        target=/bundles/Symfony/Bundle/DoctrineMigrationsBundle

Em seguida, atualizamos os ``Vendors`` para refletir essas alterações.

.. code-block:: bash

    $ php bin/vendors install

Isso vai baixar e instalar a versão mais recente de cada um dos repositórios do GitHub nos locais corretos.

.. note::

    Se você estiver usando uma máquina que não tem Git instalado, você terá que baixar e instalar a extensão e o pacote 
    manualmente.

    Extensão doctrine-migrations: `Faça o download <http://github.com/doctrine/migrations>`_ da versão atual do pacote e 
    extraia para a seguinte localização: ``vendor/doctrine-migrations``.

    DoctrineMigrationsBundle: `Faça o download <http://github.com/symfony/DoctrineMigrationsBundle>`_ da versão atual do 
    pacote e extraia para a seguinte localização: ``vendor/bundles/Symfony/Bundle/DoctrineMigrationsBundle``.

Atualize o arquivo ``app/autoload.php`` para registrar o novo namespace. Como as migrações do Doctrine 2 estão no 
namespace ``Doctrine\DBAL``, eles devem ser colocados acima das configurações ``Doctrine\DBAL`` existentes, especificando 
um novo caminho. 

Namespaces são verificados de cima para baixo para namespaces. Então, namespaces mais específicos precisam ser 
registrados antes dos menos específicos.

.. code-block:: php

    // app/autoload.php
    // ...
    $loader->registerNamespaces(array(
    // ...
    'Doctrine\\DBAL\\Migrations' => __DIR__.'/../vendor/doctrine-migrations/lib',
    'Doctrine\\DBAL'             => __DIR__.'/../vendor/doctrine-dbal/lib',
    // ...
    ));

Agora, vamos registrar o pacote no kernel. Vá em ``app/AppKernel.php``.

.. code-block:: php

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Symfony\Bundle\DoctrineMigrationsBundle\DoctrineMigrationsBundle(),
            // ...
        );
        // ...
    }

.. warning::

    A biblioteca Doctrine 2 Migrations, ainda está em em fase de teste. Seu uso em servidores de produção deve ser 
    desencorajado, por enquanto.

Agora, estamos prontos para atualizar o banco de dados para refletir as alterações da entidade. Este é o segundo passo do 
processo. 

Primeiro, precisamos fazer com que as Migrações do Doctrine 2 trabalhem as diferenças entre as entidades e o esquema do 
banco de dados atual. Isto é feito com a funcionalidade ``doctrine:migrations:diff``. 

Segundo, precisamos, realmente, fazer a migração com base no dif criado anteriormente. Isto é feito com a funcionalidade 
``doctrine:migrations: migrate``.

Execute os comandos abaixo, para atualizar o esquema do banco de dados:

.. code-block:: bash

    $ php app/console doctrine:migrations:diff
    $ php app/console doctrine:migrations:migrate

Seu banco de dados vai refletir as alterações mais recentes das entidade e irá conter a nova tabela comment.

.. note::

    Você deve ter notado uma nova tabela no banco de dados, chamada ``migration_versions``. Esta tabela, armazena os 
    números das versões das migrações, para a funcionalidade de migração ser capaz de saber qual é a versão atual do 
    banco de dados.
    
.. tip::

    As migrações do Doctrine 2 são uma ótima maneira de atualizar o banco de dados de produção, pois as mudanças podem 
    ser feitas de forma programada. Isto significa que podemos integrar esta funcionalidade em um script de 
    desenvolvimento para que o banco de dados seja atualizado, automaticamente, quando implantamos uma nova versão da 
    aplicação. 

    As migrações do Doctrine 2 permitem reverter as alterações, pois cada migração tem criado um método ``up`` e 
    ``down``. Para reverter para uma versão anterior, você precisa especificar o número da versão que você gostaria de 
    reverter, executando o seguinte código:
    
    .. code-block:: bash
    
        $ php app/console doctrine:migrations:migrate 20110806183439
        
Data Fixtures: Revisão
----------------------

Agora que temos a entidade ``Comment`` criada, vamos adicionar alguns fixtures para ela. É sempre uma boa ideia 
adicionar alguns fixtures, cada vez que criamos uma entidade. 

Sabemos que um comentário deve ter uma entidade ``Blog`` relacionada, de acordo com o que foi configurado nos metadados, 
portanto, quando criamos Data Fixtures para a entidade ``Comments``, vamos ter de especificar a entidade ``Blog``. 

Já criamos os fixtures para a entidade ``Blog``, então, vamos  simplesmente atualizar esse arquivo para adicionar a 
entidade ``comment``. 

Isso é viável agora, mas, o que acontece quando, posteriormente, adicionarmos usuários, categorias do blog, e outras 
entidades para o nosso pacote (Bundle)? 

A melhor maneira, seria criar um novo arquivo para a entidade ``Comment``. O problema com esta abordagem é que vamos 
acessar a entidade ``blog`` através dos fixtues do blog.

Felizmente, conseguimos, facilmente, ajustar as referências à objetos em um arquivo de fixture para que possa ser 
acessado. 

Atualize a entidade ``Blog DataFixtures``, localizado em ``src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php``, 
com o código baixo. 

As mudanças que devemos observados aqui são, a extensão da classe ``AbstractFixture`` e a implementação do 
``OrderedFixtureInterface``. Observe também, o uso das declarações de importação dessas classes.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php

    namespace Blogger\BlogBundle\DataFixtures\ORM;

    use Doctrine\Common\DataFixtures\AbstractFixture;
    use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
    use Doctrine\Common\Persistence\ObjectManager;
    use Blogger\BlogBundle\Entity\Blog;

    class BlogFixtures extends AbstractFixture implements OrderedFixtureInterface
    {
        public function load(ObjectManager $manager)
        {
            // ..

            $manager->flush();

            $this->addReference('blog-1', $blog1);
            $this->addReference('blog-2', $blog2);
            $this->addReference('blog-3', $blog3);
            $this->addReference('blog-4', $blog4);
            $this->addReference('blog-5', $blog5);
        }

        public function getOrder()
        {
            return 1;
        }
    }

Adicionamos as referências às entidades de blog usando o método ``addReference()``. Este primeiro parâmetro é um 
identificador de referência que podemos usar para recuperar o objeto a qualquer momento. 

Finalmente, implementamos o método ``getOrder()`` para especificar a ordem de carregamento dos fixtures. 

Blogs deve ser carregado antes dos comentários para que retorne 1.

Fixtures de Comentários
~~~~~~~~~~~~~~~~~~~~~~~

Agora, estamos prontos para definir alguns fixtures para a nossa entidade ``Comment``. 

Crie um arquivo de fixture em ``src/Blogger/BlogBundle/DataFixtures/ORM/CommentFixtures.php`` e adicione o seguinte 
conteúdo:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/DataFixtures/ORM/CommentFixtures.php
    
    namespace Blogger\BlogBundle\DataFixtures\ORM;
    
    use Doctrine\Common\DataFixtures\AbstractFixture;
    use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
    use Doctrine\Common\Persistence\ObjectManager;
    use Blogger\BlogBundle\Entity\Comment;
    use Blogger\BlogBundle\Entity\Blog;
    
    class CommentFixtures extends AbstractFixture implements OrderedFixtureInterface
    {
        public function load(ObjectManager $manager)
        {
            $comment = new Comment();
            $comment->setUser('symfony');
            $comment->setComment('Para fazer um pequena longa história. Você não se arrependerá, escolhendo Symfony! E ninguém jamais foi demitido por usar Symfony.');
            $comment->setBlog($manager->merge($this->getReference('blog-1')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('David');
            $comment->setComment('Para fazer um pequena longa história. Escolher um framework não deve ser uma escolha feita sem pensar, é um compromisso de longo prazo. Certifique-se de fazer a escolha certa!');
            $comment->setBlog($manager->merge($this->getReference('blog-1')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('Algo mais mãe? Você quer que eu corte a grama? Oops! Eu esqueci, New York, sem grama.');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('Você está me desafiando? ');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:15:20"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('Façam as suas apostas.');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:18:35"));
            $manager->persist($comment);
            
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('Se eu ganhar, vocẽ será mey escravo.');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:22:53"));
            $manager->persist($comment);
            
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('Seu ESCRAVO?');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:25:15"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('Você quer! Você vai fazer o trabalho de merda, digitalização, os direitos autorais de crack...');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 06:46:08"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('E se EU ganhar?');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 10:22:46"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('Tornarei isso o meu primogênito!');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-23 11:08:08"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Dade');
            $comment->setComment('Tornaremos isso o nosso primogênito!');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-24 18:56:01"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Kate');
            $comment->setComment('Eu não procuro encontros. Mas eu não perco, por isso você está aqui!');
            $comment->setBlog($manager->merge($this->getReference('blog-2')));
            $comment->setCreated(new \DateTime("2011-07-25 22:28:42"));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Stanley');
            $comment->setComment('Isso não vai terminar bem');
            $comment->setBlog($manager->merge($this->getReference('blog-3')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Gabriel');
            $comment->setComment('Oh, vamos lá, Stan. Nem tudo termina do jeito que você acha que deveria. Além disso, o público adora finais felizes.');
            $comment->setBlog($manager->merge($this->getReference('blog-3')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Mile');
            $comment->setComment('Será que Bill Gates tem algo como isso aqui?');
            $comment->setBlog($manager->merge($this->getReference('blog-5')));
            $manager->persist($comment);
    
            $comment = new Comment();
            $comment->setUser('Gary');
            $comment->setComment('Bill quem?');
            $comment->setBlog($manager->merge($this->getReference('blog-5')));
            $manager->persist($comment);
    
            $manager->flush();
        }
    
        public function getOrder()
        {
            return 2;
        }
    }
        
Tal como acontece com as modificações que fizemos na classe ``BlogFixtures``, a classe ``CommentFixtures`` também 
estende a classe ``AbstractFixture`` e implementa a ``OrderedFixtureInterface``. Isso significa que também devemos 
implementar o método ``getOrder()``. 

Desta vez, vamos definir o valor de retorno para 2, garantindo que esses fixtures serão carregados depois dos fixtures 
do blog.

Podemos ver, como as referências para a entidade ``Blog``, que criamos anteriormente, estão sendo utilizadas.

.. code-block:: php

    $comment->setBlog($manager->merge($this->getReference('blog-2')));

Agora, estamos prontos para carregar os fixtures para o banco de dados.

.. code-block:: bash

    $ php app/console doctrine:fixtures:load
    
Exibindo Comentários
--------------------

Agora, podemos exibir os comentários relacionados a cada post do blog. 

Vamos atualizar o ``CommentRepository`` com um método para recuperar os comentários aprovados mais recentes de um post 
do blog.

Repositório de Comentários 
~~~~~~~~~~~~~~~~~~~~~~~~~~

Abra a classe ``CommentRepository``, localizada em ``src/Blogger/BlogBundle/Repository/CommentRepository.php`` e 
substitua o seu conteúdo pelo seguinte código:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Repository/CommentRepository.php

    namespace Blogger\BlogBundle\Repository;

    use Doctrine\ORM\EntityRepository;

    /**
     * CommentRepository
     *
     * This class was generated by the Doctrine ORM. Add your own custom
     * repository methods below.
     */
    class CommentRepository extends EntityRepository
    {
        public function getCommentsForBlog($blogId, $approved = true)
        {
            $qb = $this->createQueryBuilder('c')
                       ->select('c')
                       ->where('c.blog = :blog_id')
                       ->addOrderBy('c.created')
                       ->setParameter('blog_id', $blogId);
            
            if (false === is_null($approved))
                $qb->andWhere('c.approved = :approved')
                   ->setParameter('approved', $approved);
                   
            return $qb->getQuery()
                      ->getResult();
        }
    }
    
O método que criamos, irá recuperar comentários de um post do blog. Para isso, precisamos adicionar uma cláusula 
``where`` em nossa consulta. A cláusula ``where`` usa um parâmetro nomeado que é definido usando o método 
``setParameter()``. 

Você deve sempre usar parâmetros, em vez de definir os valores diretamente na consulta, como o exemplo abaixo:
    
.. code-block:: php

    ->where('c.blog = ' . blogId)

Neste exemplo, o valor de ``$blogId`` não será tratado e poderia deixar a consulta aberta para um ataque de 
`SQL injection <http://en.wikipedia.org/wiki/SQL_injection>`_.

O Controlador do Blog
---------------------

Agora, precisamos atualizar a ação ``show`` do controlador do ``Blog`` para recuperar os comentários. 

Atualize o controlador do ``Blog``, localizado em ``src/Blogger/BlogBundle/controller/BlogController.php``, com o 
seguinte código:

.. code-block:: php
    
    // src/Blogger/BlogBundle/Controller/BlogController.php
    
    public function showAction($id)
    {
        // ..

        if (!$blog) {
            throw $this->createNotFoundException('Unable to find Blog post.');
        }
        
        $comments = $em->getRepository('BloggerBlogBundle:Comment')
                       ->getCommentsForBlog($blog->getId());
        
        return $this->render('BloggerBlogBundle:Blog:show.html.twig', array(
            'blog'      => $blog,
            'comments'  => $comments
        ));
    }

Usamos o novo método ``CommentRepository`` para recuperar os comentários aprovados para o blog. A coleção ``$comments`` 
também é passado para o template.

O template Show do Blog
~~~~~~~~~~~~~~~~~~~~~~~

Agora que temos uma lista dos comentários para o blog, podemos atualizar o template ``show`` do blog para exibir os 
comentários. 

Poderíamos, simplesmente, colocar a renderização dos comentários diretamente no template ``show`` do blog, mas, como os 
comentários tem a sua própria entidade, seria melhor separar a renderização em outro template para a inclusão do 
comentário. Com isso, é possível reutilizar o template renderizado de comentários em outras partes do aplicação. 

Atualize o template ``show`` do blog, localizado em ``src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig``, com 
o seguinte código:

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig #}
    
    {# .. #}
    
    {% block body %}
        {# .. #}
    
        <section class="comments" id="comments">
            <section class="previous-comments">
                <h3>Comentários</h3>
                {% include 'BloggerBlogBundle:Comment:index.html.twig' with { 'comments': comments } %}
            </section>
        </section>
    {% endblock %}
    
Como você pôde ver, usamos uma nova tag do Twig , a tag ``include``. Assim, iremos incluir o conteúdo do template 
especificado por ``BloggerBlogBundle:Comment:index.html.twig``.

Podemos passar qualquer número de argumentos para o template. Neste caso, foi passado uma coleção de entidades de 
``Comment`` para ser renderizado.

O Template Show dos Comentarios
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

O ``BloggerBlogBundle:Comment:index.html.twig``, que estavamos incluindo acima, ainda não existe, precisamos criá-lo. 
Como é apenas um template, não precisamos criar uma rota ou um controlador para ele, precisamos apenas do arquivo de 
template. 

Crie um novo arquivo, localizado em ``src/Blogger/BlogBundle/Recursos/views/Resources/index.html.twig``, e cole o 
seguinte código:

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Comment/index.html.twig #}
    
    {% for comment in comments %}
        <article class="comment {{ cycle(['odd', 'even'], loop.index0) }}" id="comment-{{ comment.id }}">
            <header>
                <p><span class="highlight">{{ comment.user }}</span> comentou <time datetime="{{ comment.created|date('c') }}">{{ comment.created|date('l, F j, Y') }}</time></p>
            </header>
            <p>{{ comment.comment }}</p>
        </article>
    {% else %}
        <p>Não existem comentários para este post. Seja o primeiro a comentar...</p>
    {% endfor %}

Como você pôde ver, iteramos uma coleção de entidades ``Comment`` e exibimos os comentários. Mostramos também, uma outra 
função útil do Twig, a função de ``ciclo``. Esta função irá percorrer os valores do array passado em cada iteração da 
execução do loop. 

O valor atual da iteração do loop é obtido através da variável especial ``loop.index0``. Esta variável mantém uma 
contagem de iterações do loop, começando de 0. 

Temos outras `Variáveis especiais <http://www.twig-project.org/doc/templates.html#for>`_ disponíveis, quando precisamos 
usar um bloco de código de loop. 

Você também pôde perceber que precisamos informar um ID para o elemento HTML ``article``. Assim, podemos criar links 
para os comentário criados, quando necessário.

CSS do template Show dos Comentários
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 

Finalmente, vamos adicionar um pouco de CSS para manter o layout dos comentários mais elegante. 

Atualize a folha de estilos localizada em ``src/Blogger/BlogBundle/Resorces/public/css/blog.css`` com o seguinte código:

.. code-block:: css

    /** src/Blogger/BlogBundle/Resorces/public/css/blog.css **/
    .comments { clear: both; }
    .comments .odd { background: #eee; }
    .comments .comment { padding: 20px; }
    .comments .comment p { margin-bottom: 0; }
    .comments h3 { background: #eee; padding: 10px; font-size: 20px; margin-bottom: 20px; clear: both; }
    .comments .previous-comments { margin-bottom: 20px; }

.. note::

    Se você não estiver usando o método de link simbólico para referenciar os ``assets`` do pacote para a pasta ``web``, 
    você deve re-instalar os ``assets`` para aplicar as alterações no seu CSS.

    .. code-block:: bash

        $ php app/console assets:install web
        
Se você der uma olhada em uma das páginas de exibição do blog, por exemplo, ``http://symblog.dev/app_dev.php/2``, você 
deve ver a página de comentários do blog como a ilustrada abaixo:

.. image:: /_static/images/part_4/comments.jpg
    :align: center
    :alt: Exibição dos comentários do Symblog
    
Adicionando comentários
-----------------------

Para a última parte deste capítulo, iremos adicionar a funcionalidade para os usuários poderem adicionar comentários a um 
post do blog. Isso será possível através de um formulário na página de apresentação do blog. 

Já sabemos como criar um formulário em Symfony 2, isso foi mostrado quando criamos o formulário de contato. Em vez de 
criar manualmente o formulário de comentário, podemos usar Symfony 2 para fazer isso. 

Execute o seguinte código para gerar a classe ``CommentType`` para a entidade ``Comment``.

.. code-block:: bash
    
    $ php app/console generate:doctrine:form BloggerBlogBundle:Comment
    
Perceba, novamente, a utilização de atalhos para especificar a entidade ``Comment``.

.. tip::

    Você deve ter percebido que a funcionalidade ``doctrine:generate:form`` também está disponível. É a mesma coisa, só 
    foi adicionado o namespace de forma diferente.
    
A classe ``CommentType`` do formulário, foi criada em ``src/Blogger/BlogBundle/Form/CommentType.php``.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Form/CommentType.php
    
    namespace Blogger\BlogBundle\Form;
    
    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;
    
    class CommentType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder
                ->add('user')
                ->add('comment')
                ->add('approved')
                ->add('created')
                ->add('updated')
                ->add('blog')
            ;
        }
    
        public function getName()
        {
            return 'blogger_blogbundle_commenttype';
        }
    }

Já vimos o que acontece aqui, ao ter criado a classe ``EnquiryType``. 

Poderíamos personalizar esta classe agora, mas vamos passar para a exibição do formulário primeiro.

Exibindo o formulário de comentário
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 

Como queremos que o usuário adicione seus comentários da página de exibição do blog, poderíamos criar o formulário na 
ação ``show`` do controlador ``Blog`` e renderizar o formulário diretamente no template ``show``. No entanto, seria 
melhor separar este código, como fizemos com a exibição dos comentários. 

A diferença entre mostrar os comentários e apresentar o formulário de comentário é que o formulário de comentário 
precisa ser processado, então, um controlador é necessário. 

Esse método é ligeiramente diferente do abordado acima, por isso, vamos apenas incluir o template.

Rota
~~~~

Precisamos criar uma nova rota para lidar com o processamento dos formulários enviados. 

Adicione uma nova rota no arquivo de rota localizado em ``src/Blogger/BlogBundle/resources/config/routing.yml``.

.. code-block:: yaml

    BloggerBlogBundle_comment_create:
        pattern:  /comment/{blog_id}
        defaults: { _controller: BloggerBlogBundle:Comment:create }
        requirements:
            _method:  POST
            blog_id: \d+
        
O controlador
~~~~~~~~~~~~~

Agora, precisamos criar o novo controlador ``Comment`` que mencionamos acima. 

Crie um arquivo localizado em ``src/Blogger/BlogBundle/controller/CommentController.php`` e cole o seguinte código:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/CommentController.php
    
    namespace Blogger\BlogBundle\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Blogger\BlogBundle\Entity\Comment;
    use Blogger\BlogBundle\Form\CommentType;
    
    /**
     * Comment controller.
     */
    class CommentController extends Controller
    {
        public function newAction($blog_id)
        {
            $blog = $this->getBlog($blog_id);
            
            $comment = new Comment();
            $comment->setBlog($blog);
            $form   = $this->createForm(new CommentType(), $comment);
    
            return $this->render('BloggerBlogBundle:Comment:form.html.twig', array(
                'comment' => $comment,
                'form'   => $form->createView()
            ));
        }
    
        public function createAction($blog_id)
        {
            $blog = $this->getBlog($blog_id);
            
            $comment  = new Comment();
            $comment->setBlog($blog);
            $request = $this->getRequest();
            $form    = $this->createForm(new CommentType(), $comment);
            $form->bindRequest($request);
    
            if ($form->isValid()) {
                // TODO: Persist the comment entity
    
                return $this->redirect($this->generateUrl('BloggerBlogBundle_blog_show', array(
                    'id' => $comment->getBlog()->getId())) .
                    '#comment-' . $comment->getId()
                );
            }
    
            return $this->render('BloggerBlogBundle:Comment:create.html.twig', array(
                'comment' => $comment,
                'form'    => $form->createView()
            ));
        }
        
        protected function getBlog($blog_id)
        {
            $em = $this->getDoctrine()
                        ->getEntityManager();
    
            $blog = $em->getRepository('BloggerBlogBundle:Blog')->find($blog_id);
    
            if (!$blog) {
                throw $this->createNotFoundException('Unable to find Blog post.');
            }
            
            return $blog;
        }
       
    }
    
Nós criamos 2 ações no controlador ``Comment``, uma para ``new`` e um para ``create``. A ação ``new`` está preocupada em 
exibir o formulário de comentário. A ação ``create`` está preocupada em processar a apresentação do formulário de 
comentário. 

Embora isso possa parecer estranho, não há nada novo aqui, tudo foi abordado no capítulo 2, quando criamos o formulário 
de contato. 

No entanto, antes de seguirmos, certifique-se de ter entendido completamente o que está acontecendo no controlador 
``Comment``.

Validação do formulário
~~~~~~~~~~~~~~~~~~~~~~~ 

Não queremos que os usuários enviem comentários do blog com o ``usuário`` ou ``comentário`` com  valores em branco ou 
vazios. Assim, voltemos aos validadores que foram mostrados na parte 2 ao criar o formulário de contato. 

Atualize a entidade ``Comment``, localizada em ``src/Blogger/BlogBundle/Entity/Comment.php``, com o seguinte código:

.. code-block:: php
    
    <?php
    // src/Blogger/BlogBundle/Entity/Comment.php
    
    // ..
    
    use Symfony\Component\Validator\Mapping\ClassMetadata;
    use Symfony\Component\Validator\Constraints\NotBlank;
    
    // ..
    class Comment
    {
        // ..
        
        public static function loadValidatorMetadata(ClassMetadata $metadata)
        {
            $metadata->addPropertyConstraint('user', new NotBlank(array(
                'message' => 'Vocẽ deve fornecer um nome'
            )));
            $metadata->addPropertyConstraint('comment', new NotBlank(array(
                'message' => 'Você deve fornecer um comentário'
            )));
        }
        
        // ..
    }

As restrições garantem que, tanto o usuário e o comentário, não possam ser passados em branco.

Temos, também, que definir as opções das ``mensagems`` para estas restrições, para substituir as mensagens padrões. 

Lembre-se de adicionar o namespace para ``ClassMetadata`` e ``NotBlank`` como mostrado acima.

A View
~~~~~~

Precisamos criar os 2 templates para as 2 ações do controlador ``new`` e ``create``. 

Crie um novo arquivo em ``src/Blogger/BlogBundle/Resources/views/Comment/form.html.twig`` e cole o seguinte código:

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/Comment/form.html.twig #}
    
    <form action="{{ path('BloggerBlogBundle_comment_create', { 'blog_id' : comment.blog.id } ) }}" method="post" {{ form_enctype(form) }} class="blogger">
        {{ form_widget(form) }}
        <p>
            <input type="submit" value="Enviar">
        </p>
    </form>

O objetivo deste template é simples, ele apenas renderiza o formulário de comentário. Perceba que a ``ação`` do 
formulário é ``POST`` para a nova rota que criamos em ``BloggerBlogBundle_comment_create``.

Agora, vamos adicionar o template para o ``create``. 

Crie um novo arquivo em ``src/Blogger/BlogBundle/Resources/views/Comment/create.html.twig`` e cole o seguinte código:

.. code-block:: html

    {% extends 'BloggerBlogBundle::layout.html.twig' %}
    
    {% block title %}Add Comment{% endblock%}
    
    {% block body %}
        <h1>Add comment for blog post "{{ comment.blog.title }}"</h1>
        {% include 'BloggerBlogBundle:Comment:form.html.twig' with { 'form': form } %}    
    {% endblock %}

À medida que a ação ``create`` do controlador ``Comment`` processa o formulário, ela também precisa ser capaz de 
exibi-lo, caso existam erros. 

Reutilizaremos o ``BloggerBlogBundle:Comment:form.html.twig`` para renderizar o formulário atual, para evitar a 
duplicação de código.

Agora, vamos atualizar o template de exibição do blog para renderizar o formulário de inserção de comentário do blog. 

Atualize o template localizado em ``src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig`` com o seguinte código:

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Blog/show.html.twig #}
    
    {# .. #}
    
    {% block body %}
    
        {# .. #}
        
        <section class="comments" id="comments">
            {# .. #}
            
            <h3>Adicione um Comentário</h3>
            {% render 'BloggerBlogBundle:Comment:new' with { 'blog_id': blog.id } %}
        </section>
    {% endblock %}

Usamos uma outra tag nova do Twig, a tag ``render``. Esta tag irá processar o conteúdo de um controlador para o template. 
No nosso caso, renderizamos o conteúdo da ação ``BloggerBlogBundle:Comment:new`` do controlador.

Se você acessar uma das páginas de exibição do blog, como ``http://symblog.dev/app_dev.php/2``, você verá uma página de 
exceção do Symfony 2.

.. image:: /_static/images/part_4/to_string_error.jpg
    :align: center
    :alt: Exceção toString() do Symfony 2
    
Essa exceção está sendo exibida pelo template ``BloggerBlogBundle:Blog:show.html.twig``. Se formos para a linha 25 do 
template ``BloggerBlogBundle:Blog:show.html.twig``, veremos que o problema da linha realmente existe.

.. code-block:: html

    {% render 'BloggerBlogBundle:Comment:create' with { 'blog_id': blog.id } %}
    
Se observarmos a mensagem de exceção, ela ainda nos dá mais algumas informações sobre o por que essa exceção ocorreu.

    Entities passed to the choice field must have a "__toString()" method defined
    (Entidades transformadas em campo de escolha, devem ter um método "__toString ()" definido)

Esta mensagem, nos diz que um campo de escolha que nós estamos tentando renderizar, não tem um método ``__toString()`` 
definido para a entidade, cujo campo de escolha está associado. 

Um campo de escolha é um elemento de formulário que dá ao usuário uma série de opções de escolha, como um ``select`` 
(drop down). 

Você pode estar se perguntando, onde estamos renderizando o campo de escolha do formulário de comentário? Se você 
observar o template do formulário de comentário novamente, você vai perceber que renderizamos o formulário usando a 
função Twig ``{{form_widget (form)}}``. Esta função gera todos os elementos básicos do formulário. 

Então, vamos voltar para a classe que cria o formulário ``CommentType``. Podemos ver que uma série de campos estão sendo 
adicionados ao formulário através do objeto ``FormBuilder``. Em particular, estamos adicionando campos do ``blog``.

Se você se lembra do capítulo 2, falamos sobre como o ``FormBuilder`` tenta descobrir o tipo de campo, baseado nos 
metadados relacionados a este campo.  

Como configuramos a relação entre as entidades ``Comment`` e ``Blog``, o ``FormBuilder`` já descobriu que o blog poderia 
ser um ``campo de escolha``, permitindo que o usuário escolha para qual post do blog o comentário vai. É por isso que 
temos um ``campo de escolha`` no formmulário e um erro de exceção do Symfony 2. 

Podemos resolver este problema adicionando o método ``__toString()`` na entidade ``Blog``.

.. code-block:: php
    
    // src/Blogger/BlogBundle/Entity/Blog.php
    public function __toString()
    {
        return $this->getTitle();
    }

.. tip::

    As mensagens de erro do Symfony 2 são bem informativas, quando se trata de descrever o problema que ocorreu. Leia 
    sempre as mensagens de erro, pois elas tornam o processo de depuração muito mais fácil. 

    As mensagens de erro, também, fornecem uma relação completa do que causou o erro.
    
Agora, quando você atualizar a página, você deve ver o formulário de comentário. Você irá notar que alguns campos 
indesejáveis foram retornados, como ``approved``, ``create``, ``updated`` e ``blog``. Isto é porque nós não 
personalizamos a classe ``CommentType``, gerada anteriormente.

.. tip::

    Os campos a serem renderizados, terão a saída correta de acordo com o tipo de campo. O campo ``user`` é um campo de 
    texto ``text``, o campo ``comment`` é um ``textarea``, os 2 campos ``datetime`` são vários campos ``select`` 
    permitindo especificar a data completa com horário, etc.
    
    Isto é possível, graças à capacidade do ``FormBuilder`` descobrir o tipo de campo do membro que está renderizando. 
    Ele consegue fazer isso baseado em metadados fornecidos. Como especificamos os metadados para a entidade 
    ``Comment``, o ``FormBuilder`` é capaz de fazer estimativas precisas dos tipos de campo.

Vamos, agora, atualizar esta classe, localizada em ``src/Blogger/BlogBundle/Form/CommentType.php``, para exibir somente 
os campos que precisamos. 

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Form/CommentType.php
    
    // ..
    class CommentType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder
                ->add('user')
                ->add('comment')
            ;
        }
    
        // ..
    }

Agora, quando você atualizar a página, somente o usuário e campo de comentários são exibidos. 

Se você enviar o formulário agora, o comentário não seria realmente salvo no banco de dados. Isso ocorre, porque o 
controlador do formulário, não faz nada com a entidade ``Comment`` para que possa ser validado. 

Então, como vamos trabalhar com a entidade ``Comment``, para usar o banco de dados? Você já viu como fazer isso ao criar 
``DataFixtures``. 

Atualize a ação ``create`` do controlador ``Comment``, para trabalhar com a entidade do banco de dados ``Comment``.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/CommentController.php
    
    // ..
    class CommentController extends Controller
    {
        public function createAction($blog_id)
        {
            // ..
            
            if ($form->isValid()) {
                $em = $this->getDoctrine()
                           ->getEntityManager();
                $em->persist($comment);
                $em->flush();
                    
                return $this->redirect($this->generateUrl('BloggerBlogBundle_blog_show', array(
                    'id' => $comment->getBlog()->getId())) .
                    '#comment-' . $comment->getId()
                );
            }
        
            // ..
        }
    }



Agora, conseguimos adicionar comentários aos posts do blog.

Persistir a entidade ``Comment`` é tão simples quanto chamar ``persist()`` e ``flush()``. Lembre-se, o formulário só 
lida com objetos do PHP, e Doctrine 2 gerencia a persistência desses objetos. Não há conexão direta entre a apresentação 
de um formulário e os dados apresentados .

.. image:: /_static/images/part_4/add_comments.jpg
    :align: center
    :alt: symblog add blog comments
    
Conclusão
---------

Nós progredimos bem neste capítulo. Nosso site está começando a ficar do jeito que esperamos que funcione. Agora, temos o 
básico da página inicial criada e a entidade do comentário. 

Os usuários, agora, podem postar comentários em posts do blog e ler os comentários deixado por outro utilizador. Vimos 
como criar fixtures que podem ser referenciados em multiplos arquivos de fixtures e usamos as Migrações do Doctrine 2 
para manipular o esquema do banco de dados com as alterações da entidade.

No próximo capítulo, vamos construir a barra lateral para incluir a nuvem de tags e os comentários recentes. Vamos 
estender o Twig, criando nossos próprios filtros personalizados. 

Finalmente, vamos usar a biblioteca ``asset`` para nos auxiliar na gestão da nossos assets.