[Parte 3] - O Model do Blog: Usando Doctrine 2 e Data Fixtures
==============================================================

Visão geral
-----------

Este capítulo vai começar a explorar o Model do blog. O Model será implementado usando Mapa de Objeto Relacional (ORM) 
`Doctrine 2 <http://www.doctrine-project.org/projects/orm>`_.  

Doctrine 2 oferece persistência para os objetos PHP. Ele também fornece um dialeto SQL próprio chamado de Doctrine Query 
Language (DQL). 

Além disso, vamos apresentar alguns conceitos de Data Fixtures. Data Fixtures é um mecanismo para preencher o nosso 
banco de dados em ambientes de desenvolvimento e de teste. 

No final deste capítulo você terá definido o modelo do blog, terá o banco de dados atualizado para refletir o novo 
modelo, e terá criado alguns data fixtures. Você irá também construir alguns métodos básicos da página ``show`` do blog.

Doctrine 2: O Modelo (Model)
----------------------------

Para o nosso blog funcionar, precisamos de algo para trabalhar com os dados. Doctrine 2 fornece uma biblioteca ORM 
exatamente para esta finalidade. 

O Doctrine 2 ORM se baseia em uma 
`Camada de abstração de banco de dados <http://www.doctrine-project.org/projects/dbal>`_ que nos dá uma abstração de 
armazenamento graças ao PHP PDO. 

Assim, podemos usar um número variado de ferramentas de armazenamento, incluindo MySQL, PostgreSQL e SQLite. Iremos usar 
o MySQL como nossa ferramenta de armazenamento, mas qualquer outra poderia ser facilmente utilizada. 

.. tip::

    Se você não estiver familiarizado com ORMs, vamos explicar o princípio básico deles.
    
    A `Wikipedia <http://en.wikipedia.org/wiki/Object-relational_mapping>`_ define o seguinte:
    "Mapeamento objeto-relacional (ORM, O/RM, e mapeamento O/R) em software de computador é uma técnica de programação 
    para conversão de dados entre o tipo incompatível de sistemas em linguagens orientadas a objetos de programação. 
    Isto cria, uma ``Base de dados de objeto virtual`` que pode ser usado a partir da linguagem de programação."
    
    O que as funcionalidades do ORM nos diz é que os dados de um banco de dados relacional, como o MySQL, podem ser 
    manipulados como objetos do PHP. Isso nos permite encapsular a funcionalidade necessária em uma tabela dentro de uma 
    classe. 

    Pense em um tabela de usuário, que provavelmente tem campos como nome de usuario, senha, primeiro_nome, ultimo_nome 
    e email. Com um ORM, isso se torna uma classe com membros usuario, senha, primeiro_nome, etc, o que nos permite 
    chamar métodos como ``getUsername()`` e ``setSenha()``. 

    Os ORMs podem ir muito mais além do que isso, eles também são capazes de recuperar tabelas relacionadas, seja ao 
    mesmo tempo como nós recuperamos o objeto de usuário, ou mais de forma mais lenta. 

    Agora, considere que nosso usuário tem alguns amigos relacionados a ele. Teríamos uma tabela de amigos, armazenando 
    a chave primária da tabela do usuário. Utilizando ORM, podemos agora fazer uma chamada do tipo 
    ``$user->getFriends()`` para recuperar objetos da tabela de amigos. 

    E também, o ORM lida com persistência, assim, podemos criar objetos em PHP, chamar um método como ``save()`` e 
    deixar o ORM lidar com os detalhes da persistência atual dos dados para o banco de dados. 

    Como estamos usando a biblioteca Doctrine 2 ORM, você ficará muito mais familiarizado com o que é um ORM à medida 
    que progredimos com o tutorial.

.. note::

    Apesar deste tutorial usar a biblioteca Doctrine 2 ORM, você pode optar por usar a biblioteca Doctrine 2 Document 
    Object Mapper (ODM). Há um número de variações desta biblioteca, incluindo implementações para 
    `MongoDB <http://www.mongodb.org/>`_ e `CouchDB <http://couchdb.apache.org/>`_. Conheça os projetos do 
    `Doctrine <http://www.doctrine-project.org/projects>`_ para maiores informações.

    Existe também um artigo
    `CookBook <http://symfony.com/doc/current/cookbook/doctrine/mongodb.html>`_ que explica como configurar ODM com 
    Symfony 2.

A Entidade Blog
~~~~~~~~~~~~~~~

