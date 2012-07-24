[Parte 2] - Página Contato: Validadores, Formulários e E-mail
=============================================================

Visão geral
-----------

Agora temos os templates básicos de HTML em seus devidos lugares, é hora de fazer uma página mais funcional. 

Vamos começar com uma página simples, a página de contato. No final deste capítulo, você terá uma página de contatos 
que permite que os usuários enviem informações de contato para um e-mail de contato. 

As seguintes áreas serão abordadas neste capítulo:

    1. Validadores
    2. Forms
    3. Definir valores de configuração do pacote (Bundle)

Página de Contato
-----------------

Rota
~~~~

Tal como foi feito com a página Sobre que nós criamos no último capítulo, vamos começar por definir a rota da página de 
contato. 

Abra o arquivo de rotas do``BloggerBlogBundle`` localizado em ``src/Blogger/BlogBundle/Resources/config/routing.yml`` e 
acrescente a seguinte regra de rota.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_contact:
        pattern:  /contact
        defaults: { _controller: BloggerBlogBundle:Page:contact }
        requirements:
            _method:  GET

Não há nada novo aqui, a regra corresponde ao padrão ``/contact``, para o método HTTP ``GET`` que executa a ação 
``contact`` do controlador ``Page`` em ``BloggerBlogBundle``.

Controlador
~~~~~~~~~~~

Agora vamos adicionar a ação para a página de contato no controlador ``Page`` em ``BloggerBlogBundle`` localizado em 
``src/Blogger/BlogBundle/Controller/PageController.php``.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    // ..
    public function contactAction()
    {
        return $this->render('BloggerBlogBundle:Page:contact.html.twig');
    }
    // ..

Até o momento, a ação é muito simples, apenas renderiza a visualização da página de contato. Voltaremos para o 
controlador depois.

Visão
~~~~~

Crie a página de visualização da página de contato em ``src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig`` 
e adicione o seguinte conteúdo:

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}Contact{% endblock%}

    {% block body %}
        <header>
            <h1>Contact symblog</h1>
        </header>

        <p>Want to contact symblog?</p>
    {% endblock %}

Este template também é bastante simples. Ele extende o  layout do tamplate de ``BloggerBlogBundle``, substitui o bloco 
de título para definir um título personalizado e define algum conteúdo para o ``body block`` (o corpo da página).

Linkando para a página
~~~~~~~~~~~~~~~~~~~~~~

Por último, precisamos atualizar o link no template da aplicação localizada em ``app/Resources/views/base.html.twig`` 
para vincular a página de contato.

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    {% block navigation %}
        <nav>
            <ul class="navigation">
                <li><a href="{{ path('BloggerBlogBundle_homepage') }}">Home</a></li>
                <li><a href="{{ path('BloggerBlogBundle_about') }}">About</a></li>
                <li><a href="{{ path('BloggerBlogBundle_contact') }}">Contact</a></li>
            </ul>
        </nav>
    {% endblock %}

Se você acessar em seu navegador ``http://symblog.dev/app_dev.php/`` e clicar no link de contato na barra de navegação, 
você deve ver uma página de contato com um conteúdo básico. 

Agora que temos a página corretamente configurada, é hora de começar a trabalhar no formulário de contato. Este tópico 
está dividido em 2 partes distintas: Os validadores e Os Formulários. 

Antes de podermos abordar o conceito de Validadores e Formulários, precisamos pensar sobre como vamos lidar com os dados 
do formulário de contato.

A Entidade Contato
------------------

Vamos começar criando uma classe que representa um formulário de contato de um usuário. Nós vamos usar algumas 
informações básicas, tais como ``name``, ``subject`` e ``body`` da mensagem. Crie um novo arquivo localizado em 
``src/Blogger/BlogBundle/Entity/Enquiry.php`` e cole o seguinte conteúdo:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Enquiry.php

    namespace Blogger\BlogBundle\Entity;

    class Enquiry
    {
        protected $name;

        protected $email;

        protected $subject;

        protected $body;

        public function getName()
        {
            return $this->name;
        }

        public function setName($name)
        {
            $this->name = $name;
        }

        public function getEmail()
        {
            return $this->email;
        }

        public function setEmail($email)
        {
            $this->email = $email;
        }

        public function getSubject()
        {
            return $this->subject;
        }

        public function setSubject($subject)
        {
            $this->subject = $subject;
        }

        public function getBody()
        {
            return $this->body;
        }

        public function setBody($body)
        {
            $this->body = $body;
        }
    }

Como você pôde ver, esta classe apenas define alguns métodos protegidos e os métodos de acesso a estes métodos 
protegidos. 

