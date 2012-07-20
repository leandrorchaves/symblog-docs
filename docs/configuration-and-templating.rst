[Parte 1] - Configuração do Symfony 2 e Criação de layout 
========================================================= 

Visão geral 
----------- 

Este capítulo cobrirá os primeiros passos na criação de um site utilizando Symfony 2. 

Vamos baixar e configurar a distribuição `Standard <http://symfony.com/doc/current/glossary.html#term-distribution>`_ 
do Framework, criar o pacote (bundle) Blog e montar os principai layouts utilizando HTML. 

No final deste capítulo, você terá configurado um site em Symfony 2 que estará disponível através de um domínio local, 
por exemplo: ``http://symblog.dev/``. 

O site irá conter a estrutura principal HTML do blog, juntamente com algum conteúdo fictício. 

As seguintes áreas serão abordadas neste capítulo: 

    1. Configurando um aplicativo Symfony 2 
    2. Configurando um domínio do desenvolvimento 
    3. Pacotes (Bundles) Symfony 2 
    4. Roteamento 
    5. Controladores 
    6. Geração de layout com Twig 

Download e Instalação 
--------------------- 

Como dito acima, estaremos usando a distribuição Standard do Framework. Esta distribuição vem completa com as principais 
bibliotecas e pacotes mais comuns necessários para o desenvolvimento de sites. 

Você pode fazer o `Download <http://symfony.com/download>`_ do pacote Symfony 2 diretamente do site do Symfony 2. 

Como não queremos repetir a excelente documentação fornecida pelo livro do Symfony 2, por favor, consulte o capítulo 
`Instalando e configurando Symfony 2  <http://symfony.com/doc/current/book/installation.html>`_ para obter maiores 
detalhes, como por exemplo, como instalar os vendors necessários, e como designar corretamente as permissões para as 
pastas corretas.

.. warning::
    
    É importante prestar atenção em 
    `Configurando Permissões <http://symfony.com/doc/current/book/installation.html#configuration-and-setup>`_ na seção 
    do capítulo de instalação. Esta seção explica várias maneiras como você pode designar permissão para as pastas 
    ``app/cache`` e ``app/logs`` para que os usuário do servidor e os usuário de linha de comando possam ter acesso de 
    escrita nestas pastas. 

Criando um Domínio de Desenvolvimento 
------------------------------------- 

Como padrão deste tutorial, iremos utilizar o domínio local ``Http://symblog.dev/``, porém você pode escolher qualquer 
domínio que você deseja. 

Estas instruções são específicas para `Apache <http://httpd.apache.org/>`_ . Assim, assumimos que você já tenha o Apache 
instalado e funcionando em sua máquina. Se você utiliza outro servidor para utilização em seus domínios locais, como 
`Nginx <http://nginx.net/>`_, por exemplo, você pode pular esta seção.

.. note::
    :title: Observação

     Estas etapas foram realizadas na distribuição Fedora Linux. Então, nomes de caminho, etc, podem ser diferentes 
    dependendo do seu sistema operacional. 

Vamos começar criando um domínio virtual com o Apache. 

Localize o arquivo de configuração do Apache e acrescente as seguintes configurações, certificando-se de mudar o 
``DocumentRoot`` e os caminhos de diretório. 

A localização e nome da configuração do Apache pode variar muito dependendo do seu sistema operacional. No Fedora , sua 
localização é ``/etc/httpd/conf/httpd.conf``. Você terá que editar este arquivo com privilégio de ``sudo``.

.. code-block:: text

    # /etc/httpd/conf/httpd.conf

    NameVirtualHost 127.0.0.1

    <VirtualHost 127.0.0.1>
      ServerName symblog.dev
      DocumentRoot "/var/www/html/symblog.dev/web"
      DirectoryIndex app.php
      <Directory "/var/www/html/symblog.dev/web">
        AllowOverride All
        Allow from All
      </Directory>
    </VirtualHost>

Em seguida, adicione um novo domínio no arquivo host localizado em ``/etc/hosts``. Novamente, você terá que editar este 
arquivo com privilégios ``sudo``. 

.. code-block:: text

    # /etc/hosts
    127.0.0.1     symblog.dev

Por último não se esqueça de reiniciar o serviço Apache. Assim, o servidor irá recarregar as definições de configuração 
que fizemos. 

.. code-block:: bash

    $ sudo service httpd restart

.. tip::

    Se você cria seus próprios domínios virtuais, você pode simplificar seu processo, usando 
    `Hosts virtuais dinâmicos <http://blog.dsyph3r.com/2010/11/apache-dynamic-virtual-hosts.html>`_. 