Vamos começar a criar a classe de entidade ``Blog``. Nós já sabemos sobre entidades mostrado no capítulo anterior, 
quando criamos a entidade ``Enquiry``. Como o objetivo da entidade é armazenar dados, faz todo o sentido usar uma para 
representar uma entrada do blog. Ao definir uma entidade, não estamos automaticamente dizendo que os dados serão 
mapeados para o banco de dados. Vimos isso com a nossa entidade de ``Enquiry``, onde os dados contidos na entidade foram 
apenas enviados para o E-mail de contato.

Crie um novo arquivo em ``src/Blogger/BlogBundle/Entidade/blog.php`` e cole o seguinte código:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    namespace Blogger\BlogBundle\Entity;

    class Blog
    {
        protected $title;

        protected $author;

        protected $blog;

        protected $image;

        protected $tags;

        protected $comments;

        protected $created;

        protected $updated;
    }


Como você pôde perceber, esta é uma classe PHP simples. Ela não extende de ninguém e não tem nenhum assessor. Cada um 
dos membros é declarado como protegido, assim, se tornam inacessíveis quando estivermos trabalhando com um objeto desta 
classe. 

Poderíamos declarar os ``getters`` e ``setters`` manualmente, mas o Doctrine 2 tem uma funcionalidade que faz isso. 
Afinal, escrever assessores não é uma tarefa de codificação empolgante.

Antes de executar esta tarefa, precisamos informar ao Doctrine 2 como a entidade ``Blog`` deve ser mapeada para o banco 
de dados. A informação é especificada como metadados usando mapeamentos do Doctrine 2. 

Os metadados podem ser especificados em vários formatos incluindo ``YAML``, ``PHP``, ``XML`` e ``Anotations``. Usaremos 
``Anotations`` neste tutorial. 

É importante notar que nem todos os membros na entidade precisam ser persistentes, por isso não vamos fornecer metadados 
para eles. Assim, conseguimos flexibilidade de escolher somente os membros que exigem mapeamento do Doctrine 2 para o 
banco de dados. 

Substitua o conteúdo da classe da entidade ``Blog`` localizada em ``src/Blogger/BlogBundle/Entidade/blog.php`` com o 
seguinte código:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    namespace Blogger\BlogBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity
     * @ORM\Table(name="blog")
     */
    class Blog
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
        protected $title;

        /**
         * @ORM\Column(type="string", length=100)
         */
        protected $author;

        /**
         * @ORM\Column(type="text")
         */
        protected $blog;

        /**
         * @ORM\Column(type="string", length="20")
         */
        protected $image;

        /**
         * @ORM\Column(type="text")
         */
        protected $tags;

        protected $comments;

        /**
         * @ORM\Column(type="datetime")
         */
        protected $created;

        /**
         * @ORM\Column(type="datetime")
         */
        protected $updated;
    }


Primeiro importamos e linkamos o namespace do mapeamento ORM do Doctrine 2. Isto permite usar ``anotations`` para 
descrever os metadados para a entidade. 

Os metadados fornecem informações sobre como os membros devem ser mapeados para o banco de dados.

.. tip::

    Usamos somente um pequeno subconjunto dos tipos de mapeamento existentes para Doctrine 2. A lista completa de 
    `Tpos de mapeamento <http://www.doctrine-project.org/docs/orm/2.0/en/reference/basic-mapping.html#doctrine-mapping-types>`_ 
    pode ser encontrada no site do Doctrine 2. 

    Outros tipos de mapeamento serão introduzidos mais tarde no tutorial.

Se você prestou bem a atenção, os ``$comments`` não tem metadados anexados. Isto ocorre porque nós não precisamos de 
persistir seus dados, ela só vai trazer uma coleção de comentários relacionados a um ``post`` do blog. Se você percebeu, 
estamos sem banco de dados. Veja alguns exemplos:

.. code-block:: php

    // Criando um objeto do blog.
    $blog = new Blog();
    $blog->setTitle("Symblog - Um tutorial de Symfony 2");
    $blog->setAuthor("dsyph3r");
    $blog->setBlog("Symblog é um site de blogs com todos os recursos ...");

    // Criando um comentário e adicionando-o ao nosso blog
    $comment = new Comment();
    $comment->setComment("Symfony 2 é de mais!");
    $blog->addComment($comment);

O trecho acima demonstra o comportamento normal que você gostaria que uma classe ``blog`` e ``comment`` tivesse. 
Internamente, o método ``$blog->addComment()`` poderia ser implementada como se segue:

.. code-block:: php

    class Blog
    {
        protected $comments = array();

        public function addComment(Comment $comment)
        {
            $this->comments[] = $comment;
        }
    }

O método ``addComment`` apenas adiciona um novo objeto comentário para a variável ``$comment`` do blog. Recuperar os 
comentários também se torna bem simples.

.. code-block:: php

    class Blog
    {
        protected $comments = array();

        public function getComments()
        {
            return $this->comments;
        }
    }

