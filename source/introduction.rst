Introduction
============

Simulating hypothetical scenarios is a very useful way to analyse and predict the behaviour of complex phenomena such as the spread of COVID-19 through a network of individuals. An infectious disease outbreak is heavily influenced by factors pertaining to both the disease (like the transmissibility of the disease, or how it is affected by vaccines), and the network on which it spreads (such as the age structure, population density, network contacts, and geography). These factors can be responsible for heterogeneity in the size and severity of outbreaks in different locations that might otherwise appear similar.

For this reason the results from models developed for other countries may not be able to achieve the desirable level of predictive ability. Such models might not account for factors pertinent to India -- its unique demographics, the state (or absence) of health care, stratified social structures, and complex ecological gradients. The inputs for such factors must be collated from multiple sources, including imperfect and poorly usable government data, which requires both local knowledge and considerable experience in data analysis and interpretation. The result is a pressing need for an epidemic modelling framework tailored to Indian needs, with potential applications beyond its immediate use for COVID-19 modelling.

Given these design requirements, building and implementing detailed agent-based models is an unsurprisingly daunting task. A useful model should ideally predict impending potential surges in health care requirements, enabling for timely resource allocation. It should also allow the modeler to compare the merits and consequences of different non-pharmaceutical interventions at different stages of the epidemic. As a specific example, a useful model might help the modeler untangle how asymptomatic infections play a role in the spread of the disease.

BharatSim has been designed keeping the aforementioned design requirements and challenges of the Indian context in mind. BharatSim's vision is to build an India scale agent-based framework that would enable modellers from different disciplines -- ranging from epidemiology, disaster management, and economics -- to advise policy makers and decision makers across institutions. No other framework in India is as generalizable or scalable in this context.



BharatSim has been developed with 3 goals in mind:

1. **Flexibility:** Since it is a social simulation framework, a researcher can develop models for a variety of disciplines including epidemiology, economics, climate science.

2. **Scalability:** It can keep track of millions of agents in an efficient manner.

3. **Customisation:** It has been developed to suit India's needs, via its support for synthetic population.


Synthetic Population
====================

A synthetic population is a simplified individual-level representation of the actual population. This means that while every person is represented individually in it, not all of their attributes are included (for example, hair colour or shoe-size are deemed to be irrelevant for modelling epidemic spread, and are thus ignored, while the presence of commodities like diabetes would be included). As such, a synthetic population does not aim to be identical to the actual population, but instead attempts to match its various statistical distributions and correlations, thereby being sufficiently close to the true population to be used in modelling.

In the table below, you can see an example of a section of a synthetic population. Each row represents an individual with a unique ID, as well as certain attributes. These attributes could be related to the individual themselves (like their gender, age, and height and so on), or their network (details pertaining to their homes, workplaces, and possibly schools). Additionally, the population could also contain information regarding the individual's comorbidities (for example, whether they have diabetes or other preexisting conditions), if this is deemed relevant to the modelling exercise.


.. csv-table:: sample_synthetic_population.csv
   :file: _static/csvs/sample_synthetic_population.csv
   :header-rows: 1
   :class: longtable

All of these attributes are strongly correlated with each other and a good synthetic population will ideally be able reproduce the correlations that occur in the real world. However, this is a monumental task; real world data is complex, and often contains many artifacts that need to be addressed.