Agora você deve ser capaz de acessar ``http://symblog.dev/app_dev.php/ ``. 

.. image:: /_static/images/part_1/welcome.jpg
    :align: center
    :alt: Symfony2 welcome page

Se esta é primeira vez que você vê a página de boas-vindas do Symfony 2, gaste um tempo observando as páginas de 
demonstração. Cada página exibe trechos de código que demonstram como cada página funciona.

.. note::
    :title: Observação

    Você também vai perceber uma barra de ferramentas na parte inferior da tela de boas-vindas. Esta é a barra de 
    ferramentas do desenvolvedor e fornece a você informações muito importantes sobre o estado da aplicação como, por 
    exemplo, o tempo de execução da página, uso de memória, as consultas de banco de dados, estado de autenticação e 
    muito mais pode ser visto a partir desta barra de ferramentas. 

    Por padrão, a barra de ferramentas só é visível quando estamos no ambiente ``dev``, pois exibir a barra de 
    ferramentas no ambiente de produção seria um grande risco à segurança porque ela expõe muitas informações da sua 
    aplicação. 

    As referências à barra de ferramentas serão feitas no decorrer deste tutorial à medida que formos introduzindo novas 
    funcionalidades.

Configurando Symfony: Interface Web 
----------------------------------- 

Symfony 2 possui uma interface web para configurar vários aspectos relacionados ao site, tais como configurações de 
banco de dados. Precisamos de um banco de dados para este projeto então vamos começar a usar o configurador. 

Acesse ``http://symblog.dev/app_dev.php/`` e clique no botão ``Configure``. Forneça os detalhes para configurar o banco 
de dados (este tutorial assume o uso do MySQL, mas você pode escolher qualquer outro banco de dados de sua preferência). Na próxima página, gere um token de segurança CSRF. Será apresentado a você as definições dos parâmetros que o Symfony 2 gerou. Preste atenção ao aviso que possa surgir na página, pois é bem provável que você não tenha acesso de escrita no seu arquivo ``app/config/parameters.ini`` sendo necessário copiar e colar as configurações neste arquivo (Estas configurações podem substituir as definições já existentes neste arquivo). 


Pacotes (Bundles): Construindo blocos com Symfony 2 
--------------------------------------------------- 

Os pacotes (bundles) são blocos básicos de construção de qualquer aplicação Symfony 2, e só pra constar, o Symfony 2 é 
um pacote. Pacotes nos permitem separar funcionalidades para fornecer unidades de código reutilizáveis. Eles encapsulam 
as entradas afim de dar suporte aos propósitos dos pacotes incluindo controladores, o modelo, os layouts e diversos 
outros recursos, tais como imagens e CSS. Criaremos um pacote para o nosso site com namespace ``Blogger``. Se você não 
estiver familiarizado com ``namespaces`` em PHP você deve gastar um tempo lendo sobre eles pois eles são muito usados em 
Symfony 2. Leia o `Symfony 2 autoloader <http://symfony.com/doc/current/cookbook/tools/autoloader.html>`_ para maiores 
detalhes sobre como Symfony 2 trabalha com autoloading.

.. tip::
    :title: Dica!

    Um bom entendimento de namespaces pode ajudar a eliminar problemas comuns que você pode enfrentar ao ter de mapear 
    corretamente as estruturas de pastas sem namespace. 

Criando o pacote 
~~~~~~~~~~~~~~~~ 

Para encapsular funcionalidades para o blog, vamos criar um pacote Blog. Este pacote irá abrigar todos os arquivos 
necessários para o trabalho da aplicação Symfony 2. 

Symfony 2 fornece uma série de ferramentas para nos auxiliar na execução de operações comuns. Uma dessas ferramentas é o 
gerador de pacote. 

Para iniciar o gerador de pacote execute o seguinte comando. Você verá uma série de instruções que permitem configurar a 
forma como o pacote pode ser configurado. Cada solicitação deve seguir um padrão.

.. code-block:: bash

    $ php app/console generate:bundle --namespace=Blogger/BlogBundle --format=yml

Após a execução do código acima, o gerador do Symfony 2 terá construído o pacote base. Algumas mudanças importantes são 
observadas aqui.

.. tip::
    :title: Dica!

    Você não precisa usar as opções do gerador de pacote do Symfony 2, elas são simplesmente para ajudá-lo. 

    Você poderia ter criado manualmente a estrutura de pastas e arquivos. Embora não seja obrigatório o uso do gerador, 
    ele fornece alguns benefícios como agilidade e execução de todas as tarefas básicas para deixar o pacote instalado e 
    funcionando. Um exemplo disso é o registrando do pacote. 

