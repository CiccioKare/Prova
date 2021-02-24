---
description: '#optimization #scheduling #inventory #routing #tests #bestpractises'
---

# Functional Tests & Best Practices

This template has the purpose to **schematize** all the **steps** every **optimization project** needs to have.

First of all, let us recap the **three different kinds of optimization projects** we have encountered in Ammagamma \(or EnergyWay\):

* **Scheduling** \(optimize the cycle of production of a set of machineries in order to maximize the production or minimize the time spent, or minimize the energy consumed\)
* **Inventory Optimization** \(optimize the orders to suppliers in order to minimize the storage of goods or to maximize the service level or to minimize the expenses\)
* **Routing Optimization** \(choose the best route for a vehicle or a fleet of vehicles in order to minimize the time of journey or the fuel consumed\)

### Functional Tests Common to all Optimization Projects

For each kind of optimization project, there are some steps that are really advisable to follow. We summarize them in the following:

1. Define the **KPIs** you are going to use to evaluate your solution \(for example: number of product made in time unit; time spent to finish the production of all orders, ...\)
2. Write methodically your **mathematical modelling** and update it as soon as you procede with the development
3. Define an appropriate way to **visualize** the results
4. Build a **TOY MODEL** that you can solve by hand \(using pen and paper!\) and that you can use to check whether your code is doing what you are expecting while running on it
5. Start with a **reduced dataset**, or with a **little time window**, in order to still hold the control on the functionality of the system
6. Extend to the **entire problem**
7. Perform some **stress tests** in order to test the robustness of your model:
   1. **Little perturbations** on solutions found, in order to check the **sensitivity** and to show that the obtained solution is "good"
   2. Try **different weights** \(also **unbalanced**\) for your objective function
   3. Define some **tests with the client**, in order to keep him/her involved in the project and confident on the results
8. For the solution obtained, **check all the constraints** written \(**giving a name** to each constraint\), using **charts** on the solution optimized

Now, for each family of optimization project, we see a few examples relative to some of these steps.

#### Scheduling

\[KPI\] Once you have written the models and tried that on past data, verify that the number of product made per week/month is higher in the optimized solution than in the old scheduled production

\[Stress Test\] Try to perform a solution reducing the number of presses

\[Stress Test\] Perform a solution with the production time increased/reduced

\[Stress Test\] Perform a solution with the time of preparation of the presses reduced

\[Viz\] Gantt with the time on x axis, all the machineries on the y axis and the processing time \(or preparation time\) as coloured or grey blocks

#### Inventory Optimization

\[KPI\] Some kinds of KPIs for evaluating a good inventory optimization are: verify the level of storage is lower with the optimized entrance; verify the number of orders to suppliers are less than before the optimization; verify the costs of orders are lower.

\[Stress Test\] First, perform the model using the real exits from the store \(in these cases in must obtain a storage level lower than the historical\)

\[Stress Test\] Secondly, perform the model using the real exits from the store, modified with a random error, of the same magnitude of the Forecasting error

\[Stress Test\] Perform a solution where impose that all the articols in storage will expiry the day after they enter

\[Stress Test\] Perform a solution where impose the minimum orders to the supliers are 0

\[Stress Test\] Perform a solution where add some fake, really big orders

\[Viz\] Gantt with the time on the x axis and the storage \[units or Euros\] on y axis. And check that the safety is \(almost always\) respected and at the same time the level is lower than at the beginning, when the orders were not optimized 

#### Routing

\[Viz\] Gantt with the time on x axis, the vehicles on the y axis and the routing time \(or permanence time\) as coloured or grey blocks