Não há nada aqui que define como vamos validar os métodos, ou como os métodos se relacionam com os elementos do 
formulários. Iremos voltar a este ponto mais tarde.


.. note::

    Vamos falar rapidamente sobre o uso de ``namespaces`` em Symfony 2. A classe de entidade que criamos define o 
    namespace para ``Blogger\BlogBundle\Entity``. 

    Como o autoloading do Symfony 2 suporta o 
    `Padrão PSR-0 <http://groups.google.com/group/php-standards/web/psr-0-final-proposal?pli=1>`_,  o namespace irá 
    mapear diretamente para a estrutura de pastas do Bundle. 

    A classe da entidade do formulário está localizado em ``src/Blogger/BlogBundle/Entity/Enquiry.php`` assegurando o 
    Symfony 2 de fazer corretamente o autoload da classe.

    Como o autoloader do Symfony 2 sabe que o namespace do ``Blogger`` pode ser encontrado no diretério ``src``?? Isto é 
    possível graças as configurações no autoloader em ``app/autoloader.php``.

    .. code-block:: php

        // app/autoloader.php
        $loader->registerNamespaceFallbacks(array(
            __DIR__.'/../src',
        ));

    Esta declaração é uma alternativa para quaisquer namespaces que ainda não foram registrados.
    
    Como o namespace do ``Blogger`` não está registrado, o autoloader do Symfony 2 vai procurar os arquivos necessários 
    no diretório ``src``.

    Autoloading e namespaces são conceitos muito poderosos em Symfony 2. Se está acontecendo erros onde o PHP é incapaz 
    de encontrar classes, é bem provável que você tenha um erro em seu namespace ou na estrutura de pastas. Verifique 
    também se o namespace foi registrado com o autoloader, como mostrado acima. 

    Não tente corrigir isso usando as diretivas ``require`` ou ``include``.

Forms
-----

Agora, vamos criar o formulário. 

Symfony 2 vem com um Framework de formulário muito poderoso que torna a tarefa de lidar com o formulário mais fácil. 
Tal como acontece com todos os componentes do Symfony 2, pode-se usar fora do Symfony 2 em seus próprios projetos.
O `Componente Formulário <https://github.com/symfony/Form>`_ está disponível no Github. 

Vamos começar criando uma classe ``AbstractType`` que representa o formulário. Poderíamos ter criado o formulário 
diretamente no controlador sem se preocupar com essa classe, no entanto, separar o formulário em suas próprias classes 
permite-nos reutilizar o formulário em toda a aplicação. 

Ele também impede-nos de ocupar ainda mais o controlador. Afinal, o controlador é supostamente simples. O objetivo dele 
é proporcionar a ligação entre o modelo e a visão.

EnquiryType
~~~~~~~~~~~

Crie um novo arquivo localizado em ``src/Blogger/BlogBundle/Form/EnquiryType.php`` e cole o seguinte conteúdo.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Form/EnquiryType.php

    namespace Blogger\BlogBundle\Form;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;

    class EnquiryType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('name');
            $builder->add('email', 'email');
            $builder->add('subject');
            $builder->add('body', 'textarea');
        }

        public function getName()
        {
            return 'contact';
        }
    }

A classe ``EnquiryType`` acrescenta a classe ``FormBuilder``. A classe ``FormBuilder`` é a sua melhor amiga, quando se 
trata de criar formulários. É capaz de simplificar o processo de definição de campos com base nos metadados que o campo 
tem. 

Como a nossa Entidade ``Enquiry`` ainda é muito simples, pois nós não definimos nenhum metadado ainda, o ``FormBuilder``, 
por padrão, vai adicionar o tipo básico de campo para entrada de texto. Isto é adequado para a maioria dos campos
exceto para o corpo pois queremos um ``textarea``, e e-mail onde queremos tirar vantagem do tipo de campo e-mail do 
HTML5.

.. note::

    Um ponto chave para mencionar aqui é que o método ``getName`` deve retornar um identificador único.

Criando o formulário no controlador
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Agora que temos definido a Entidade do formulário e ``EnquiryType``, podemos atualizar a ação de contato para
utilizá-los. 

Substitua o conteúdo da ação de contato localizado em ``src/Blogger/BlogBundle/controller/PageController.php`` pelo 
seguinte conteúdo:

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    public function contactAction()
    {
        $enquiry = new Enquiry();
        $form = $this->createForm(new EnquiryType(), $enquiry);

        $request = $this->getRequest();
        if ($request->getMethod() == 'POST') {
            $form->bindRequest($request);

            if ($form->isValid()) {
                // Perform some action, such as sending an email

                // Redirect - This is important to prevent users re-posting
                // the form if they refresh the page
                return $this->redirect($this->generateUrl('BloggerBlogBundle_contact'));
            }
        }

        return $this->render('BloggerBlogBundle:Page:contact.html.twig', array(
            'form' => $form->createView()
        ));
    }