Registrando o pacote 
.................... 

O nosso novo pacote ``BloggerBlogBundle`` foi registrado no Kernel da aplicação localizado em 
``App/AppKernel.php``. O Symfony 2 nos obriga a registrar todos os pacotes que a aplicação precisa usar. 

Você também vai notar que alguns pacotes só são registrados quando estão em ambientes ``dev`` ou ``test``. 

Carregando estes pacotes no ambiente``prod``(Produção) iria provocar sobrecarga adicional para a funcionalidade que não 
seriam utilizados. O trecho abaixo mostra como o ``BloggerBlogBundle`` foi registrado.

.. code-block:: php

    // app/AppKernel.php
    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            $bundles = array(
            // ..
                new Blogger\BlogBundle\BloggerBlogBundle(),
            );
            // ..

            return $bundles;
        }

        // ..
    }

Criando rotas
............. 

A rota do pacote foi inserido arquivo principal de rotas das aplicações, localizado em ``app/config/routing.yml``.

.. code-block:: yaml

    # app/config/routing.yml
    BloggerBlogBundle:
        resource: "@BloggerBlogBundle/Resources/config/routing.yml"
        prefix:   /

A possibilidade de utilizar prefixos nos permite montar toda a rota de ``BloggerBlogBundle``. No nosso caso, optamos por 
montar a rota utilizando o padrão, que é ``/``. Se, por exemplo, você quiser que todos os caminhos sejam prefixados com 
``/blogger`` mude o prefixo para ``:/blogger``. 

Estrutura padrão 
................

O pacote foi criado no diretório ``src`` com uma estrutura padrão começando no nível mais alto com a pasta ``Blogger`` 
que mapeia diretamente para o namespace do pacote que criamos dentro de ``Blogger``. 

Dentro desta pasta temos a pasta``BlogBundle`` que contém o pacote atual. Os conteúdos desta pasta serão analisados com 
o aprofundamento do tutorial. 

Se você já é familiarizado com a estrutura MVC, algumas das pastas serão auto-explicativas. 

O Controlador padrão 
~~~~~~~~~~~~~~~~~~~~ 

Como padrão do gerador de pacote, Symfony 2 criou um controlador padrão. Nós podemos executar este controlador, 
acessando ``Http://symblog.dev/app_dev.php/hello/symblog``. Você deverá ver uma página de saudação simples. 

Tente alterar o ``symblog`` da parte final da URL pelo seu nome. Vamos examinar com um nível elevado, como esta página 
foi gerada. 
 
Rota 
.... 

O arquivo de roteamento ``BloggerBlogBundle`` localizado em ``src/Blogger/BlogBundle/Resources/config/routing.yml`` 
contém a seguinte regra de roteamento.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_homepage:
        pattern:  /hello/{name}
        defaults: { _controller: BloggerBlogBundle:Default:index }

O roteamento é composto de um padrão e outras opções padrão. 

O padrão é verificado em relação a URL, e as opções padrão dizem para o controlador executar se as rotas coincidirem. 

No padrão ``/Olá/{nome}``, o ``{nome}`` é um local específico que irá corresponder a qualquer valor uma vez que os 
requisitos específicos não foram definidos. 

A rota também não especifica os métodos de língua ou HTTP. Como não temos métodos HTTP definidos, as solicitações de GET, 
POST, PUT, etc serão todos elegíveis para casamento de padrões. 

Se a rota satisfaz todos os critérios especificados, as opções padrão do _controller será invocado. As opções 
_controller especificam o Nome lógico do controlador que permite o Symfony 2 mapear para um arquivo específico. 

O exemplo acima fará com que a ação ``index`` do controlador padrão localizado em 
``src/Blogger/BlogBundle/Controller/DefaultController.php`` seja executada. 

O Controlador 
............. 

O controlador neste exemplo é muito simples.  A classe ``DefaultController``estende a classe ``Controller`` que fornece 
alguns métodos úteis, como a renderização, método utilizado a seguir. 

Como a nossa rota define um local específico que é passado para a ação com o argumento ``$nome``, a ação faz nada mais 
do que chamar o método de renderização especificando o template ``index.html.twig`` na pasta padãro de visão ``View`` 
dentro de ``BloggerBlogBundle``. 

