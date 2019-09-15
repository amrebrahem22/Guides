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

#### Caching and QuerySets
Each QuerySet contains a cache to minimize database access. Understanding how it works will allow you to write the most efficient code.
</br>
In a newly created QuerySet, the cache is empty. The first time a QuerySet is evaluated – and, hence, a database query happens – Django saves the query results in the QuerySet’s cache and returns the results that have been explicitly requested (e.g., the next element, if the QuerySet is being iterated over). Subsequent evaluations of the QuerySet reuse the cached results.
</br>
Keep this caching behavior in mind, because it may bite you if you don’t use your QuerySets correctly. For example, the following will create two QuerySets, evaluate them, and throw them away:
``` python
>>> print([e.headline for e in Entry.objects.all()])
>>> print([e.pub_date for e in Entry.objects.all()])
```
That means the same database query will be executed twice, effectively doubling your database load. Also, there’s a possibility the two lists may not include the same database records, because an Entry may have been added or deleted in the split second between the two requests.
</br>
To avoid this problem, simply save the QuerySet and reuse it:
``` python
>>> queryset = Entry.objects.all()
>>> print([p.headline for p in queryset]) # Evaluate the query set.
>>> print([p.pub_date for p in queryset]) # Re-use the cache from the evaluation.
```
#### When QuerySets are not cached
Querysets do not always cache their results. When evaluating only part of the queryset, the cache is checked, but if it is not populated then the items returned by the subsequent query are not cached. Specifically, this means that limiting the queryset using an array slice or an index will not populate the cache.
</br>
For example, repeatedly getting a certain index in a queryset object will query the database each time:
``` python
>>> queryset = Entry.objects.all()
>>> print(queryset[5]) # Queries the database
>>> print(queryset[5]) # Queries the database again
```
However, if the entire queryset has already been evaluated, the cache will be checked instead:
``` python
>>> queryset = Entry.objects.all()
>>> [entry for entry in queryset] # Queries the database
>>> print(queryset[5]) # Uses cache
>>> print(queryset[5]) # Uses cache
```
Here are some examples of other actions that will result in the entire queryset being evaluated and therefore populate the cache:
``` python
>>> [entry for entry in queryset]
>>> bool(queryset)
>>> entry in queryset
>>> list(queryset)
```

### Complex lookups with Q objects
Keyword argument queries – in filter(), etc. – are “AND”ed together. If you need to execute more complex queries (for example, queries with OR statements), you can use Q objects.
</br>
A Q object (django.db.models.Q) is an object used to encapsulate a collection of keyword arguments. These keyword arguments are specified as in “Field lookups” above.
</br>
For example, this Q object encapsulates a single LIKE query:
``` python
from django.db.models import Q
Q(question__startswith='What')
```
Q objects can be combined using the & and | operators. When an operator is used on two Q objects, it yields a new Q object.
</br>
For example, this statement yields a single Q object that represents the “OR” of two "question__startswith" queries:
``` python
Q(question__startswith='Who') | Q(question__startswith='What')
```
This is equivalent to the following SQL WHERE clause:
``` sql
WHERE question LIKE 'Who%' OR question LIKE 'What%'
```
You can compose statements of arbitrary complexity by combining Q objects with the & and | operators and use parenthetical grouping. Also, Q objects can be negated using the ~ operator, allowing for combined lookups that combine both a normal query and a negated (NOT) query:
``` python
Q(question__startswith='Who') | ~Q(pub_date__year=2005)
```
Each lookup function that takes keyword-arguments (e.g. filter(), exclude(), get()) can also be passed one or more Q objects as positional (not-named) arguments. If you provide multiple Q object arguments to a lookup function, the arguments will be “AND”ed together. For example:
``` python
Poll.objects.get(
    Q(question__startswith='Who'),
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
)
```
… roughly translates into the SQL:
``` python
SELECT * from polls WHERE question LIKE 'Who%'
    AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')
 ```
