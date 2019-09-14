# Making queries
``` python
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):
        return self.headline
```
## Creating objects
To represent database-table data in Python objects, Django uses an intuitive system: A model class represents a database table, and an instance of that class represents a particular record in the database table.
</br>
To create an object, instantiate it using keyword arguments to the model class, then call save() to save it to the database.
</br>
Assuming models live in a file mysite/blog/models.py, here’s an example:
``` python
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```
This performs an INSERT SQL statement behind the scenes. Django doesn’t hit the database until you explicitly call save().
</br>
The save() method has no return value.

### Saving changes to objects
To save changes to an object that’s already in the database, use save().
</br>
Given a Blog instance b5 that has already been saved to the database, this example changes its name and updates its record in the database:
``` python
>>> b5.name = 'New name'
>>> b5.save()
```
This performs an UPDATE SQL statement behind the scenes. Django doesn’t hit the database until you explicitly call save().

### Saving ForeignKey and ManyToManyField fields
Updating a ForeignKey field works exactly the same way as saving a normal field – simply assign an object of the right type to the field in question. This example updates the blog attribute of an Entry instance entry, assuming appropriate instances of Entry and Blog are already saved to the database (so we can retrieve them below):
``` python
>>> from blog.models import Blog, Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
```
Updating a ManyToManyField works a little differently – use the add() method on the field to add a record to the relation. This example adds the Author instance joe to the entry object:
``` python
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```
To add multiple records to a ManyToManyField in one go, include multiple arguments in the call to add(), like this:
``` python
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```
Django will complain if you try to assign or add an object of the wrong type.

### Retrieving objects
To retrieve objects from your database, construct a QuerySet via a Manager on your model class.
</br>
A QuerySet represents a collection of objects from your database. It can have zero, one or many filters. Filters narrow down the query results based on the given parameters. In SQL terms, a QuerySet equates to a SELECT statement, and a filter is a limiting clause such as WHERE or LIMIT.
</br>
You get a QuerySet by using your model’s Manager. Each model has at least one Manager, and it’s called objects by default. Access it directly via the model class, like so:
``` python
>>> Blog.objects
<django.db.models.manager.Manager object at ...>
>>> b = Blog(name='Foo', tagline='Bar')
>>> b.objects
Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."
```
Note
</br>
Managers are accessible only via model classes, rather than from model instances, to enforce a separation between “table-level” operations and “record-level” operations.
</br>
The Manager is the main source of QuerySets for a model. For example, Blog.objects.all() returns a QuerySet that contains all Blog objects in the database.

#### Retrieving all objects
The simplest way to retrieve objects from a table is to get all of them. To do this, use the all() method on a Manager:
</br>
`>>> all_entries = Entry.objects.all()` 
</br>
The all() method returns a QuerySet of all the objects in the database.
</br>
#### Retrieving specific objects with filters
The QuerySet returned by all() describes all objects in the database table. Usually, though, you’ll need to select only a subset of the complete set of objects.
</br>
To create such a subset, you refine the initial QuerySet, adding filter conditions. The two most common ways to refine a QuerySet are:
</br>
**`filter(**kwargs)`** </br>
Returns a new QuerySet containing objects that match the given lookup parameters. </br>
**`exclude(**kwargs)`** </br>
Returns a new QuerySet containing objects that do not match the given lookup parameters.
The lookup parameters (**kwargs in the above function definitions) should be in the format described in Field lookups below.
</br>
For example, to get a QuerySet of blog entries from the year 2006, use filter() like so:
``` python
Entry.objects.filter(pub_date__year=2006)
With the default manager class, it is the same as:

Entry.objects.all().filter(pub_date__year=2006)
```
#### Chaining filters
The result of refining a QuerySet is itself a QuerySet, so it’s possible to chain refinements together. For example:
``` python
>>> Entry.objects.filter(
...     headline__startswith='What'
... ).exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(
...     pub_date__gte=datetime.date(2005, 1, 30)
... )
```
This takes the initial QuerySet of all entries in the database, adds a filter, then an exclusion, then another filter. The final result is a QuerySet containing all entries with a headline that starts with “What”, that were published between January 30, 2005, and the current day.

#### Filtered QuerySets are unique
Each time you refine a QuerySet, you get a brand-new QuerySet that is in no way bound to the previous QuerySet. Each refinement creates a separate and distinct QuerySet that can be stored, used and reused.
</br>
Example:
``` python
>>> q1 = Entry.objects.filter(headline__startswith="What")
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
```
These three QuerySets are separate. The first is a base QuerySet containing all entries that contain a headline starting with “What”. The second is a subset of the first, with an additional criteria that excludes records whose pub_date is today or in the future. The third is a subset of the first, with an additional criteria that selects only the records whose pub_date is today or in the future. The initial QuerySet (q1) is unaffected by the refinement process.

