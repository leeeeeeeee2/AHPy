# AHPy

**AHPy** 是分析层次过程（[AHP](https://en.wikipedia.org/wiki/Analytic_hierarchy_process)）的实现，这是一种用于结构化、综合化和评估决策问题元素的方法。AHP由[Thomas Saaty](http://www.creativedecisions.org/about/ThomasLSaaty.php)在20世纪70年代开发，它在操作研究以外的领域的广泛使用证明了其简单而强大的心理学和数学的结合。

 AHPy试图提供一个库，它不仅使用简单，而且能够直观地在AHP可以应用的众多概念框架内工作。出于这个原因，在编程界面中，一般的术语比更具体的术语更受欢迎。

#### Installing AHPy

AHPy is available on the Python Package Index ([PyPI](https://pypi.org/)):

```
python -m pip install ahpy
```

AHPy requires [Python 3.7+](https://www.python.org/), as well as [numpy](https://numpy.org/) and [scipy](https://scipy.org/).

## Table of Contents

#### Examples

[Relative consumption of drinks in the United States](#relative-consumption-of-drinks-in-the-united-states)

[Choosing a leader](#choosing-a-leader)

[Purchasing a vehicle](#purchasing-a-vehicle)

[Purchasing a vehicle reprised: normalized weights and the Compose class](#purchasing-a-vehicle-reprised-normalized-weights-and-the-compose-class)


#### Details

[The Compare Class](#the-compare-class)

[Compare.add_children()](#compareadd_children)

[Compare.report()](#comparereport)

[The Compose Class](#the-compose-class)

[Compose.add_comparisons()](#composeadd_comparisons)

[Compose.add_hierarchy()](#composeadd_hierarchy)

[Compose.report()](#composereport)

[A Note on Weights](#a-note-on-weights)

[Missing Pairwise Comparisons](#missing-pairwise-comparisons)

[Development and Testing](#development-and-testing)

---

## Examples

The easiest way to learn how to use AHPy is to *see* it used, so this README begins with worked examples of gradually increasing complexity.

### Relative consumption of drinks in the United States

在Saaty对AHP的阐述中，这个例子经常被用来作为AHP方法的一个简短而清晰的演示；正是这个例子让我第一次看到了AHP的广泛用途（以及众人的智慧！）。我在这里使用的版本来自他2008年的文章"[用分析层次结构过程进行决策](https://doi.org/10.1504/IJSSCI.2008.017590)".
如果你对这个例子不熟悉，30名参与者被要求比较美国饮料的相对消费。例如，他们认为咖啡的消费量比葡萄酒多得多，但与牛奶的消费量相同。从他们的答案中得出的矩阵如下。

||Coffee|Wine|Tea|Beer|Soda|Milk|Water|
|-|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|Coffee|1|9|5|2|1|1|1/2|
|Wine|1/9|1|1/3|1/9|1/9|1/9|1/9|
|Tea|1/5|3|1|1/3|1/4|1/3|1/9|
|Beer|1/2|9|3|1|1/2|1|1/3|
|Soda|1|9|4|2|1|2|1/2|
|Milk|1|9|3|1|1/2|1|1/3|
|Water|2|9|9|3|2|3|1|

下表显示了使用AHP计算出的饮料相对消费量，以及从美国统计摘要中获得的*实际*饮料相对消费量，给定的矩阵。

|:exploding_head:|Coffee|Wine|Tea|Beer|Soda|Milk|Water|
|-|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|AHP|0.177|0.019|0.042|0.116|0.190|0.129|0.327|
|Actual|0.180|0.010|0.040|0.120|0.180|0.140|0.330|

我们可以用下面的代码用AHPy重新创建这个分析。

```python
>>> drink_comparisons = {('coffee', 'wine'): 9, ('coffee', 'tea'): 5, ('coffee', 'beer'): 2, ('coffee', 'soda'): 1,
			 ('coffee', 'milk'): 1, ('coffee', 'water'): 1 / 2,
                         ('wine', 'tea'): 1 / 3, ('wine', 'beer'): 1 / 9, ('wine', 'soda'): 1 / 9,
                         ('wine', 'milk'): 1 / 9, ('wine', 'water'): 1 / 9,
                         ('tea', 'beer'): 1 / 3, ('tea', 'soda'): 1 / 4, ('tea', 'milk'): 1 / 3,
                         ('tea', 'water'): 1 / 9,
                         ('beer', 'soda'): 1 / 2, ('beer', 'milk'): 1, ('beer', 'water'): 1 / 3,
                         ('soda', 'milk'): 2, ('soda', 'water'): 1 / 2,
                         ('milk', 'water'): 1 / 3}

>>> drinks = ahpy.Compare(name='Drinks', comparisons=drink_comparisons, precision=3, random_index='saaty')

>>> print(drinks.target_weights)
{'water': 0.327, 'soda': 0.19, 'coffee': 0.177, 'milk': 0.129, 'beer': 0.116, 'tea': 0.042, 'wine': 0.019}

>>> print(drinks.consistency_ratio)
0.022
```

1. 首先，我们用上面矩阵中的值创建一个配对比较的字典。
2. 然后我们创建一个**Compare**对象，用一个唯一的名字和刚才的字典来初始化它。我们还改变精度和随机索引，使结果与Saaty提供的结果一致。
3. 最后，我们打印比较对象的目标权重和一致性比率，看看我们的分析结果。

Brilliant!

### Choosing a leader

这个例子可以在[维基百科的AHP条目的附录中]找到(https://en.wikipedia.org/wiki/Analytic_hierarchy_process_-_leader_example)。为了向[原来的说法](https://www.grammarphobia.com/blog/2009/06/tom-dick-and-harry-part-2.html)表示敬意，名称已经改变，但输入的比较值保持不变。

#### N.B.

你可能会注意到，在某些情况下，AHPy的结果会与维基百科页面上的结果不一致。这不是AHPy的计算错误，而是[用于计算维基百科例子中显示的数值的方法](https://en.wikipedia.org/wiki/Analytic_hierarchy_process_-_car_example#Pairwise_comparing_the_criteria_with_respect_to_the_goal)的结果。

> You can duplicate this analysis at this online demonstration site...**IMPORTANT: The demo site is designed for convenience, not accuracy. The priorities it returns may differ somewhat from those returned by rigorous AHP calculations.**

In this example, we'll be judging job candidates by their experience, education, charisma and age. Therefore, we need to compare each potential leader to the others, given each criterion...

```python
>>> experience_comparisons = {('Moll', 'Nell'): 1/4, ('Moll', 'Sue'): 4, ('Nell', 'Sue'): 9}
>>> education_comparisons = {('Moll', 'Nell'): 3, ('Moll', 'Sue'): 1/5, ('Nell', 'Sue'): 1/7}
>>> charisma_comparisons = {('Moll', 'Nell'): 5, ('Moll', 'Sue'): 9, ('Nell', 'Sue'): 4}
>>> age_comparisons = {('Moll', 'Nell'): 1/3, ('Moll', 'Sue'): 5, ('Nell', 'Sue'): 9}
```

...as well as compare the importance of each criterion to the others:

```python
>>> criteria_comparisons = {('Experience', 'Education'): 4, ('Experience', 'Charisma'): 3, ('Experience', 'Age'): 7,
			    ('Education', 'Charisma'): 1/3, ('Education', 'Age'): 3,
			    ('Charisma', 'Age'): 5}
```

在继续之前，重要的是要注意，构成字典键的元素的*顺序是有意义的。例如，使用Saaty的尺度，比较"（'经验'，'教育'）：4 "意味着 "经验比*教育更*重要"。

现在我们已经创建了所有必要的成对比较字典，我们将创建它们相应的比较对象，并使用字典作为输入。

```python
>>> experience = ahpy.Compare('Experience', experience_comparisons, precision=3, random_index='saaty')
>>> education = ahpy.Compare('Education', education_comparisons, precision=3, random_index='saaty')
>>> charisma = ahpy.Compare('Charisma', charisma_comparisons, precision=3, random_index='saaty')
>>> age = ahpy.Compare('Age', age_comparisons, precision=3, random_index='saaty')
>>> criteria = ahpy.Compare('Criteria', criteria_comparisons, precision=3, random_index='saaty')
```

请注意，在上面的`criteria_comparisons`字典中，经验、教育、魅力和年龄对象的名称是重复的。这是必要的，以便正确地将比较对象连接成一个层次结构，如下所示。

In the final step, we need to link the Compare objects together into a hierarchy, such that Criteria is the *parent* object and the other objects form its *children*:

```python
>>> criteria.add_children([experience, education, charisma, age])
```

Now that the hierarchy represents the decision problem, we can print the target weights of the parent Criteria object to see the results of the analysis:

```python
>>> print(criteria.target_weights)
{'Nell': 0.493, 'Moll': 0.358, 'Sue': 0.15}
```

我们还可以打印任何其他比较对象中的元素的局部和全局权重，以及它们的比较的一致性比率。

```python
>>> print(experience.local_weights)
{'Nell': 0.717, 'Moll': 0.217, 'Sue': 0.066}

>>> print(experience.consistency_ratio)
0.035

>>> print(education.global_weights)
{'Sue': 0.093, 'Moll': 0.024, 'Nell': 0.01}

>>> print(education.consistency_ratio)
0.062
```

The global and local weights of the Compare objects themselves are likewise available:

```python
>>> print(experience.global_weight)
0.548

>>> print(education.local_weight)
0.127
```

在一个比较对象上调用`report()`提供了一个标准的方法来了解该对象的信息。在下面的代码中，变量`report`包含一个[Python字典](#comparereport)的重要信息，而`show=True`参数则以JSON格式将同样的信息打印到控制台。

```python
>>> report = criteria.report(show=True)
{
    "Criteria": {
        "global_weight": 1.0,
        "local_weight": 1.0,
        "target_weights": {
            "Nell": 0.493,
            "Moll": 0.358,
            "Sue": 0.15
        },
        "elements": {
            "global_weights": {
                "Experience": 0.548,
                "Charisma": 0.27,
                "Education": 0.127,
                "Age": 0.056
            },
            "local_weights": {
                "Experience": 0.548,
                "Charisma": 0.27,
                "Education": 0.127,
                "Age": 0.056
            },
            "consistency_ratio": 0.044
        }
    }
}
```

### Purchasing a vehicle

这个例子也可以在[维基百科的AHP条目的附录中找到](https://en.wikipedia.org/wiki/Analytic_hierarchy_process_-_car_example)。像以前一样，在某些情况下，AHPy的结果会与维基百科上的结果不一致，即使输入的比较值是相同的。重申一下，这是由于方法的不同，而不是AHPy的错误。

在这个例子中，我们将根据成本、安全、风格和容量来选择要购买的车辆。成本将进一步取决于车辆的购买价格、燃料成本、维护成本和转售价值的组合；容量将取决于车辆的货物和乘客容量的组合。

首先，我们将高层次的标准相互比较。

```python
>>> criteria_comparisons = {('Cost', 'Safety'): 3, ('Cost', 'Style'): 7, ('Cost', 'Capacity'): 3,
			    ('Safety', 'Style'): 9, ('Safety', 'Capacity'): 1,
			    ('Style', 'Capacity'): 1/7}
```

如果我们为标准创建一个比较对象，我们可以查看其报告。

```python
>>> criteria = ahpy.Compare('Criteria', criteria_comparisons, precision=3)
>>> report = criteria.report(show=True)
{
    "Criteria": {
        "global_weight": 1.0,
        "local_weight": 1.0,
        "target_weights": {
            "Cost": 0.51,
            "Safety": 0.234,
            "Capacity": 0.215,
            "Style": 0.041
        },
        "elements": {
            "global_weights": {
                "Cost": 0.51,
                "Safety": 0.234,
                "Capacity": 0.215,
                "Style": 0.041
            },
            "local_weights": {
                "Cost": 0.51,
                "Safety": 0.234,
                "Capacity": 0.215,
                "Style": 0.041
            },
            "consistency_ratio": 0.08
        }
    }
}
```

Next, we compare the *sub*criteria of Cost to one another...

```python
>>> cost_comparisons = {('Price', 'Fuel'): 2, ('Price', 'Maintenance'): 5, ('Price', 'Resale'): 3,
			('Fuel', 'Maintenance'): 2, ('Fuel', 'Resale'): 2,
			('Maintenance', 'Resale'): 1/2}
```

...as well as the subcriteria of Capacity:

```python
>>> capacity_comparisons = {('Cargo', 'Passenger'): 1/5}
```

我们还需要在每个标准下，将每个潜在的车辆与其他车辆进行比较。我们将首先建立一个所有可能的两车组合的列表。

```python
>>> import itertools
>>> vehicles = ('Accord Sedan', 'Accord Hybrid', 'Pilot', 'CR-V', 'Element', 'Odyssey')
>>> vehicle_pairs = list(itertools.combinations(vehicles, 2))
>>> print(vehicle_pairs)
[('Accord Sedan', 'Accord Hybrid'), ('Accord Sedan', 'Pilot'), ('Accord Sedan', 'CR-V'), ('Accord Sedan', 'Element'), ('Accord Sedan', 'Odyssey'), ('Accord Hybrid', 'Pilot'), ('Accord Hybrid', 'CR-V'), ('Accord Hybrid', 'Element'), ('Accord Hybrid', 'Odyssey'), ('Pilot', 'CR-V'), ('Pilot', 'Element'), ('Pilot', 'Odyssey'), ('CR-V', 'Element'), ('CR-V', 'Odyssey'), ('Element', 'Odyssey')]
```

然后，我们可以简单地将车辆对和它们对每个标准的比较值拉在一起。

```python
>>> price_values = (9, 9, 1, 1/2, 5, 1, 1/9, 1/9, 1/7, 1/9, 1/9, 1/7, 1/2, 5, 6)
>>> price_comparisons = dict(zip(vehicle_pairs, price_values))
>>> print(price_comparisons)
{('Accord Sedan', 'Accord Hybrid'): 9, ('Accord Sedan', 'Pilot'): 9, ('Accord Sedan', 'CR-V'): 1, ('Accord Sedan', 'Element'): 0.5, ('Accord Sedan', 'Odyssey'): 5, ('Accord Hybrid', 'Pilot'): 1, ('Accord Hybrid', 'CR-V'): 0.1111111111111111, ('Accord Hybrid', 'Element'): 0.1111111111111111, ('Accord Hybrid', 'Odyssey'): 0.14285714285714285, ('Pilot', 'CR-V'): 0.1111111111111111, ('Pilot', 'Element'): 0.1111111111111111, ('Pilot', 'Odyssey'): 0.14285714285714285, ('CR-V', 'Element'): 0.5, ('CR-V', 'Odyssey'): 5, ('Element', 'Odyssey'): 6}

>>> safety_values = (1, 5, 7, 9, 1/3, 5, 7, 9, 1/3, 2, 9, 1/8, 2, 1/8, 1/9)
>>> safety_comparisons = dict(zip(vehicle_pairs, safety_values))

>>> passenger_values = (1, 1/2, 1, 3, 1/2, 1/2, 1, 3, 1/2, 2, 6, 1, 3, 1/2, 1/6)
>>> passenger_comparisons = dict(zip(vehicle_pairs, passenger_values))

>>> fuel_values = (1/1.13, 1.41, 1.15, 1.24, 1.19, 1.59, 1.3, 1.4, 1.35, 1/1.23, 1/1.14, 1/1.18, 1.08, 1.04, 1/1.04)
>>> fuel_comparisons = dict(zip(vehicle_pairs, fuel_values))

>>> resale_values = (3, 4, 1/2, 2, 2, 2, 1/5, 1, 1, 1/6, 1/2, 1/2, 4, 4, 1)
>>> resale_comparisons = dict(zip(vehicle_pairs, resale_values))

>>> maintenance_values = (1.5, 4, 4, 4, 5, 4, 4, 4, 5, 1, 1.2, 1, 1, 3, 2)
>>> maintenance_comparisons = dict(zip(vehicle_pairs, maintenance_values))

>>> style_values = (1, 7, 5, 9, 6, 7, 5, 9, 6, 1/6, 3, 1/3, 7, 5, 1/5)
>>> style_comparisons = dict(zip(vehicle_pairs, style_values))

>>> cargo_values = (1, 1/2, 1/2, 1/2, 1/3, 1/2, 1/2, 1/2, 1/3, 1, 1, 1/2, 1, 1/2, 1/2)
>>> cargo_comparisons = dict(zip(vehicle_pairs, cargo_values))
```

Now that we've created all of the necessary pairwise comparison dictionaries, we can create their corresponding Compare objects:

```python
>>> cost = ahpy.Compare('Cost', cost_comparisons, precision=3)
>>> capacity = ahpy.Compare('Capacity', capacity_comparisons, precision=3)
>>> price = ahpy.Compare('Price', price_comparisons, precision=3)
>>> safety = ahpy.Compare('Safety', safety_comparisons, precision=3)
>>> passenger = ahpy.Compare('Passenger', passenger_comparisons, precision=3)
>>> fuel = ahpy.Compare('Fuel', fuel_comparisons, precision=3)
>>> resale = ahpy.Compare('Resale', resale_comparisons, precision=3)
>>> maintenance = ahpy.Compare('Maintenance', maintenance_comparisons, precision=3)
>>> style = ahpy.Compare('Style', style_comparisons, precision=3)
>>> cargo = ahpy.Compare('Cargo', cargo_comparisons, precision=3)
```

最后一步是将所有的比较对象连接成一个层次结构。首先，我们将使价格、燃料、维修和转售对象成为成本对象的子对象......

```python
>>> cost.add_children([price, fuel, maintenance, resale])
```

...and do the same to link the Cargo and Passenger objects to the Capacity object...

```python
>>> capacity.add_children([cargo, passenger])
```

...then finally make the Cost, Safety, Style and Capacity objects the children of the Criteria object:

```python
>>> criteria.add_children([cost, safety, style, capacity])
```

现在，层次结构代表了决策问题，我们可以打印*最高级别*标准对象的目标权重，以查看分析结果。

```python
>>> print(criteria.target_weights)
{'Odyssey': 0.219, 'Accord Sedan': 0.215, 'CR-V': 0.167, 'Accord Hybrid': 0.15, 'Element': 0.144, 'Pilot': 0.106}
```

对于层次结构中任何一个比较对象的详细信息，我们可以调用该对象的`report()`，参数为`verbose=True`。

```python
>>> report = criteria.report(show=True, verbose=True)
{
    "name": "Criteria",
    "global_weight": 1.0,
    "local_weight": 1.0,
    "target_weights": {
        "Odyssey": 0.219,
        "Accord Sedan": 0.215,
        "CR-V": 0.167,
        "Accord Hybrid": 0.15,
        "Element": 0.144,
        "Pilot": 0.106
    },
    "elements": {
        "global_weights": {
            "Cost": 0.51,
            "Safety": 0.234,
            "Capacity": 0.215,
            "Style": 0.041
        },
        "local_weights": {
            "Cost": 0.51,
            "Safety": 0.234,
            "Capacity": 0.215,
            "Style": 0.041
        },
        "consistency_ratio": 0.08,
        "random_index": "Donegan & Dodd",
        "count": 4,
        "names": [
            "Cost",
            "Safety",
            "Style",
            "Capacity"
        ]
    },
    "children": {
        "count": 4,
        "names": [
            "Cost",
            "Safety",
            "Style",
            "Capacity"
        ]
    },
    "comparisons": {
        "count": 6,
        "input": {
            "Cost, Safety": 3,
            "Cost, Style": 7,
            "Cost, Capacity": 3,
            "Safety, Style": 9,
            "Safety, Capacity": 1,
            "Style, Capacity": 0.14285714285714285
        },
        "computed": null
    }
}
```

在层次结构较低的比较对象上调用`report(show=True, verbose=True)`会提供不同的信息，这取决于它们所处的层次。

```python
>>> report = cost.report(show=True, verbose=True)
{
    "name": "Cost",
    "global_weight": 0.51,
    "local_weight": 0.51,
    "target_weights": null,
    "elements": {
        "global_weights": {
            "Price": 0.249,
            "Fuel": 0.129,
            "Resale": 0.082,
            "Maintenance": 0.051
        },
        "local_weights": {
            "Price": 0.488,
            "Fuel": 0.252,
            "Resale": 0.161,
            "Maintenance": 0.1
        },
        "consistency_ratio": 0.016,
        "random_index": "Donegan & Dodd",
        "count": 4,
        "names": [
            "Price",
            "Fuel",
            "Maintenance",
            "Resale"
        ]
    },
    "children": {
        "count": 4,
        "names": [
            "Price",
            "Fuel",
            "Resale",
            "Maintenance"
        ]
    },
    "comparisons": {
        "count": 6,
        "input": {
            "Price, Fuel": 2,
            "Price, Maintenance": 5,
            "Price, Resale": 3,
            "Fuel, Maintenance": 2,
            "Fuel, Resale": 2,
            "Maintenance, Resale": 0.5
        },
        "computed": null
    }
}

>>> report = price.report(show=True, verbose=True)
{
    "name": "Price",
    "global_weight": 0.249,
    "local_weight": 0.488,
    "target_weights": null,
    "elements": {
        "global_weights": {
            "Element": 0.091,
            "Accord Sedan": 0.061,
            "CR-V": 0.061,
            "Odyssey": 0.023,
            "Accord Hybrid": 0.006,
            "Pilot": 0.006
        },
        "local_weights": {
            "Element": 0.366,
            "Accord Sedan": 0.246,
            "CR-V": 0.246,
            "Odyssey": 0.093,
            "Accord Hybrid": 0.025,
            "Pilot": 0.025
        },
        "consistency_ratio": 0.072,
        "random_index": "Donegan & Dodd",
        "count": 6,
        "names": [
            "Accord Sedan",
            "Accord Hybrid",
            "Pilot",
            "CR-V",
            "Element",
            "Odyssey"
        ]
    },
    "children": null,
    "comparisons": {
        "count": 15,
        "input": {
            "Accord Sedan, Accord Hybrid": 9,
            "Accord Sedan, Pilot": 9,
            "Accord Sedan, CR-V": 1,
            "Accord Sedan, Element": 0.5,
            "Accord Sedan, Odyssey": 5,
            "Accord Hybrid, Pilot": 1,
            "Accord Hybrid, CR-V": 0.1111111111111111,
            "Accord Hybrid, Element": 0.1111111111111111,
            "Accord Hybrid, Odyssey": 0.14285714285714285,
            "Pilot, CR-V": 0.1111111111111111,
            "Pilot, Element": 0.1111111111111111,
            "Pilot, Odyssey": 0.14285714285714285,
            "CR-V, Element": 0.5,
            "CR-V, Odyssey": 5,
            "Element, Odyssey": 6
        },
        "computed": null
    }
}
```

最后，在层次结构中的任何一个比较对象上调用`report(complete=True)`将返回一个字典，其中包含层次结构中*每个*比较对象的报告，字典的键是比较对象的名称。

```python
>>> complete_report = cargo.report(complete=True)

>>> print([key for key in complete_report])
['Criteria', 'Cost', 'Price', 'Fuel', 'Maintenance', 'Resale', 'Safety', 'Style', 'Capacity', 'Cargo', 'Passenger']

>>> print(complete_report['Cargo'])
{'name': 'Cargo', 'global_weight': 0.0358, 'local_weight': 0.1667, 'target_weights': None, 'elements': {'global_weights': {'Odyssey': 0.011, 'Pilot': 0.006, 'CR-V': 0.006, 'Element': 0.006, 'Accord Sedan': 0.003, 'Accord Hybrid': 0.003}, 'local_weights': {'Odyssey': 0.311, 'Pilot': 0.17, 'CR-V': 0.17, 'Element': 0.17, 'Accord Sedan': 0.089, 'Accord Hybrid': 0.089}, 'consistency_ratio': 0.002}}

>>> print(complete_report['Criteria']['target_weights'])
{'Odyssey': 0.219, 'Accord Sedan': 0.215, 'CR-V': 0.167, 'Accord Hybrid': 0.15, 'Element': 0.144, 'Pilot': 0.106}
```

Calling `report(complete=True, verbose=True)` will return a similar dictionary, but with the detailed version of the reports.

```python
>>> complete_report = style.report(complete=True, verbose=True)

>>> print(complete_report['Price']['comparisons']['count'])
15
```

We could also print all of the reports to the console with the `show=True` argument.

### Purchasing a vehicle reprised: normalized weights and the Compose class

在读完[维基百科上的车辆决策问题](https://en.wikipedia.org/wiki/Analytic_hierarchy_process_-_car_example)的解释后，你可能会想，用于表示纯数字标准（如乘客容量）的数据是否可以在相互比较车辆时*直接使用，而不是要求转变成 "强度 "的判断。在这个例子中，我们将解决与之前相同的决策问题，只是现在我们将对乘客容量、燃料成本、转售价值和货物容量的测量值进行标准化处理，以便为这些标准得出一套不同的权重。

我们还将使用一个**组合**对象来构建决策问题。Compose对象允许我们使用问题层次的抽象表示，而不是用代码动态地建立它，当我们不在交互式环境中使用AHPy时，这很有价值。要使用Compose对象，我们需要首先添加比较信息，然后是层次结构，*依次为*。但后面会有更多关于这个问题的内容。

使用前面例子中的车辆列表，我们首先将车辆和它们的测量值压缩在一起，然后为每个规范化的标准创建一个比较对象。


```python
>>> passenger_measured_values = (5, 5, 8, 5, 4, 8)
>>> passenger_data = dict(zip(vehicles, passenger_measured_values))
>>> print(passenger_data)
{'Accord Sedan': 5, 'Accord Hybrid': 5, 'Pilot': 8, 'CR-V': 5, 'Element': 4, 'Odyssey': 8}

>>> passenger_normalized = ahp.Compare('Passenger', passenger_data, precision=3)

>>> fuel_measured_values = (31, 35, 22, 27, 25, 26)
>>> fuel_data = dict(zip(vehicles, fuel_measured_values))
>>> fuel_normalized = ahp.Compare('Fuel', fuel_data, precision=3)

>>> resale_measured_values = (0.52, 0.46, 0.44, 0.55, 0.48, 0.48)
>>> resale_data = dict(zip(vehicles, resale_measured_values))
>>> resale_normalized = ahp.Compare('Resale', resale_data, precision=3)

>>> cargo_measured_values = (14, 14, 87.6, 72.9, 74.6, 147.4)
>>> cargo_data = dict(zip(vehicles, cargo_measured_values))
>>> cargo_normalized = ahp.Compare('Cargo', cargo_data, precision=3)
```

让我们打印一下新的乘客对象的归一化局部权重，以便与前面例子中的局部权重进行比较。

```python
>>> print(passenger_normalized.local_weights)
{'Pilot': 0.229, 'Odyssey': 0.229, 'Accord Sedan': 0.143, 'Accord Hybrid': 0.143, 'CR-V': 0.143, 'Element': 0.114}

>>> print(passenger.local_weights)
{'Accord Sedan': 0.493, 'Accord Hybrid': 0.197, 'Odyssey': 0.113, 'Element': 0.091, 'CR-V': 0.057, 'Pilot': 0.049}
```

当我们直接使用测量值时，我们看到车辆的排名与之前不同。然而，这是否会影响目标变量的*合成*排名，还有待观察。

我们接下来创建一个组合对象并开始添加比较信息。

```python
>>> compose = ahpy.Compose()

>>> compose.add_comparisons([passenger_normalized, fuel_normalized, resale_normalized, cargo_normalized])
```

我们可以通过几种不同的方式将比较信息添加到Compose对象中。如上所示，我们可以提供一个比较对象的列表；我们也可以一次提供一个，或者存储在一个元组中。使用我们前面例子中的比较对象。

```python
>>> compose.add_comparisons(cost)

>>> compose.add_comparisons((safety, style, capacity))
```

我们甚至可以把Compose对象当作Compare对象，直接添加数据。再次使用前面例子中的代码。

```python
>>> compose.add_comparisons('Price', price_comparisons, precision=3)
```

Finally, we can provide an ordered list or tuple containing the data needed to construct a Compare object:

```python
>>> comparisons = [('Maintenance', maintenance_comparisons, 3), ('Criteria', criteria_comparisons)]
>>> compose.add_comparisons(comparisons)
```

现在，所有的比较信息都已被添加，我们接下来需要创建层次结构并将其添加到组合对象中。层次结构是一个简单的字典，其中的键是*父*比较对象的名称，值是其*子*的名称列表。

```python
>>> hierarchy = {'Criteria': ['Cost', 'Safety', 'Style', 'Capacity'],
		 'Cost': ['Price', 'Fuel', 'Resale', 'Maintenance'],
		 'Capacity': ['Passenger', 'Cargo']}

>>> compose.add_hierarchy(hierarchy)
```

完成这两个步骤后，我们现在可以查看分析的综合结果。

我们查看合成对象的报告的方式与查看比较对象的方式相同。唯一不同的是，合成对象默认显示一个完整的报告；为了查看层次结构中单个比较对象的报告，我们需要指定其名称。

```python
>>> criteria_report = compose.report('Criteria', show=True)
{
    "name": "Criteria",
    "global_weight": 1.0,
    "local_weight": 1.0,
    "target_weights": {
        "Odyssey": 0.218,
        "Accord Sedan": 0.21,
        "Element": 0.161,
        "Accord Hybrid": 0.154,
        "CR-V": 0.149,
        "Pilot": 0.108
    },
    "elements": {
        "global_weights": {
            "Cost": 0.51,
            "Safety": 0.234,
            "Capacity": 0.215,
            "Style": 0.041
        },
        "local_weights": {
            "Cost": 0.51,
            "Safety": 0.234,
            "Capacity": 0.215,
            "Style": 0.041
        },
        "consistency_ratio": 0.08
    }
}
```

我们可以使用点或括号符号来访问我们添加到Compose对象中的比较信息的公共属性。

```python
>>> print(compose.Criteria.target_weights)
{'Odyssey': 0.218, 'Accord Sedan': 0.21, 'Element': 0.161, 'Accord Hybrid': 0.154, 'CR-V': 0.149, 'Pilot': 0.108}

>>> print(compose['Resale']['local_weights'])
{'CR-V': 0.188, 'Accord Sedan': 0.177, 'Element': 0.164, 'Odyssey': 0.164, 'Accord Hybrid': 0.157, 'Pilot': 0.15}
```

我们可以看到，将数字标准归一化会导致一组略有不同的目标权重，尽管奥德赛和雅阁轿车仍然是考虑购买的前两款车。

## Details

Keep reading to learn the details of the AHPy library's API...

### The Compare Class

The Compare class computes the weights and consistency ratio of a positive reciprocal matrix, created using an input dictionary of pairwise comparison values. Optimal values are computed for any [missing pairwise comparisons](#missing-pairwise-comparisons). Compare objects can also be [linked together to form a hierarchy](#compareadd_children) representing the decision problem: the target weights of the problem elements are then derived by synthesizing all levels of the hierarchy.

`Compare(name, comparisons, precision=4, random_index='dd', iterations=100, tolerance=0.0001, cr=True)`

`name`: *str (required)*, the name of the Compare object
- This property is used to link a child object to its parent and must be unique

`comparisons`: *dict (required)*, the elements and values to be compared, provided in one of two forms:

1. A dictionary of pairwise comparisons, in which each key is a tuple of two elements and each value is their pairwise comparison value
    - `{('a', 'b'): 3, ('b', 'c'): 2, ('a', 'c'): 5}`
    - **The order of the elements in the key matters: the comparison `('a', 'b'): 3` means "a is moderately more important than b"**

2. A dictionary of measured values, in which each key is a single element and each value is that element's measured value
    - `{'a': 1.2, 'b': 2.3, 'c': 3.4}`
    - Given this form, AHPy will automatically create consistent, normalized target weights

`precision`: *int*, the number of decimal places to take into account when computing both the target weights and the consistency ratio of the Compare object
- The default precision value is 4

`random_index`: *'dd'* or *'saaty'*, the set of random index estimates used to compute the Compare object's consistency ratio
- 'dd' supports the computation of consistency ratios for matrices less than or equal to 100 &times; 100 in size and uses estimates from:

  >Donegan, H.A. and Dodd, F.J., 'A Note on Saaty's Random Indexes,' *Mathematical and Computer Modelling*, 15:10, 1991, pp. 135-137 (DOI: [10.1016/0895-7177(91)90098-R](https://doi.org/10.1016/0895-7177(91)90098-R))
- 'saaty' supports the computation of consistency ratios for matrices less than or equal to 15 &times; 15 in size and uses estimates from:

  >Saaty, T., *Theory And Applications Of The Analytic Network Process*, Pittsburgh: RWS Publications, 2005, p. 31
- The default random index is 'dd'

`iterations`: *int*, the stopping criterion for the algorithm used to compute the Compare object's target weights
- If target weights have not been determined after this number of iterations, the algorithm stops and the last principal eigenvector to be computed is used as the target weights
- The default number of iterations is 100

`tolerance`: *float*, the stopping criterion for the cycling coordinates algorithm used to compute the optimal value of missing pairwise comparisons
- The algorithm stops when the difference between the norms of two cycles of coordinates is less than this value
- The default tolerance value is 0.0001

`cr`: *bool*, whether to compute the target weights' consistency ratio
- Set `cr=False` to compute the target weights of a matrix when a consistency ratio cannot be determined due to the size of the matrix
- The default value is True

The properties used to initialize the Compare class are intended to be accessed directly, along with a few others:

`Compare.global_weight`: *float*, the global weight of the Compare object within the hierarchy

`Compare.local_weight`: *float*, the local weight of the Compare object within the hierarchy

`Compare.global_weights`: *dict*, the global weights of the Compare object's elements; each key is an element and each value is that element's computed global weight
- `{'a': 0.25, 'b': 0.25}`

`Compare.local_weights`: *dict*, the local weights of the Compare object's elements; each key is an element and each value is that element's computed local weight
- `{'a': 0.5, 'b': 0.5}`

`Compare.target_weights`: *dict*, the target weights of the elements in the lowest level of the hierarchy; each key is an element and each value is that element's computed target weight; *if the global weight of the Compare object is less than 1.0, the value will be `None`*
- `{'a': 0.5, 'b': 0.5}`

`Compare.consistency_ratio`: *float*, the consistency ratio of the Compare object's pairwise comparisons

### Compare.add_children()

Compare objects can be linked together to form a hierarchy representing the decision problem. To link Compare objects together into a hierarchy, call `add_children()` on the Compare object intended to form the *upper* level (the *parent*) and include as an argument a list or tuple of one or more Compare objects intended to form its *lower* level (the *children*).

**In order to properly synthesize the levels of the hierarchy, the `name` of each child object MUST appear as an element in its parent object's input `comparisons` dictionary.**

`Compare.add_children(children)`

`children`: *list* or *tuple (required)*, the Compare objects that will form the lower level of the current Compare object

```python
>>> child1 = ahpy.Compare(name='child1', ...)
>>> child2 = ahpy.Compare(name='child2', ...)

>>> parent = ahpy.Compare(name='parent', comparisons={('child1', 'child2'): 5})
>>> parent.add_children([child1, child2])
```

The precision of the target weights is updated as the hierarchy is constructed: each time `add_children()` is called, the precision of the target weights is set to equal that of the Compare object with the lowest precision in the hierarchy. Because lower precision propagates up through the hierarchy, *the target weights will always have the same level of precision as the hierarchy's least precise Compare object*. This also means that it is possible for the precision of a Compare object's target weights to be different from the precision of its local and global weights.

### Compare.report()

A standard report on the details of a Compare object is available. To return the report as a dictionary, call `report()` on the Compare object; to simultaneously print the information to the console in JSON format, set `show=True`. The report is available in two levels of detail; to return the most detailed report, set `verbose=True`.

`Compare.report(complete=False, show=False, verbose=False)`

`complete`: *bool*, whether to return a report for every Compare object in the hierarchy
- This returns a dictionary of reports, with the keys of the dictionary being the names of the Compare objects
  - `{'a': {'name': 'a', ...}, 'b': {'name': 'b', ...}}`
- The default value is False

`show`: *bool*, whether to print the report to the console in JSON format
  - The default value is False

`verbose`: *bool*, whether to include full details of the Compare object in the report
  - The default value is False

The keys of the report take the following form:

`name`: *str*, the name of the Compare object

`global_weight`: *float*, the global weight of the Compare object within the hierarchy

`local_weight`: *float*, the local weight of the Compare object within the hierarchy

`target_weights`: *dict*, the target weights of the elements in the lowest level of the hierarchy; each key is an element and each value is that element's computed target weight
- `{'a': 0.5, 'b': 0.5}`
- *If the global weight of the Compare object is less than 1.0, the value will be `None`*

`elements`: *dict*, information regarding the elements compared by the Compare object
- `global_weights`: *dict*, the global weights of the Compare object's elements; each key is an element and each value is that element's computed global weight
  - `{'a': 0.25, 'b': 0.25}`
- `local_weights`: *dict*, the local weights of the Compare object's elements; each key is an element and each value is that element's computed local weight
  - `{'a': 0.5, 'b': 0.5}`
- `consistency_ratio`: *float*, the consistency ratio of the Compare object's pairwise comparisons

The remaining dictionary keys are only displayed when `verbose=True`:

- `random_index`: *'Donegan & Dodd' or 'Saaty'*, the random index used to compute the consistency ratio
- `count`: *int*, the number of elements compared by the Compare object
- `names`: *list*, the names of the elements compared by the Compare object

`children`: *dict*, the children of the Compare object
- `count`: *int*, the number of the Compare object's children
- `names`: *list*, the names of the Compare object's children
- If the Compare object has no children, the value will be `None`

`comparisons`: *dict*, the comparisons of the Compare object
- `count`: *int*, the number of comparisons made by the Compare object, *not counting reciprocal comparisons*
- `input`: *dict*, the comparisons input to the Compare object; this is identical to the input `comparisons` dictionary
- `computed`: *dict*, the comparisons computed by the Compare object; each key is a tuple of two elements and each value is their computed pairwise comparison value
  - `{('c', 'd'): 0.730297106886979}, ...}`
- If the Compare object has no computed comparisons, the value will be `None`

### The Compose Class

The Compose class can store and structure all of the information making up a decision problem. After first [adding comparison information](#composeadd_comparisons) to the object, then [adding the problem hierarchy](#composeadd_hierarchy), the analysis results of the multiple different Compare objects can be accessed through the single Compose object.

`Compose()`
 
After adding all necessary information, the public properties of any stored Compare object can be accessed directly through the Compose object using either dot or bracket notation:

```python
>>> my_compose_object.a.global_weights

>>> my_compose_object['a']['global_weights']
```

### Compose.add_comparisons()

The comparison information of a decision problem can be added to a Compose object in any of the several ways listed below. Always add comparison information *before* adding the problem hierarchy.

`Compose.add_comparisons(item, comparisons=None, precision=4, random_index='dd', iterations=100, tolerance=0.0001, cr=True)`

`item`: *Compare object, list or tuple, or string (required)*, this argument allows for multiple input types:

1. A single Compare object
    - `Compare('a', comparisons=a, ...)`

2. A list or tuple of Compare objects
    - `[Compare('a', ...), Compare('b', ...)]`
  
3. The data necessary to create a Compare object
    - `'a', comparisons=a, precision=3, ...`
    - The method signature mimics that of the Compare class for this reason

4. A nested list or tuple of the data necessary to create a Compare object
    - `(('a', a, 3, ...), ('b', b, 3, ...))`

All other arguments are identical to those of the [Compare class](#the-compare-class).

### Compose.add_hierarchy()

The Compose class uses an abstract representation of the problem hierarchy to automatically link its Compare objects together. When a hierarchy is added, the elements of the decision problem are synthesized and the analysis results are immediately available for use or viewing.

**`Compose.add_hierarchy()` should only be called AFTER all comparison information has been added to the Compose object.**

`Compose.add_hierarchy(hierarchy)`

`hierarchy`: *dict*, a representation of the hierarchy as a dictionary, in which the keys are the names of parent Compare objects and the values are lists of the names of their children
- `{'a': ['b', 'c'], 'b': ['d', 'e']}`

### Compose.report()

The standard report available for a Compare object can be accessed through the Compose object. Calling `report()` on a Compose object is equivalent to calling `report(complete=True)` on a Compare object and will return a dictionary of all the reports within the hierarchy; calling `report(name='a')` on a Compose object is equivalent to calling `a.report()` on the named Compare object.

`Compose.report(name=None, show=False, verbose=False)`

`name`: *str*, the name of a Compare object report to return; if None, returns a dictionary of reports, with the keys of the dictionary being the names of the Compare objects in the hierarchy
- `{'a': {'name': 'a', ...}, 'b': {'name': 'b', ...}}`
- The default value is None

All other arguments are identical to the [Compare class's `report()` method](#comparereport).

### A Note on Weights

Compare objects compute up to three kinds of weights for their elements: global weights, local weights and target weights.
Compare objects also compute their own global and local weight, given their parent.

- **Global** weights display the computed weights of a Compare object's elements **dependent on** that object's global weight within the current hierarchy
  - Global weights are derived by multiplying the local weights of the elements within a Compare object by that object's *own* global weight in the current hierarchy

- **Local** weights display the computed weights of a Compare object's elements **independent of** that object's global weight within the current hierarchy
  - The local weights of the elements within a Compare object will always (approximately) sum to 1.0

- **Target** weights display the synthesized weights of the problem elements described in the *lowest level* of the current hierarchy
  - Target weights are only available from the Compare object at the highest level of the hierarchy (*i.e.* the only Compare object without a parent)

#### N.B.

A Compare object that does not have a parent will have identical global and local weights; a Compare object that has neither a parent nor children will have identical global, local and target weights.

In many instances, the sum of the local or target weights of a Compare object will not equal 1.0 *exactly*. This is due to rounding. If it's critical that the sum of the weights equals 1.0, it's recommended to simply divide the weights by their cumulative sum: `x = x / np.sum(x)`. Note, however, that the resulting values will contain a false level of precision, given their inputs.

### Missing Pairwise Comparisons

When a Compare object is initialized, the elements forming the keys of the input `comparisons` dictionary are permuted. Permutations of elements that do not contain a value within the input `comparisons` dictionary are then optimally solved for using the cyclic coordinates algorithm described in:

>Bozóki, S., Fülöp, J. and Rónyai, L., 'On optimal completion of incomplete pairwise comparison matrices,' *Mathematical and Computer Modelling*, 52:1–2, 2010, pp. 318-333 (DOI: [10.1016/j.mcm.2010.02.047](https://doi.org/10.1016/j.mcm.2010.02.047))

As the paper notes, "The number of *necessary* pairwise comparisons ... depends on the characteristics of the real decision problem and provides an exciting topic of future research" (29). In other words, don't rely on the algorithm to fill in a comparison dictionary that has a large number of missing values: it certainly might, but it also very well might not. **Caveat emptor!**

The example below demonstrates this functionality of AHPy using the following matrix:

||a|b|c|d|
|-|:-:|:-:|:-:|:-:|
|a|1|1|5|2|
|b|1|1|3|4|
|c|1/5|1/3|1|**3/4**|
|d|1/2|1/4|**4/3**|1|

We'll first compute the target weights and consistency ratio for the complete matrix, then repeat the process after removing the **(c, d)** comparison marked in bold. We can view the computed value in the Compare object's detailed report:

```python
>>> comparisons = {('a', 'b'): 1, ('a', 'c'): 5, ('a', 'd'): 2,
		   ('b', 'c'): 3, ('b', 'd'): 4,
		   ('c', 'd'): 3 / 4}

>>> complete = ahpy.Compare('Complete', comparisons)

>>> print(complete.target_weights)
{'b': 0.3917, 'a': 0.3742, 'd': 0.1349, 'c': 0.0991}

>>> print(complete.consistency_ratio)
0.0372

>>> del comparisons[('c', 'd')]

>>> missing_cd = ahpy.Compare('Missing_CD', comparisons)

>>> print(missing_cd.target_weights)
{'b': 0.392, 'a': 0.3738, 'd': 0.1357, 'c': 0.0985}

>>> print(missing_cd.consistency_ratio)
0.0372

>>> report = missing_cd.report(verbose=True)
>>> print(report['comparisons']['computed'])
{('c', 'd'): 0.7302971068355002}
```

## Development and Testing

To set up a development environment and run the included tests, you can use the following commands:

```
virtualenv .venv
source .venv/bin/activate
python setup.py develop
pip install pytest
pytest
```