Lookup functions can mix the use of Q objects and keyword arguments. All arguments provided to a lookup function (be they keyword arguments or Q objects) are “AND”ed together. However, if a Q object is provided, it must precede the definition of any keyword arguments. For example:
``` python
Poll.objects.get(
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
    question__startswith='Who',
)
```
… would be a valid query, equivalent to the previous example; but:
``` python
# INVALID QUERY
Poll.objects.get(
    question__startswith='Who',
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
)
```
… would not be valid.

### Comparing objects
To compare two model instances, just use the standard Python comparison operator, the double equals sign: ==. Behind the scenes, that compares the primary key values of two models.
</br>
Using the Entry example above, the following two statements are equivalent:
``` python
>>> some_entry == other_entry
>>> some_entry.id == other_entry.id
```
If a model’s primary key isn’t called id, no problem. Comparisons will always use the primary key, whatever it’s called. For example, if a model’s primary key field is called name, these two statements are equivalent:
``` python
>>> some_obj == other_obj
>>> some_obj.name == other_obj.name
```
### Deleting objects
The delete method, conveniently, is named delete(). This method immediately deletes the object and returns the number of objects deleted and a dictionary with the number of deletions per object type. Example:
``` python
>>> e.delete()
(1, {'weblog.Entry': 1})
```
You can also delete objects in bulk. Every QuerySet has a delete() method, which deletes all members of that QuerySet.

For example, this deletes all Entry objects with a pub_date year of 2005:
``` python
>>> Entry.objects.filter(pub_date__year=2005).delete()
(5, {'webapp.Entry': 5})
```
Keep in mind that this will, whenever possible, be executed purely in SQL, and so the delete() methods of individual object instances will not necessarily be called during the process. If you’ve provided a custom delete() method on a model class and want to ensure that it is called, you will need to “manually” delete instances of that model (e.g., by iterating over a QuerySet and calling delete() on each object individually) rather than using the bulk delete() method of a QuerySet.
</br>
When Django deletes an object, by default it emulates the behavior of the SQL constraint ON DELETE CASCADE – in other words, any objects which had foreign keys pointing at the object to be deleted will be deleted along with it. For example:
``` python
b = Blog.objects.get(pk=1)
# This will delete the Blog and all of its Entry objects.
b.delete()
```
This cascade behavior is customizable via the on_delete argument to the ForeignKey.
</br>
Note that delete() is the only QuerySet method that is not exposed on a Manager itself. This is a safety mechanism to prevent you from accidentally requesting Entry.objects.delete(), and deleting all the entries. If you do want to delete all the objects, then you have to explicitly request a complete query set:
``` python
Entry.objects.all().delete()
```

### Copying model instances
Although there is no built-in method for copying model instances, it is possible to easily create new instance with all fields’ values copied. In the simplest case, you can just set pk to None. Using our blog example:
``` python
blog = Blog(name='My blog', tagline='Blogging is easy')
blog.save() # blog.pk == 1

blog.pk = None
blog.save() # blog.pk == 2
```
Things get more complicated if you use inheritance. Consider a subclass of Blog:
``` python
class ThemeBlog(Blog):
    theme = models.CharField(max_length=200)

django_blog = ThemeBlog(name='Django', tagline='Django is easy', theme='python')
django_blog.save() # django_blog.pk == 3
```
Due to how inheritance works, you have to set both pk and id to None:
``` python
django_blog.pk = None
django_blog.id = None
django_blog.save() # django_blog.pk == 4
```
This process doesn’t copy relations that aren’t part of the model’s database table. For example, Entry has a ManyToManyField to Author. After duplicating an entry, you must set the many-to-many relations for the new entry:
``` python
entry = Entry.objects.all()[0] # some previous entry
old_authors = entry.authors.all()
entry.pk = None
entry.save()
entry.authors.set(old_authors)
```
For a OneToOneField, you must duplicate the related object and assign it to the new object’s field to avoid violating the one-to-one unique constraint. For example, assuming entry is already duplicated as above:
``` python
detail = EntryDetail.objects.all()[0]
detail.pk = None
detail.entry = entry
detail.save()
```

