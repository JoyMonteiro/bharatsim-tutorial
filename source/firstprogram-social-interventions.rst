Introduction of Social Interventions
====================================

Various social interventions can be made such as quarantine as explained in the previous section, or vaccine drives. 

Quarantine
^^^^^^^^^^

Quarantine can be brought into effect by forcing a schedule onto the people where everyone stays at their respective house. In ``create24HourSchedules`` everyone can be made to stay at home from 0 to 23, and this can be given the number 1 priority. When brought into effect, the school and office schedules will be ignored and the quarantine schedules will be abided by. 

.. code-block:: scala

  private def create24HourSchedules()(implicit context: Context): Unit = {
    val employeeSchedule = (myDay, myTick)
      .add[House](0, 8)
      .add[Office](9, 17)
      .add[House](18,23)

    val studentSchedule = (myDay, myTick)
      .add[House](0, 8)
      .add[Office](9, 16)
      .add[House](17, 23)

    val quarantinedSchedule = (myDay, myTick)
      .add[House](0, 23)

    registerSchedules(
      (quarantinedSchedule, (agent: Agent, _: Context) => agent.asInstanceOf[Person].isInfected, 1),
      (employeeSchedule, (agent: Agent, _: Context) => agent.asInstanceOf[Person].age >= 18, 2),
      (studentSchedule, (agent: Agent, _: Context) => agent.asInstanceOf[Person].age < 18, 3)
    )
  }