#### QuerySets are lazy
QuerySets are lazy – the act of creating a QuerySet doesn’t involve any database activity. You can stack filters together all day long, and Django won’t actually run the query until the QuerySet is evaluated. Take a look at this example:
``` python
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")
>>> print(q)
```
Though this looks like three database hits, in fact it hits the database only once, at the last line (print(q)). In general, the results of a QuerySet aren’t fetched from the database until you “ask” for them. When you do, the QuerySet is evaluated by accessing the database. For more details on exactly when evaluation takes place, see When QuerySets are evaluated.

#### Retrieving a single object with get()
filter() will always give you a QuerySet, even if only a single object matches the query - in this case, it will be a QuerySet containing a single element.
</br>
If you know there is only one object that matches your query, you can use the get() method on a Manager which returns the object directly:
</br>
`>>> one_entry = Entry.objects.get(pk=1)`
</br>
You can use any query expression with get(), just like with filter() - again, see Field lookups below.
</br>
Note that there is a difference between using get(), and using filter() with a slice of [0]. If there are no results that match the query, get() will raise a DoesNotExist exception. This exception is an attribute of the model class that the query is being performed on - so in the code above, if there is no Entry object with a primary key of 1, Django will raise Entry.DoesNotExist.
</br>
Similarly, Django will complain if more than one item matches the get() query. In this case, it will raise MultipleObjectsReturned, which again is an attribute of the model class itself.

#### Limiting QuerySets
Use a subset of Python’s array-slicing syntax to limit your QuerySet to a certain number of results. This is the equivalent of SQL’s LIMIT and OFFSET clauses.
</br>
For example, this returns the first 5 objects (LIMIT 5):
</br>
`>>> Entry.objects.all()[:5]`
</br>
This returns the sixth through tenth objects (OFFSET 5 LIMIT 5):
</br>
`>>> Entry.objects.all()[5:10]`
</br>
Negative indexing (i.e. Entry.objects.all()[-1]) is not supported.
</br>
Generally, slicing a QuerySet returns a new QuerySet – it doesn’t evaluate the query. An exception is if you use the “step” parameter of Python slice syntax. For example, this would actually execute the query in order to return a list of every second object of the first 10:
</br>
`>>> Entry.objects.all()[:10:2]`
</br>
Further filtering or ordering of a sliced queryset is prohibited due to the ambiguous nature of how that might work.
</br>
To retrieve a single object rather than a list (e.g. SELECT foo FROM bar LIMIT 1), use a simple index instead of a slice. For example, this returns the first Entry in the database, after ordering entries alphabetically by headline:
</br>
`>>> Entry.objects.order_by('headline')[0]`
</br>
This is roughly equivalent to:
</br>
`>>> Entry.objects.order_by('headline')[0:1].get()`
</br>
Note, however, that the first of these will raise IndexError while the second will raise DoesNotExist if no objects match the given criteria

### Field lookups
Field lookups are how you specify the meat of an SQL WHERE clause. They’re specified as keyword arguments to the QuerySet methods filter(), exclude() and get().
</br>
Basic lookups keyword arguments take the form field__lookuptype=value. (That’s a double-underscore). For example:
</br>
`>>> Entry.objects.filter(pub_date__lte='2006-01-01')` 
</br>
translates (roughly) into the following SQL:
</br>
`SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';` 
</br>

The field specified in a lookup has to be the name of a model field. There’s one exception though, in case of a ForeignKey you can specify the field name suffixed with _id. In this case, the value parameter is expected to contain the raw value of the foreign model’s primary key. For example:
</br>
`>>> Entry.objects.filter(blog_id=4)`
</br>
If you pass an invalid keyword argument, a lookup function will raise TypeError.
</br>
**exact** </br>
An “exact” match. For example:
</br>
`>>> Entry.objects.get(headline__exact="Cat bites dog")` </br>
Would generate SQL along these lines:
</br>
`SELECT ... WHERE headline = 'Cat bites dog';` </br>
If you don’t provide a lookup type – that is, if your keyword argument doesn’t contain a double underscore – the lookup type is assumed to be exact.
</br>
For example, the following two statements are equivalent:

`>>> Blog.objects.get(id__exact=14)  # Explicit form` </br>
`>>> Blog.objects.get(id=14)         # __exact is implied` </br>
This is for convenience, because exact lookups are the common case.
</br>
**iexact** </br>
A case-insensitive match. So, the query:
</br>
>>> Blog.objects.get(name__iexact="beatles blog")
Would match a Blog titled "Beatles Blog", "beatles blog", or even "BeAtlES blOG".
</br>
**contains** </br>
Case-sensitive containment test. For example:
</br>
`Entry.objects.get(headline__contains='Lennon')` 
</br>
Roughly translates to this SQL:
</br>
`SELECT ... WHERE headline LIKE '%Lennon%';`
</br>
Note this will match the headline 'Today Lennon honored' but not 'today lennon honored'.
</br>
There’s also a case-insensitive version, icontains.
</br>
**startswith, endswith** </br>
Starts-with and ends-with search, respectively. There are also case-insensitive versions called istartswith and iendswith.

