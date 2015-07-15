# Django selectreverse #

This module contains a set of helpers to reduce the number of queries needed to access m2m relations or reverse foreign key relations.

## Introduction ##

Consider a parent-child relationship building-apartements. It is created using a foreign key field to the Building model in the Apartment model.

A common pattern is to list some or all buildings and for each of the buildings list the apartments.

An example:

In the template:

```
	{% for building in buildinglist %}
	  <p>Building: {{ building }}</p>
	  <p>Apartments:</p>
	  <ul>
	    {% for apartment in building.apartment_set.all %}
	      <li>{{ apartment.number }}</li>
	    {% endfor %}
	  </ul>
	{% endfor %}
```

In the view:

```
	buildinglist = Building.objects.all()
```

The problem is that this causes an extra query for each building, when the list of apartments for each building gets loaded from the database.

Using the ReverseManager in this package, this can be reduced to only one extra query.

How ?

In the Building model, use the ReverseManager instead (or in addition to) the default manager.

```
	objects = ReverseManager()
```

In the view use the select\_reverse method to prefetch the appartments:

```
	buildinglist = Building.objects.select_reverse({'apartments': 'apartment_set'})
```

The select\_reverse method accepts a dictionary, mapping a reverse (or m2m) relationship to a new attribute name.

Now when retrieving the buildinglist from the database, one extra query will be performed to get the corresponding apartments. Each building will get an 'apartments' attribute, which contains a list of the corresponding apartment objects.

All we need to do to use this in the template is replace the 'building.apartment\_set.all' query with a reference to the new attribute: 'building.apartments', like this:

```
	{% for building in buildinglist %}
	  <p>Building: {{ building }}</p>
	  <p>Apartments:</p>
	  <ul>
	    {% for apartment in building.apartments %}
	      <li>{{ apartment.number }}</li>
	    {% endfor %}
	  </ul>
	{% endfor %}
```

Instead of many queries, this only requires two queries, regardless of the number of buildings in the list.


## API ##

The package contains a custom manager, called ReverseManager.

ReverseManager contains a custom method 'select\_reverse', which accepts one argument: a dictionary mapping relationships to a new attribute name.

```
	buildinglist = Building.objects.select_reverse({'apartments': 'apartment_set'})
```

This will use one single extra query to get the apartment objects and adds to each building in the set an attribute 'apartments' which is a list of apartments, equivalent to the result of a building.apartment\_set.all() query.

Mind that overriding an existing attribute name is not allowed and will raise a ImproperlyConfigured exception.

The following relationships are currently supported:

  * reverse foreign key
  * m2m
  * reverse m2m

It is also possible to define a default mapping when defining the manager. Just pass the mapping in the manager constructor like this:

```
	objects = ReverseManager({'apartments': 'apartment_set'})
```

Now when you use Building.objects.all() or Building.objects.select\_reverse(), this default mapping will be used to prefetch the objects.

Of course select\_reverse() can be chained after filter(), e.g. Building.objects.filter(pklt=20).select\_reverse().

Look at the tests for examples of m2m and reverse m2m relationships.
