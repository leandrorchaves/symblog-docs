[Parte 6] - Teste: Teste de Unidade e Teste Funcional com PHPUnit
=================================================================

Visão geral
-----------

Até agora, exploramos vários conceitos fundamentais com relação ao desenvolvimento com Symfony 2. Antes de continuarmos 
a adicionar funcionalidades, é hora de realizarmos testes. 

Vamos ver como testar funções individualmente com testes de unidade e como garantir se múltiplos componentes estão 
funcionando corretamente em conjunto, com testes funcionais. 

A biblioteca de testes PHP, `PHPUnit <http://www.phpunit.de/manual/current/en/>`_, será o centro dos testes em Symfony 2. 
Como ``teste`` é um tema abrangente, será abordado em capítulos posteriores. 

Até o final deste capítulo, você vai ter escrito vários testes que abrangem tanto o teste de unidade quanto testes 
funcionais. Você terá simulado solicitações do navegador, terá preenchido formulários com dados e verificado as 
respostas para garantir que as páginas do site retornam as saídas da forma correta. Você ainda poderá verificar o quanto 
seus testes são eficazes no código base de suas aplicações.

Testes em Symfony2
------------------

`PHPUnit <http://www.phpunit.de/manual/current/en/>`_ tornou-se o padrão para a escrita de testes em PHP, então, esta 
aprendizagem irá beneficiar você em todos os seus projetos em PHP. Não podemos esquecer que, a maioria dos tópicos 
abordados neste capítulo, são independentes de linguagem e, assim, podem ser utilizados em outras línguas. 

.. tip::

    Se você está pensando em escrever a seus próprios pacotes de código fonte aberto em Symfony 2, você irá ter mais 
    vantagem se você testar (e documentar) bem seus pacotes. Dê uma olhada nos pacotes existentes do Symfony 2, 
    disponíveis em `Symfony2Bundles <http://symfony2bundles.org/>`_.

Testes Unitários
~~~~~~~~~~~~~~~~

O teste de unidade, está preocupado em garantir que unidades individuais de código funcionem corretamente, quando usados 
isoladamente. Em uma base de código Orientado à objetos, como Symfony 2, uma unidade seria uma classe e seus métodos. 
Por exemplo, poderíamos escrever testes para as classes de entidade ``Blog`` e ``Comments``. 

Ao escrevermos testes de unidade, os casos de testes devem ser escrito independentemente de outros casos de teste, isto 
é, o resultado do teste do caso B não depende do resultado do teste do caso A. Isso é útil quando o teste de unidade for 
capaz de criar objetos fictícios que permitem você, facilmente, teste as funções que têm dependências externas, 
unicamente. 

Esses objetos fictícios, permitem simular uma chamada de uma função em vez de realmente executá-la. Um exemplo disso 
seria testar unicamente uma classe que encapsula uma API externa. A classe API pode usar uma camada de transporte para 
se comunicar com a API externa. 

Poderíamos simular um método de requisição da camada de transporte para retornar os resultados que especificamos, em vez 
de realmente atingir a API externa. 

O teste unitário não testa os componentes de uma função de uma aplicação, corretamente, em conjunto. Este assunto é 
abordado pelo próximo tópico, Testes Funcionais.

Testes Funcionais
~~~~~~~~~~~~~~~~~

O teste funcional, verifica a integração dos diferentes componentes dentro da aplicação, tais como rota, controladores e 
as visões (Views). 

Testes funcionais são semelhante aos testes manuais que você executa no próprio navegador, como requisitar a página 
inicial, clicar em um link do blog e verificar se o blog correto está sendo exibido. 

O teste funcional, possibilita a capacidade de automatizar esse processo. O Symfony 2 vem repleto de várias classes 
úteis que auxiliam no teste funcional, incluindo um ``Cliente`` que é capaz de requisitar páginas, enviar formulários e 
inspecionar o DOM ``DOM Crawler``, que podemos usar para interpretar a ``resposta`` do cliente.

.. tip::

    Há vários processos de desenvolvimento de software que são direcionados a testes. Estes processos incluem Test 
    Driven Development (TDD) e Behavioral Driven Development (BDD). 

    Estes testes estão fora do escopo deste tutorial, mas, você deve estar ciente da biblioteca escrita por 
    `everzet <https://twitter.com/#!/everzet>`_ que facilita o BDD, chamada `Behat <http://behat.org/>`_. 

    Existe também um pacote Symfony 2 `BehatBundle <http://docs.behat.org/bundle/index.html>`_ disponível para se 
    integrar facilmente o Behat em seu projeto Symfony 2.

PHPUnit
-------

Como dito acima, os testes em Symfony 2, são escritos utilizando PHPUnit. Você vai precisar instalar PHPUnit a fim de 
executar esses testes e os testes deste capítulo. Para informações sobre uma 
`Instalação detalhada <http://www.phpunit.de/manual/current/en/installation.html>`_, leia a documentação oficial no site 
do PHPUnit. 

Para executar os testes no Symfony 2, você precisa instalar PHPUnit 3.5.11 ou posterior. PHPUnit é uma biblioteca de 
teste muito grande, então, referências à documentação oficial serão feitas onde uma leitura adicional é recomendada.

Afirmações (Assertions)
~~~~~~~~~~~~~~~~~~~~~~~

Testes de escrita, estão relacionados com a verificação do resultado do teste atual, se é igual ao resultado do teste 
esperado. Há um certo número de métodos de afirmação (assertion) disponíveis em PHPUnit para ajudá-lo com esta tarefa. 

Vários métodos comun de afirmação, que serão usados, estão listados abaixo:

.. code-block:: php

    // Check 1 === 1 is true
    $this->assertTrue(1 === 1);

    // Check 1 === 2 is false
    $this->assertFalse(1 === 2);

    // Check 'Hello' equals 'Hello'
    $this->assertEquals('Hello', 'Hello');

    // Check array has key 'language'
    $this->assertArrayHasKey('language', array('language' => 'php', 'size' => '1024'));

    // Check array contains value 'php'
    $this->assertContains('php', array('php', 'ruby', 'c++', 'JavaScript'));