Como você pôde ver, a variável ``$comment`` é apenas uma lista de objetos ``Comment``. Doctrine 2 não escolhe como isso 
funciona. 

Doctrine 2 vai automaticamente preencher essa variável ``$comments`` com objetos relacionados com o objeto ``Blog``.

Agora que dissemos como o Doctrine 2 deve mapear os membros da entidade, podemos gerar os métodos de acesso usando o 
seguinte código:

.. code-block:: bash

    $ php app/console doctrine:generate:entities Blogger


Você vai perceber que a entidade ``Blog`` foi atualizada com os métodos de acesso. Cada vez que fizermos uma alteração 
nos metadados do ORM para nossa classe de entidade, temos que executar este código para gerar quaisquer métodos de 
acesso adicionais. 

Este comando não vai fazer alterações os assessores já existentes na entidade, então, seus métodos de acesso já 
existentes nunca serão substituídos utilizando este comando. 

Isto é importante porque você pode personalizar mais tarde alguns assessores padrão.

.. tip::

    Embora tenhamos utilizado ``anotations`` no nosso entidade, é possível converter a informação de mapeamento para os 
    outros formatos de mapeamento suportados usando o comando ``doctrine:mapping:converter``. 

    Por exemplo, o seguinte comando converte os mapeamentos da entidade acima no formato ``yaml``.

    .. code-block:: bash

        $ php app/console doctrine:mapping:convert --namespace="Blogger\BlogBundle\Entity\Blog" yaml src/Blogger/BlogBundle/Resources/config/doctrine

    O resultado é um arquivo criado em
    ``src/Blogger/BlogBundle/Resources/config/doctrine/Blogger.BlogBundle.Entity.Blog.orm.yml``
    que conterá os mapeamentos da entidade do ``blog`` no formato ``yaml``.

O banco de dados
~~~~~~~~~~~~~~~~

Criando o banco de dados
........................

Assim como no capítulo 1, você deve ter usado o configurador web para definir as configurações de banco de dados. Se 
você não tiver feito isso, atualize as opções ``database_*`` no arquivo de parâmetros localizado em 
``app/config/parameters.ini``.

Agora é hora de criar o banco de dados usando outra funcionalidade do Doctrine 2. Esta, só cria o banco de dados, não 
cria as tabelas dentro dele. 

Se um banco de dados com o mesmo nome já existir, um erro será exibido e o banco de dados existente não será alterado.

.. code-block:: bash

    $ php app/console doctrine:database:create

Agora estamos prontos para criar a representação da entidade do banco de dados do ``Blog``. Existem 2 maneiras para se 
fazer isso. 

Podemos usar os esquemas do Doctrine 2 para atualizar o banco de dados ou podemos usar as migrações (Migrations) do 
Doctrine 2. Por agora, vamos usar a funcionalidade de esquema. 

As Migrações do Doctrine, serão apresentadas no capítulo seguinte.

Criando a tabela blog
.....................

Para criar a tabela blog em nosso banco de dados podemos executar o seguinte comando Doctrine.

.. code-block:: bash

    $ php app/console doctrine:schema:create

Esse comando executará o SQL necessário para gerar o esquema de banco de dados para a entidade do ``blog``. Você também 
pode passar a opção ``--dump sql`` para a tarefa de salvar o SQL em vez de executá-lo na base de dados. 

Se você ver o seu banco de dados, você verá que a tabela blog foi criada, com os campos que configuramos com informações 
do mapeamento.

.. tip::

    Nós usamos vários comandos do Symfony 2 agora, e, na verdade, cada comando tem uma ajuda associada, basta digitar a 
    opção ``--help``. Para ver os detalhes da ajuda para ``doctrine:schema:create``, execute o seguinte comando:

    .. code-block:: bash

        $ php app/console doctrine:schema:create --help

    As informações de ajuda serão exibidas mostrando o uso e várias outras opções disponíveis. A maioria das 
    funcionalidades vêm com uma série de opções que podem ser definidas para personalizar sua execução.

Integrando o Model com a Visão. Mostrando uma entrada do blog
-------------------------------------------------------------

Agora temos a entidade ``Blog`` criada e o banco de dados atualizado. Podemos começar a integrar o Model com a View. 

Nós vamos começar construindo a página ``show`` do nosso blog.

A Rota da página show do Blog
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Começaremos criando a rota para a action ``show``. 

Um blog será identificados pelo seu ``id`` único, de modo que este id deverá estar presente na URL. Atualize o arquivo 
de rotas de ``BloggerBlogBundle`` localizado em ``src/Blogger/BlogBundle/Resources/config/routing.yml`` com o seguinte 
código:

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_blog_show:
        pattern:  /{id}
        defaults: { _controller: BloggerBlogBundle:Blog:show }
        requirements:
            _method:  GET
            id: \d+

