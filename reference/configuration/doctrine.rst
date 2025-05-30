Doctrine Configuration Reference (DoctrineBundle)
=================================================

The DoctrineBundle integrates both the :doc:`DBAL </doctrine/dbal>` and
:doc:`ORM </doctrine>` Doctrine projects in Symfony applications. All these
options are configured under the ``doctrine`` key in your application
configuration.

.. code-block:: terminal

    # displays the default config values defined by Symfony
    $ php bin/console config:dump-reference doctrine

    # displays the actual config values used by your application
    $ php bin/console debug:config doctrine

.. note::

    When using XML, you must use the ``http://symfony.com/schema/dic/doctrine``
    namespace and the related XSD schema is available at:
    ``https://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd``

.. _`reference-dbal-configuration`:

Doctrine DBAL Configuration
---------------------------

DoctrineBundle supports all parameters that default Doctrine drivers
accept, converted to the XML or YAML naming standards that Symfony
enforces. See the Doctrine `DBAL documentation`_ for more information.
The following block shows all possible configuration keys:

.. configuration-block::

    .. code-block:: yaml

        doctrine:
            dbal:
                dbname:               database
                host:                 localhost
                port:                 1234
                user:                 user
                password:             secret
                driver:               pdo_mysql
                # if the url option is specified, it will override the above config
                url:                  mysql://db_user:db_password@127.0.0.1:3306/db_name
                # the DBAL driverClass option
                driver_class:         App\DBAL\MyDatabaseDriver
                # the DBAL driverOptions option
                options:
                    foo: bar
                path:                 '%kernel.project_dir%/var/data/data.sqlite'
                memory:               true
                unix_socket:          /tmp/mysql.sock
                # the DBAL wrapperClass option
                wrapper_class:        App\DBAL\MyConnectionWrapper
                charset:              utf8mb4
                logging:              '%kernel.debug%'
                platform_service:     App\DBAL\MyDatabasePlatformService
                server_version:       '8.0.37'
                mapping_types:
                    enum: string
                types:
                    custom: App\DBAL\MyCustomType

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd
                http://symfony.com/schema/dic/doctrine
                https://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd">

            <doctrine:config>
                <doctrine:dbal
                    name="default"
                    dbname="database"
                    host="localhost"
                    port="1234"
                    user="user"
                    password="secret"
                    driver="pdo_mysql"
                    driver-class="App\DBAL\MyDatabaseDriver"
                    path="%kernel.project_dir%/var/data/data.sqlite"
                    memory="true"
                    unix-socket="/tmp/mysql.sock"
                    wrapper-class="App\DBAL\MyConnectionWrapper"
                    charset="utf8mb4"
                    logging="%kernel.debug%"
                    platform-service="App\DBAL\MyDatabasePlatformService"
                    server-version="8.0.37">

                    <doctrine:option key="foo">bar</doctrine:option>
                    <doctrine:mapping-type name="enum">string</doctrine:mapping-type>
                    <doctrine:type name="custom">App\DBAL\MyCustomType</doctrine:type>
                </doctrine:dbal>
            </doctrine:config>
        </container>

.. note::

    The ``server_version`` option was added in Doctrine DBAL 2.5, which
    is used by DoctrineBundle 1.3. The value of this option should match
    your database server version (use ``postgres -V`` or ``psql -V`` command
    to find your PostgreSQL version and ``mysql -V`` to get your MySQL
    version).

    If you are running a MariaDB database, you must prefix the ``server_version``
    value with ``mariadb-`` (e.g. ``server_version: mariadb-10.4.14``). This will
    change in Doctrine DBAL 4.x, where you must define the version as output by
    the server (e.g. ``10.4.14-MariaDB``).

    Always wrap the server version number with quotes to parse it as a string
    instead of a float number. Otherwise, the floating-point representation
    issues can make your version be considered a different number (e.g. ``5.7``
    will be rounded as ``5.6999999999999996447286321199499070644378662109375``).

    If you don't define this option and you haven't created your database
    yet, you may get ``PDOException`` errors because Doctrine will try to
    guess the database server version automatically and none is available.

