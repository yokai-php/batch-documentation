Bridge with ``symfony/framework-bundle``
============================================================

Refer to the `official documentation <https://symfony.com/doc/current/index.html>`__ on Symfony's website.

This bridge provides library configuration, and services registering in a Symfony framework project.


Configure the Job launcher(s)
------------------------------------------------------------

You can use many different job launcher in your application,
you will be able to register these using configuration:

.. code-block:: yaml

    # config/packages/yokai_batch.yaml
    yokai_batch:
        launcher:
            default: simple
            launchers:
              simple: ...
              async: ...

.. note::
   If you do not configure anything here, you will be using the
   `SimpleJobLauncher <https://github.com/yokai-php/batch/blob/0.x/src/Launcher/SimpleJobLauncher.php>`__.

| The ``default`` job launcher, must reference a launcher name, defined in the ``launchers`` list.
| The ``default`` job launcher will be the autowired instance of job launcher when you ask for one.
| All ``launchers`` will be registered as a service, and an autowire named alias will be configured for it.
| For instance, in the example below, you will be able to register all these launchers like this:

.. literalinclude:: symfony-framework/job-launcher-usage.php
   :language: php

All ``launchers`` are configured using a DSN, every scheme has it’s own associated factory:

* ``simple://simple``: a `SimpleJobLauncher <https://github.com/yokai-php/batch/blob/0.x/src/Launcher/SimpleJobLauncher.php>`__, no configuration allowed
* ``messenger://messenger``: a `DispatchMessageJobLauncher <https://github.com/yokai-php/batch-symfony-messenger/blob/0.x/src/DispatchMessageJobLauncher.php>`__, no configuration allowed
* ``console://console``: a `RunCommandJobLauncher <https://github.com/yokai-php/batch-symfony-console/blob/0.x/src/RunCommandJobLauncher.php>`__, configurable options:

  * ``log``: the filename where command output will be redirected (defaults to ``batch_execute.log``)

* ``service://service``: pointing to a service of your choice, configurable options:

  * ``service``: the id of the service to use (required, an exception will be thrown otherwise)

.. seealso::
   | :doc:`What is a job launcher? </core-concepts/job-launcher>`


Configure the JobExecution storage
------------------------------------------------------------

You can have only one storage for your ``JobExecution``, and you have several options:

* ``filesystem`` will create a file for each ``JobExecution`` in
  ``%kernel.project_dir%/var/batch/{execution.jobName}/{execution.id}.json``
* ``dbal`` will create a row in a table for each ``JobExecution``
* ``service`` will use a service you have defined in your application

.. code-block:: yaml

    # config/packages/yokai_batch.yaml
    yokai_batch:
        storage:
            filesystem: ~
            # Or with yokai/batch-doctrine-dbal (& doctrine/dbal)
            # dbal: ~
            # Or with a service of yours
            # service: ~

.. note::
   | The default storage is ``filesystem``, because it only requires a writeable filesystem.
   | But if you already have ``doctrine/dbal`` in your project, it is highly recommended to use it instead.
   | Because querying ``JobExecution`` in a filesystem might be slow, specially if you are planing to add UIs on top.

.. seealso::
   | :doc:`What is a job execution? </core-concepts/job-execution>`
   | :doc:`What is a job execution storage? </core-concepts/job-execution-storage>`

Configure the JobExecution id generator
------------------------------------------------------------

When it is created, every ``JobExecution`` is assigned with a unique identifier.
You can configure what your id will be like:

* ``uniqid``: a `UniqidJobExecutionIdGenerator <https://github.com/yokai-php/batch/blob/0.x/src/Factory/UniqidJobExecutionIdGenerator.php>`__
* ``symfony.uuid.random``: a `RandomBasedUuidJobExecutionIdGenerator <https://github.com/yokai-php/batch-symfony-uid/blob/0.x/src/Factory/RandomBasedUuidJobExecutionIdGenerator.php>`__
* ``symfony.uuid.time``: a `TimeBasedUuidJobExecutionIdGenerator <https://github.com/yokai-php/batch-symfony-uid/blob/0.x/src/Factory/TimeBasedUuidJobExecutionIdGenerator.php>`__
* ``symfony.ulid``: a `UlidJobExecutionIdGenerator <https://github.com/yokai-php/batch-symfony-uid/blob/0.x/src/Factory/UlidJobExecutionIdGenerator.php>`__

.. note::
   | The default storage is ``uniqid``, because it only requires the function with the same name that is a PHP standard.

.. code-block:: yaml

    # config/packages/yokai_batch.yaml
    yokai_batch:
        id: uniqid
        # Or with yokai/batch-symfony-uid (& symfony/uid)
        # id: symfony.uuid.random
        # id: symfony.uuid.time
        # id: symfony.ulid

User interface to visualize ``JobExecution``
------------------------------------------------------------

The package is shipped with few routes that will allow you and your users, to watch for ``JobExecution``.

.. image:: /_static/images/symfony/ui/bootstrap4-list.png
.. image:: /_static/images/symfony/ui/bootstrap4-details.png
.. image:: /_static/images/symfony/ui/bootstrap4-children.png
.. image:: /_static/images/symfony/ui/bootstrap4-warnings.png

Installation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For the UI to be enabled, it is required that you install some dependencies:

.. code-block:: shell

    composer require symfony/translation symfony/twig-bundle

