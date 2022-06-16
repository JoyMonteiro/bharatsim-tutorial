Known Issues
============

java.util.NoSuchElementException on fetchActiveState
----------------------------------------------------

This issue will lead to the following output:

.. code-block::

  [ticks-loop-akka.actor.default-dispatcher-46] ERROR akka.actor.LocalActorRefProvider - guardian failed, shutting down system

.. tip::
 
  Note that the ``46`` in the above error could be any number, as this error arises from parallelism

Below this, you will find one of the following errors:

.. code-block::

  java.util.NoSuchElementException: null
	at scala.collection.concurrent.TrieMap.apply(TrieMap.scala:878)

.. code-block::
  
  java.lang.Exception: Something went wrong, each StatefulAgent must have one active state all the times
	at com.bharatsim.engine.models.StatefulAgent.fetchActiveState(StatefulAgent.scala:69)

The error is not easily reproducible, and appears to occur randomly.

A preliminary investigation suggests a problem with the underlying ``TrieMap`` data structure. Edges on the map do not appear to be created, and cannot be referenced when called.  Note that this behaviour only occurs if bharatsim is parallelized.

A potential workaround is to run bharatsim without parallelism. To do so, you need to edit the ``application.conf`` file, which can be found in ``/src/main/resources/``. The first couple of lines should be

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

Line 4 needs to be changed to

.. code-block:: 

  mode = "no-parallelism"

This will fix the issue, but be aware that removing parallelism could lead to your code taking longer to run.

.. note::
  For the latest on the error, please check the `github issue <https://github.com/debayanLab/BharatSim/issues/3>`_
