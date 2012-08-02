Criando um blog com Symfony2
============================

Introdução
------------

Este tutorial irá guiá-lo no processo de aprendizado, através da criação de um blog completo usando 
`Symfony 2 <http://symfony.com/>`_. 

A distribuição do Framework que será utilizada será a Standard, pois inclui os principais componentes que serão 
necessários para a construção de seus sites. 

O tutorial está dividido em partes cobrindo diferentes aspectos do Symfony 2 e seus componentes. Este tutorial tem, por 
objetivo, ser similar ao tutorial `Jobeet <http://www.symfony-project.org/jobeet/1_4/Doctrine/en/>`_ utilizado no 
Symfony 1.

Partes do tutorial
~~~~~~~~~~~~~~~~~~

.. toctree::
    :maxdepth: 1

    docs/configuration-and-templating
    docs/validators-and-forms
    docs/doctrine-2-the-blog-model
    docs/extending-the-model-blog-comments
    docs/customising-the-view-more-with-twig
    docs/testing-unit-and-functional-phpunit

Website de exemplo
------------------

O site Symblog pode ser acessado em
`http://symblog.co.uk <http://symblog.co.uk/>`_. O código fonte está disponível via 
`Github <https://github.com/dsyph3r/symblog>`_. Estes códigos, serão mostrados em cada parte do tutorial.

Conteúdo do tutorial
---------------------

Este tutorial tem, como objetivo, cobrir as tarefas comuns ao desenvolvimento de sites usando Symfony 2.

    1.  Pacotes (Bundles)
    2.  Controladores
    3.  Criação de interfaces (Usando TWIG)
    4.  Model - Doctrine 2
    5.  Migrações
    6.  Data Fixtures
    7.  Validadores
    8.  Formulários
    9.  Rotas
    10. Gerenciamento de Assets
    11. Envio de Email
    12. Ambientes
    13. Customização de páginas de erro
    14. Segurança
    15. Usuário & Sessão
    16. Geração de CRUD
    17. Memória Cache
    18. Testes
    19. Desenvolvimento

Symfony2 é altamente personalizável e fornece várias maneiras diferentes para realizar a mesma tarefa. Alguns exemplos 
disso são, a opção de escrever configurações em YAML, XML, PHP, ou Annotation, e criar interfaces utilizando Twig ou
PHP. 

Para manter este tutorial o mais simples possível, vamos usar YAML e Annotations para configuração e Twig para as 
interfaces. 

O `Symfony Book <http://symfony.com/doc/current/book/index.html>`_ pode ser utilizado como um excelente recurso com 
exemplos de como usar os outros métodos. 

Se outras pessoas quiserem contribuir para a realização de outros métodos alternativos, simplesmente faça o fork do 
repositório em `Github <https://github.com/dsyph3r/symblog-docs>`_ e envie os pedidos de alteração. :)

Traduções
---------

Espanhol
~~~~~~~~

Symblog foi traduzido para o `Espanhol <http://symblog.site90.net/>`_  graças à contribuição de 
`Lisper <https://twitter.com/#!/esymfony>`_.

Francês
~~~~~~~

Symblog foi traduzido para o `Frances <http://keiruaprod.fr/symblog-fr/>`_  graças à contribuição de 
`Clement Keirua <https://twitter.com/clemkeirua>`_.

Português
~~~~~~~~~

Symblog foi traduzido para o `Português Brasileiro <http://symblog-ptbr.totlab.com.br/>`_  graças à contribuição de 
`TotLab - Pesquisa e Desenvolvimento Web <http://totlab.com.br>`_.


Autor
------

Este tutorial, foi inicialmente criado por `dsyph3r <http://twitter.com/#!/dsyph3r>`_.

Contribuindo
------------

O `código <https://github.com/dsyph3r/symblog-docs>`_ para este tutorial, está disponível no Github. Se você gostaria de 
melhorar e/ou ampliar este tutorial, simplesmente faça o fork do projeto e envie-nos os pedidos para alteração. 

Você também pode propor questões usando o `GitHub Issue Tracker <https://github.com/dsyph3r/symblog-docs/issues>`_. 

Se alguém está interessado em criar um design muito mais agradável visualmente, por favor entre em 
`contato <http://twitter.com/#!/dsyph3r>`_!

Créditos
--------

Um agradecimento especial à todos os que contribuiram com a 
`Documentação oficial do Symfony 2 <http://symfony.com/doc/current/>`_. Esta ação proporcionou um recurso inestimável de 
informação.

Os ícones das bandeiras, podem ser encontrados em `famfamfam <http://www.famfamfam.com/lab/icons/flags/>`_.

Busca
-----

Procurando por um tópico específico? Use a busca :ref:`search`.