If you want to configure multiple connections in YAML, put them under the
``connections`` key and give them a unique name:

.. code-block:: yaml

    doctrine:
        dbal:
            default_connection:       default
            connections:
                default:
                    dbname:           Symfony
                    user:             root
                    password:         null
                    host:             localhost
                    server_version:   '8.0.37'
                customer:
                    dbname:           customer
                    user:             root
                    password:         null
                    host:             localhost
                    server_version:   '8.2.0'

The ``database_connection`` service always refers to the *default* connection,
which is the first one defined or the one configured via the
``default_connection`` parameter.

Each connection is also accessible via the ``doctrine.dbal.[name]_connection``
service where ``[name]`` is the name of the connection. In a :doc:`controller </controller>`
you can access it using the ``getConnection()`` method and the name of the connection::

    // src/Controller/SomeController.php
    use Doctrine\Persistence\ManagerRegistry;

    class SomeController
    {
        public function someMethod(ManagerRegistry $doctrine): void
        {
            $connection = $doctrine->getConnection('customer');
            $result = $connection->fetchAllAssociative('SELECT name FROM customer');

            // ...
        }
    }

Doctrine ORM Configuration
--------------------------

This following configuration example shows all the configuration defaults
that the ORM resolves to:

.. code-block:: yaml

    doctrine:
        orm:
            auto_mapping: false
            # the standard distribution overrides this to be true in debug, false otherwise
            auto_generate_proxy_classes: false
            proxy_namespace: Proxies
            proxy_dir: '%kernel.cache_dir%/doctrine/orm/Proxies'
            default_entity_manager: default
            metadata_cache_driver: array
            query_cache_driver: array
            result_cache_driver: array
            naming_strategy: doctrine.orm.naming_strategy.default

There are lots of other configuration options that you can use to overwrite
certain classes, but those are for very advanced use-cases only.

Shortened Configuration Syntax
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When you are only using one entity manager, all config options available
can be placed directly under ``doctrine.orm`` config level.

.. code-block:: yaml

    doctrine:
        orm:
            # ...
            query_cache_driver:
                # ...
            metadata_cache_driver:
                # ...
            result_cache_driver:
                # ...
            connection: ~
            class_metadata_factory_name:  Doctrine\ORM\Mapping\ClassMetadataFactory
            default_repository_class:  Doctrine\ORM\EntityRepository
            auto_mapping: false
            naming_strategy: doctrine.orm.naming_strategy.default
            hydrators:
                # ...
            mappings:
                # ...
            dql:
                # ...
            filters:
                # ...

This shortened version is commonly used in other documentation sections.
Keep in mind that you can't use both syntaxes at the same time.

Caching Drivers
~~~~~~~~~~~~~~~

Use any of the existing :doc:`Symfony Cache </cache>` pools or define new pools
to cache each of Doctrine ORM elements (queries, results, etc.):

.. code-block:: yaml

    # config/packages/prod/doctrine.yaml
    framework:
        cache:
            pools:
                doctrine.result_cache_pool:
                    adapter: cache.app
                doctrine.system_cache_pool:
                    adapter: cache.system

    doctrine:
        orm:
            # ...
            metadata_cache_driver:
                type: pool
                pool: doctrine.system_cache_pool
            query_cache_driver:
                type: pool
                pool: doctrine.system_cache_pool
            result_cache_driver:
                type: pool
                pool: doctrine.result_cache_pool

            # in addition to Symfony Cache pools, you can also use the
            # 'type: service' option to use any service as the cache
            query_cache_driver:
                type: service
                id: App\ORM\MyCacheService

Mapping Configuration
~~~~~~~~~~~~~~~~~~~~~

Explicit definition of all the mapped entities is the only necessary
configuration for the ORM and there are several configuration options that
you can control. The following configuration options exist for a mapping:

``type``
........

