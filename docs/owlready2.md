## Owlready2 with GraphDB 

### Install
```bash
pip3 install git+https://github.com/csse-uoft/owlready2.git
```

### Connect to a GraphDB repository
```python
from owlready2 import default_world

# If authentication is not required, remove username and password.
default_world.set_backend(backend="sparql-endpoint", 
                          endpoint="http://3.97.83.181:7200/repositories/compass-test",
                          username='grapdb login username', password='graphdb login password')
```

### Load local ontology file into GraphDB

#### With Owlready2
Owlready2 keep track of the Modified time of an ontology file.

- If the ontology file is pre-imported into GraphDB and the local ontology file is not modified , it will skip the import and load the ontology to Python from GraphDB.
- If the local ontology file is newer, it will update the ontology in GraphDB and load the ontology to Python from GraphDB.

```python
compass = default_world.get_ontology('./data/compass_ontology.owl').load()
```
#### With GraphDB Interface
You can directly import an ontology file to the GraphDB interface.
Make sure setting **Target Graph** to **NamedGraph** and enter the ontology IRI, i.e. `http://ontology.eil.utoronto.ca/Compass/compass`
Make sure you are importing to the correct named graph.


### Load ontology from GraphDB
By default, owlready2 load all properties along with the ontology.
```python
compass = default_world.get_ontology('http://ontology.eil.utoronto.ca/Compass/compass').load()
```
Load properties may take some time. You can skip the properties loading process by setting `load_all_properties=False`.
```python
compass = default_world.get_ontology('http://ontology.eil.utoronto.ca/Compass/compass').load(load_all_properties=False)
```

### interacting with Classes and Individuals
> Please refer to https://owlready2.readthedocs.io/en/latest/class.html for full usage.

#### Instantiate a Class
```python
# This is needed because we want to load a Class that is defined in the cids namespace.
cids = default_world.get_namespace('http://ontology.eil.utoronto.ca/cids/cids#')
ic = default_world.get_namespace('http://ontology.eil.utoronto.ca/tove/icontact#')

# Instantiate the cids:Organization Class
organization = cids.Organization()

# Assign the hasName Property
# Same as cids.hasName[organization] = ['Orgazation name']
organization.hasName.append('Orgazation name')


# Instantiate a ic:PhoneNumber Class
telephone = ic.PhoneNumber()
telephone.hasPhoneNumber.append('123456789')

# Assign the PhoneNumber instance to the Organization instance
ic.hasTelephone[organization].append(telephone)
```

#### Set a property
There are two ways to set property value.
##### Simplified way
This does not specify where the `hasName` property is from. 
It could be problematic when there are multiple `hasName` properties defined in different namespaces.
```python
organization.hasName.append('Orgazation name')
```
##### Strict Way
This specify to use `cids.HasName`.
```python
cids.hasName[organization] = ['Orgazation name']
```

#### SPARQL Queries
> When integrated with GraphDB, owlready2 uses the GraphDB SPARQL engine instead of its buggy native SPARQL engine.

The result will be mapped into owlready2 objects.
```python
result = default_world.sparql('''
    PREFIX cids: <http://ontology.eil.utoronto.ca/cids/cids#>
    SELECT ?service WHERE {
        ?service a cids:Organization.
    }
''')
print(result)
# -> [org1, org2, org3, ...]

print(result[0].hasName)
# -> 'Some organization name'
```

#### Owlready2 Queries
```python
orgs = list(default_world.search(is_a=cids.Organization))

print(list(orgs))
# -> [org1, org2, org3, ...]
```
