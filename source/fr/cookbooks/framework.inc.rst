
.. _utilisation-avec-frameworks:

Utilisation avec des frameworks
****************************

.. _utilisation-avec-ezpublish:

Utilisation avec ez Publish
=========================


Étape 1 : Installation d'atoum au sein d'eZ Publish
-------------------------------------------------------

Le framework eZ Publish possède déjà un répertoire dédié aux tests, nommé logiquement tests. C'est donc dans ce répertoire que devra être placé l':ref:`archive PHAR <archive-phar>` d’atoum. Les fichiers de tests unitaires utilisant atoum seront quant à eux placés dans un sous-répertoire *tests/atoum* afin qu'ils ne soient pas en conflit avec l'existant.


Étape 2 : Création de la classe de test de base
---------------------------------------------------

Une classe de test basée sur atoum doit étendre la classe ``\mageekguy\atoum\test``. However, this one doesn't take into account of *eZ Publish* specificities. It's therefore mandatory to
Il est donc nécessaire de définir une classe de test de base, dérivée de ``\mageekguy\atoum\test``, qui prendra en compte ces spécifités et donc dérivera l'ensemble des classes de tests unitaires. Pour cela, il suffit de définir la classe suivante dans le fichier ``tests\atoum\test.php`` :

.. code-block:: php

	<?php

	namespace ezp;

	use mageekguy\atoum;

	require_once __DIR__ . '/mageekguy.atoum.phar';

	// Autoloading : eZ
	require 'autoload.php';

	if ( !ini_get( "date.timezone" ) )
	{
		date_default_timezone_set( "UTC" );
	}

	require_once( 'kernel/common/i18n.php' );

	\eZContentLanguage::setCronjobMode();

	/**
	 * @abstract
	 */
	abstract class test extends atoum\test
	{
	}

	?>



Étape 3 : Création d'une classe de test
-----------------------------

Par défaut, atoum demande à ce que les classes de tests unitaires soient dans un espace de noms contenant *test(s)\unit(s)*, afin de pouvoir déduire le nom de la classe testée. À titre d'exemple, l'espace de noms *\nomprojet* sera utilisé dans ce qui suit. Pour plus de simplicité, il est de plus conseillé de calquer l'arborescence des classes de test sur celle des classes testées, afin de pouvoir localiser rapidement la classe de test d'une classe, et inversement.

.. code-block:: php

	<?php

	namespace nomdeprojet\tests\units;

	require_once '../test.php';

	use ezp;

	class cache extends ezp\test
	{
	   public function testClass()
	   {
		  $this->assert->hasMethod('__construct');
	   }
	}


Étapes 4 : Exécution des tests unitaires
------------------------------

Une fois une classe de test créée, il suffit d'exécuter en ligne de commande l'instruction ci-dessous pour lancer le test, en se plaçant à la racine du projet :

.. code-block:: shell

	# php tests/atoum/mageekguy.atoum.phar -d tests/atoum/units


Merci `Jérémy Poulain <https://github.com/Tharkun>`_ pour ce tutoriel.


.. _utilisation-avec-symfony-2:

Utilisation avec Symfony 2
=========================

Si vous souhaitez utiliser atoum au sein de vos projets Symfony, vous pouvez installer le Bundle `AtoumBundle <https://github.com/atoum/AtoumBundle>`_.

Si vous souhaitez installer et configurer atoum manuellement, voici comment faire.


Étape 1: installation d'atoum
-----------------------------

Si vous utilisez Symfony 2.0, :ref:`téléchargez l'archive PHAR <archive-phar>`_ et placez-la dans le répertoire vendor qui est à la racine de votre projet.

Si vous utilisez Symfony 2.1+, :ref:`ajoutez atoum dans votre fichier composer.json <installation-par-composer>`_.


Étape 2: création de la classe de test
-----------------------------

Imaginons que nous voulions tester cet Entity:

.. code-block:: php

   <?php
   // src/Acme/DemoBundle/Entity/Car.php
   namespace Acme\DemoBundle\Entity;

   use Doctrine\ORM\Mapping as ORM;

   /**
    * Acme\DemoBundle\Entity\Car
    * @ORM\Table(name="car")
    * @ORM\Entity(repositoryClass="Acme\DemoBundle\Entity\CarRepository")
    */
   class Car
   {
       /**
        * @var integer $id
        * @ORM\Column(name="id", type="integer")
        * @ORM\Id
        * @ORM\GeneratedValue(strategy="AUTO")
        */
       private $id;

       /**
        * @var string $name
        * @ORM\Column(name="name", type="string", length=255)
        */
       private $name;

       /**
        * @var integer $max_speed
        * @ORM\Column(name="max_speed", type="integer")
        */

       private $max_speed;
   }