O formato do nome do template é ``bundle:controller:template``. Em nosso exemplo, 
``BloggerBlogBundle:Default:index.html.twig`` que mapeia para o tamplate ``index.html.twig``, na pasta de visão padrão 
de ``BloggerBlogBundle``, ou fisicamente para o arquivo 
``src/Blogger/BlogBundle/resources/views/default/index.html.twig``. 

Diferentes formatos de templates podem ser usados para renderizar os templates em diferentes locais dentro das 
aplicações dos seus pacotes. Veremos isso mais tarde neste capítulo. 

Nós também podemos passar a variavel ``$name`` para o template por meio de ``array``.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/DefaultController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class DefaultController extends Controller
    {
        public function indexAction($name)
        {
            return $this->render('BloggerBlogBundle:Default:index.html.twig', array('name' => $name));
        }
    }

O template (A View) 
................... 

Como você pode ver, o template é muito simples. Ela imprime Olá seguido pelo argumento ``name`` passado pelo controlador.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Default/index.html.twig #}
    Hello {{ name }}!

Limpando 
~~~~~~~~ 

Como alguns arquivos padrão, criados pelo gerador, não são necessários podemos excluí-los. 

O arquivo ``src/Blogger/BlogBundle/Controller/DefaultController.php`` pode ser excluído, juntamente com a pasta View e o 
seu conteúdo localizdo em `` Src/Blogger/BlogBundle/resources/views/Default/``. Finalmente, remova a rota definida em 
``src/Blogger/BlogBundle/Resources/config/routing.yml``. 

Gerando os layouts 
------------------ 

Com Symfony 2 podemos criar os layouts usando 2 padrões; 
`Twig <http://www.twig-project.org/>`_ e PHP. 

Você poderia optar por não utilizar as opções citadas acima e escolher usar outra biblioteca. Isso é possível graças ao 
`Conteúdo de Injeção de Dependencia <http://symfony.com/doc/current/book/service_container.html>`_. 

Iremos utilizar Twig para gerar nossos layouts por alguns motivos: 

    1. Twig é rápido – Templates feitos com Twig tem um baixo custo para compilar as classes PHP o que gera pouca 
       sobrecarga. 
    2. Twig é conciso - Twig nos permite executar a funcionalidade de templates com pouco código. Compare isso com o PHP, 
       onde algumas declarações tornam-se muito detalhadas. 
    3. Twig suporta herança de templates – Templates têm a capacidade de ampliar e substituir outros templates 
       permitindo templates filhos alterar os padrões estabelecidos pelos templates de seus pais. 
    4. Twig é seguro - Twig tem saída ativa por padrão e ainda fornece um pacote de ambientes para templates importados. 
    5. Twig é extensível - Twig vem com um monte de funcionalidades comuns que você esperava de um gerador de templates, 
       mas para aquelas ocasiões em que você precisa de mais algumas funcionalidades extras, o Twig pode ser facilmente 
       estendido. 

Estes são apenas alguns dos benefícios do Twig. Para mais motivos pelos quais você deve usar Twig, veja o site oficial 
do `Twig <http://www.twig-project.org/>`_. 

Estrutura de layout 
~~~~~~~~~~~~~~~~~~~ 

Como Twig suporta herança de templates, vamos usar a abordagem de  
`Herança de Três níveis <http://symfony.com/doc/current/book/templating.html#three-level-inheritance>`_. Essa abordagem 
nos permite modificar a visão em 3 níveis distintos dentro da aplicação, nos dando muito espaço para personalizações. 

Template Principal - Nível 1 
............................ 