One of ``attribute`` (for PHP attributes; it's the default value),
``xml``, ``php`` or ``staticphp``. This specifies which
type of metadata type your mapping uses.

.. versionadded:: 3.0

    The ``yml`` mapping configuration is deprecated and was removed in Doctrine ORM 3.0.

See `Doctrine Metadata Drivers`_ for more information about this option.

``dir``
.......

Absolute path to the mapping or entity files (depending on the driver).

``prefix``
..........

A common namespace prefix that all entities of this mapping share. This prefix
should never conflict with prefixes of other defined mappings otherwise some of
your entities cannot be found by Doctrine.

``alias``
.........

Doctrine offers a way to alias entity namespaces to simpler, shorter names
to be used in DQL queries or for Repository access.

``is_bundle``
.............

This option is ``false`` by default and it's considered a legacy option. It was
only useful in previous Symfony versions, when it was recommended to use bundles
to organize the application code.

.. _doctrine_auto-mapping:

Custom Mapping Entities in a Bundle
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine's ``auto_mapping`` feature loads attribute configuration from
the ``Entity/`` directory of each bundle *and* looks for other formats (e.g.
YAML, XML) in the ``Resources/config/doctrine`` directory.

If you store metadata somewhere else in your bundle, you can define your
own mappings, where you tell Doctrine exactly *where* to look, along with
some other configurations.

If you're using the ``auto_mapping`` configuration, you just need to overwrite
the configurations you want. In this case it's important that the key of
the mapping configurations corresponds to the name of the bundle.

For example, suppose you decide to store your ``XML`` configuration for
``AppBundle`` entities in the ``@AppBundle/SomeResources/config/doctrine``
directory instead:

.. configuration-block::

    .. code-block:: yaml

        doctrine:
            # ...
            orm:
                # ...
                auto_mapping: true
                mappings:
                    # ...
                    AppBundle:
                        type: xml
                        dir: SomeResources/config/doctrine

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd">

            <doctrine:config>
                <doctrine:orm auto-mapping="true">
                    <mapping name="AppBundle" dir="SomeResources/config/doctrine" type="xml"/>
                </doctrine:orm>
            </doctrine:config>
        </container>

    .. code-block:: php

        use Symfony\Config\DoctrineConfig;

        return static function (DoctrineConfig $doctrine): void {
            $emDefault = $doctrine->orm()->entityManager('default');

            $emDefault->autoMapping(true);
            $emDefault->mapping('AppBundle')
                ->type('xml')
                ->dir('SomeResources/config/doctrine')
            ;
        };

Mapping Entities Outside of a Bundle
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For example, the following looks for entity classes in the ``Entity``
namespace in the ``src/Entity`` directory and gives them an ``App`` alias
(so you can say things like ``App:Post``):

.. configuration-block::

    .. code-block:: yaml

        doctrine:
                # ...
                orm:
                    # ...
                    mappings:
                        # ...
                        SomeEntityNamespace:
                            type: attribute
                            dir: '%kernel.project_dir%/src/Entity'
                            is_bundle: false
                            prefix: App\Entity
                            alias: App

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd">

            <doctrine:config>
                <doctrine:orm>
                    <mapping name="SomeEntityNamespace"
                        type="attribute"
                        dir="%kernel.project_dir%/src/Entity"
                        is-bundle="false"
                        prefix="App\Entity"
                        alias="App"
                    />
                </doctrine:orm>
            </doctrine:config>
        </container>

    .. code-block:: php

        use Symfony\Config\DoctrineConfig;

        return static function (DoctrineConfig $doctrine): void {
            $emDefault = $doctrine->orm()->entityManager('default');

            $emDefault->autoMapping(true);
            $emDefault->mapping('SomeEntityNamespace')
                ->type('attribute')
                ->dir('%kernel.project_dir%/src/Entity')
                ->isBundle(false)
                ->prefix('App\Entity')
                ->alias('App')
            ;
        };

Detecting a Mapping Configuration Format
........................................

If the ``type`` on the bundle configuration isn't set, the DoctrineBundle
will try to detect the correct mapping configuration format for the bundle.

DoctrineBundle will look for files matching ``*.orm.[FORMAT]`` (e.g.
``Post.orm.yaml``) in the configured ``dir`` of your mapping (if you're mapping
a bundle, then ``dir`` is relative to the bundle's directory).

The bundle looks for (in this order) XML, YAML and PHP files.
Using the ``auto_mapping`` feature, every bundle can have only one
configuration format. The bundle will stop as soon as it locates one.

If it wasn't possible to determine a configuration format for a bundle,
the DoctrineBundle will check if there is an ``Entity`` folder in the bundle's
root directory. If the folder exist, Doctrine will fall back to using
attributes.

Default Value of Dir
....................

If ``dir`` is not specified, then its default value depends on which configuration
driver is being used. For drivers that rely on the PHP files (attribute,
``staticphp``) it will be ``[Bundle]/Entity``. For drivers that are using
configuration files (XML, YAML, ...) it will be
``[Bundle]/Resources/config/doctrine``.

If the ``dir`` configuration is set and the ``is_bundle`` configuration
is ``true``, the DoctrineBundle will prefix the ``dir`` configuration with
the path of the bundle.

SSL Connection with MySQL
~~~~~~~~~~~~~~~~~~~~~~~~~

To securely configure an SSL connection to MySQL in your Symfony application
with Doctrine, you need to specify the SSL certificate options. Here's how to
set up the connection using environment variables for the certificate paths:

.. configuration-block::

    .. code-block:: yaml

        doctrine:
            dbal:
                url: '%env(DATABASE_URL)%'
                server_version: '8.0.31'
                driver: 'pdo_mysql'
                options:
                    # SSL private key
                    !php/const 'PDO::MYSQL_ATTR_SSL_KEY': '%env(MYSQL_SSL_KEY)%'
                    # SSL certificate
                    !php/const 'PDO::MYSQL_ATTR_SSL_CERT': '%env(MYSQL_SSL_CERT)%'
                    # SSL CA authority
                    !php/const 'PDO::MYSQL_ATTR_SSL_CA': '%env(MYSQL_SSL_CA)%'

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd
                http://symfony.com/schema/dic/doctrine
                https://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd">

            <doctrine:config>
                <doctrine:dbal
                    url="%env(DATABASE_URL)%"
                    server-version="8.0.31"
                    driver="pdo_mysql">

                    <doctrine:option key-type="constant" key="PDO::MYSQL_ATTR_SSL_KEY">%env(MYSQL_SSL_KEY)%</doctrine:option>
                    <doctrine:option key-type="constant" key="PDO::MYSQL_ATTR_SSL_CERT">%env(MYSQL_SSL_CERT)%</doctrine:option>
                    <doctrine:option key-type="constant" key="PDO::MYSQL_ATTR_SSL_CA">%env(MYSQL_SSL_CA)%</doctrine:option>
                </doctrine:dbal>
            </doctrine:config>
        </container>

    .. code-block:: php

        // config/packages/doctrine.php
        use Symfony\Config\DoctrineConfig;

        return static function (DoctrineConfig $doctrine): void {
            $doctrine->dbal()
                ->connection('default')
                ->url(env('DATABASE_URL')->resolve())
                ->serverVersion('8.0.31')
                ->driver('pdo_mysql');

            $doctrine->dbal()->defaultConnection('default');

            $doctrine->dbal()->option(\PDO::MYSQL_ATTR_SSL_KEY, '%env(MYSQL_SSL_KEY)%');
            $doctrine->dbal()->option(\PDO::MYSQL_SSL_CERT, '%env(MYSQL_ATTR_SSL_CERT)%');
            $doctrine->dbal()->option(\PDO::MYSQL_SSL_CA, '%env(MYSQL_ATTR_SSL_CA)%');
        };

Ensure your environment variables are correctly set in the ``.env.local`` or
``.env.local.php`` file as follows:

.. code-block:: bash

    MYSQL_SSL_KEY=/path/to/your/server-key.pem
    MYSQL_SSL_CERT=/path/to/your/server-cert.pem
    MYSQL_SSL_CA=/path/to/your/ca-cert.pem

This configuration secures your MySQL connection with SSL by specifying the paths to the required certificates.


.. _DBAL documentation: https://www.doctrine-project.org/projects/doctrine-dbal/en/current/reference/configuration.html
.. _`Doctrine Metadata Drivers`: https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/metadata-drivers.html