### Lookups that span relationships
Django offers a powerful and intuitive way to “follow” relationships in lookups, taking care of the SQL JOINs for you automatically, behind the scenes. To span a relationship, just use the field name of related fields across models, separated by double underscores, until you get to the field you want.
</br>
This example retrieves all Entry objects with a Blog whose name is 'Beatles Blog':
</br>
`>>> Entry.objects.filter(blog__name='Beatles Blog')`
</br>
This spanning can be as deep as you’d like.
</br>
It works backwards, too. To refer to a “reverse” relationship, just use the lowercase name of the model.
</br>
This example retrieves all Blog objects which have at least one Entry whose headline contains 'Lennon':
</br>
`>>> Blog.objects.filter(entry__headline__contains='Lennon')`
</br>
If you are filtering across multiple relationships and one of the intermediate models doesn’t have a value that meets the filter condition, Django will treat it as if there is an empty (all values are NULL), but valid, object there. All this means is that no error will be raised. For example, in this filter:
</br>
`Blog.objects.filter(entry__authors__name='Lennon')`
</br>
(if there was a related Author model), if there was no author associated with an entry, it would be treated as if there was also no name attached, rather than raising an error because of the missing author. Usually this is exactly what you want to have happen. The only case where it might be confusing is if you are using isnull. Thus:
</br>
`Blog.objects.filter(entry__authors__name__isnull=True)`
</br>
will return Blog objects that have an empty name on the author and also those which have an empty author on the entry. If you don’t want those latter objects, you could write:
</br>
`Blog.objects.filter(entry__authors__isnull=False, entry__authors__name__isnull=True)`
</br>

### Filters can reference fields on the model
In the examples given so far, we have constructed filters that compare the value of a model field with a constant. But what if you want to compare the value of a model field with another field on the same model?
</br>
Django provides F expressions to allow such comparisons. Instances of F() act as a reference to a model field within a query. These references can then be used in query filters to compare the values of two different fields on the same model instance.
</br>
For example, to find a list of all blog entries that have had more comments than pingbacks, we construct an F() object to reference the pingback count, and use that F() object in the query:
``` python
>>> from django.db.models import F
>>> Entry.objects.filter(n_comments__gt=F('n_pingbacks'))
```
Django supports the use of addition, subtraction, multiplication, division, modulo, and power arithmetic with F() objects, both with constants and with other F() objects. To find all the blog entries with more than twice as many comments as pingbacks, we modify the query:
``` python
>>> Entry.objects.filter(n_comments__gt=F('n_pingbacks') * 2)
```
To find all the entries where the rating of the entry is less than the sum of the pingback count and comment count, we would issue the query:
``` python
>>> Entry.objects.filter(rating__lt=F('n_comments') + F('n_pingbacks'))
```
You can also use the double underscore notation to span relationships in an F() object. An F() object with a double underscore will introduce any joins needed to access the related object. For example, to retrieve all the entries where the author’s name is the same as the blog name, we could issue the query:
``` python
>>> Entry.objects.filter(authors__name=F('blog__name'))
```
For date and date/time fields, you can add or subtract a timedelta object. The following would return all entries that were modified more than 3 days after they were published:
``` python
>>> from datetime import timedelta
>>> Entry.objects.filter(mod_date__gt=F('pub_date') + timedelta(days=3))
```
The F() objects support bitwise operations by .bitand(), .bitor(), .bitrightshift(), and .bitleftshift(). For example:
``` python
>>> F('somefield').bitand(16)
```

### The pk lookup shortcut
For convenience, Django provides a pk lookup shortcut, which stands for “primary key”.
</br>
In the example Blog model, the primary key is the id field, so these three statements are equivalent:
``` python
>>> Blog.objects.get(id__exact=14) # Explicit form
>>> Blog.objects.get(id=14) # __exact is implied
>>> Blog.objects.get(pk=14) # pk implies id__exact
```
The use of pk isn’t limited to `__exact` queries – any query term can be combined with pk to perform a query on the primary key of a model:
``` python
# Get blogs entries with id 1, 4 and 7
>>> Blog.objects.filter(pk__in=[1,4,7])

# Get all blog entries with id > 14
>>> Blog.objects.filter(pk__gt=14)
```
pk lookups also work across joins. For example, these three statements are equivalent:
``` python
>>> Entry.objects.filter(blog__id__exact=3) # Explicit form
>>> Entry.objects.filter(blog__id=3)        # __exact is implied
>>> Entry.objects.filter(blog__pk=3)        # __pk implies __id__exact
```















































