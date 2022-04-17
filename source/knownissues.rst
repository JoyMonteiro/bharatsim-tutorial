Known Issues
============

OneForOneStrategy - null
------------------------

This issue will lead to the following output:

.. code-block::

  [ticks-loop-akka.actor.default-dispatcher-46] ERROR akka.actor.LocalActorRefProvider - guardian failed, shutting down system

.. tip::
 
  Note that the ``46`` in the above error could be any number, as this error arises from parallelism

You will also see the following error below the first one:

.. code-block::

  [ticks-loop-akka.actor.default-dispatcher-46] ERROR akka.actor.OneForOneStrategy - null
  java.util.NoSuchElementException: null

The error is not easily reproducible, and appears to occur randomly.

To fix the error, you need to edit the ``application.conf`` file. It can be found in ``/src/main/resources/``. The first couple of lines should be

.. code-block:: scala

  bharatsim {
    engine {
        execution {
            mode = "actor-based"
            mode = ${?EXECUTION_MODE}
            simulation-steps = 4000
            simulation-steps = ${?SIMULATION_STEPS}
            actor-based {
                num-processing-actors = 100
            }
        }

You need to change line 4 to 

.. code-block:: 

  mode = "no-parallelism"

This will fix the issue, but also remove parallelism, which can lead to your code taking longer to run.