Vamos começar criando o nosso template de blocos básico para symblog. Precisamos de 2 arquivos aqui, o layout e o CSS. 
Como Symfony 2 suporta `HTML5 <http://diveintohtml5.org/>`_, também vamos usá-lo. 

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    <!DOCTYPE html>
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html"; charset=utf-8" />
            <title>{% block title %}symblog{% endblock %} - symblog</title>
            <!--[if lt IE 9]>
                <script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
            <![endif]-->
            {% block stylesheets %}
                <link href='http://fonts.googleapis.com/css?family=Irish+Grover' rel='stylesheet' type='text/css'>
                <link href='http://fonts.googleapis.com/css?family=La+Belle+Aurore' rel='stylesheet' type='text/css'>
                <link href="{{ asset('css/screen.css') }}" type="text/css" rel="stylesheet" />
            {% endblock %}
            <link rel="shortcut icon" href="{{ asset('favicon.ico') }}" />
        </head>
        <body>

            <section id="wrapper">
                <header id="header">
                    <div class="top">
                        {% block navigation %}
                            <nav>
                                <ul class="navigation">
                                    <li><a href="#">Home</a></li>
                                    <li><a href="#">About</a></li>
                                    <li><a href="#">Contact</a></li>
                                </ul>
                            </nav>
                        {% endblock %}
                    </div>

                    <hgroup>
                        <h2>{% block blog_title %}<a href="#">symblog</a>{% endblock %}</h2>
                        <h3>{% block blog_tagline %}<a href="#">creating a blog in Symfony2</a>{% endblock %}</h3>
                    </hgroup>
                </header>

                <section class="main-col">
                    {% block body %}{% endblock %}
                </section>
                <aside class="sidebar">
                    {% block sidebar %}{% endblock %}
                </aside>

                <div id="footer">
                    {% block footer %}
                        Symfony2 blog tutorial - created by <a href="https://github.com/dsyph3r">dsyph3r</a>
                    {% endblock %}
                </div>
            </section>

            {% block javascripts %}{% endblock %}
        </body>
    </html>

.. note::
    :title: Observações

    Existem 3 arquivos externos referenciados para o modelo, 1 arquivo JavaScript  e 2 arquivos CSS. O arquivo 
    JavaScript corrige a falta de suporte ao HTML5 das versões do IE anteriores ao IE9. Os 2 arquivos CSS importam 
    fontes do `Google Web Font <http://www.google.com/webfonts>`_. 

Este layout representa a estrutura principal do nosso site. A maior parte do layout consiste em HTML, com umas diretivas 
Twig estranhas. Vamos examinar estas diretivas agora. 

Vamos começar com o cabeçalho do documento. Vamos começar pelo título: 

.. code-block:: html

    <title>{% block title %}symblog{% endblock %} - symblog</title>