Como o id do blog deve estar presente na URL, especificamos um espaço reservado para o ``id``. Isto quer dizer que URLs 
como ``http://symblog.co.uk/1`` e ``http://symblog.co.uk/my-blog`` irão corresponder a esta rota. No entanto, sabemos 
que o id do blog deve ser um inteiro (é definido desta forma nos mapeamentos de entidade), então devemos adicionar uma 
restrição para especificar que esta rota só pode coincidir apenas quando o parâmetro ``id`` contém um número inteiro.
Isto é feito com a rota desejada ``id: \d+``. 

Agora, como exemplo, a URL anterior ``http://symblog.co.uk/my-blog`` deixaria de funcionar para esta rota. Você também 
pode ver que uma rota correspondente irá executar a ação ``show`` do controlador do ``Blog`` em ``BloggerBlogBundle``. 

Este controlador ainda está para ser criado.

A Ação Show do Controlador
~~~~~~~~~~~~~~~~~~~~~~~~~~

O responsável por ligar o modelo e a visão é o controlador, deste modo, aqui é o lugar onde nós começaremos a criar a 
página show. 

Poderíamos acrescentar a ação ``show`` em nosso controlador ``Page`` já existente, mas como esta página serve para 
mostrar as entidades do ``blog``, seria mais adequado criar a ação ``show`` no controlador ``Blog``.

Crie um novo arquivo em ``src/Blogger/BlogBundle/Controller/BlogController.php`` e cole o seguinte código:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/BlogController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    /**
     * Blog controller.
     */
    class BlogController extends Controller
    {
        /**
         * Show a blog entry
         */
        public function showAction($id)
        {
            $em = $this->getDoctrine()->getEntityManager();

            $blog = $em->getRepository('BloggerBlogBundle:Blog')->find($id);

            if (!$blog) {
                throw $this->createNotFoundException('O post do blog não pode ser enconrado.');
            }

            return $this->render('BloggerBlogBundle:Blog:show.html.twig', array(
                'blog'      => $blog,
            ));
        }
    }

Nós criamos um novo controlador para a entidade ``Blog`` e definimos a ação ``show``. Como especificamos um parâmetro 
``id`` na regra de rota ``BloggerBlogBundle_blog_show`` do arquivo de rotas, ele será passado como um argumento para o 
método ``showAction``. 

Se tivéssemos especificado mais parâmetros na regra de roteamento, eles também seriam passados como argumentos separados.

.. tip::

    As ações do controlador também vão passar por um objeto ``Symfony\Component\HttpFoundation\Request`` se você 
    especificar isso como um parâmetro. Isto pode ser útil quando se lida com formulários. 

    Formulários foram vistos no capítulo 2, mas nós não iremos usar esse método como foi utilizado em 
    ``Symfony\Bundle\FrameworkBundle\Controller\Controller`` utilizando métodos auxiliares como mostrado abaixo:

    .. code-block:: php

        // src/Blogger/BlogBundle/Controller/PageController.php
        public function contactAction()
        {
            // ..
            $request = $this->getRequest();
        }

    Poderíamos ter usado este:

    .. code-block:: php

        // src/Blogger/BlogBundle/Controller/PageController.php

        use Symfony\Component\HttpFoundation\Request;

        public function contactAction(Request $request)
        {
            // ..
        }
    
    Ambos fazem a mesma coisa. Se o controlador não estender a classe auxiliar 
    ``Symfony\Bundle\FrameworkBundle\Controller\Controller`` você não poderia utilizar o primeiro método.

Agora, precisamos recuperar a entidade ``Blog`` do banco de dados . Nós, primeiramente, iremos usar outro método 
auxiliar da classe ``Symfony\Bundle\FrameworkBundle\Controller\Controller`` para obter o Gerenciador de Entidade 
Doctrine 2. O trabalho do 
`Gerente de Entidade <http://www.doctrine-project.org/docs/orm/2.0/en/reference/working-with-objects.html>`_ é lidar 
com a recuperação e persistência de objetos de e para o banco de dados. 

Nós, em seguida, usaremos o objeto ``EntityManager`` para obter o ``Repositório`` do Doctrine 2 para a entidade 
``BloggerBlogBundle:Blog``. A sintaxe especificada aqui é simplesmente uma amostra do que podemos usa com Doctrine 2 ao 
invés de especificar o nome completo da entidade, ou seja, ``Blogger\BlogBundle\Entity\Blog``. 

Com o objeto repositório, podemos invocar o método ``find()`` passando o argumento ``$id``. Este método irá recuperar o 
objeto pela sua chave primária.