A lista completa de 
`Afirmações <http://www.phpunit.de/manual/current/en/writing-tests-for-phpunit.html#writing-tests-for-phpunit.assertions>`_ 
está disponível na documentação do PHPUnit.

Executando Testes em Symfony 2
------------------------------

Antes de começar a escrever alguns testes, vamos ver como podemos executar testes em Symfony 2. 

PHPUnit pode ser configurado para executar, usando um arquivo de configuração. Em nosso projeto Symfony 2, este arquivo 
está localizado em ``app/phpunit.xml.dist``. Como este arquivo está com o sufixo ``.dist``, você precisa copiar o seu 
conteúdo para um arquivo chamado ``app/phpunit.xml``.

.. tip::

   Se você estiver usando um VCS como Git, você deve adicionar o novo arquivo ``app/phpunit.xml`` na lista de VCS's 
    ignorados.

Se você observar o conteúdo do arquivo de configuração do PHPUnit, você vai ver o seguinte:

.. code-block:: xml

    <!-- app/phpunit.xml -->
    
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>../src/*/*Bundle/Tests</directory>
            <directory>../src/*/Bundle/*Bundle/Tests</directory>
        </testsuite>
    </testsuites>

O código acima, configura alguns diretórios que fazem parte de nosso pacote de teste. Ao executarmos o PHPUnit, ele irá 
procurar, nos diretórios acima, testes para serem executados. 

Você também pode passar argumentos adicionais, em linha de comando, para o PHPUnit para executar testes em diretórios 
específicos, em vez de usar o pacote de testes. Você vai ver como fazer isso depois.

Perceba que a configuração está especificando o arquivo de inicialização (bootstrap) localizada em
``app/bootstrap.php.cache``. Este arquivo é usado pelo PHPUnit para obter a configuração do ambiente de teste.

.. code-block:: xml

    <!-- app/phpunit.xml -->
    
    <phpunit
        bootstrap                   = "bootstrap.php.cache" >

.. tip::

    Para mais informações sobre como configurar PHPUnit com um arquivo XML, veja a 
    `Documentação do PHPUnit <http://www.phpunit.de/manual/current/en/organizing-tests.html#organizing-tests.xml-configuration>`_.

Executando os testes atuais
---------------------------

Assim como nós usamos o gerador de tarefas do Symfony 2 para criar o ``BloggerBlogBundle``, no capítulo 1, ele também 
criou um controlador de teste para a classe ``DefaultController``. Podemos executar este teste, executando o seguinte 
comando, a partir do diretório raiz do projeto. 

A opção ``-c`` especifica que o PHPUnit deve carregar a sua configuração a partir do diretório ``app``.

.. code-block:: bash

    $ phpunit -c app

Depois que o teste foi completado, você poderá ser notificado de que os testes falharam. 

Se você observar a classe ``DefaultControllerTest`` localizado em 
``src/Blogger/BlogBundle/Tests/Controller/DefaultControllerTest.php``, você vai ver o seguinte conteúdo:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Controller/DefaultControllerTest.php

    namespace Blogger\BlogBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class DefaultControllerTest extends WebTestCase
    {
        public function testIndex()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/hello/Fabien');

            $this->assertTrue($crawler->filter('html:contains("Hello Fabien")')->count() > 0);
        }
    }

Este é um teste funcional para a classe ``DefaultController`` que o Symfony 2 gerou. Se você se lembrar do capítulo 1, 
este controlador tem a ação de tratar as requisições para ``/Hello/{name}``. Removemos este controlador, pois o teste 
acima está falhando. Tente acessar ``http://symblog.dev/app_dev.php/hello/Fabien`` em seu navegador. Você deve ser 
informado de que o percurso não pôde ser encontrado. 

Como o teste acima faz uma requisição para a mesma URL, teremos a mesma resposta, daí, o porque do teste falhar. O teste 
funcional é uma parte grande deste capítulo e será abordado em detalhe mais tarde.

Como a classe ``DefaultController`` foi removida, você também pode remover esta classe de teste. Exclua a classe 
``DefaultControllerTest`` localizado em ``src/Blogger/BlogBundle/Tests/Controller/DefaultControllerTest.php``.

Testes Unitários
----------------

Como explicado anteriormente, teste de unidade está preocupado em testar unidades individuais de sua aplicação de forma 
isolada. Ao escrever testes de unidade, é recomendável que você replique a estrutura de pastas do pacote (Bundle) na 
pasta ``Tests``. Por exemplo, se você quiser testar a classe de entidade  ``Blog`` localizada em 
``src/Blogger/BlogBundle/Entity/blog.php``, o arquivo de teste deve estar em 
``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php``. 

Um exemplo da pasta de layout, seria como se segue:

.. code-block:: text

    src/Blogger/BlogBundle/
                    Entity/
                        Blog.php
                        Comment.php
                    Controller/
                        PageController.php
                    Twig/
                        Extensions/
                            BloggerBlogExtension.php
                    Tests/
                        Entity/
                            BlogTest.php
                            CommentTest.php
                        Controller/
                            PageControllerTest.php
                        Twig/
                            Extensions/
                                BloggerBlogExtensionTest.php

Observe que cada um dos arquivos de teste estão sufixados por ``Test``.

Testando a Entidade Blog - método Slugify
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Começaremos a testar o método slugify da entidade ``Blog``. Vamos escrever alguns testes para garantir que este método 
está funcionando corretamente. 

Crie um novo arquivo localizado em ``src/Blogger/BlogBundle/tests/Entity/BlogTest.php`` e adicione o seguinte código:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    namespace Blogger\BlogBundle\Tests\Entity;

    use Blogger\BlogBundle\Entity\Blog;

    class BlogTest extends \PHPUnit_Framework_TestCase
    {

    }

Nós criamos uma classe de teste para a entidade ``Blog``. Note que a localização do arquivo está em conformidade com a 
estrutura da pasta mencionada acima. 

A classe ``BlogTest`` estende a classe base do PHPUnit ``PHPUnit_Framework_TestCase``. Todos os testes que você escreve 
para PHPUnit, será um filho (child) da classe. Você vai se lembrar de capítulos anteriores que  ``\`` deve ser colocado 
na frente do nome da classe ``PHPUnit_Framework_TestCase`` pois a classe é declarada com namespace PHP público.

Agora que temos a classe esqueleto para testar a nossa entidade ``Blog``, vamos escrever um caso de teste. Os casos de 
testes em PHPUnit, são métodos da classe Test, prefixadas com ``test``, como ``testSlugify()``. 

Atualize o ``BlogTest`` localizado em ``src/Blogger/BlogBundle/Teste/Entity/BlogTest.php`` com o seguinte código:

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    class BlogTest extends \PHPUnit_Framework_TestCase
    {
        public function testSlugify()
        {
            $blog = new Blog();

            $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        }
    }

Este é um caso de teste muito simples. Ele instancia uma nova entidade ``Blog`` e executa um ``assertEquals()`` sobre o 
resultado do método ``slugify``. 

O método ``assertEquals()`` leva 2 argumentos obrigatórios, o resultado esperado e o resultado atual. Um terceiro 
argumento opcional, pode ser passado para especificar uma mensagem a ser exibida quando o caso de teste falhar.

Vamos executar o nosso novo teste de unidade executando o seguinte na linha de comando:

.. code-block:: bash

    $ phpunit -c app

Você deve ver a seguinte saída:

.. code-block :: bash

    PHPUnit 3.5.11 by Sebastian Bergmann.

    .

    Time: 1 second, Memory: 4.25Mb

    OK (1 test, 1 assertion)

A saída do PHPUnit é muito simples. Primeiro, exibe algumas informações sobre o PHPUnit e devolve um número de ``.`` 
para cada teste executado, no nosso caso, estamos executando apenas um teste, então, apenas 1 ``.`` é a exibido. 

A última instrução nos informa do resultado dos testes. Para o nosso ``BlogTest``, nós só executamos um teste com 1 
afirmação (assertion). 

Se seu prompt de comando exibir saídas com cores, você verá que a última linha exibida está com um fundo verde, 
informando que tudo está OK. 

Vamos atualizar o método ``testSlugify()`` para ver o que acontece quando os testes falham.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSlugify()
    {
        $blog = new Blog();

        $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        $this->assertEquals('a day with symfony2', $blog->slugify('A Day With Symfony2'));
    }

Re execute os testes de unidade como antes. A saída será apresentada como a exibida baixo:

.. code-block :: bash

    PHPUnit 3.5.11 by Sebastian Bergmann.

    F

    Time: 0 seconds, Memory: 4.25Mb

    There was 1 failure:

    1) Blogger\BlogBundle\Tests\Entity\BlogTest::testSlugify
    Failed asserting that two strings are equal.
    --- Expected
    +++ Actual
    @@ @@
    -a day with symfony2
    +a-day-with-symfony2

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Entity/BlogTest.php:15

    FAILURES!
    Tests: 1, Assertions: 2, Failures: 1.

A saída é um pouco mais envolvente neste momento. Podemos ver que o ``.`` para a execução dos testes foi substituído por 
um ``F``. Isto nos diz que o teste falhou. Você também verá o caractere ``E`` se o teste contém erros. 

Depois, o PHPUnit nos informa sobre as falhas em detalhes, neste caso, a falha 1. 

Nós podemos ver o método ``Blogger\BlogBundle\Tests\Entity\BlogTest::testSlugify`` falhou porque o valor esperado e o 
valores atual eram diferentes. 

Se seu prompt de comando exibir saídas com cores, você verá que a última linha exibida está em vermelho informando que 
houve falhas em seu teste. 

Corrija o método ``testSlugify()`` para que os testes sejam executados com êxito.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSlugify()
    {
        $blog = new Blog();

        $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        $this->assertEquals('a-day-with-symfony2', $blog->slugify('A Day With Symfony2'));
    }

Antes de seguirmos, adicione mais alguns testes para o método ``slugify()``.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSlugify()
    {
        $blog = new Blog();

        $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        $this->assertEquals('a-day-with-symfony2', $blog->slugify('A Day With Symfony2'));
        $this->assertEquals('hello-world', $blog->slugify('Hello    world'));
        $this->assertEquals('symblog', $blog->slugify('symblog '));
        $this->assertEquals('symblog', $blog->slugify(' symblog'));
    }

Agora que nós testamos o método ``slugify`` da entidade ``Blog``, é preciso garantir que o membro ``$slug`` de ``Blog`` 
está definido corretamente quando o membro ``$title`` do ``Blog`` é atualizado. 

Adicione os métodos a seguir no arquivo ``BlogTest`` localizado em ``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php``.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSetSlug()
    {
        $blog = new Blog();

        $blog->setSlug('Symfony2 Blog');
        $this->assertEquals('symfony2-blog', $blog->getSlug());
    }

    public function testSetTitle()
    {
        $blog = new Blog();

        $blog->setTitle('Hello World');
        $this->assertEquals('hello-world', $blog->getSlug());
    }

Começamos a testar o método ``setSlug`` para garantir que o membro ``$slug`` é executado (slugified) corretamente, 
quando atualizado. Depois, verifficamos que o membro ``$slug`` é corretamente atualizado quando o método ``setTitle`` é 
chamado na entidade ``Blog``.

Execute os testes para verificar que a entidade ``Blog`` está funcionando corretamente.

Testando a extensão do Twig
~~~~~~~~~~~~~~~~~~~~~~~~~~~

No capítulo anterior, criamos uma extensão do Twig para converter uma instância ``\DateTime`` em uma string detalhando o 
período de existência do post. 

Crie um novo arquivo de teste localizado em 
``src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php`` e o atualize com o seguinte conteúdo:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php

    namespace Blogger\BlogBundle\Tests\Twig\Extensions;

    use Blogger\BlogBundle\Twig\Extensions\BloggerBlogExtension;

    class BloggerBlogExtensionTest extends \PHPUnit_Framework_TestCase
    {
        public function testCreatedAgo()
        {
            $blog = new BloggerBlogExtension();

            $this->assertEquals("0 seconds ago", $blog->createdAgo(new \DateTime()));
            $this->assertEquals("34 seconds ago", $blog->createdAgo($this->getDateTime(-34)));
            $this->assertEquals("1 minute ago", $blog->createdAgo($this->getDateTime(-60)));
            $this->assertEquals("2 minutes ago", $blog->createdAgo($this->getDateTime(-120)));
            $this->assertEquals("1 hour ago", $blog->createdAgo($this->getDateTime(-3600)));
            $this->assertEquals("1 hour ago", $blog->createdAgo($this->getDateTime(-3601)));
            $this->assertEquals("2 hours ago", $blog->createdAgo($this->getDateTime(-7200)));

            // Cannot create time in the future
            $this->setExpectedException('\InvalidArgumentException');
            $blog->createdAgo($this->getDateTime(60));
        }

        protected function getDateTime($delta)
        {
            return new \DateTime(date("Y-m-d H:i:s", time()+$delta));
        }
    }

A classe está configurada da mesma forma como antes, criando um método ``testCreatedAgo()`` para testar a extensão do 
Twig. Nós introduzimos um outro método PHPUnit neste caso de teste, o método ``setExpectedException()``. Este método 
deve ser chamado antes da execução de um método que você espera lançar uma exceção. 

Sabemos que o método ``createdAgo`` da extensão do Twig, não pode lidar com datas no futuro, então, irá lançar uma 
``\Exception``. 

O método ``getDateTime()`` é simplesmente um método auxiliar para criar uma instância ``\DateTime``. Observe que não é 
prefixado com o ``test``, assim, o PHPUnit não vai tentar executá-lo como um caso de teste. 

Abra a linha de comando e execute os testes para esse arquivo. Nós poderíamos simplesmente executar o teste como antes, 
mas, também podemos dizer ao PHPUnit para executar testes para uma pasta específica (e suas sub-pastas) ou um arquivo. 

Execute o seguinte comando:

.. code-block:: bash

    $ phpunit -c app src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php

Isto irá executar os testes somente para o arquivo ``BloggerBlogExtensionTest``. O PHPUnit nos informa que os testes 
falharam. A saída é mostrada abaixo:

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Twig\Extension\BloggerBlogExtensionTest::testCreatedAgo
    Failed asserting that two strings are equal.
    --- Expected
    +++ Actual
    @@ @@
    -0 seconds ago
    +0 second ago

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php:14

Esperavamos que a primeira afirmação retornasse ``0 segundos atrás (0 seconds ago)``, mas não o fez, a palavra 
``segundo`` não estava no plural. 

Vamos atualizar a Extensão do Twig, localizado em ``src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php``, para 
corrigir isso.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php

    namespace Blogger\BlogBundle\Twig\Extensions;

    class BloggerBlogExtension extends \Twig_Extension
    {
        // ..

        public function createdAgo(\DateTime $dateTime)
        {
            // ..
            if ($delta < 60)
            {
                // Seconds
                $time = $delta;
                $duration = $time . " second" . (($time === 0 || $time > 1) ? "s" : "") . " ago";
            }
            // ..
        }

        // ..
    }

Re execute os testes PHPUnit. Você deverá ver que, a primeira afirmação é passanda corretamente, mas o nosso caso de 
teste ainda continua a falhar. Vamos examinar a próxima saída:

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Twig\Extension\BloggerBlogExtensionTest::testCreatedAgo
    Failed asserting that two strings are equal.
    --- Expected
    +++ Actual
    @@ @@
    -1 hour ago
    +60 minutes ago

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php:18

Podemos ver agora que a quinta afirmação está falhando (observe o ``18`` no final da saída, isso nos dá o número da 
linha no arquivo onde a afirmação falhou). 

Observando o caso de teste, podemos ver que a extensão do Twig tem funcionado incorretamente. 1 hora atrás 
``1 hour ago`` deveria ter sido devolvido, mas em vez disso, foi retornado 60 minutos atrás ``60 minutes ago``. 

Se examinarmos o código da extensão Twig ``BloggerBlogExtension``, podemos ver a razão. Nós comparamos o tempo para ser 
inclusivo, ou seja, usamos ``<=`` ao invés de ``<``. Observe que isso causa a verificação em horas. 

Atualize a extensão Twig, localizado em ``src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php`` para corrigir 
este problema.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php

    namespace Blogger\BlogBundle\Twig\Extensions;

    class BloggerBlogExtension extends \Twig_Extension
    {
        // ..

        public function createdAgo(\DateTime $dateTime)
        {
            // ..

            else if ($delta < 3600)
            {
                // Mins
                $time = floor($delta / 60);
                $duration = $time . " minute" . (($time > 1) ? "s" : "") . " ago";
            }
            else if ($delta < 86400)
            {
                // Hours
                $time = floor($delta / 3600);
                $duration = $time . " hour" . (($time > 1) ? "s" : "") . " ago";
            }

            // ..
        }

        // ..
    }

Agora, re execute todos os testes usando o seguinte comando:

.. code-block:: bash

    $ phpunit -c app

Este comando executa todos os testes e mostra que todos os testes passaram com sucesso. 

Embora tenhamos escrito poucos testes de unidade, você deve estar percebendo como os testes são importantes, quando se 
escreve código. Apesar dos erros acima serem pequenos, eles ainda eram erros. 

Teste também ajuda, a qualquer funcionalidade futura adicionada ao projeto, romper características anteriores. 

Concluímos o teste de unidade por agora. Veremos mais sobre teste de unidade nos capítulos seguintes. 

Tente adicionar algum de seus próprios testes de unidade, para testar as funcionalidade que não foram abordadas aqui.

Testes Funcionais
-----------------

Agora que nós escrevemos alguns testes de unidade, vamos passar para teste de vários componentes simultâneos. 

A primeira seção do teste funcional, envolverá simulação de requisições ao navegador para testar as respostas geradas.

Testando a página Sobre
~~~~~~~~~~~~~~~~~~~~~~~

Começamos testando a classe para a página sobre em ``PageController``. Como a página sobre é muito simples, este é um 
bom lugar para começar. 

Crie um novo arquivo localizado em ``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php`` e adicione o 
seguinte conteúdo:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    namespace Blogger\BlogBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class PageControllerTest extends WebTestCase
    {
        public function testAbout()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/about');

            $this->assertEquals(1, $crawler->filter('h1:contains("About symblog")')->count());
        }
    }

Nós já vimos um controlador de teste muito semelhante a este quando observamos a classe ``DefaultControllerTest``. Esta 
classe está testando a página Sobre de Symblog, verificando se a string ``About Symblog`` está presente no HTML gerado, 
especificamente, dentro da tag ``H1``. 

A classe ``PageControllerTest``, não estende ``\PHPUnit_Framework_TestCase``, como vimos com os exemplos de testes de 
unidade. Em vez disso, estende a classe ``WebTestCase``. Essa classe é parte do pacote do Framework Symfony 2.

Como explicado anteriormente, classes de teste PHPUnit devem estender a ``\PHPUnit_Framework_TestCase``, mas, quando uma 
funcionalidade extra ou comum é necessária para vários casos de teste, é melhor encapsular esta funcionalidade na sua 
própria classe e fazer com que estas classes de teste estendam dela. 

O ``WebTestCase`` faz exatamente isso, ele fornece vários métodos úteis para a execução de testes funcionais em 
Symfony 2. 

Observe o arquivo ``WebTestCase`` localizado em 
``vendor/symfony/src/Symfony/Bundle/FrameworkBundle/Test/WebTestCase.php``, você vai ver que esta classe é, na verdade, 
uma extensão da classe ``\PHPUnit_Framework_TestCase``.

.. code-block:: php

    // vendor/symfony/src/Symfony/Bundle/FrameworkBundle/Test/WebTestCase.php

    abstract class WebTestCase extends \PHPUnit_Framework_TestCase
    {
        // ..
    }

Se você observar o método ``createClient()`` na classe ``WebTestCase``, você pode ver que ele cria uma instância do 
Kernel do Symfony 2. Seguindo os métodos passados, você também vai perceber que o ``ambiente`` ``test`` é definido (A 
menos que seja substituído como um dos argumentos para ``createClient()``). Este é o ambiente de ``test`` que falamos no 
capítulo anterior.

Observando nossa classe de teste, podemos ver que o método ``createClient()`` é chamado a fazer o teste e executá-lo. 

Então, chamamos o método ``request()`` no cliente para simular uma solicitação HTTP GET do navegador para a url 
``/about`` (o mesmo que você faz para visitar ``http://symblog.dev/about`` no seu navegador). A requisição nos dá um 
objeto ``Crawler``, que contém a resposta. 

A classe ``Crawler`` é muito útil, pois nos permite percorrer o HTML retornado. Usamos a instância do ``Crawler`` para 
verificar que a tag ``H1`` na resposta HTML, contém as palavras ``About Symblog``. 

Observe que, apesar de estarmos estendendo a classe ``WebTestCase``, ainda usamos o método de afirmação como antes 
(Lembre-se, a classe ``PageControllerTest`` ainda é filha da classe ``\PHPUnit_Framework_TestCase``).

Vamos executar ``PageControllerTest`` usando o seguinte comando. Quando escrevemos testes, é melhor executar os testes 
somente para o arquivo que você está trabalhando atualmente. Quando o seu pacote de testes se torna grande, a execução 
de testes pode ser uma tarefa demorada.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Você pode observar a seguinte mensagem ``OK (1 test, 1 assertion)`` nos mostrando que um teste (o ``testAbout()``) foi 
executado com 1 afirmação (o ``assertEquals()``).

Tente alterar a string ``About Symblog`` por ``Contato`` e execute novamente o teste. O teste irá falhar pois 
``Contato`` não vai ser encontrada, fazendo com que ``asertEquals`` equivalha a false.

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Controller\PageControllerTest::testAbout
    Failed asserting that 0 matches expected 1.

Retorne o valor da string  para ``About Symblog`` antes de prosseguirmos.

A instância do ``Crawler`` utilizado, permite percorrer documentos HTML ou XML (o que significa que o ``Crawler`` só vai 
funcionar com as respostas que retornam HTML ou XML). 

Podemos usar o ``Crawler`` para passar a resposta gerada usando outros métodos, tais como ``filter()``, ``first()``, 
``last()``, e ``parents()``. Se você usa `jQuery <http://jquery.com/>`_, você deve estar se sentindo familiarizado com a 
classe ``Crawler``. 

A lista completa de métodos de passagens da classe ``Crawler``, pode ser encontrada no capítulo 
`Testes  <http://symfony.com/doc/current/book/testing.html#traversing>`_ do livro dio Symfony 2. 

Vamos explorar outros recursos do ``Crawler`` à medida que prosseguimos.

Página inicial
~~~~~~~~~~~~~~

Apesar do teste para a página Sobre ser simples, delineamos os princípios básicos de testes funcionais das páginas do 
site.

 1. Crie o cliente
 2. Solicite uma página
 3. Verifique a resposta

Esta é uma visão geral simples do processo. De fato, existem vários outros passos que também poderíamos fazer, como 
clicar em links e preencher e enviar formulários.

Vamos criar um método para testar a página inicial. Sabemos que a página inicial está disponível através da URL ``/`` e 
que deve exibir as mensagens mais recentes dos posts do blog. 

Adicione um novo método ``testIndex()`` para a classe ``PageControllerTest`` localizada em 
``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php`` como mostrado abaixo:

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testIndex()
    {
        $client = static::createClient();

        $crawler = $client->request('GET', '/');

        // Check there are some blog entries on the page
        $this->assertTrue($crawler->filter('article.blog')->count() > 0);
    }

Você pôde observar que são os mesmos passos tomados com os testes para a página Sobre. Execute o teste para garantir que 
tudo está funcionando como esperado.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Vamos agora levar o teste um pouco mais adiante. Parte do teste funcional envolve ser capaz de reproduzir o que um 
usuário faria no site. 

Para que os usuários naveguem entre as páginas do seu site, eles devem clicar em links. Vamos simular esta ação agora 
para testar os links para a página do blog mostrando que funcionam corretamente quando o título do blog é clicado. 

Atualize o método ``testIndex()`` na classe ``PageControllerTest`` com o seguinte código:

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testIndex()
    {
        // ..

        // Find the first link, get the title, ensure this is loaded on the next page
        $blogLink   = $crawler->filter('article.blog h2 a')->first();
        $blogTitle  = $blogLink->text();
        $crawler    = $client->click($blogLink->link());

        // Check the h2 has the blog title in it
        $this->assertEquals(1, $crawler->filter('h2:contains("' . $blogTitle .'")')->count());
    }

A primeira coisa que fizemos foi usar o ``Crawler`` para extrair o texto dentro do primeiro link do título do Blog. Isso 
é feito usando o filtro ``article.blog h2 a``. Este filtro é usado para retornar a tag ``a`` dentro da tag ``H2`` do 
artigo ``article.blog``. 

Para entender isso melhor, dê um olhar na marcação usada na página inicial para a exibição de blogs.

.. code-block:: html

    <article class="blog">
        <div class="date"><time datetime="2011-09-05T21:06:19+01:00">Monday, September 5, 2011</time></div>
        <header>
            <h2><a href="/app_dev.php/1/a-day-with-symfony2">A day with Symfony2</a></h2>
        </header>

        <!-- .. -->
    </article>
    <article class="blog">
        <div class="date"><time datetime="2011-09-05T21:06:19+01:00">Monday, September 5, 2011</time></div>
        <header>
            <h2><a href="/app_dev.php/2/the-pool-on-the-roof-must-have-a-leak">The pool on the roof must have a leak</a></h2>
        </header>

        <!-- .. -->
    </article>

Você pode ver a estrutura do filtro ``article.blog h2 a`` no lugar da marcação, na página principal. Você também vai 
notar que há mais de um ``<article class="blog">`` na marcação, o que significa que o filtro do ``Crawler`` irá retornar 
uma coleção. 

Como só queremos o primeiro link, usamos o método ``first()`` na coleção. Finalmente, usamos o método ``text()`` para 
extrair o texto do link, neste caso, será o texto ``Um dia com Symfony2``. 

Em seguida, o link do título do blog é clicado para navegar para a página exibição do blog. O método cliente ``click()`` 
utiliza um objeto de ligação e retorna o ``Response`` em uma instância do ``Crawler``. 

Percebendo que o objeto ``Crawler`` é uma peça chave para o teste funcional.

O objeto ``Crawler``, agora, contém a resposta para a página de apresentação do blog. Precisamos testar se o link que 
clicamos nos levou para a página correta. Podemos usar o valor de ``$BlogTitle``, que recuperamos mais cedo, para 
verificar se há um título na Resposta.

Execute os testes para garantir que a navegação, entre a página inicial e a página de exibição do blog, está funcionando 
corretamente.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Agora que você tem uma compreensão de como navegar através das páginas do site utilizando o teste funcional, vamos 
passar para os testes de formulários.

Testando a Página Contato
~~~~~~~~~~~~~~~~~~~~~~~~~

Usuários do Symblog são capazes de enviar informações de contato através do preenchimento do formulário da página de 
contato ``http://symblog.dev/contact``. Vamos testar se as submissões do formmulário funcionam corretamente. 

Primeiro, precisamos delinear o que deve acontecer quando o formulário é submetido corretamente (submetido com êxito, 
neste caso, significa não há erros presentes no formulário).

 1. Navegue até a página de contato
 2. Preencher formulário de contato com os valores
 3. Enviar formulário
 4. Verifique se o e-mail foi enviado para Symblog
 5. Confira se a resposta para o cliente, contém notificação de contato bem sucedido

Até agora, sabemos o suficiente para completar os passos 1 e 5 apenas. Iremos, agora, saber como testar os 3 passos 
intermediários.

Adicione um novo método ``testContact()`` para a classe ``PageControllerTest`` localizada em 
``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php``.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testContact()
    {
        $client = static::createClient();

        $crawler = $client->request('GET', '/contact');

        $this->assertEquals(1, $crawler->filter('h1:contains("Contact symblog")')->count());

        // Select based on button value, or id or name for buttons
        $form = $crawler->selectButton('Submit')->form();

        $form['blogger_blogbundle_enquirytype[name]']       = 'name';
        $form['blogger_blogbundle_enquirytype[email]']      = 'email@email.com';
        $form['blogger_blogbundle_enquirytype[subject]']    = 'Subject';
        $form['blogger_blogbundle_enquirytype[body]']       = 'The comment body must be at least 50 characters long as there is a validation constrain on the Enquiry entity';

        $crawler = $client->submit($form);

        $this->assertEquals(1, $crawler->filter('.blogger-notice:contains("Your contact enquiry was successfully sent. Thank you!")')->count());
    }

Começamos na forma usual, fazendo uma solicitação para a URL ``/contact`` e verificamos se a página contém o titulo 
``H1`` correto. Depois, usamos o ``Crawler`` para selecionar o botão enviar do formulário. 

A razão pela qual selecionamos o botão e não o formulário, é que um formulário pode conter vários botões que podemos 
querer clicar de forma independente. A partir do botão selecionado, somos capazes de recuperar o formulário. Somos 
capazes de definir os valores do formulário usando o array de subscrita ``[]``. 

Finalmente, o formulário é passado para o método cliente ``submit()`` para realmente enviar o formulário. Como de 
costume, recebemos um retorno da instância do ``Crawler``. Usando essa resposta, vamos verificar para garantir que as 
mensagens estão presentes no retorno da resposta. 

Execute o teste para verificar se tudo está funcionando corretamente.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Os testes falharam. Nós recebemos a seguinte saída do PHPUnit:

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Controller\PageControllerTest::testContact
    Failed asserting that <integer:0> matches expected <integer:1>.

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php:53

    FAILURES!
    Tests: 3, Assertions: 5, Failures: 1.

A saída está nos informando que a mensagem de texto não pôde ser encontrado na resposta do formulário de envio. Isto 
ocorreu porque, quando nós estamos no ambiente de ``teste``, redirecionamentos não são seguidos. Quando o formulário for 
validado com sucesso na classe ``PageController``, um redirecionamento acontece. Esse redirecionamento não está sendo 
seguido. Precisamos dizer, explicitamente, que o redirecionamento deve ser seguido. 

O motivo pelo qual o redirecionamento não é seguido, é simples. Você pode querer verificar a atual resposta primeiro. 

Vamos demonstrar isso em breve para verificar se o e-mail foi enviado. Atualize a classe ``PageControllerTest`` para 
configurar o cliente para acompanhar o redirecionamento.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testContact()
    {
        // ..

        $crawler = $client->submit($form);

        // Need to follow redirect
        $crawler = $client->followRedirect();

        $this->assertEquals(1, $crawler->filter('.blogger-notice:contains("Your contact enquiry was successfully sent. Thank you!")')->count());
    }

Agora, quando você executa os testes PHPUnit, eles devem passar. Vamos, agora, observar a etapa final do processo de 
envio do formulário de contato, o passo 4, verificar se um e-mail foi enviado para Symblog. 

Nós já sabemos que e-mails não serão entregues no ambiente de ``test`` , devido à seguinte configuração.

.. code-block:: yaml

    # app/config/config_test.yml

    swiftmailer:
        disable_delivery: true

Podemos testar se os e-mails foram enviados utilizando a informação recolhida pelo web profiler. Este arquivo é que diz 
ao cliente não fazer redirecionamentos. 

A verificação do profiler, precisa ser feita antes que o redirecionamento acontecer, se não, as informações no perfil 
serão perdidas. Atualize o método ``testContact()`` com o seguinte código:

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testContact()
    {
        // ..

        $crawler = $client->submit($form);

        // Check email has been sent
        if ($profile = $client->getProfile())
        {
            $swiftMailerProfiler = $profile->getCollector('swiftmailer');

            // Only 1 message should have been sent
            $this->assertEquals(1, $swiftMailerProfiler->getMessageCount());

            // Get the first message
            $messages = $swiftMailerProfiler->getMessages();
            $message  = array_shift($messages);

            $symblogEmail = $client->getContainer()->getParameter('blogger_blog.emails.contact_email');
            // Check message is being sent to correct address
            $this->assertArrayHasKey($symblogEmail, $message->getTo());
        }

        // Need to follow redirect
        $crawler = $client->followRedirect();

        $this->assertTrue($crawler->filter('.blogger-notice:contains("Your contact enquiry was successfully sent. Thank you!")')->count() > 0);
    }

Após o envio do formulário, vamos verificar se o perfil está disponível, pois ele pode ter sido desativado por uma 
configuração do ambiente atual.

.. tip::

    Lembre-se, testes não tem que ser executados no ambiente de ``teste``, eles poderiam ser executados no ambiente de 
    ``produção``, onde as informações do profiler estarão disponíveis.

Se somos capazes de obter o profiler, faremos um pedido para recuperar o coletor ``SwiftMailer``. O coletor 
``SwiftMailer`` trabalha nos bastidores para coletar informações sobre como o serviço de e-mail é usado. Podemos usar 
isso para obter informações sobre quais e-mails foram enviados.

Agora, usaremos o método ``getMessageCount()`` para verificar se um e-mail foi enviado. Este método talvez seja o 
suficiente para garantir que pelo menos um e-mail vai ser enviado, mas não verifica que o e-mail será enviado para o 
local correto. Poderia ser muito constrangedor ou até mesmo prejudicial, se e-mails fossem enviados para o endereço de 
e-mail errado. Para verificar isso, não é o caso, vamos verificar se o e-mail que vai receber a mensagem, está correto.

Agora, re-execute os testes para verificar se tudo está funcionando corretamente.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Teste de Adição de comentários do blog
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Vamos, agora, usar o conhecimento que adquirimos a partir dos testes anteriores, na página de contato, para testar o 
processo de envio de um comentário no blog. 

Mais uma vez, destacamos o que deve acontecer quando o formulário é enviado com sucesso:

 1. Navegue até uma página de blog
 2. Preencher formulário de comentar com os valores
 3. Enviar formulário
 4. Confira se o novo comentário é adicionado ao fim da lista de comentários do blog
 5. Além disso, verifique os comentários mais recentes da barra lateral para assegurar que o comentário está no topo da 
    lista

Crie um novo arquivo localizado em ``src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php`` e adicione o 
seguinte código:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php

    namespace Blogger\BlogBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class BlogControllerTest extends WebTestCase
    {
        public function testAddBlogComment()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/1/a-day-with-symfony');

            $this->assertEquals(1, $crawler->filter('h2:contains("A day with Symfony2")')->count());

            // Select based on button value, or id or name for buttons
            $form = $crawler->selectButton('Submit')->form();

            $crawler = $client->submit($form, array(
                'blogger_blogbundle_commenttype[user]'          => 'name',
                'blogger_blogbundle_commenttype[comment]'       => 'comment',
            ));

            // Need to follow redirect
            $crawler = $client->followRedirect();

            // Check comment is now displaying on page, as the last entry. This ensure comments
            // are posted in order of oldest to newest
            $articleCrawler = $crawler->filter('section .previous-comments article')->last();

            $this->assertEquals('name', $articleCrawler->filter('header span.highlight')->text());
            $this->assertEquals('comment', $articleCrawler->filter('p')->last()->text());

            // Check the sidebar to ensure latest comments are display and there is 10 of them

            $this->assertEquals(10, $crawler->filter('aside.sidebar section')->last()
                                            ->filter('article')->count()
            );

            $this->assertEquals('name', $crawler->filter('aside.sidebar section')->last()
                                                ->filter('article')->first()
                                                ->filter('header span.highlight')->text()
            );
        }
    }

Antes de começar a dissecar o código, execute os testes para este arquivo para garantir que tudo está funcionando 
corretamente.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php

PHPUnit deve informá-lo que o teste 1 foi executado com êxito. 

Observando o código de ``testAddBlogComment()``, podemos ver as coisas acontecendo da seguinte forma: criamos um cliente, 
solicitamos uma página e verificamos se a página que estamos é a correta. 

Precisamos prosseguir para obter adição do formulário de comentário e enviá-lo. Desta vez, vamos preencher os valores do 
formulário de uma forma um pouco diferente da versão anterior. Desta vez, usaremos o segundo argumento do método cliente 
``submit()`` para passar os valores para o formulário.

.. tip::

    Poderíamos, também, utilizar a interface Orientada a Objetos para definir os valores dos campos do formulário. 
    Alguns exemplos são mostrados abaixo:

    .. code-block:: php

        // Tick a checkbox
        $form['show_emal']->tick();
        
        // Select an option or a radio
        $form['gender']->select('Male');

Após enviar o formulário, solicitamos que o cliente siga o redirecionamento para que possamos verificar a resposta. 
Usamos o ``Crawler`` novamente para obter o último comentário no blog, que deve ser o único que acabamos de enviar. 

Por fim, verifique os últimos comentários na barra lateral para verificar se o comentário, que acabamos de enviar, é, 
também, o primeiro na lista.

Repositório do Blog
~~~~~~~~~~~~~~~~~~~ 

Na última parte do teste funcional, que abordamos neste capítulo, testaremos um repositório do Doctrine 2. 

Crie um novo arquivo localizado em ``src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php`` e adicione o 
seguinte conteúdo:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php

    namespace Blogger\BlogBundle\Tests\Repository;

    use Blogger\BlogBundle\Repository\BlogRepository;
    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class BlogRepositoryTest extends WebTestCase
    {
        /**
         * @var \Blogger\BlogBundle\Repository\BlogRepository
         */
        private $blogRepository;

        public function setUp()
        {
            $kernel = static::createKernel();
            $kernel->boot();
            $this->blogRepository = $kernel->getContainer()
                                           ->get('doctrine.orm.entity_manager')
                                           ->getRepository('BloggerBlogBundle:Blog');
        }

        public function testGetTags()
        {
            $tags = $this->blogRepository->getTags();

            $this->assertTrue(count($tags) > 1);
            $this->assertContains('symblog', $tags);
        }

        public function testGetTagWeights()
        {
            $tagsWeight = $this->blogRepository->getTagWeights(
                array('php', 'code', 'code', 'symblog', 'blog')
            );

            $this->assertTrue(count($tagsWeight) > 1);

            // Test case where count is over max weight of 5
            $tagsWeight = $this->blogRepository->getTagWeights(
                array_fill(0, 10, 'php')
            );

            $this->assertTrue(count($tagsWeight) >= 1);

            // Test case with multiple counts over max weight of 5
            $tagsWeight = $this->blogRepository->getTagWeights(
                array_merge(array_fill(0, 10, 'php'), array_fill(0, 2, 'html'), array_fill(0, 6, 'js'))
            );

            $this->assertEquals(5, $tagsWeight['php']);
            $this->assertEquals(3, $tagsWeight['js']);
            $this->assertEquals(1, $tagsWeight['html']);

            // Test empty case
            $tagsWeight = $this->blogRepository->getTagWeights(array());

            $this->assertEmpty($tagsWeight);
        }
    }

