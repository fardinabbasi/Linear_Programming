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

This optimization problem is solved using the **Pyomo** package in **Python**, based on the following code.
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

## Q3: Dynamic Programming
The following image is a model of Tehran's popular places. The cost of transportation between each two places is shown on the arcs.
<img src="/readme_images/model.jpg">
The problem is to find the best route from an arbitrary starting point to an arbitrary destination point.

Utilizing the **PuLP** package in **Python**, the following class is written to solve the problem.
```python
class traveling_path_problem():
  def __init__(self, num_points, s, t, C):
    self.prob = LpProblem('prob', LpMinimize)
    self.N = [str(i) for i in range(num_points)]
    self.D = {node: 1 if node == s else -1 if node == t else 0 for node in self.N}
    self.E = [(i,j) for i in self.N for j in self.N if i in C.keys() if j in C[i].keys()]

  def solve(self):
    x = LpVariable.dicts('x', self.E,  lowBound = 0, upBound = 1, cat = LpInteger)
    self.prob += lpSum([C[i][j]*x[i,j] for (i,j) in self.E])

    for i in self.N:
        self.prob += (lpSum([x[i,j] for j in self.N if (i,j) in self.E])
                - lpSum([x[k,i] for k in self.N if (k,i) in self.E])) == self.D[i]

    status = self.prob.solve()
    print(f'STATUS\n{LpStatus[status]}\n')
    path = []

    for i in self.N:
        for j in self.N:
            if (i, j) in self.E and value(x[i, j]) == 1:
                path.append((i, j))

    total_cost = value(self.prob.objective)
    return path, total_cost

  def plotter(self, position):
    flow = [v.varValue*3 for v in self.prob.variables()]
    G = nx.Graph()
    G.add_nodes_from(self.N)
    G.add_edges_from(self.E)
    fig, ax = plt.subplots()
    nx.draw(G, pos=position, with_labels=True, ax=ax)
    lines = []
    for (i,j) in self.E:
        lines.append([(position[i][0], position[i][1]),(position[j][0], position[j][1])])
    lc = collections.LineCollection(lines, linewidth=flow, colors='r')
    ax.add_collection(lc)

    ax.set_xlabel("Longitude")
    ax.set_ylabel("Latitude")
    ax.set_title("Tehran Map")
    plt.axis('on')
    ax.tick_params(left=True, bottom=True, labelleft=True, labelbottom=True)
    plt.show()
```
Based on the following properties:
```python
C = {
    '0': {'1': 8.28, '2': 1.28, '6': 4.48},
    '1': {'0': 3.64, '5': 7.52, '7': 7.56},
    '2': {'0': 0.66},
    '3': {'9': 10},
    '4': {'5': 19.38, '8': 44, '9': 42},
    '5': {'1': 11.38, '4': 24.96, '7': 10.44},
    '6': {'0': 1.6, '7': 11.04},
    '7': {'1': 9.12, '5': 9.12, '6': 8.4},
    '8': {'4': 37.6, '9': 29.58},
    '9': {'3': 4.32, '4': 36.12, '8': 28.6}
}

num_points = 10
s = '1' #start
t  = '9' #terminal
```
the best path is shown in the following image.
```python
problem = traveling_path_problem(num_points, s, t, C)

path, total_cost = problem.solve()
```
```python
position = {'0': [35.702, 51.369], '1': [35.721, 51.388], '2': [35.701,51.39],'3': [35.784, 51.398], '4': [35.753, 51.426],
            '5': [35.742, 51.399], '6': [35.703, 51.41], '7': [35.728, 51.417], '8': [35.79, 51.457], '9': [35.777, 51.41]}
problem.plotter(position)
```

<img src="/readme_images/sol.png">

The best path is $1\rightarrow5\rightarrow4\rightarrow9$ with a **total cost** of 74.48.

## Course Project for Operational Research
- **Course**: Operational Research [ECE 145]
- **Semester**: Fall 2023
- **Institution:** [School of Electrical & Computer Engineering](https://ece.ut.ac.ir/en/), [College of Engineering](https://eng.ut.ac.ir/en), [University of Tehran](https://ut.ac.ir/en)
- **Instructor:** Dr. Ramezani Moghaddam
