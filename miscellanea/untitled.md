# Set operations

This is a list of useful set operations in Python.

## Output and Full Code​

The examples below show how to use Python set objects, which implement all mathematical set operations. The set can contains almost every kind of variable, but they cannot contain duplicated elements.

```python
cities = {'Madrid', 'Valencia', 'Munich'}

# add an element to the set (already present) - the element is not added, since duplicates are not allowed
cities.add('Madrid')
# {'Madrid', 'Munich', 'Valencia'}

# remove Stuttgart from the set - the element does not exist 
cities.discard('Stuttgart')
# {'Madrid', 'Munich', 'Valencia'}
```

```python
## OPERATIONS
# two sets - one containing mammals and another containing aquatic animals
mammals = {'Tiger', 'Camel', 'Sheep', 'Whale', 'Walrus'}
aquatic = {'Octopus', 'Squid', 'Crab', 'Whale', 'Walrus'}

# union
animals = mammals.union(aquatic)
animals = mammals | aquatic

# intersection
aquatic_mammals = mammals.intersection(aquatic)
aquatic_mammals = mammals & aquatic

# set difference
mammals_no_acquatic = mammals.difference(aquatic)
mammals_no_acquatic = mammals - aquatic

# set symmetric difference = (A | B) - (A & B) 
no_acquatic_mammals = mammals.symmetric_difference(aquatic)
no_acquatic_mammals = mammals ^ aquatic

# set relations
mammals.issuperset(aquatic_mammals)
# True

aquatic.issubset(mammals)
# False

mammals.isdisjoint(aquatic)
# False
```