Como queremos realizar testes que requerem uma conexão válida ao banco de dados, estendemos o ``WebTestCase`` novamente 
pois nos permitir inicializar o Kernel do Symfony 2. Execute este teste para este arquivo usando o seguinte comando:

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php

Cobertura dos Testes nos Códigos
--------------------------------

Antes de prosseguirmos, vamos abordar rapidamente a cobertura de código. Cobertura de código nos dá um insight sobre 
quais partes do código que são executados quando os testes são chamados. Assim, podemos ver as partes do nosso código 
que não têm testes sendo executados sobre eles e determinar se precisamos escrever teste para eles.

Para emitir a análise de cobertura de código para a sua aplicação, execute o seguinte comando:

.. code-block:: bash

    $ phpunit --coverage-html ./phpunit-report -c app/

A saída será a análise de cobertura de código para a pasta ``phpunit-report``. Abra o arquivo ``index.html`` no seu 
navegador para ver o resultado da análise.

Leia o capítulo `Análise da Cobertura de Código <http://www.phpunit.de/manual/current/en/code-coverage-analysis.html>`_ 
na documentação do PHPUnit para maiores informações.

Conclusão
---------

Nós cobrimos várias áreas-chave no que diz respeito aos testes. Nós exploramos tanto o teste de unidade quanto teste 
funcional, para garantir que o nosso site está funcionando corretamente. Vimos como simular solicitações do navegador e 
como usar a classe ``Crawler`` do Symfony 2 para verificarmos as respostas à essas solicitações.

Em seguida, vamos abordar o componente de segurança do Symfony 2, e, mais especificamente, como usá-lo para 
gerenciamento de usuários. Também vamos integrar o ``FOSUserBundle`` para que possamos trabalhar na seção admin do 
Symblog.
