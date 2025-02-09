Bridge with ``symfony/messenger``
============================================================

Refer to the `official documentation <https://symfony.com/doc/current/messenger.html>`__ on Symfony's website.

| This bridge provides a way launch job using message bus.
| Along with ways to interact with messenger inside jobs.


Launch Job through Messenger dispatcher
------------------------------------------------------------

The
`DispatchMessageJobLauncher <https://github.com/yokai-php/batch-symfony-messenger/blob/0.x/src/DispatchMessageJobLauncher.php>`__
execute jobs via a symfony command message dispatch.

A `LaunchJobMessage <https://github.com/yokai-php/batch-symfony-messenger/blob/0.x/src/LaunchJobMessage.php>`__
message will be dispatched and handled by the
`LaunchJobMessageHandler <https://github.com/yokai-php/batch-symfony-messenger/blob/0.x/src/LaunchJobMessageHandler.php>`__
will be called with that message after being routed.

How to configure an async transport for the launcher?
------------------------------------------------------------

You first need to configure an async transport, like explained
`in Symfony’s doc <https://symfony.com/doc/current/messenger.html#transports-async-queued-messages>`__.

Then you will have to route the message to this async transport you created, like explained
`in Symfony’s doc <https://symfony.com/doc/current/messenger.html#routing-messages-to-a-transport>`__.

You will end with something like:

.. code-block:: yaml

    # config/packages/messenger.yaml
    framework:
        messenger:
            transports:
                async: "%env(MESSENGER_TRANSPORT_DSN)%"
            routing:
                'Yokai\Batch\Bridge\Symfony\Messenger\LaunchJobMessage':  async

.. seealso::
   | :doc:`What is a job launcher? </core-concepts/job-launcher>`
   | :doc:`Getting started with Symfony Framework </getting-started/with-symfony>`
   | :doc:`Bridge with Symfony Framework </bridges/symfony-framework>`


Dispatch item with messenger writer
------------------------------------------------------------

| This item writer will dispatches each item through Symfony's messenger bus.
| Items must be objects that messenger can handle.

.. literalinclude:: symfony-messenger/dispatch-each-item-as-message-writer.php
   :language: php

.. seealso::
   | :doc:`What is an item writer? </core-concepts/item-job/item-writer>`