Começamos criando uma instância da entidade ``Enquiry``. Esta entidade representa os dados de um formulário de contato. 
Em seguida, criamos o formulário real. Nós especificamos o ``EnquiryType`` que criamos anteriormente e passamos para o 
nosso objeto de entidade do formulário. 

O método ``CreateForm`` é capaz de usar estes 2 método para criar um formulário.

Com esta ação, o controlador irá lidar com a exibição e irá processar o formulário, assim, nós precisamos verificar o 
método HTTP. Formulários são normalmente enviados enviados via ``POST``, e nosso formulário não será exceção. 

Se o método de solicitação é ``POST``, uma chamada para ``BindRequest`` irá transformar os dados enviados de volta para 
o objeto ``$enquiry``. Neste momento, o objeto ``$enquiry`` representa o que o usuário enviou.

Agora, faremos uma verificação para ver se o formulário é válido. Como não especificamos nenhum validador até agora, o 
formulário será sempre válido.

Finalmente, especificamos o template a ser renderizado. 

Observe que agora estamos também passando uma representação do formulário para o modelo. Este objeto permite-nos 
processar o formulário na View.

Como usamos 2 novas classes em nosso controller, precisamos importar os namespaces. Atualize o arquivo controlador 
localizado em ``src/Blogger/BlogBundle/Controller/PageController.php`` com o seguinte conteúdo. 

As declarações devem ser colocados sob a forma ``use statement``.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/PageController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    // Import new namespaces
    use Blogger\BlogBundle\Entity\Enquiry;
    use Blogger\BlogBundle\Form\EnquiryType;

    class PageController extends Controller
    // ..

Renderizando o formulário
~~~~~~~~~~~~~~~~~~~~~~~~~ 

Graças aos métodos do Twig, renderização de formulários torna-se simples. O Twig fornece um sistema de camadas de 
renderização de formulários que lhe permite processar o formulário como uma entrada da entidade, ou como erros 
individuais e elementos, dependendo do nível de personalização que você forneceu.

Para demonstrar o poder dos métodos do Twig, podemos usar o seguinte trecho de código para processar todo o formulário.

.. code-block:: html

    <form action="{{ path('BloggerBlogBundle_contact') }}" method="post" {{ form_enctype(form) }}>
        {{ form_widget(form) }}

        <input type="submit" />
    </form>

Embora isso seja muito útil para prototipagem de formulários simples, há limitações quando precisamos de personalizações 
grandes, o que acontece com frequência com os formulários.

Para o nosso formulário de contato, vamos optar pelo meio termo. 

Substitua o código do template localizado em ``src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig`` pelo 
seguinte código:

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}Contact{% endblock%}

    {% block body %}
        <header>
            <h1>Contact symblog</h1>
        </header>

        <p>Want to contact symblog?</p>

        <form action="{{ path('BloggerBlogBundle_contact') }}" method="post" {{ form_enctype(form) }} class="blogger">
            {{ form_errors(form) }}

            {{ form_row(form.name) }}
            {{ form_row(form.email) }}
            {{ form_row(form.subject) }}
            {{ form_row(form.body) }}

            {{ form_rest(form) }}

            <input type="submit" value="Submit" />
        </form>
    {% endblock %}

Como você pôde ver, usamos 4 novos métodos do Twig para processar o formulário.

    O primeiro método ``form_enctype`` define o tipo de conteúdo do formulário. Isso deve ser definido quando o seu 
    formulário lida com upload de arquivos. O nosso formulário não tem nenhum uso para este método, mas a sua prática é 
    aconselhada caso você opte por adicionar o upload de arquivos no futuro. Depurar um formulário que manipula arquivos 
    de uploads que não tem nenhum tipo de conteúdo definido pode ser uma verdadeira dor de cabeça!

    O segundo método ``form_errors`` irá renderizar quaisquer erros cuja validação do formulário tenha falhado.

    O terceiro método ``form_row`` exibe as entradas dos elementos relacionados a cada campo de formulário. Isto inclui 
    todos os erros para o campo, o label e o elemento do campo atual .

    Finalmente, usamos o método ``form_rest``. É sempre importante usar o método no final do formulário para renderizar 
    quaisquer campos que você possa ter esquecido, incluindo campos hidden e o token CSRF do formulário Symfony 2.

.. note::

    Cross-site request forgery (CSRF) é explicado em detalhes no capítulo
    `Formulários <http://symfony.com/doc/current/book/forms.html#csrf-protection>`_  do livro do Symfony 2.