The UI is disabled by default, you must enable it explicitly:

.. code-block:: yaml

    # config/packages/yokai_batch.yaml
    yokai_batch:
      ui:
        enabled: true

You will also need to import bundle routes:

.. code-block:: yaml

    # config/routes/yokai_batch.yaml
    _yokai_batch:
      resource: "@YokaiBatchBundle/Resources/routing/ui.xml"

Templating
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| The templating service is used by the
  `JobController <https://github.com/yokai-php/batch-symfony-framework/blob/0.x/src/UserInterface/Controller/JobController.php>`__
  to render its templates.
| It’s a wrapper around `Twig <https://twig.symfony.com/>`__, for you to control templates used,
  and variables passed.

By default

* the templating will find templates like ``@YokaiBatch/bootstrap4/*.html.twig``
* the template base view will be ``base.html.twig``

You can configure a prefix for all templates:


.. code-block:: yaml

    # config/packages/yokai_batch.yaml
    yokai_batch:
      ui:
        templating:
          prefix: 'batch/job/'

.. note::
   With this configuration, we will look for templates like ``batch/job/*.html.twig``.

You can also configure the name of the base template for the root views of that bundle:

.. code-block:: yaml

    # config/packages/yokai_batch.yaml
    yokai_batch:
      ui:
        templating:
          base_template: 'layout.html.twig'

.. note::
   With this configuration, the template base view will be ``layout.html.twig``.

If these are not enough, or if you need to add more variables to context, you can configure a service:

.. code-block:: yaml

    # config/packages/yokai_batch.yaml
    yokai_batch:
      ui:
        templating:
          service: 'App\Batch\AppTemplating'

And create the class that will cover the templating:

.. literalinclude:: symfony-framework/ui-templating-custom.php
   :language: php

.. note::
   You can also use the
   ``Yokai\Batch\Bridge\Symfony\Framework\UserInterface\Templating\ConfigurableTemplating``
   that will cover both prefix and static variables at construction.

Filtering
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``JobExecution`` list includes a filter form, but you will need another optional dependency:

.. code-block:: shell

    composer require symfony/form

Security
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There is no access control over ``JobExecution`` by default, you will need another optional dependency:

.. code-block:: shell

    composer require symfony/security-bundle

Every security attribute the bundle is using is configurable:

.. code-block:: yaml

    # config/packages/yokai_batch.yaml
    yokai_batch:
      ui:
        security:
          attributes:
            list: ROLE_JOB_LIST # defaults to IS_AUTHENTICATED
            view: ROLE_JOB_VIEW # defaults to IS_AUTHENTICATED
            traces: ROLE_JOB_TRACES # defaults to IS_AUTHENTICATED
            logs: ROLE_JOB_LOGS # defaults to IS_AUTHENTICATED

| Optionally, you can register a voter for these attributes.
| This is especially useful if you need different access control rules per ``JobExecution``.

.. literalinclude:: symfony-framework/ui-voter.php
   :language: php

Integration with SonataAdminBundle
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| If you are on a
  `SonataAdmin <https://symfony.com/bundles/SonataAdminBundle/current/index.html>`__
  project.
| The bundle got you covered with a dedicated templating services
  and templates.

.. image:: /_static/images/symfony/ui/sonata-list.png
.. image:: /_static/images/symfony/ui/sonata-details.png
.. image:: /_static/images/symfony/ui/sonata-children.png
.. image:: /_static/images/symfony/ui/sonata-warnings.png

.. code-block:: shell

    composer require sonata-project/admin-bundle

.. code-block:: yaml

    # config/packages/yokai_batch.yaml
    yokai_batch:
      ui:
        templating: sonata

.. note::
   With this configuration, we will look for templates like ``@YokaiBatch/sonata/*.html.twig``.

Customizing templates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| You can override templates like
  `described it Symfony’s documentation <https://symfony.com/doc/current/bundles/override.html>`__.
| Examples:

* ``templates/bundles/YokaiBatchBundle/bootstrap4/list.html.twig``
* ``templates/bundles/YokaiBatchBundle/bootstrap4/show/_parameters.html.twig``

But you can also register job name dedicated templates if you need some specific view for one of your jobs:

* ``templates/bundles/YokaiBatchBundle/bootstrap4/show/{job name}/_children-executions.html.twig``
* ``templates/bundles/YokaiBatchBundle/bootstrap4/show/{job name}/_failures.html.twig``
* ``templates/bundles/YokaiBatchBundle/bootstrap4/show/{job name}/_general.html.twig``
* ``templates/bundles/YokaiBatchBundle/bootstrap4/show/{job name}/_information.html.twig``
* ``templates/bundles/YokaiBatchBundle/bootstrap4/show/{job name}/_parameters.html.twig``
* ``templates/bundles/YokaiBatchBundle/bootstrap4/show/{job name}/_summary.html.twig``
* ``templates/bundles/YokaiBatchBundle/bootstrap4/show/{job name}/_warnings.html.twig``


Logger service that log in ``JobExecution``
------------------------------------------------------------

| The batch logger will log inside the JobExecution.
| In a Symfony project, you can use that with the symfony autowiring
  by naming your variable as ``$yokaiBatchLogger``

.. literalinclude:: symfony-framework/logger-usage.php
   :language: php

.. seealso::
   | :doc:`What is the job execution? </core-concepts/job-execution>`
