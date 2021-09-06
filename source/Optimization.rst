Optimisation
===============


PerTickCache
----------------
PerTickCache is a Map-based temporary data store, which is used as a cache. It is a time and resource consuming process to access 
the data store for each agent, most of which may even be redundant. We can use the PerTickCache for accessing such data. 
If the value is already computed, we can acces it from there or compute the result and then store it as cache. 
The PerTickCache is auto cleaned at the beginning of the tick. Example: In epidemiology, if we are trying to compute whom all 
might get infected based on their area(Home, office, etc.), the total number of infected in the area remains constant for a 
particular tick, and the same can be used for all susceptible agents.

The user has to determine a unique key such as placeType to avoid overriding the results. A new value will be computed only
if the value of that key is not present.
Consider a case where a particular placeType, say office_1, has 1000 agents in a particular tick. So if we have to find the infected
fraction for each agent, we have to query the data store for all the 1000 agents. But if we are using PerTickCache, we need to 
access the data store only once. This significantly speeds up our simulation.

Example for PerTickCache implementation:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Here we define the function which computes the value to be stored as cache.

.. code-block:: Scala

    private def computeTheValue(node: Node): Double = {
    val total = node.getConnectionCount(node.getRelation[Person]().get)
    if (total == 0)
      return 0.0
    val infected = node.getConnectionCount(node.getRelation[Person]().get, "infectionState" equ Infected)

    infected.toDouble/total.toDouble
    }

The below function computes the infected fraction at a place, i.e. the number of infected people divided by the total number of people
in that particular place. 

.. code-block:: Scala

    private  def fetchInfectedFraction(node: Node, placeType: String, context: Context): Double = {
    val cache = context.perTickCache
    val uniqueKey = (placeType, node.internalId)
    cache.getOrUpdate(uniqueKey, () => computeTheValue(node)).asInstanceOf[Double]
    }

The unique key used here is place type. Since in a particular tick, all agents in the same place will
have the same infected fraction, we do not have to compute this for every agent; instead, we use the value from the cache. This improves the
computation speed and minimises resource usage.