A primeira coisa que você notará é a tag estranha ``{%``. Não é HTML, e definitivamente não é PHP. Esta tag é um das três 
tags do Twig. Esta tag é o Twig ``Faça algo``. Ela é usada para executar comandos, como instruções de controle e para a 
definição de elementos de bloco. 

A lista completa de 
`Estruturas de controle <http://www.twig-project.org/doc/templates.html#list-of-control-structures>`_ pode ser 
encontrada na Documentação do Twig. 

O bloco Twig que definimos no título faz 2 coisas; 
Ele define o identificador do bloco de título, e fornece uma saída padrão entre as diretivas ``block`` e ``endblock``. 
Através da definição de um bloco, podemos tirar proveito do modelo de herança do Twig. Por exemplo, em uma página para 
exibir um post do blog que gostariamos que o título da página refletisse o título do blog. 

Podemos conseguir isso estendendo o layout e substituir o bloco de título. 

.. code-block:: html

    {% extends '::base.html.twig' %}

    {% block title %}The blog title goes here{% endblock %}

No exemplo acima, estendemos o layout base das aplicações que primeiro definiu o bloco de título. Você notará que o 
formato de layout usado com a diretiva ``extends`` está faltando as partes do pacote (Bundle) e do Controlador, 
lembrando que o formato de layout é ``bundle:controller:template``. 

Excluindo partes do pacote e do controlador, estamos especificando o uso de níveis de templates por aplicativo definido 
em ``app/Recursos/views/``. 

Em seguida, temos definido um outro bloco de título e colocamos um conteúdo, neste caso, o título do blog. Como o modelo 
pai já contém um bloco de título, ele é substituído por esse bloco novo. O título seria, agora, algo como 
'O título do blog vai aqui - symblog'. 

Esta funcionalidade fornecida pelo Twig será bastante usada na criação de layouts. 

No bloco de folhas de estilo, foi introduzidos a próxima tag do Twig, a tag ``{{``,  ou a tag ``Diga algo``. 

.. code-block:: html

    <link href="{{ asset('css/screen.css') }}" type="text/css" rel="stylesheet" />

Esta tag é usada para imprimir o valor da variável ou expressão. No exemplo acima ela mostra o valor de retorno da 
função ``_asset``, que nos fornece uma forma portátil de vincular a aplicação dos ativos, tais como CSS, JavaScript e 
imagens. 

A tag ``{{`` pode também ser combinado com filtros para manipular os retornos antes da impressão.

.. code-block:: html

    {{ blog.created|date("d-m-Y") }}

Para uma lista completa de filtros, verifique a 
`Documentação do  Twig <http://www.twig-project.org/doc/templates.html#list-of-built-in-filters>`_. 

A ultima tag Twig, que não vimos nos layouts é a tag de comentário ``{#``. Veja o exemplo de sua utilização:

.. code-block:: html

    {# The quick brown fox jumps over the lazy dog #}

Não há outros conceitos introduzidos neste template. Ele fornece o principal Layout pronto para que possamos 
personalizá-lo de acordo com nossa necessidade. 

Agora, vamos adicionar alguns estilos. Crie uma folha de estilo em ``web/css/screen.css`` e adicione o seguinte conteúdo. 
Isto irá adicionar estilos para o layout principal.

.. code-block:: css

    html,body,div,span,applet,object,iframe,h1,h2,h3,h4,h5,h6,p,blockquote,pre,a,abbr,acronym,address,big,cite,code,del,dfn,em,img,ins,kbd,q,s,samp,small,strike,strong,sub,sup,tt,var,b,u,i,center,dl,dt,dd,ol,ul,li,fieldset,form,label,legend,table,caption,tbody,tfoot,thead,tr,th,td,article,aside,canvas,details,embed,figure,figcaption,footer,header,hgroup,menu,nav,output,ruby,section,summary,time,mark,audio,video{border:0;font-size:100%;font:inherit;vertical-align:baseline;margin:0;padding:0}article,aside,details,figcaption,figure,footer,header,hgroup,menu,nav,section{display:block}body{line-height:1}ol,ul{list-style:none}blockquote,q{quotes:none}blockquote:before,blockquote:after,q:before,q:after{content:none}table{border-collapse:collapse;border-spacing:0}

    body { line-height: 1;font-family: Arial, Helvetica, sans-serif;font-size: 12px; width: 100%; height: 100%; color: #000; font-size: 14px; }
    .clear { clear: both; }

    #wrapper { margin: 10px auto; width: 1000px; }
    #wrapper a { text-decoration: none; color: #F48A00; }
    #wrapper span.highlight { color: #F48A00; }

    #header { border-bottom: 1px solid #ccc; margin-bottom: 20px; }
    #header .top { border-bottom: 1px solid #ccc; margin-bottom: 10px; }
    #header ul.navigation { list-style: none; text-align: right; }
    #header .navigation li { display: inline }
    #header .navigation li a { display: inline-block; padding: 10px 15px; border-left: 1px solid #ccc; }
    #header h2 { font-family: 'Irish Grover', cursive; font-size: 92px; text-align: center; line-height: 110px; }
    #header h2 a { color: #000; }
    #header h3 { text-align: center; font-family: 'La Belle Aurore', cursive; font-size: 24px; margin-bottom: 20px; font-weight: normal; }

    .main-col { width: 700px; display: inline-block; float: left; border-right: 1px solid #ccc; padding: 20px; margin-bottom: 20px; }
    .sidebar { width: 239px; padding: 10px; display: inline-block; }

    .main-col a { color: #F48A00; }
    .main-col h1,
    .main-col h2
        { line-height: 1.2em; font-size: 32px; margin-bottom: 10px; font-weight: normal; color: #F48A00; }
    .main-col p { line-height: 1.5em; margin-bottom: 20px; }

    #footer { border-top: 1px solid #ccc; clear: both; text-align: center; padding: 10px; color: #aaa; }

Pacote Template - Nível 2 
......................... 

Vamos agora avançar para a criação do layout para o pacote (Bundle) Blog. Crie um arquivo em 
``src/Blogger/BlogBundle/Recursos/views/layout.html.twig`` e adicione o seguinte conteúdo.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    {% extends '::base.html.twig' %}

    {% block sidebar %}
        Sidebar content
    {% endblock %}

À primeira vista, este modelo pode parecer um pouco simples, mas sua simplicidade é a chave. 

Em primeiro lugar, amplia o template base das aplicações, que criamos anteriormente. Em segundo lugar, substitui o bloco 
pai lateral com algum conteúdo fictício. À medida que o bloco lateral vai aparecendo em todas as páginas de nosso blog, 
faz sentido executar sua personalização. 

Você pode perguntar por que não colocamos a personalização no templates de aplicação uma vez que irá estar presente em 
todas as páginas. Simples, a aplicação não sabe nada sobre o pacote e não deveria. O pacote deve conter toda a sua 
funcionalidade e tornar o bloco lateral parte de suas funcionalidades. 

OK, então por que não colocar a barra lateral em cada da página de template? Novamente, isto é simples, teríamos que 
duplicar a barra lateral a cada vez que nós adicionamos uma página. Além disso, este modelo de nível 2 nos dará 
flexibilidade no futuro, para adicionarmos personalizações que todos os outros templates filhos herdarão.  

Por exemplo, poderíamos querer mudar a cópia de rodapé de todas as páginas, este seria um ótimo lugar para fazer isso. 

Template da Página - Nível 3 
............................ 

Finalmente estamos prontos para o layout do controlador. Estes layouts vão ser comumente relacionados com uma ação do 
controlador, isto é, a ação do blog de exibição terá um tempĺate ``show`` do blog. 

Vamos começar criando o controlador para a página inicial e seu template. Como esta é a primeira página que estamos 
criando, precisamos criar o controlador. 

Crie o controlador em ``src/Blogger/BlogBundle/Controller/PageController.php`` e adicione o seguinte conteúdo: 

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/PageController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class PageController extends Controller
    {
        public function indexAction()
        {
            return $this->render('BloggerBlogBundle:Page:index.html.twig');
        }
    }

Agora vamos criar o modelo para esta ação. 

Como você pode ver na ação do controlador, nós estamos indo para renderizar o template de Page, o Index. 

Crie o template em ``src/Blogger/BlogBundle/Recursos/views/Page/index.html.twig``

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block body %}
        Blog homepage
    {% endblock %}

Este formato mostra o template final que podemos especificar. 

Neste exemplo o template ``BloggerBlogBundle::layout.html.twig`` é estendido onde parte do nome do template é omitida 
pelo Controlador. 

Excluindo partes do Controlador, estamos especificando a utilização de nível de template do pacote (bundle) criado em 
``src/Blogger/BlogBundle/Recursos/views/layout.html.twig``. 

Agora vamos adicionar uma rota para a nossa homepage. 

Atualize o arquivo de configuração de rotas localizado em ``src/Blogger/BlogBundle/Recursos/config/routing.yml``.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_homepage:
        pattern:  /
        defaults: { _controller: BloggerBlogBundle:Page:index }
        requirements:
            _method:  GET

Por último precisamos remover a rota padrão para a tela de boas-vindas do Symfony 2. Retire a rota ``_welcome`` no topo 
do arquivo de rota ``dev`` localizado em ``app/config/routing_dev.yml``. 

Agora estamos prontos para ver o nosso template do blog. Acesse ``http://symblog.dev/app_dev.php/``. 

.. image:: /_static/images/part_1/homepage.jpg
    :align: center
    :alt: symblog main template layout

Você deverá ver o layout básico do blog, com o conteúdo principal e lateral refletindo os blocos que substituímos nos 
respectivos templates. 

A página Sobre 
-------------- 

A tarefa final nesta parte do tutorial será a criação de uma página estática de nome Sobre. Isso vai demonstrar como 
vincular páginas em conjunto, e reforçam ainda mais a abordagem de herança de Três Níveis que adotamos. 

A Rota 
~~~~~~ 

Ao criar uma nova página, uma das primeiras tarefas que devemos fazer é criar a rota para ela. 

Abra o arquivo de rotas de ``BloggerBlogBundle`` localizado em ``src/Blogger/BlogBundle/Resources/config/routing.yml`` e 
acrescente a seguinte regra de rota. 

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_about:
        pattern:  /about
        defaults: { _controller: BloggerBlogBundle:Page:about }
        requirements:
            _method:  GET

O Controlador 
~~~~~~~~~~~~~ 

Em seguida, abra o controlador de ``Page`` localizado em ``src/Blogger/BlogBundle/controller/PageController.php`` e 
adicione a ação para lidar com a página Sobre. 

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    class PageController extends Controller
    {
        //  ..

        public function aboutAction()
        {
            return $this->render('BloggerBlogBundle:Page:about.html.twig');
        }
    }

A Visão 
~~~~~~~ 

Para a visão, crie um novo arquivo localizado em ``src/Blogger/BlogBundle/Recursos/views/Page/about.html.twig`` e copie 
o seguinte conteúdo. 

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/about.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}About{% endblock%}

    {% block body %}
        <header>
            <h1>About symblog</h1>
        </header>
        <article>
            <p>Donec imperdiet ante sed diam consequat et dictum erat faucibus. Aliquam sit
            amet vehicula leo. Morbi urna dui, tempor ac posuere et, rutrum at dui.
            Curabitur neque quam, ultricies ut imperdiet id, ornare varius arcu. Ut congue
            urna sit amet tellus malesuada nec elementum risus molestie. Donec gravida
            tellus sed tortor adipiscing fringilla. Donec nulla mauris, mollis egestas
            condimentum laoreet, lacinia vel lorem. Morbi vitae justo sit amet felis
            vehicula commodo a placerat lacus. Mauris at est elit, nec vehicula urna. Duis a
            lacus nisl. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices
            posuere cubilia Curae.</p>
        </article>
    {% endblock %}

A página Sobre não é nada espetacular. Sua ação é apenas para processar um arquivo de template com algum conteúdo 
fictício. Isto, contudo, leva-nos para a próxima tarefa. 

Ligando as páginas 
~~~~~~~~~~~~~~~~~~ 

Agora temos  a página Sobre pronta para ser acessada. Dê uma olhada em ``http://symblog.dev/app_dev.php/about``. 

Do jeito que está não há como um usuário do seu blog ir para a página sobre, somente se digitar a URL completa, tal como 
fizemos. 

Como seria de esperar, Symfony 2 fornece 2 lados da equação de roteamento. Pode corresponder a rotas, como vimos, e 
também pode gerar URLs a partir destas rotas. 

Você deve sempre usar as regras de roteamento do Symfony 2. Nunca em sua aplicação você deve usar o seguinte: 

.. code-block:: html+php

    <a href="/contact">Contact</a>

    <?php $this->redirect("/contact"); ?>

Você pode estar se perguntando o que há de errado com esta abordagem, pode ser o jeito que você sempre vinculou suas 
páginas em conjunto. No entanto, há um certo número de problemas com esta abordagem. 

    1. Ele usa um link absoluto e ignora o sistema de roteamento Symfony 2 inteiramente. Se você quiser mudar a 
       localização da página Sobre em qualquer ponto você teria que encontrar todas as referências para o link e 
       alterá-las. 
    2. Ele vai ignorar seus controladores de ambiente. Ambiente é algo que realmente não explicamos ainda mas você tem 
       de usá-los. O controlador de frente ``app_dev.php`` nos dá acesso a nossa aplicação no ambiente ``dev``. Se você 
       quisesse substituir o ``app_dev.php`` por ``app.php``, você vai estar executando a aplicação no ambiente ``prod``. 
       A importância desses ambientes será explicada neste tutorial, mas por agora, é importante notar que o link 
       absoluto definido acima não mantem o ambiente atual que estamos e que o controlador de frente não é prefixado na 
       URL. 

A maneira correta de vincular páginas em conjunto é com os métodos fornecidos pelo Twig ``path`` e ``url``. Os 2 são 
muito semelhante, exceto o método ``url`` que irá nos fornecer URLs absolutas. 

Vamos atualizar o template principal de aplicações, localizado em ``app/resources/views/base.html.twig`` para linkar 
para a página Sobre e página Inicial em conjunto. 

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    {% block navigation %}
        <nav>
            <ul class="navigation">
                <li><a href="{{ path('BloggerBlogBundle_homepage') }}">Home</a></li>
                <li><a href="{{ path('BloggerBlogBundle_about') }}">About</a></li>
                <li><a href="#">Contact</a></li>
            </ul>
        </nav>
    {% endblock %}

Agora atualize seu navegador para ver os links das páginas Inicial e Sobre funcionando conforme o esperado. Se você ver 
o código fonte das páginas, você vai perceber que o link foi prefixado com ``/app_dev.php/``. Este é o controlador de 
frente que explicamos acima, e como você pode ver o uso do ``path`` está mantido. 

Finalmente vamos atualizar os links do logotipo para redirecioná-lo de volta para a página inicial. Atualize o template 
localizado em ``app/resources/views/base.html.twig``.

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    <hgroup>
        <h2>{% block blog_title %}<a href="{{ path('BloggerBlogBundle_homepage') }}">symblog</a>{% endblock %}</h2>
        <h3>{% block blog_tagline %}<a href="{{ path('BloggerBlogBundle_homepage') }}">creating a blog in Symfony2</a>{% endblock %}</h3>
    </hgroup>
    
Conclusão 
--------- 

Nós cobrimos as áreas básicas no que diz respeito a uma aplicação Symfony 2 inclusive recebendo a aplicação configurada 
e funcional. Começamos a explorar os conceitos fundamentais atrás de uma aplicação Symfony 2, incluindo roteamento e da 
maquina geradora de tamplete, Twig. 

No próximo capítulo, vamos criar a página de Contato. Esta página é um pouco mais envolvente do que a página Sobre uma 
vez que permite aos usuários interagir com um formulário web para enviar-nos dúvidas. 

O próximo capítulo irá introduzir alguns conceitos como Validadores e Formulários.