.. note::
   Pour plus d'informations sur la création d'Entity dans Symfony 2, reportez-vous au `manuel Symfony <http://symfony.com/fr/doc/current/book/doctrine.html#creer-une-classe-entite>`_.


Créez le répertoire Tests/Units dans votre Bundle (par exemple src/Acme/DemoBundle/Tests/Units). C'est dans ce répertoire que seront stockés tous les tests de ce Bundle.

Créez un fichier Test.php qui servira de base à tous les futurs tests de ce Bundle.

.. code-block:: php

   <?php
   // src/Acme/DemoBundle/Tests/Units/Test.php
   namespace Acme\DemoBundle\Tests\Units;

   // On inclus et active le class loader
   require_once __DIR__ . '/../../../../../vendor/symfony/symfony/src/Symfony/Component/ClassLoader/UniversalClassLoader.php';

   $loader = new \Symfony\Component\ClassLoader\UniversalClassLoader();

   $loader->registerNamespaces(
       array(
           'Symfony'         => __DIR__ . '/../../../../../vendor/symfony/src',
           'Acme\DemoBundle' => __DIR__ . '/../../../../../src'
       )
   );

   $loader->register();

   use mageekguy\atoum;

   // Pour Symfony 2.0 uniquement !
   require_once __DIR__ . '/../../../../../vendor/mageekguy.atoum.phar';

   abstract class Test extends atoum
   {
       public function __construct(
           adapter $adapter = null,
           annotations\extractor $annotationExtractor = null,
           asserter\generator $asserterGenerator = null,
           test\assertion\manager $assertionManager = null,
           \closure $reflectionClassFactory = null
       )
       {
           $this->setTestNamespace('Tests\Units');
           parent::__construct(
               $adapter,
               $annotationExtractor,
               $asserterGenerator,
               $assertionManager,
               $reflectionClassFactory
           );
       }
   }

.. note::
   L'inclusion de l'archive PHAR d'atoum n'est nécessaire que pour Symfony 2.0. Supprimez cette ligne dans le cas où vous utilisez Symfony 2.1+.


.. note::
   Par défaut, atoum utilise le namespace tests/units pour les tests. Or Symfony 2 et son class loader exige des majuscules au début des noms. Pour cette raison, nous changeons le namespace des tests grâce à la méthode setTestNamespace('Tests\Units').


Étape 3: écriture d'un test
--------------------

In the Tests/Units directory, simply recreate the tree of the classes that you want to test (for example src/Acme/DemoBundle/Tests/Units/Entity/Car.php).

Create our test file:

.. code-block:: php

   <?php
   // src/Acme/DemoBundle/Tests/Units/Entity/Car.php
   namespace Acme\DemoBundle\Tests\Units\Entity;

   require_once __DIR__ . '/../Test.php';

   use Acme\DemoBundle\Tests\Units\Test;

   class Car extends Test
   {
       public function testGetName()
       {
           $this
               ->if($car = new \Acme\DemoBundle\Entity\Car())
               ->and($car->setName('Batmobile'))
                   ->string($car->getName())
                       ->isEqualTo('Batmobile')
                       ->isNotEqualTo('De Lorean')
           ;
       }
   }


Step 4: launch tests
--------------------

If you use Symfony 2.0:

.. code-block:: shell

   # Launch tests of one file
   $ php vendor/mageekguy.atoum.phar -f src/Acme/DemoBundle/Tests/Units/Entity/Car.php

   # Launch all tests of the Bundle
   $ php vendor/mageekguy.atoum.phar -d src/Acme/DemoBundle/Tests/Units

If you use Symfony 2.1+:

.. code-block:: shell

   # Launch tests of one file
   $ ./bin/atoum -f src/Acme/DemoBundle/Tests/Units/Entity/Car.php

   # Launch all tests of the Bundle
   $ ./bin/atoum -d src/Acme/DemoBundle/Tests/Units

.. note::
   You can get more information on the :ref:`test launch <lancement-des-tests>` in the chapter which is dedicated to.


In any case, this is what you should get:

.. code-block:: shell

   > PHP path: /usr/bin/php
   > PHP version:
   > PHP 5.3.15 with Suhosin-Patch (cli) (built: Aug 24 2012 17:45:44)
   ===================================================================
   > Copyright (c) 1997-2012 The PHP Group
   =======================================
   > Zend Engine v2.3.0, Copyright (c) 1998-2012 Zend Technologies
   ===============================================================
   >     with Xdebug v2.1.3, Copyright (c) 2002-2012, by Derick Rethans
   ====================================================================
   > Acme\DemoBundle\Tests\Units\Entity\Car...
   [S___________________________________________________________][1/1]
   > Test duration: 0.01 second.
   =============================
   > Memory usage: 0.50 Mb.
   ========================
   > Total test duration: 0.01 second.
   > Total test memory usage: 0.50 Mb.
   > Code coverage value: 42.86%
   > Class Acme\DemoBundle\Entity\Car: 42.86%
   ==========================================
   > Acme\DemoBundle\Entity\Car::getId(): 0.00%
   ---------------------------------------------------
   > Acme\DemoBundle\Entity\Car::setMaxSpeed(): 0.00%
   --------------------------------------------------
   > Acme\DemoBundle\Entity\Car::getMaxSpeed(): 0.00%
   --------------------------------------------------
   > Running duration: 0.24 second.
   Success (1 test, 1/1 method, 0 skipped method, 4 assertions) !


.. _utilisation-avec-symfony-1-4:

Use with symfony 1.4
====================

If you want to use atoum inside your Symfony 1.4 project, you can install the plugins sfAtoumPlugin. It's available on this address: `https://github.com/atoum/sfAtoumPlugin <https://github.com/atoum/sfAtoumPlugin>`_.


Installation
------------

There are several ways to install this plugin in your project:

* installation via composer
* installation via git submodules


Using composer
""""""""""""""

Add this lines inside the composer.json file:

.. code-block:: json

   "require"     : {
     "atoum/sfAtoumPlugin": "*"
   },

After a ``php composer.phar update`` the plugin should be in the plugin folder and atoum in the ``vendor`` folder.

Then, in your ProjectConfiguration file, you have to activate the plugin and define the atoum path.

.. code-block:: php

   <?php
   sfConfig::set('sf_atoum_path', dirname(__FILE__) . '/../vendor/atoum/atoum');

   if (sfConfig::get('sf_environment') != 'prod')
   {
     $this->enablePlugins('sfAtoumPlugin');
   }


Using a git submodule
"""""""""""""""""""""

First, install atoum as a submodule:

.. code-block:: shell

   $ git submodule add git://github.com/atoum/atoum.git lib/vendor/atoum

Then install sfAtoumPlugin as a git submodule:

.. code-block:: shell

   $ git submodule add git://github.com/atoum/sfAtoumPlugin.git plugins/sfAtoumPlugin

Finally, enable the plugin in in your ProjectConfiguration file:

.. code-block:: php

   <?php
   if (sfConfig::get('sf_environment') != 'prod')
   {
     $this->enablePlugins('sfAtoumPlugin');
   }


Write tests
-----------

Tests must include the bootstrap file from the plugin:

.. code-block:: php

   <?php
   require_once __DIR__ . '/../../../../plugins/sfAtoumPlugin/bootstrap/unit.php';


Launch tests
------------

The symfony command ``atoum:test`` is available. The tests can then be launched in this way:

.. code-block:: shell

   $ ./symfony atoum:test

All the arguments of atoum are available.

It's therefore, for example, possible to give a configuration file like this :

.. code-block:: php

   <?php
   php symfony atoum:test -c config/atoum/hudson.php


.. _framework-symfony-1-plugin:

Symfony 1 plugin
================

To use atoum within a symfony project 1, a plug-in exists and is available at the following address: `https://github.com/atoum/sfAtoumPlugin <https://github.com/atoum/sfAtoumPlugin>`_.

The instructions for installation and use are the cookbook  :ref:`utilisation-avec-symfony-1-4` as well as on the github page.


.. _framework-symfony-2-bundle:

Symfony 2 bundle
================

To use atoum inside a Symfony 2 project, the bundle `AtoumBundle <https://github.com/atoum/AtoumBundle>`_  is available.

The instructions for installation and use are the cookbook :ref:`utilisation-avec-symfony-2` as well as on the github page.

.. _framework-zend-framework-2:

Zend Framework 2 component
==========================

If you want to use atoum within a Zend Framework 2 project, a component exists and is available at the `following address <https://github.com/blanchonvincent/zend-framework-test-atoum>`_.

The instructions for installation and usage are available on this page.