### Updating multiple objects at once
Sometimes you want to set a field to a particular value for all the objects in a QuerySet. You can do this with the update() method. For example:
``` python
# Update all the headlines with pub_date in 2007.
Entry.objects.filter(pub_date__year=2007).update(headline='Everything is the same')
```
You can only set non-relation fields and ForeignKey fields using this method. To update a non-relation field, provide the new value as a constant. To update ForeignKey fields, set the new value to be the new model instance you want to point to. For example:
``` python
>>> b = Blog.objects.get(pk=1)

# Change every Entry so that it belongs to this Blog.
>>> Entry.objects.all().update(blog=b)
```
The update() method is applied instantly and returns the number of rows matched by the query (which may not be equal to the number of rows updated if some rows already have the new value). The only restriction on the QuerySet being updated is that it can only access one database table: the model’s main table. You can filter based on related fields, but you can only update columns in the model’s main table. Example:
``` python
>>> b = Blog.objects.get(pk=1)

# Update all the headlines belonging to this Blog.
>>> Entry.objects.select_related().filter(blog=b).update(headline='Everything is the same')
```
Be aware that the update() method is converted directly to an SQL statement. It is a bulk operation for direct updates. It doesn’t run any save() methods on your models, or emit the pre_save or post_save signals (which are a consequence of calling save()), or honor the auto_now field option. If you want to save every item in a QuerySet and make sure that the save() method is called on each instance, you don’t need any special function to handle that. Just loop over them and call save():
``` python
for item in my_queryset:
    item.save()
```
Calls to update can also use F expressions to update one field based on the value of another field in the model. This is especially useful for incrementing counters based upon their current value. For example, to increment the pingback count for every entry in the blog:
``` python
>>> Entry.objects.all().update(n_pingbacks=F('n_pingbacks') + 1)
```
However, unlike F() objects in filter and exclude clauses, you can’t introduce joins when you use F() objects in an update – you can only reference fields local to the model being updated. If you attempt to introduce a join with an F() object, a FieldError will be raised:
``` python
# This will raise a FieldError
>>> Entry.objects.update(headline=F('blog__name'))
```

### Related objects
When you define a relationship in a model (i.e., a ForeignKey, OneToOneField, or ManyToManyField), instances of that model will have a convenient API to access the related object(s).
</br>
Using the models at the top of this page, for example, an Entry object e can get its associated Blog object by accessing the blog attribute: e.blog.
</br>
(Behind the scenes, this functionality is implemented by Python descriptors. This shouldn’t really matter to you, but we point it out here for the curious.)
</br>
Django also creates API accessors for the “other” side of the relationship – the link from the related model to the model that defines the relationship. For example, a Blog object b has access to a list of all related Entry objects via the entry_set attribute: b.entry_set.all().
</br>
All examples in this section use the sample Blog, Author and Entry models defined at the top of this page.