Finalmente verificamos se uma entidade foi encontrada e passamos esta entidade para a View. Se nenhuma entidade foi 
encontrada um ``createNotFoundException`` é exibido, ou seja, um ``404 Not Found`` é exibido como resposta.

.. tip::

    O objeto repositório dá acesso à uma série de métodos auxiliares úteis, incluindo:

    .. code-block:: php

        // Retorna entidades onde o 'autor' casa com o termo 'dsyph3r'
        $em->getRepository('BloggerBlogBundle:Blog')->findBy(array('author' => 'dsyph3r'));

        // Retorna entidades onde o 'slug' casa com o termo 'symblog-tutorial'
        $em->getRepository('BloggerBlogBundle:Blog')->findOneBySlug('symblog-tutorial');

    Nós vamos criar nossos próprios Repositório personalizados no próximo capítulo, quando precisarmos de pesquisas mais 
    complexas.

A View Show
~~~~~~~~~~~

Agora que temos a ação ``show`` para o controlador ``Blog``, podemos focar em apresentar a entidade do ``Blog``. 

Conforme especificado na ação ``show``, o template ``BloggerBlogBundle:Blog:show.html.twig`` será renderizado. Vamos 
criar este template em ``src/Blogger/BlogBundle/Resouces/views/Blog/show.html.twig`` e cole no seguinte código:

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resouces/views/Blog/show.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}{{ blog.title }}{% endblock %}

    {% block body %}
        <article class="blog">
            <header>
                <div class="date"><time datetime="{{ blog.created|date('c') }}">{{ blog.created|date('l, F j, Y') }}</time></div>
                <h2>{{ blog.title }}</h2>
            </header>
            <img src="{{ asset(['images/', blog.image]|join) }}" alt="{{ blog.title }} imagem não encontrada" class="large" />
            <div>
                <p>{{ blog.blog }}</p>
            </div>
        </article>
    {% endblock %}

Como seria de esperar, começamos estendendo o layout principal de ``BloggerBlogBundle``. Em seguida, sobrescrevemos o 
título da página com o título do blog. Esta é útil para atividades de SEO pois o título da página do blog é mais 
descritiva do que o título padrão que está definido. 

Por último, substituimos o body block pelo conteúdo da entidade do ``Blog``. Nós usamos a função ``assets`` novamente 
aqui para renderizar a imagem do blog. As imagens blog devem ser colocadas na pasta ``web/images``.

CSS
...

Para deixarmos a página ``show`` mais bonita, precisamos adicionar algum estilo. 

Atualize a folha de estilo em ``src/Blogger/BlogBundle/Resouces/public/css/blog.css`` com o seguinte código:

.. code-block:: css

    .date { margin-bottom: 20px; border-bottom: 1px solid #ccc; font-size: 24px; color: #666; line-height: 30px }
    .blog { margin-bottom: 20px; }
    .blog img { width: 190px; float: left; padding: 5px; border: 1px solid #ccc; margin: 0 10px 10px 0; }
    .blog .meta { clear: left; margin-bottom: 20px; }
    .blog .snippet p.continue { margin-bottom: 0; text-align: right; }
    .blog .meta { font-style: italic; font-size: 12px; color: #666; }
    .blog .meta p { margin-bottom: 5px; line-height: 1.2em; }
    .blog img.large { width: 300px; min-height: 165px; }

.. note::

    Se você não estiver usando o método de ligação simbólica para referenciar os pacotes de assets para a pasta ``web``, 
    você deve re-executar o instalador de assets agora para copiar as alterações no seu CSS

    .. code-block:: bash

        $ php app/console assets:install web


Como já construímos o controlador e a visão para a ação ``show`` vamos dar uma olhada na página de show. Acesse 
``http://symblog.dev/app_dev.php/1``. Não é a página que você estava esperando?

.. image:: /_static/images/part_3/404_not_found.jpg
    :align: center
    :alt:Exceção Symfony 2 404 Não Encontrado

O Symfony 2 gerou uma resposta 404 ``Não Encontrado``. Isto aconteceu porque não temos dados em nosso banco de dados. 
Assim, nenhuma entidade com ``id`` igual a 1 poderia ser encontrada.

Você poderia simplesmente inserir uma linha na tabela blog de seu banco de dados, mas vamos usar um método muito melhor; 
Data Fixtures.

Data Fixtures
-------------

Podemos usar os Data Fixtures para popular o banco de dados com alguns dados de amostra/teste. Para fazer isso usamos o 
pacote de extensões Doctrine Data Fixtures. 

O pacote de extensões Doctrine Data Fixtures não vem com a distribuição Standard do Symfony 2, precisamos instalar 
manualmente. Felizmente, esta é uma tarefa fácil. 

Abra o arquivo ``deps`` localizado na raiz do projeto e adicione os pacotes e extensões Doctrine Data Fixtures como se 
segue:

.. code-block:: text

    [doctrine-fixtures]
        git=http://github.com/doctrine/data-fixtures.git

    [DoctrineFixturesBundle]
        git=http://github.com/symfony/DoctrineFixturesBundle.git
        target=/bundles/Symfony/Bundle/DoctrineFixturesBundle

Em seguida, devemos atualizar os vendors para atualizar essas alterações.

.. code-block:: bash

    $ php bin/vendors install

Assim, faremos a atualização dos repositórios mais recente do Github e iremos instalá-los no local desejado.

.. note::

    Se você estiver usando uma máquina que não tem o Git instalado, você terá que baixar e instalar manualmente as 
    extensões e pacotes.

    doctrine-fixtures extension: Faça o `Download <https://github.com/doctrine/data-fixtures>`_ da versão atual do 
    pacote e extraia ``vendor/doctrine-fixtures``.

    DoctrineFixturesBundle: Faça o `Download  <https://github.com/symfony/DoctrineFixturesBundle>`_ da versão atual do 
    pacote e extraia em ``vendor/bundles/Symfony/Bundle/DoctrineFixturesBundle``.

Depois, atualize o arquivo ``app/autoloader.php`` para registrar o novo namespace. 

Como DataFixtures também estão no namespace ``Doctrine\Common``, eles devem ser colocados acima da diretiva 
``Doctrine\Common`` existente para especificar um novo caminho. 

Namespaces são verificados de cima para baixo. Para namespaces mais específicos, precisamos registrar antes dos menos 
específicos.

.. code-block:: php

    // app/autoloader.php
    // ...
    $loader->registerNamespaces(array(
    // ...
    'Doctrine\\Common\\DataFixtures'    => __DIR__.'/../vendor/doctrine-fixtures/lib',
    'Doctrine\\Common'                  => __DIR__.'/../vendor/doctrine-common/lib',
    // ...
    ));

Agora vamos registrar o ``DoctrineFixturesBundle`` no kernel em ``app/AppKernel.php``

.. code-block:: php

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Symfony\Bundle\DoctrineFixturesBundle\DoctrineFixturesBundle(),
            // ...
        );
        // ...
    }

Blog Fixtures
~~~~~~~~~~~~~

Agora estamos prontos para definir algumas fixtures para os nossos blogs. Crie um arquivo de fixture em 
``src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php`` e adicione o seguinte conteúdo:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/DataFixtures/ORM/BlogFixtures.php
    
    namespace Blogger\BlogBundle\DataFixtures\ORM;
    
    use Doctrine\Common\DataFixtures\FixtureInterface;
    use Doctrine\Common\Persistence\ObjectManager;
    use Blogger\BlogBundle\Entity\Blog;
    
    class BlogFixtures implements FixtureInterface
    {
        public function load(ObjectManager $manager)
        {
            $blog1 = new Blog();
            $blog1->setTitle('Um dia com Symfony 2');
            $blog1->setBlog('Lorem ipsum dolor sit amet, consectetur adipiscing eletra electrify denim vel ports.\nLorem ipsum dolor sit amet, consectetur adipiscing elit. Morbi ut velocity magna. Etiam vehicula nunc non leo hendrerit commodo. Vestibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque. Nulla consectetur tempus nisl vitae viverra. Cras el mauris eget erat congue dapibus imperdiet justo scelerisque. Nulla consectetur tempus nisl vitae viverra. Cras elementum molestie vestibulum. Morbi id quam nisl. Praesent hendrerit, orci sed elementum lobortis, justo mauris lacinia libero, non facilisis purus ipsum non mi. Aliquam sollicitudin, augue id vestibulum iaculis, sem lectus convallis nunc, vel scelerisque lorem tortor ac nunc. Donec pharetra eleifend enim vel porta.');
            $blog1->setImage('beach.jpg');
            $blog1->setAuthor('dsyph3r');
            $blog1->setTags('symfony2, php, paradise, symblog');
            $blog1->setCreated(new \DateTime());
            $blog1->setUpdated($blog1->getCreated());
            $manager->persist($blog1);
    
            $blog2 = new Blog();
            $blog2->setTitle('A piscina no telhado tem que ter um vazamento');
            $blog2->setBlog('Vestibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque. Na. Cras elementum molestie vestibulum. Morbi id quam nisl. Praesent hendrerit, orci sed elementum lobortis.');
            $blog2->setImage('pool_leak.jpg');
            $blog2->setAuthor('Zero Cool');
            $blog2->setTags('pool, leaky, hacked, movie, hacking, symblog');
            $blog2->setCreated(new \DateTime("2011-07-23 06:12:33"));
            $blog2->setUpdated($blog2->getCreated());
            $manager->persist($blog2);
    
            $blog3 = new Blog();
            $blog3->setTitle('Desorientação. O que os olhos vêem e os ouvidos ouvem, a mente acredita');
            $blog3->setBlog('Lorem ipsumvehicula nunc non leo hendrerit commodo. Vestibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque.');
            $blog3->setImage('misdirection.jpg');
            $blog3->setAuthor('Gabriel');
            $blog3->setTags('misdirection, magic, movie, hacking, symblog');
            $blog3->setCreated(new \DateTime("2011-07-16 16:14:06"));
            $blog3->setUpdated($blog3->getCreated());
            $manager->persist($blog3);
    
            $blog4 = new Blog();
            $blog4->setTitle('A Grade - Uma fronteira digital');
            $blog4->setBlog('Lorem commodo. Vestibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque. Nulla consectetur tempus nisl vitae viverra.');
            $blog4->setImage('the_grid.jpg');
            $blog4->setAuthor('Kevin Flynn');
            $blog4->setTags('grid, daftpunk, movie, symblog');
            $blog4->setCreated(new \DateTime("2011-06-02 18:54:12"));
            $blog4->setUpdated($blog4->getCreated());
            $manager->persist($blog4);
    
            $blog5 = new Blog();
            $blog5->setTitle('Ou você é um ou zero. Vivo ou morto');
            $blog5->setBlog('Lorem ipsum dolor sit amet, consectetur adipiscing elittibulum vulputate mauris eget erat congue dapibus imperdiet justo scelerisque.');
            $blog5->setImage('one_or_zero.jpg');
            $blog5->setAuthor('Gary Winston');
            $blog5->setTags('binary, one, zero, alive, dead, !trusting, movie, symblog');
            $blog5->setCreated(new \DateTime("2011-04-25 15:34:18"));
            $blog5->setUpdated($blog5->getCreated());
            $manager->persist($blog5);
    
            $manager->flush();
        }
    
    }

O arquivo de fixtures demonstra uma série de características importantes quando se utiliza Doctrine 2, incluindo como 
persistir entidades para o banco de dados.

Vejamos como podemos criar uma entrada no blog.

.. code-block:: php

    $blog1 = new Blog();
    $blog1->setTitle('Um dia no paraiso - Um dia com Symfony 2');
    $blog1->setBlog('Lorem ipsum dolor sit d us imperdiet justo scelerisque. Nulla consectetur...');
    $blog1->setImage('beach.jpg');
    $blog1->setAuthor('dsyph3r');
    $blog1->setTags('symfony2, php, paradise, symblog');
    $blog1->setCreated(new \DateTime());
    $blog1->setUpdated($this->getCreated());
    $manager->persist($blog1);
    // ..

    $manager->flush();

Começamos criando um objeto do ``Blog`` e definimos alguns valores para seus membros. 

Neste ponto, Doctrine 2 não sabe nada sobre o objeto ``Entity``. Só quando fazemos uma chamada 
``$manager>persist($blog1)`` que instruimos o Doctrine 2 a começar a gerir o objeto da entidade. 

O objeto ``$manager`` aqui, é uma instância do objeto ``EntityManager`` que vimos anteriormente ao recuperar entidades 
do banco de dados. 

É importante notar que, enquanto Doctrine 2 está, agora, sabendo da existência do objeto de entidade, ainda não é 
mantido pelo o banco de dados. É  necessário fazer uma chamada para ``$manager->flush()``. 

O método flush faz o Doctrine 2 realmente interagir com o banco de dados e aciona todas as entidades que serão mantidas. 

Para um melhor desempenho, você deve agrupar as operações do Doctrine 2 em conjunto e executar todas as ações de uma só 
vez. É assim que temos feito em nossos Data Fixture. Criamos as entidade, pedimos ao Doctrine 2 para manipula e, em 
seguida, executamos todas as operações no final.

.. tip:

    Você deve ter percebido a definição dos membros ``created`` e ``updated``. Esta não é a forma ideal de definir esses 
    campos. 

    Espera-se, que eles sejam atualizados automaticamente quando um objeto é criado ou atualizado. Doctrine 2 dispõe de 
    uma método para que possamos alcançar este objetivo. 

    Vamos explorar este método brevemente.

Carregando os Data Fixtures
~~~~~~~~~~~~~~~~~~~~~~~~~~~ 

Agora estamos prontos para carregar os fixtures para o banco de dados.

.. code-block:: bash

    $ php app/console doctrine:fixtures:load

Se recarregarmos a página de show do blog no navegador, ``http://symblog.dev/app_dev.php/1``, veremos um blog completo e 
com estilos.

.. image:: /_static/images/part_3/blog_show.jpg
    :align: center
    :alt: A página de exibição do blog do Symblog

Tente alterar o parâmetro ``id`` na URL para 2. Você deve ver a outra entrada do blog. Se você tentar acessar 
``http://symblog.dev/app_dev.php/100`` você verá uma exceção ``404 Not Found``. Calro,  não há entidade de ``Blog`` com 
um ``id`` de 100. Agora tente acessar ``http://symblog.dev/app_dev.php/symfony2-blog``. 

Por que não temos uma exceção ``404 Not Found``? Isto é porque a ação ``show`` nunca é executado. A URL falhou em 
coincidir qualquer rota na aplicação por causa da especificação ``\d+`` que definimos na rota 
``BloggerBlogBundle_blog_show``. É por isso que você viu uma exceção ``No route found for "GET /symfony2-blog"``.

Timestamps
----------

Finalmente, vamos analisar os 2 membros timestamp na entidade ``Blog``; ``created`` e ``updated``. 

A funcionalidade destes 2 membros é referida como um comportamento ``Timestampable``. Estes membros guardam o horário em 
que o blog foi criado e atualizado, respectivamente. 

Como não queremos ter que configurar manualmente estes campos cada vez que criamos ou atualizamos uma entidade do blog, 
podemos utilizar o Doctrine 2 para nos ajudar.

Doctrine 2 vem com um `Sistema de Eventos <http://www.doctrine-project.org/docs/orm/2.0/en/reference/events.html>`_ que 
fornece `Callbacks do ciclo de vida <http://www.doctrine-project.org/docs/orm/2.0/en/reference/events.html#lifecycle-callbacks>`_.

Podemos usar esses eventos de callbacks para registrar nossas entidades para ser notificado sobre eventos durante o 
período de vida da entidade. 

Alguns exemplos de eventos de notificação são  utilizados para ilustrar que algo aconteceu antes de uma atualização, 
depois de uma persistência e depois de uma exclusão. 

Para utilizar Callbacks do ciclo de vida em nossa entidade temos que registrar a entidade para eles. Isso é feito usando 
metadados na entidade. Atualize a entidade ``Blog`` em ``src/Blogger/BlogBundle/Entity/blog.php`` com o seguinte código:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    // ..

    /**
     * @ORM\Entity
     * @ORM\Table(name="blog")
     * @ORM\HasLifecycleCallbacks()
     */
    class Blog
    {
        // ..
    }

Agora vamos adicionar um método na entidade ``Blog`` que registra o evento ``preUpdate``. Nós também adicionamos um 
construtor para definir valores padrão para os membros ``created`` e o ``updated``.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Blog.php

    // ..

    /**
     * @ORM\Entity
     * @ORM\Table(name="blog")
     * @ORM\HasLifecycleCallbacks()
     */
    class Blog
    {
        // ..

        public function __construct()
        {
            $this->setCreated(new \DateTime());
            $this->setUpdated(new \DateTime());
        }

        /**
         * @ORM\preUpdate
         */
        public function setUpdatedValue()
        {
           $this->setUpdated(new \DateTime());
        }

        // ..
    }

Nós registramos a entidade ``Blog`` para ser notificado sobre o evento ``preUpdate`` para definir o valor de ``updated``. 
Agora, quando você executar novamente a inserção do fixtures, você vai notar que os membros ``created`` e ``updated`` 
são definidos automaticamente.

.. tip::

    Como membros Timestampable são membros comuns para as entidades, existe um pacote disponível que os suporta. O 
    `StofDoctrineExtensionsBundle <https://github.com/stof/StofDoctrineExtensionsBundle>`_ fornece uma série de extensões 
    úteis do Doctrine 2, incluindo Timestampable, Sluggable e Sortable.

    Vamos integrar este pacote mais tarde no tutorial. Fique a vontade para estudar a respeito deste tema. Vá até o 
    `CookBokk <http://symfony.com/doc/current/cookbook/doctrine/common_extensions.html>`_.

Conclusão
---------

Nós cobrimos uma série de conceitos para lidar com Model em Doctrine 2. Também vimos a definição de Data Fixtures que 
nos proporciona uma maneira fácil de inserir dados em nosso ambiente de desenvolvimento e teste.

Vamos estender o Model um pouco mais, acrescentando a entidade comentário. Vamos construir uma página inicial e criar um 
repositório comum. Também vamos introduzir o conceito de Migrações do Doctrine e como formulários interagem com Doctrine 
2 para permitir que os comentários sejam postados para um blog