Estilizando o formulário
~~~~~~~~~~~~~~~~~~~~~~~~

Se você visualizar o formulário de contato agora, acessando ``http://symblog.dev/app_dev.php/contact``, você vai notar 
que não parece tão atraente. Vamos adicionar alguns estilos para melhorar esta exibição. 

Como os estilos são específicos para o formulário dentro de nosso pacote (Bundle) Blog, iremos criar os estilos em uma 
nova folha de estilos. 

Crie um novo arquivo localizado em ``src/Blogger/BlogBundle/Resources/public/css/blog.css`` e cole o seguinte conteúdo:

.. code-block:: css

    .blogger-notice { text-align: center; padding: 10px; background: #DFF2BF; border: 1px solid; color: #4F8A10; margin-bottom: 10px; }
    form.blogger { font-size: 16px; }
    form.blogger div { clear: left; margin-bottom: 10px; }
    form.blogger label { float: left; margin-right: 10px; text-align: right; width: 100px; font-weight: bold; vertical-align: top; padding-top: 10px; }
    form.blogger input[type="text"],
    form.blogger input[type="email"]
        { width: 500px; line-height: 26px; font-size: 20px; min-height: 26px; }
    form.blogger textarea { width: 500px; height: 150px; line-height: 26px; font-size: 20px; }
    form.blogger input[type="submit"] { margin-left: 110px; width: 508px; line-height: 26px; font-size: 20px; min-height: 26px; }
    form.blogger ul li { color: #ff0000; margin-bottom: 5px; }


Precisamos fazer com que o aplicativo saiba que nós queremos usar este estilo. Poderíamos importar a folha de estilo 
para o template de contato, mas como outros templates também podem vir a usar este estilo mais tarde, faz sentido 
importá-lo para o layout de ``BloggerBlogBundle`` que criamos anteriormente no capítulo 1. 

Abra o layout de ``BloggerBlogBundle`` localizado em ``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` e 
substitua seu conteúdo com o seguinte código:

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    {% extends '::base.html.twig' %}

    {% block stylesheets %}
        {{ parent() }}
        <link href="{{ asset('bundles/bloggerblog/css/blog.css') }}" type="text/css" rel="stylesheet" />
    {% endblock %}

    {% block sidebar %}
        Sidebar content
    {% endblock %}

Você pôde ver que temos definido um bloco de folhas de estilo para substituir o bloco de folhas de estilo definido no 
modelo pai. No entanto, é importante notar a chamada para o método ``Pai``. Isto irá importar o conteúdo do bloco de 
folhas de estilo do template ``Pai`` localizado em ``app/Resources/base.html.twig``, permitindo-nos anexar o nosso estilo 
novo. Afinal, não queremos substituir as folhas de estilo existentes.

Para que a função ``asset`` possa linkar corretamente o recurso, precisamos copiar ou vincular os recursos do pacote das 
aplicações para a pasta ``web``. Isto pode ser feito da seguinte forma:

.. code-block:: bash

    $ php app/console assets:install web --symlink

.. note::

    Se você estiver usando um sistema operacional que não suporta links simbólicos, tais como Windows você terá que 
    utilizar o seguinte artificio.

    .. code-block:: bash

        php app/console assets:install web

     Este método vai realmente copiar os recursos dos pacotes das pastas ``public`` na pasta ``web`` da aplicação. Como 
    os arquivos são copiados, você terá de executar esta tarefa cada vez que fizer uma alteração em um recurso público 
    do pacote.

Agora, se você atualizar a página de contato, o formulário estará estilizado conforme feito acima.

.. image:: /_static/images/part_2/contact.jpg
    :align: center
    :alt: symblog contact form

.. tip::

    Enquanto a função ``asset`` fornece a funcionalidade do recurso que desejamos utilizar, existe uma alternativa 
    melhor para isso. A biblioteca `Assetic <https://github.com/kriswallsmith/assetic>`_ de 
    `Kris Wallsmith <https://github.com/kriswallsmith>`_ é empacotado com a distribuição Standard do Symfony 2 por 
    padrão. 

    Esta biblioteca fornece a manutenção dos assets muito além das capacidades dos padrões do Symfony 2. Assetic nos 
    permite executar filtros ativos para combinar automaticamente, minify e gzip. Ele também pode executar filtros de 
    compressão de imagens. 

    Assetic ainda nos permite fazer referência a recursos diretamente dentro da pasta ``public`` do pacote sem ter que 
    executar a tarefa ``assets:install``. Vamos explorar o uso de Assetic mais adiante no tutorial.

Falha ao postar os dados
------------------------

Se você tentou enviar o formulário, vocẽ se deparou com um erro do Symfony 2.

.. image:: /_static/images/part_2/post_error.jpg
    :align: center
    :alt: No route found for "POST /contact": Method Not Allowed (Allow: GET, HEAD)

Esse erro está nos dizendo que não existe uma rota para coincidir com ``/contact`` para o método POST HTTP. A rota 
aceita somente pedidos GET e HEAD. Isto é porque nós configuramos nossa rota com a exigência de método de GET.

Vamos atualizar a rota da página de contato no arquivo localizado em 
``src/Blogger/BlogBundle/Resources/config/routing.yml`` para também permitir as requisições POST.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_contact:
        pattern:  /contact
        defaults: { _controller: BloggerBlogBundle:Page:contact }
        requirements:
            _method:  GET|POST

.. tip::

    Você talvez esteja se perguntando por que a rota permite o método HEAD onde apenas o método  GET foi especificado. 
    Isto é porque a HEAD é uma requisição GET mas apenas os cabeçalhos HTTP são retornados.

Agora, quando você enviar o formulário, deve funcionar como esperado embora, na verdade, não faça muito ainda. A página 
só vai redirecioná-lo de volta para o formulário de contato.

Validadores
-----------

Os validadores do Symfony 2 nos permitem realizar a tarefa de validação de dados. Validação é uma tarefa comum quando se 
lida com dados de formulários. 

A validação também precisa ser realizada com os dados antes que ele seja submetido a uma base de dados. O validador do 
Symfony 2 permite-nos separar a lógica de validação dos componentes que podem utilizar-la, tal como o componente do 
Formulário ou o componente de banco de dados. 

Esta abordagem significa que temos um conjunto de regras de validação para um objeto.

Vamos começar pela atualização da Entidade ``Enquiry`` localizada em ``src/Blogger/BlogBundle/Entity/Enquiry.php`` para 
especificar alguns validadores. Certifique-se de adicionar as 5 novas declarações ``use`` no topo do arquivo.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Entity/Enquiry.php

    namespace Blogger\BlogBundle\Entity;

    use Symfony\Component\Validator\Mapping\ClassMetadata;
    use Symfony\Component\Validator\Constraints\NotBlank;
    use Symfony\Component\Validator\Constraints\Email;
    use Symfony\Component\Validator\Constraints\MinLength;
    use Symfony\Component\Validator\Constraints\MaxLength;

    class Enquiry
    {
        // ..

        public static function loadValidatorMetadata(ClassMetadata $metadata)
        {
            $metadata->addPropertyConstraint('name', new NotBlank());

            $metadata->addPropertyConstraint('email', new Email());

            $metadata->addPropertyConstraint('subject', new NotBlank());
            $metadata->addPropertyConstraint('subject', new MaxLength(50));

            $metadata->addPropertyConstraint('body', new MinLength(50));
        }

        // ..

    }

Para definir os validadores, devemos implementar o método estático ``LoadValidatorMetadata``. Isso cria um objeto de 
``ClassMetadata``. Podemos usar esse objeto para definir restrições de propriedade sobre os membros da nossa entidade. 

A primeira declaração se aplica a restrição ``NotBlank`` de ``name``. O validador ``NotBlank`` é muito simples, ele só 
irá retornar ``True`` se o valor que ele está validando não está vazio. 

Em seguida, configuramos o validador para o e-mail ``email``. O serviço de validação do Symfony 2 fornece um validador 
para `E-mails <http://symfony.com/doc/current/reference/constraints/Email.html>`_ que ainda vai verificar os registros 
MX para assegurar se o domínio é válido. Sobre o ``subject`` queremos definir uma restrição ``NotBlank`` e ``MaxLength``. 
Você pode aplicar quantos validadores desejar em um determinado elemento.

A lista completa de `Restriçõs de Validadores <http://symfony.com/doc/current/reference/constraints.html>`_ está 
disponível nos documentos de referência do Symfony 2. 

É também possível 
`Criar validadores customizados <http://symfony.com/doc/current/cookbook/validation/custom_constraint.html>`_.

Agora, quando você enviar o formulário de contato, os dados apresentados serão transmitidos através dos critérios de 
validação. Tente digitar um endereço de e-mail inválido. Você deve ver uma mensagem de erro informando que o endereço de 
email é inválido. 

Cada validador fornece uma mensagem padrão que pode ser substituído se necessário. Para alterar a mensagem de saída do 
validador de e-mail, você deve fazer o seguinte:

.. code-block:: php

    $metadata->addPropertyConstraint('email', new Email(array(
        'message' => 'symblog does not like invalid emails. Give me a real one!'
    )));

.. tip::

    Se você estiver usando um browser que suporte HTML5 (é provávelmente você está), mensagens HTML5 serão exibidas 
    reforçando as restrições. 

    Esta é a validação do lado do cliente e o Symfony 2 irá definir as restrições HTML5 adequados com base nos metadados 
    da ``Entidade``. Você pode ver isso no elemento e-mail. A saída HTML é:

    .. code-block:: html

        <input type="email" value="" required="required" name="contact[email]" id="contact_email">

    Foi usado um dos novos tipos de campos de entrada do HTML5, e-mail, e estabeleceu o atributo necessário. 
    
    Validação do lado do cliente é importante pois não exige um envio para o servidor para que o servidor valide o 
    formulário. No entanto, a validação do lado cliente não devem ser usada ``sozinha``. Você deve sempre validar os 
    dados submetidos no lado servidor pois é muito fácil para um usuário contornar a validação do lado cliente.

Enviando o e-mail
-----------------

O nosso formulário de contato permitirá que os usuários enviem perguntas mas nada realmente acontece com eles ainda. 

Vamos atualizar o controlador para enviar um e-mail possa ser enviado.

Symfony2 vem com a biblioteca `Swift Mailer <http://swiftmailer.org/>`_ para envio de e-mails. Swift Mailer é uma 
biblioteca muito poderosa, vamos ver o que esta biblioteca pode realizar.

Configurar Swift Mailer
~~~~~~~~~~~~~~~~~~~~~~~

Swift Mailer já está configurado para trabalhar na distribuição Standard do Symfony 2, no entanto, precisamos definir 
algumas configurações relativas aos métodos de envio e credenciais. 

Abra o arquivo de parâmetros localizado em ``app/config/parameters.ini`` e encontre as configurações com o prefixo 
``mailer_``.

.. code-block:: text

    mailer_transport="smtp"
    mailer_host="localhost"
    mailer_user=""
    mailer_password=""

Swift Mailer fornece um número de métodos para enviar mensagens, incluindo o uso de um servidor SMTP, usando uma 
instalação local do sendmail ou mesmo usando uma conta do GMail.Para simplificar, vamos utilizar uma conta do GMail. 

Atualize os parâmetros com o seguinte: (substitua o nome de usuário e senha nos locais correspondentes)

.. code-block:: text

    mailer_transport="gmail"
    mailer_encryption="ssl"
    mailer_auth_mode="login"
    mailer_host="smtp.gmail.com"
    mailer_user="your_username"
    mailer_password="your_password"

.. warning::

    Tenha cuidado se você estiver usando um sistema de controle de versão (VCS) como Git para seu projeto, especialmente 
    se o seu repositório está acessível ao público, como o seu  nome de usuário e senha do GMail estão especificados no 
    repositório e será disponível para qualquer um ver. Você deve se certificar que o arquivo 
    ``app/config/parameters.ini`` está na lista de ignorados de seus VCS's. 

    Uma abordagem comum para este problema é sufixar o nome do arquivo que tem informações sensíveis, tais como 
    ``app/config/parameters.ini `` com ``.dist``. Você, então, fornece padrões sensíveis para as configurações deste 
    arquivo e adiciona o arquivo atual, ou seja, ``app/config/parameters.ini`` para sua lista VCS de ignorados. 

    Você pode então implantar o arquivo ``*.dist`` com o projeto e permite que o desenvolvedor remova a extensão 
    ``.dist`` e preencher as configurações necessárias.

Atualize o controlador
~~~~~~~~~~~~~~~~~~~~~~

Atualize o controlador ``Page`` localizado em ``src/Blogger/BlogBundle/Controller/PageController.php`` com o conteúdo 
abaixo:

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php

    public function contactAction()
    {
        // ..
        if ($form->isValid()) {

            $message = \Swift_Message::newInstance()
                ->setSubject('Contact enquiry from symblog')
                ->setFrom('enquiries@symblog.co.uk')
                ->setTo('email@email.com')
                ->setBody($this->renderView('BloggerBlogBundle:Page:contactEmail.txt.twig', array('enquiry' => $enquiry)));
            $this->get('mailer')->send($message);

            $this->get('session')->setFlash('blogger-notice', 'Your contact enquiry was successfully sent. Thank you!');

            // Redirect - This is important to prevent users re-posting
            // the form if they refresh the page
            return $this->redirect($this->generateUrl('BloggerBlogBundle_contact'));
        }
        // ..
    }

Quando você usa a biblioteca do Swift Mailer para criar uma instância de ``Exemplo Swift_Message``, podemos enviar um 
e-mail.

.. note::

    Como a biblioteca do Swift Mailer não usa namespaces, precisamos prefixar a classe do Swift Mailer com um ``\``. 
    Isto diz ao PHP para voltar para o `Espaço global <http://www.php.net/manual/en/language.namespaces.global.php>`_. 

    Você vai precisar prefixar todas as classes e funções que não tem namespace ``\``. Se você não colocar este prefixo 
    antes da classe ``Swift_Message``, o PHP irá pesquisar pela classe com namespace corrente, que neste exemplo é 
    ``Blogger\BlogBundle\Controlador``, causando um erro.

Também definimos uma ``flash mesage`` na sessão. As mensagens flash são mensagens que perduram por exatamente uma 
requisição. Depois disso, eles são automaticamente eliminados pelo Symfony 2. 

A ``Flash mesage`` será exibida na página de contato para informar ao usuário que o formulário foi enviado. Como a 
``Flash mesage`` apenas persistem por exatamente um pedido, elas são perfeitas para notificar o usuário do sucesso 
das ações anteriores.

Para exibir as ``Flash mesages``, precisamos atualizar o template de contato localizado em 
``src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig``. 

Atualize o conteúdo do template com o seguinte código:

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/contact.html.twig #}

    {# rest of template ... #}
    <header>
        <h1>Contact symblog</h1>
    </header>

    {% if app.session.hasFlash('blogger-notice') %}
        <div class="blogger-notice">
            {{ app.session.flash('blogger-notice') }}
        </div>
    {% endif %}

    <p>Want to contact symblog?</p>

    {# rest of template ... #}

Verificamos se uma ``flash mesage`` com o identificador ``blogger-notice`` está definido e, assim, imprimimos a mensagem.

Registre um e-mail de contato
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Symfony 2 fornece um sistema de configuração que podemos usar para definir as nossas próprias configurações. Vamos usar 
este sistema para definir o endereço de e-mail de contato ao invéz de codificar manualmente o endereço no controlador 
acima. Dessa forma, podemos facilmente reutilizar esse valor em outros lugares, sem duplicação de código. Além disso, 
quando o seu Blog gera muito tráfego, teremos muitas consultas, tornando difícil para o sistema lidar com isso. Assim, 
você pode facilmente atualizar o endereço de e-mail para passar os e-mails para seu assistente. 

Crie um novo arquivo em ``src/Blogger/BlogBundle/Resources/config/config.yml`` e cole seguinte código.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/config.yml
    parameters:
        # Blogger contact email address
        blogger_blog.emails.contact_email: contact@email.com

Ao definir parâmetros, é uma boa prática quebrar o nome do parâmetro em um número de componentes. 

A primeira parte deve ser uma versão ``lower cased`` do nome do pacote usando sublinhados para separar palavras. No 
nosso exemplo, transformamos o pacote ``BloggerBlogBundle`` em ``blogger_blog``. 

A parte restante do nome do parâmetro pode conter qualquer número de partes separadas pelo caractere ``.`` (Ponto final). 
Isso nos permite agrupar logicamente os parâmetros.

Para que a aplicação Symfony 2 use os novos parâmetros, precisamos importar a configuração para o arquivo de 
configuração principal da aplicação localizado em ``app/config/config.yml``. 

Para conseguir isso, atualize as diretivas ``imports`` na parte superior do arquivo para o seguinte código.

.. code-block:: yaml

    # app/config/config.yml
    imports:
        # .. existing import here
        - { resource: @BloggerBlogBundle/Resources/config/config.yml }

O caminho de importação é o local físico do arquivo no disco. A diretiva ``@BloggerBlogBundle`` irá dizer que o caminho 
do ``BloggerBlogBundle`` é ``src/Blogger/BlogBundle``.

Finalmente vamos atualizar a ação de contato para usar o parâmetro.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php

    public function contactAction()
    {
        // ..
        if ($form->isValid()) {

            $message = \Swift_Message::newInstance()
                ->setSubject('Contact enquiry from symblog')
                ->setFrom('enquiries@symblog.co.uk')
                ->setTo($this->container->getParameter('blogger_blog.emails.contact_email'))
                ->setBody($this->renderView('BloggerBlogBundle:Page:contactEmail.txt.twig', array('enquiry' => $enquiry)));
            $this->get('mailer')->send($message);

            // ..
        }
        // ..
    }

.. tip::

    Como o arquivo de configuração é importado, na parte superior do arquivo de configuração do aplicativo, podemos 
    facilmente substituir qualquer um dos parâmetros importados no aplicativo.

    Por exemplo, adicionar o seguinte código no fundo do arquivo ``app/config/config.yml`` substituiria o valor passado 
    do pacote pelo do parâmetro.

    .. code-block:: yaml

        # app/config/config.yml
        parameters:
            # Blogger contact email address
            blogger_blog.emails.contact_email: assistant@email.com

    Estas permissões de customizações para o pacote fornecem padrões sensíveis para os valores onde o aplicativo pode 
    substituí-los.

.. note::

    Embora seja fácil criar parâmetros de configuração do pacote usando este método, Symfony 2 também proporciona um 
    método onde você pode 
    `Expor uma configuração semântica <http://symfony.com/doc/current/cookbook/bundles/extension.html>`_ para um pacote. 
    Vamos explorar esse método no final do tutorial.

Criar um template de e-mail
~~~~~~~~~~~~~~~~~~~~~~~~~~~

O corpo do e-mail está configurado para renderizar um template. Crie este template em
``src/Blogger/BlogBundle/Resources/views/Page/contactEmail.txt.twig`` e adicione o seguinte código:

.. code-block:: text

    {# src/Blogger/BlogBundle/Resources/views/Page/contactEmail.txt.twig #}
    A contact enquiry was made by {{ enquiry.name }} at {{ "now" | date("Y-m-d H:i") }}.

    Reply-To: {{ enquiry.email }}
    Subject: {{ enquiry.subject }}
    Body:
    {{ enquiry.body }}

O conteúdo do e-mail é o formulário enviado pelo usuário.

Você também deve ter notado a extensão deste modelo é diferente dos outros templates que criamos. Ele usa a extensão 
``.txt.twig``. 

A primeira parte da extensão, ``.txt``, especifica o formato do arquivo a ser gerado. Os formatos mais comuns são 
``.txt``, ``.html``, ``.css``, ``.js``, ``.xml`` e ``.json``. 

A última parte da extensão especifica qual mecanismo de template vai ser utilizado, neste caso, Twig. Uma extensão de 
``.php`` usaria PHP para renderizar o template.

Agora, quando você enviar um formulário, um e-mail será enviado para o endereço definido no parametro 
``Blogger_blog.emails.contact_email``.

.. tip::

    Symfony 2 nos permite configurar o comportamento da biblioteca Swift Mailer em diferentes ambientes de operação do 
    Symfony 2. Já podemos ver isso em uso para o ambiente de ``test``. 

    Por padrão, a distribuição Standard do Symfony 2 configura Swift Mailer para não enviar e-mails durante a execução 
    do ambiente ``test``. Isso é definido no arquivo de configuração de teste localizado em 
    ``app/config/config_test.yml``.

    .. code-block:: yaml

        # app/config/config_test.yml
        swiftmailer:
            disable_delivery: true

    Seria bem útil duplicar essa funcionalidade para o ambiente ``dev``. Afinal, você não quer acidentalmente enviar um 
    e-mail para o endereço de e-mail errado durante o desenvolvimento. 

    Para fazer isso, adicione a configuração acima para o arquivo de configuração ``dev`` localizado em 
    ``app/config/config_dev.yml``.

    Você pode estar se perguntando como você pode testar se os e-mails estão sendo enviados e, mais especificamente, o 
    conteúdo deles, visto que eles não serão mais entregues para um endereço de e-mail real. Symfony 2 tem uma solução 
    para isso através da barra de ferramentas do desenvolvedor. Quando um e-mail é enviado um ícone de notificação de 
    e-mail aparecerá na barra de ferramentas que tem todas as informações sobre o e-mail que Swift Mailer entregaria.

    .. image:: /_static/images/part_2/email_notifications.jpg
        :align: center
        :alt: Symfony2 toolbar show email notifications

    Se você executar um redirecionamento após o envio de um e-mail, como nós fizemos para o formulário de contato, você 
    precisará definir a configuração de ``intercept_redirects`` em ``app/config/config_dev.yml`` para realmente ver o 
    e-mail de notificação na barra de ferramentas.

    Poderíamos ter configurado, ao invés do Swift Mailer enviar todos os e-mails para um determinado e-mail no ambiente 
    ``dev``, colocando a seguinte configuração no arquivo ``dev`` localizado em ``app/config/config_dev.yml``.

    .. code-block:: yaml

        # app/config/config_dev.yml
        swiftmailer:
            delivery_address:  development@symblog.dev

Conclusão
---------

Nós demonstramos os conceitos por trás da criação de um a parte mais fundamental de qualquer site: Firmulários. 
Symfony 2 vem com uma excelente biblioteca de Validadores e de Formulários que nos permite separar a lógica de validação 
do formulário para que possa ser utilizado por outras partes do aplicativo (como o modelo). Nós também mostramos como 
definir as configurações personalizadas que podem ser lidos no nosso aplicativo.

No próximo capítulo, vamos ver uma parte fundamental deste tutorial, o modelo. Vamos utilizar Doctrine 2 e usá-lo para 
definir o modelo de blog. Vamos também construir a página ``show`` do blog e explorar o conceito de Data Fixtures.
