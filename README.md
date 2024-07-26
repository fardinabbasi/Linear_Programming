# Dynamic Programming
## Q1: Linear Programming
The transportation and distribution of goods from the source to the destination is a problem that requires optimization algorithms to solve. In this context, the goal is to minimize transportation costs while meeting the constraints of available resources and the needs of each region.

Here, we illustrate the transportation problem using an example. In this example, there are two factories (`Arnhem`, `Gouda`) and six customer nodes located in six European cities, as shown in the map below. Customer nodes are marked with red labels, and factories are marked with blue labels.

<img src="/readme_images/factories.jpg">

Each city's demand (in tons) is specified as follows:
```python
factory = ["Arnhem", "Gouda"]
city = ["London", "Berlin", "Maastricht", "Amsterdam", "Utrecht", "The Hague"]
```
```python
demands = [125, 175, 225, 250, 225, 200]
```
The total supply that each factory can provide (in tons) is as follows:
```python
supply = [550, 700]
```
The transportation cost per ton (in euros) from each factory to each city is as follows. Transportation costs for city-factory pairs where transport is not possible are represented by a high value, such as 1e6.
```python
price = [
    [1e6, 2.5],
    [2.5, 1e6],
    [1.6, 2],
    [1.4, 1],
    [0.8, 1],
    [1.4, 0.8]
]
```
A graph illustrating the aforementioned details is shown below.

<img src="/readme_images/supply-demand.jpg">

The cost function to be minimized is written as follows.

$$
\text{minimize} \quad \sum_{c \in Cities} \sum_{s \in Factories} T[c,s] \cdot x[c,s]
$$

$$
\sum_{c \in Cities} x[c,s] \leq \text{Supply}[s] \quad \forall s \in Sources
$$

$$
\sum_{s \in Factories} x[c,s] = \text{Demand}[c] \quad \forall c \in Cities
$$

*where* $T[c,s]$ represents the transportation cost from factory $s$ to city $c$, and $x[c,s]$ denotes the amount of goods transported from from factory $s$ to city $c$.

This optimization problem is solved using the Pyomo package in Python, based on the following code.
```python
model = pyo.ConcreteModel()
model.x = pyo.Var(factories, cities, domain=pyo.NonNegativeReals)

model.cost = pyo.Objective(
    expr=sum(price[c][f] * model.x[f, c] for c in cities for f in factories),
    sense=pyo.minimize
)

model.demand_constraints = pyo.ConstraintList()
for c in cities:
    model.demand_constraints.add(
        sum(model.x[f, c] for f in factories) == demands[c]
    )

model.supply_constraints = pyo.ConstraintList()
for f in factories:
    model.supply_constraints.add(
        sum(model.x[f, c] for c in cities) <= supply[f]
    )
```
```python
solver = pyo.SolverFactory('glpk')
solver.solve(model)

print("Optimal Solution:")
for f in factories:
    for c in cities:
        print(f"{factory[f]} factory to {city[c]} City: {model.x[f, c]()}")

print("\nTotal Cost:", model.cost())
```
The **optimal solution** is written as follows.
| City/Factory | Arnhem | Gouda |
|---|---|---|
| London | 0 | 125 |
| Berlin | 175 | 0 |
| Maastricht | 225 | 0 |
| Amsterdam | 0 | 250 |
| Utrecht | 150 | 75 |
| The Hague | 0 | 200 |

with a **total cost** of 1715 euros.

If the total supply of each factory is considered as shown below:
```python
new_supply = [600, 650]
```
the **optimal solution** is as follows.
| City/Factory | Arnhem | Gouda |
|---|---|---|
| London | 0 | 125 |
| Berlin | 175 | 0 |
| Maastricht | 225 | 0 |
| Amsterdam | 0 | 250 |
| Utrecht | 200 | 25 |
| The Hague | 0 | 200 |

with a **total cost** of 1705 euros.
