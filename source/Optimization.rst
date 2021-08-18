Optimisation
===============


PerTickCache
----------------
This function is used to retrieve values from the cache if some value is already computed. If the value is not present in the cache, it computes the value using a user-defined function and stores it in the cache.
This is extremely useful in reducing the simulation time when we repeatedly have to query the context to compute the value for every agent in a tick.

For example, consider the case in the simple SIR model where we have to find the fraction of infected individuals around an agent in a particular place. Here the infected fraction of all agents in the same place would be the same. So while running the simulation, we have to compute the same value repeatedly for all agents in that place. We need to query the context for all agents during each tick, thereby increasing the simulation time and computational resource required.
Here when we implement PerTickCache, the infected fraction will be computed for all places as and when needed during each tick, and that value will be stored in the cache to be used for all the agents in that place.

If the value is not computed for a place, the PerTickCache function will compute the value using a user-defined function. 

.. note:: A function to compute the infected fraction for an agent in a perticular place using PerTickCache.

    .. code-block:: scala

        private  def fetchInfectedFraction(node: Node, placeType: String, context: Context): Double = {
        val cache = context.perTickCache
        val uniqueKey = (placeType, node.internalId)
        cache.getOrUpdate(uniqueKey, () => computeTheValue(node)).asInstanceOf[Double]
        }



If the value is not computed for a place, the PerTickCache function will compute the value using a user-defined function. 