#### One-to-many relationships
</br>
**Forward**
</br>
If a model has a ForeignKey, instances of that model will have access to the related (foreign) object via a simple attribute of the model.
</br>
Example:
``` python
>>> e = Entry.objects.get(id=2)
>>> e.blog # Returns the related Blog object.
```
You can get and set via a foreign-key attribute. As you may expect, changes to the foreign key aren’t saved to the database until you call save(). Example:
``` python
>>> e = Entry.objects.get(id=2)
>>> e.blog = some_blog
>>> e.save()
```
If a ForeignKey field has null=True set (i.e., it allows NULL values), you can assign None to remove the relation. Example:
``` python
>>> e = Entry.objects.get(id=2)
>>> e.blog = None
>>> e.save() # "UPDATE blog_entry SET blog_id = NULL ...;"
```
Forward access to one-to-many relationships is cached the first time the related object is accessed. Subsequent accesses to the foreign key on the same object instance are cached. Example:
``` python
>>> e = Entry.objects.get(id=2)
>>> print(e.blog)  # Hits the database to retrieve the associated Blog.
>>> print(e.blog)  # Doesn't hit the database; uses cached version.
```
Note that the select_related() QuerySet method recursively prepopulates the cache of all one-to-many relationships ahead of time. Example:
``` python
>>> e = Entry.objects.select_related().get(id=2)
>>> print(e.blog)  # Doesn't hit the database; uses cached version.
>>> print(e.blog)  # Doesn't hit the database; uses cached version.
```
</br>
**`Following relationships “backward”`**
</br>
If a model has a ForeignKey, instances of the foreign-key model will have access to a Manager that returns all instances of the first model. By default, this Manager is named FOO_set, where FOO is the source model name, lowercased. This Manager returns QuerySets, which can be filtered and manipulated as described in the “Retrieving objects” section above.
</br>
Example:
``` python
>>> b = Blog.objects.get(id=1)
>>> b.entry_set.all() # Returns all Entry objects related to Blog.

# b.entry_set is a Manager that returns QuerySets.
>>> b.entry_set.filter(headline__contains='Lennon')
>>> b.entry_set.count()
```
You can override the FOO_set name by setting the related_name parameter in the ForeignKey definition. For example, if the Entry model was altered to blog = ForeignKey(Blog, on_delete=models.CASCADE, related_name='entries'), the above example code would look like this:
``` python
>>> b = Blog.objects.get(id=1)
>>> b.entries.all() # Returns all Entry objects related to Blog.

# b.entries is a Manager that returns QuerySets.
>>> b.entries.filter(headline__contains='Lennon')
>>> b.entries.count()
```
#### Using a custom reverse manager
By default the RelatedManager used for reverse relations is a subclass of the default manager for that model. If you would like to specify a different manager for a given query you can use the following syntax:
``` python
from django.db import models

class Entry(models.Model):
    #...
    objects = models.Manager()  # Default Manager
    entries = EntryManager()    # Custom Manager

b = Blog.objects.get(id=1)
b.entry_set(manager='entries').all()
```
If EntryManager performed default filtering in its get_queryset() method, that filtering would apply to the all() call.
</br>
Of course, specifying a custom reverse manager also enables you to call its custom methods:
</br>
``` python
b.entry_set(manager='entries').is_published()
```
#### Additional methods to handle related objects
In addition to the QuerySet methods defined in “Retrieving objects” above, the ForeignKey Manager has additional methods used to handle the set of related objects. A synopsis of each is below, and complete details can be found in the related objects reference.

</br>
`add(obj1, obj2, ...)`
</br>
Adds the specified model objects to the related object set.
</br>
`create(**kwargs)`
</br>
Creates a new object, saves it and puts it in the related object set. Returns the newly created object.
</br>
`remove(obj1, obj2, ...)`
</br>
Removes the specified model objects from the related object set.
</br>
`clear()`
</br>
Removes all objects from the related object set.
</br>
`set(objs)`
</br>
Replace the set of related objects.
To assign the members of a related set, use the set() method with an iterable of object instances. For example, if e1 and e2 are Entry instances:
``` python
b = Blog.objects.get(id=1)
b.entry_set.set([e1, e2])
```
If the clear() method is available, any pre-existing objects will be removed from the entry_set before all objects in the iterable (in this case, a list) are added to the set. If the clear() method is not available, all objects in the iterable will be added without removing any existing elements.
</br>
Each “reverse” operation described in this section has an immediate effect on the database. Every addition, creation and deletion is immediately and automatically saved to the database.

#### How are the backward relationships possible?
Other object-relational mappers require you to define relationships on both sides. The Django developers believe this is a violation of the DRY (Don’t Repeat Yourself) principle, so Django only requires you to define the relationship on one end.
</br>
But how is this possible, given that a model class doesn’t know which other model classes are related to it until those other model classes are loaded?
</br>
The answer lies in the app registry. When Django starts, it imports each application listed in INSTALLED_APPS, and then the models module inside each application. Whenever a new model class is created, Django adds backward-relationships to any related models. If the related models haven’t been imported yet, Django keeps tracks of the relationships and adds them when the related models eventually are imported.
</br>
For this reason, it’s particularly important that all the models you’re using be defined in applications listed in INSTALLED_APPS. Otherwise, backwards relations may not work properly.



































