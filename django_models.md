# Models
*A model is the single, definitive source of information about your data. It contains the essential fields and behaviors of the data you’re storing. Generally, each model maps to a single database table.*
``` python 
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
```
*first_name and last_name are fields of the model. Each field is specified as a class attribute, and each attribute maps to a database column.*</br>
*The above Person model would create a database table like this:*
``` python
CREATE TABLE myapp_person (
    "id" serial NOT NULL PRIMARY KEY,
    "first_name" varchar(30) NOT NULL,
    "last_name" varchar(30) NOT NULL
);
```
### Using models
*Once you have defined your models, you need to tell Django you’re going to use those models. Do this by editing your settings file and changing the INSTALLED_APPS setting to add the name of the module that contains your models.py.*
</br>
*For example, if the models for your application live in the module myapp.models (the package structure that is created for an application by the manage.py startapp script), INSTALLED_APPS should read, in part:*
``` python
INSTALLED_APPS = [
    #...
    'myapp',
    #...
]
```
*When you add new apps to INSTALLED_APPS, be sure to run manage.py migrate, optionally making migrations for them first with manage.py makemigrations.*
### Field types
#### AutoField
#### `class AutoField(**options)`
*An IntegerField that automatically increments according to available IDs. You usually won’t need to use this directly; a primary key field will automatically be added to your model if you don’t specify otherwise.*

#### BigAutoField
#### `class BigAutoField(**options)`
*A 64-bit integer, much like an AutoField except that it is guaranteed to fit numbers from 1 to 9223372036854775807.*

#### BigIntegerField
#### `class BigIntegerField(**options)`
*A 64-bit integer, much like an IntegerField except that it is guaranteed to fit numbers from -9223372036854775808 to 9223372036854775807. The default form widget for this field is a TextInput.*

#### BinaryField
#### `class BinaryField(max_length=None, **options)`
*A field to store raw binary data. It can be assigned bytes, bytearray, or memoryview.*
</br>
*By default, BinaryField sets editable to False, in which case it can’t be included in a ModelForm.*
</br>
*Changed in Django 2.1:
Older versions don’t allow setting editable to True.*

BinaryField has one extra optional argument:

##### BinaryField.max_length
*The maximum length (in characters) of the field. The maximum length is enforced in Django’s validation using MaxLengthValidator.*

#### BooleanField
#### `class BooleanField(**options)`
*A true/false field.*

*The default form widget for this field is CheckboxInput, or NullBooleanSelect if null=True.*
</br>
*The default value of BooleanField is None when Field.default isn’t defined.*
</br>
*Changed in Django 2.1:
In older versions, this field doesn’t permit null=True, so you have to use NullBooleanField instead. Using the latter is now discouraged as it’s likely to be deprecated in a future version of Django.*

#### CharField
#### `class CharField(max_length=None, **options)`
*A string field, for small- to large-sized strings, For large amounts of text, use TextField.* </br>
*The default form widget for this field is a TextInput.* </br>
*CharField has one extra required argument:* </br>
`CharField.max_length` </br>
*The maximum length (in characters) of the field. The max_length is enforced at the database level and in Django’s validation using MaxLengthValidator.* </br>

#### DateField
#### `class DateField(auto_now=False, auto_now_add=False, **options)`
*A date, represented in Python by a datetime.date instance. Has a few extra, optional arguments:* </br>
`DateField.auto_now` </br>
*Automatically set the field to now every time the object is saved. Useful for “last-modified” timestamps. Note that the current date is always used; it’s not just a default value that you can override.* </br>
*The field is only automatically updated when calling Model.save(). The field isn’t updated when making updates to other fields in other ways such as QuerySet.update(), though you can specify a custom value for the field in an update like that.* </br>
`DateField.auto_now_add` </br>
*Automatically set the field to now when the object is first created. Useful for creation of timestamps. Note that the current date is always used; it’s not just a default value that you can override. So even if you set a value for this field when creating the object, it will be ignored. If you want to be able to modify this field, set the following instead of auto_now_add=True:* </br>
``` python
    # For DateField: default=date.today - 
    from datetime.date.today()
    # For DateTimeField: default=timezone.now  
    from django.utils.timezone.now()
```
*The default form widget for this field is a TextInput. The admin adds a JavaScript calendar, and a shortcut for “Today”. Includes an additional invalid_date error message key.* </br>
*The options auto_now_add, auto_now, and default are mutually exclusive. Any combination of these options will result in an error.* </br>
**Note** </br>
*As currently implemented, setting auto_now or auto_now_add to True will cause the field to have editable=False and blank=True set* </br>
*The auto_now and auto_now_add options will always use the date in the default timezone at the moment of creation or update. If you need something different, you may want to consider simply using your own callable default or overriding save() instead of using auto_now or auto_now_add; or using a DateTimeField instead of a DateField and deciding how to handle the conversion from datetime to date at display time.*

#### DateTimeField
#### `class DateTimeField(auto_now=False, auto_now_add=False, **options)`
*A date and time, represented in Python by a datetime.datetime instance. Takes the same extra arguments as DateField.* </br>
*The default form widget for this field is a single TextInput. The admin uses two separate TextInput widgets with JavaScript shortcuts.* </br>

#### DecimalField
#### `class DecimalField(max_digits=None, decimal_places=None, **options)`
*A fixed-precision decimal number, represented in Python by a Decimal instance. It validates the input using DecimalValidator.* </br>
*Has two required arguments:* </br>
`DecimalField.max_digits` </br>
*The maximum number of digits allowed in the number. Note that this number must be greater than or equal to decimal_places.* </br>
`DecimalField.decimal_places` </br>
*The number of decimal places to store with the number.* </br>
*For example, to store numbers up to 999 with a resolution of 2 decimal places, you’d use:* </br>
`models.DecimalField(..., max_digits=5, decimal_places=2)` </br>
*And to store numbers up to approximately one billion with a resolution of 10 decimal places:* </br>
`models.DecimalField(..., max_digits=19, decimal_places=10)` </br>
*The default form widget for this field is a NumberInput when localize is False or TextInput otherwise.* </br>

#### DurationField
#### `class DurationField(**options)`
*A field for storing periods of time - modeled in Python by timedelta. When used on PostgreSQL, the data type used is an interval and on Oracle the data type is INTERVAL DAY(9) TO SECOND(6). Otherwise a bigint of microseconds is used.* </br>
*Arithmetic with DurationField works in most cases. However on all databases other than PostgreSQL, comparing the value of a DurationField to arithmetic on DateTimeField instances will not work as expected.* </br>

#### EmailField
#### `class EmailField(max_length=254, **options)`

#### FileField
#### `class FileField(upload_to=None, max_length=100, **options)`
*A file-upload field.* </br>
*Has two optional arguments:* </br>
`FileField.upload_to`
*This attribute provides a way of setting the upload directory and file name, and can be set in two ways. In both cases, the value is passed to the Storage.save() method.* </br>
*If you specify a string value, it may contain strftime() formatting, which will be replaced by the date/time of the file upload (so that uploaded files don’t fill up the given directory). For example:* </br>
``` python
class MyModel(models.Model):
    # file will be uploaded to MEDIA_ROOT/uploads
    upload = models.FileField(upload_to='uploads/')
    # or...
    # file will be saved to MEDIA_ROOT/uploads/2015/01/30
    upload = models.FileField(upload_to='uploads/%Y/%m/%d/')
```
*If you are using the default FileSystemStorage, the string value will be appended to your MEDIA_ROOT path to form the location on the local filesystem where uploaded files will be stored. If you are using a different storage, check that storage’s documentation to see how it handles upload_to.* </br>
*upload_to may also be a callable, such as a function. This will be called to obtain the upload path, including the filename. This callable must accept two arguments and return a Unix-style path (with forward slashes) to be passed along to the storage system. The two arguments are:* </br>
**instance :  An instance of the model where the FileField is defined. More specifically, this is the particular instance where the current file is being attached.**</br>
**In most cases, this object will not have been saved to the database yet, so if it uses the default AutoField, it might not yet have a value for its primary key field.** </br>
**filename :  The filename that was originally given to the file. This may or may not be taken into account when determining the final destination path.** </br>
*For Example: * </br>
``` python
def user_directory_path(instance, filename):
    # file will be uploaded to MEDIA_ROOT/user_<id>/<filename>
    return 'user_{0}/{1}'.format(instance.user.id, filename)

class MyModel(models.Model):
    upload = models.FileField(upload_to=user_directory_path)
```
`FileField.storage`</br>

#### FloatField
#### `class FloatField(**options)`
*A floating-point number represented in Python by a float instance.* </br>
*The default form widget for this field is a NumberInput when localize is False or TextInput otherwise.* </br>
**FloatField vs. DecimalField** </br>
*The FloatField class is sometimes mixed up with the DecimalField class. Although they both represent real numbers, they represent those numbers differently. FloatField uses Python’s float type internally, while DecimalField uses Python’s Decimal type.* </br>

#### ImageField
#### `class ImageField(upload_to=None, height_field=None, width_field=None, max_length=100, **options)`
*Inherits all attributes and methods from FileField, but also validates that the uploaded object is a valid image.* </br>
*In addition to the special attributes that are available for FileField, an ImageField also has height and width attributes.* </br>
*To facilitate querying on those attributes, ImageField has two extra optional arguments:* </br>
`ImageField.height_field` </br>
*Name of a model field which will be auto-populated with the height of the image each time the model instance is saved.* </br>
`ImageField.width_field` </br>
*Name of a model field which will be auto-populated with the width of the image each time the model instance is saved.* </br>
*Requires the Pillow library.* </br>
*ImageField instances are created in your database as varchar columns with a default max length of 100 characters. As with other fields, you can change the maximum length using the max_length argument* </br>
*The default form widget for this field is a ClearableFileInput.* </br>

#### IntegerField
#### `class IntegerField(**options)`
*An integer. Values from -2147483648 to 2147483647 are safe in all databases supported by Django.* </br>
*It uses MinValueValidator and MaxValueValidator to validate the input based on the values that the default database supports.* </br>
*The default form widget for this field is a NumberInput when localize is False or TextInput otherwise.* </br>

#### PositiveIntegerField
#### `class PositiveIntegerField(**options)`
*Like an IntegerField, but must be either positive or zero (0). Values from 0 to 2147483647 are safe in all databases supported by Django. The value 0 is accepted for backward compatibility reasons.*

#### NullBooleanField
#### `class NullBooleanField(**options)`
*Like BooleanField with null=True. Use that instead of this field as it’s likely to be deprecated in a future version of Django.*

#### PositiveSmallIntegerField
#### `class PositiveSmallIntegerField(**options)`
*Like a PositiveIntegerField, but only allows values under a certain (database-dependent) point. Values from 0 to 32767 are safe in all databases supported by Django.*

#### SlugField
#### `class SlugField(max_length=50, **options)`
*Slug is a newspaper term. A slug is a short label for something, containing only letters, numbers, underscores or hyphens. They’re generally used in URLs.* </br>
*Like a CharField, you can specify max_length. If max_length is not specified, Django will use a default length of 50.* </br>
*Implies setting Field.db_index to True.* </br>
*It is often useful to automatically prepopulate a SlugField based on the value of some other value. You can do this automatically in the admin using prepopulated_fields.* </br>
*It uses validate_slug or validate_unicode_slug for validation.* </br>
`SlugField.allow_unicode` </br>
*If True, the field accepts Unicode letters in addition to ASCII letters. Defaults to False* </br>

#### SmallIntegerField
#### `class SmallIntegerField(**options)`
*Like an IntegerField, but only allows values under a certain (database-dependent) point. Values from -32768 to 32767 are safe in all databases supported by Django.*

#### TextField
#### `class TextField(**options)`
*A large text field. The default form widget for this field is a Textarea.* </br>

*If you specify a max_length attribute, it will be reflected in the Textarea widget of the auto-generated form field. However it is not enforced at the model or database level. Use a CharField for that.*

#### TimeField
#### `class TimeField(auto_now=False, auto_now_add=False, **options)`
*A time, represented in Python by a datetime.time instance. Accepts the same auto-population options as DateField.* </br>

*The default form widget for this field is a TextInput. The admin adds some JavaScript shortcuts.*

#### URLField
#### `class URLField(max_length=200, **options)`
*A CharField for a URL, validated by URLValidator.* </br>
*The default form widget for this field is a TextInput.* </br>
*Like all CharField subclasses, URLField takes the optional max_length argument. If you don’t specify max_length, a default of 200 is used.*

#### UUIDField
#### `class UUIDField(**options)`
*A field for storing universally unique identifiers. Uses Python’s UUID class. When used on PostgreSQL, this stores in a uuid datatype, otherwise in a char(32).* </br>
*Universally unique identifiers are a good alternative to AutoField for primary_key. The database will not generate the UUID for you, so it is recommended to use default:* </br>
``` python 
import uuid
from django.db import models

class MyUUIDModel(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    # other fields
```
*Note that a callable (with the parentheses omitted) is passed to default, not an instance of UUID.*

## Relationship fields
#### ForeignKey
#### `class ForeignKey(to, on_delete, **options)`
*A many-to-one relationship. Requires two positional arguments: the class to which the model is related and the on_delete option.* </br>
*To create a recursive relationship – an object that has a many-to-one relationship with itself – use models.ForeignKey('self', on_delete=models.CASCADE).* </br>
*If you need to create a relationship on a model that has not yet been defined, you can use the name of the model, rather than the model object itself:* </br>
``` python
from django.db import models

class Car(models.Model):
    manufacturer = models.ForeignKey(
        'Manufacturer',
        on_delete=models.CASCADE,
    )
    # ...

class Manufacturer(models.Model):
    # ...
    pass
```
*Relationships defined this way on abstract models are resolved when the model is subclassed as a concrete model and are not relative to the abstract model’s app_label:* </br>
*Relationships defined this way on abstract models are resolved when the model is subclassed as a concrete model and are not relative to the abstract model’s app_label:* </br>

**products/models.py**
``` python
from django.db import models

class AbstractCar(models.Model):
    manufacturer = models.ForeignKey('Manufacturer', on_delete=models.CASCADE)

    class Meta:
        abstract = True
```
**production/models.py**
``` python
from django.db import models
from products.models import AbstractCar

class Manufacturer(models.Model):
    pass

class Car(AbstractCar):
    pass

# Car.manufacturer will point to `production.Manufacturer` here.
```
##### Abstract base classes
*Abstract base classes are useful when you want to put some common information into a number of other models. You write your base class and put abstract=True in the Meta class. This model will then not be used to create any database table. Instead, when it is used as a base class for other models, its fields will be added to those of the child class.* </br>
*An example:*
``` python
from django.db import models

class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True

class Student(CommonInfo):
    home_group = models.CharField(max_length=5)
```
*The Student model will have three fields: name, age and home_group. The CommonInfo model cannot be used as a normal Django model, since it is an abstract base class. It does not generate a database table or have a manager, and cannot be instantiated or saved directly.* </br>
*Fields inherited from abstract base classes can be overridden with another field or value, or be removed with None.* </br>
*For many uses, this type of model inheritance will be exactly what you want. It provides a way to factor out common information at the Python level, while still only creating one database table per child model at the database level.* </br>
*To refer to models defined in another application, you can explicitly specify a model with the full application label. For example, if the Manufacturer model above is defined in another application called production, you’d need to use:*
``` python
class Car(models.Model):
    manufacturer = models.ForeignKey(
        'production.Manufacturer',
        on_delete=models.CASCADE,
    )
```
*This sort of reference, called a lazy relationship, can be useful when resolving circular import dependencies between two applications.* </br>

*A database index is automatically created on the ForeignKey. You can disable this by setting db_index to False. You may want to avoid the overhead of an index if you are creating a foreign key for consistency rather than joins, or if you will be creating an alternative index like a partial or multiple column index.*

##### Database Representation
*Behind the scenes, Django appends "_id" to the field name to create its database column name. In the above example, the database table for the Car model will have a manufacturer_id column. (You can change this explicitly by specifying db_column) However, your code should never have to deal with the database column name, unless you write custom SQL. You’ll always deal with the field names of your model object.*
##### Arguments
*ForeignKey accepts other arguments that define the details of how the relation works.* </br>
ForeignKey.on_delete
When an object referenced by a ForeignKey is deleted, Django will emulate the behavior of the SQL constraint specified by the on_delete argument. For example, if you have a nullable ForeignKey and you want it to be set null when the referenced object is deleted:

user = models.ForeignKey(
    User,
    models.SET_NULL,
    blank=True,
    null=True,
)
*on_delete doesn’t create a SQL constraint in the database. Support for database-level cascade options may be implemented later.* </br>

*The possible values for on_delete are found in django.db.models:* </br>

**CASCADE**
*Cascade deletes. Django emulates the behavior of the SQL constraint ON DELETE CASCADE and also deletes the object containing the ForeignKey.* </br>

*Model.delete() isn’t called on related models, but the pre_delete and post_delete signals are sent for all deleted objects.* </br>

**PROTECT**
*Prevent deletion of the referenced object by raising ProtectedError, a subclass of django.db.IntegrityError.* </br>

**SET_NULL**
*Set the ForeignKey null; this is only possible if null is True.* </br>

**SET_DEFAULT**
*Set the ForeignKey to its default value; a default for the ForeignKey must be set.* </br>

**SET()**
*Set the ForeignKey to the value passed to SET(), or if a callable is passed in, the result of calling it. In most cases, passing a callable will be necessary to avoid executing queries at the time your models.py is imported:* </br>
``` python
from django.conf import settings
from django.contrib.auth import get_user_model
from django.db import models

def get_sentinel_user():
    return get_user_model().objects.get_or_create(username='deleted')[0]

class MyModel(models.Model):
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.SET(get_sentinel_user),
    )
```
**DO_NOTHING**
*Take no action. If your database backend enforces referential integrity, this will cause an IntegrityError unless you manually add an SQL ON DELETE constraint to the database field.*

##### `ForeignKey.limit_choices_to`
*Sets a limit to the available choices for this field when this field is rendered using a ModelForm or the admin (by default, all objects in the queryset are available to choose). Either a dictionary, a Q object, or a callable returning a dictionary or Q object can be used.*

*For example:*
``` python
staff_member = models.ForeignKey(
    User,
    on_delete=models.CASCADE,
    limit_choices_to={'is_staff': True},
)
```
*causes the corresponding field on the ModelForm to list only Users that have is_staff=True. This may be helpful in the Django admin.*

*The callable form can be helpful, for instance, when used in conjunction with the Python datetime module to limit selections by date range. For example:*
``` python
def limit_pub_date_choices():
    return {'pub_date__lte': datetime.date.utcnow()}

limit_choices_to = limit_pub_date_choices
```
*If limit_choices_to is or returns a Q object, which is useful for complex queries, then it will only have an effect on the choices available in the admin when the field is not listed in raw_id_fields in the ModelAdmin for the model.*

**Note**

*If a callable is used for limit_choices_to, it will be invoked every time a new form is instantiated. It may also be invoked when a model is validated, for example by management commands or the admin. The admin constructs querysets to validate its form inputs in various edge cases multiple times, so there is a possibility your callable may be invoked several times.*
##### `ForeignKey.related_name`
*The name to use for the relation from the related object back to this one. It’s also the default value for related_query_name (the name to use for the reverse filter name from the target model). See the related objects documentation for a full explanation and example. Note that you must set this value when defining relations on abstract models; and when you do so some special syntax is available.* </br>

*If you’d prefer Django not to create a backwards relation, set related_name to '+' or end it with '+'. For example, this will ensure that the User model won’t have a backwards relation to this model:* </br>
``` python
user = models.ForeignKey(
    User,
    on_delete=models.CASCADE,
    related_name='+',
)
```
##### `ForeignKey.related_query_name`
*The name to use for the reverse filter name from the target model. It defaults to the value of related_name or default_related_name if set, otherwise it defaults to the name of the model:* </br>
``` python
# Declare the ForeignKey with related_query_name
class Tag(models.Model):
    article = models.ForeignKey(
        Article,
        on_delete=models.CASCADE,
        related_name="tags",
        related_query_name="tag",
    )
    name = models.CharField(max_length=255)

# That's now the name of the reverse filter
Article.objects.filter(tag__name="important")
```
*Like related_name, related_query_name supports app label and class interpolation via some special syntax.*

##### `ForeignKey.to_field`
*The field on the related object that the relation is to. By default, Django uses the primary key of the related object. If you reference a different field, that field must have unique=True.*

##### `ForeignKey.db_constraint`
*Controls whether or not a constraint should be created in the database for this foreign key. The default is True, and that’s almost certainly what you want; setting this to False can be very bad for data integrity. That said, here are some scenarios where you might want to do this:* </br>
*You have legacy data that is not valid.* </br>
*You’re sharding your database.* </br>
*If this is set to False, accessing a related object that doesn’t exist will raise its DoesNotExist exception.*

##### `ForeignKey.swappable`
*Controls the migration framework’s reaction if this ForeignKey is pointing at a swappable model. If it is True - the default - then if the ForeignKey is pointing at a model which matches the current value of settings.AUTH_USER_MODEL (or another swappable model setting) the relationship will be stored in the migration using a reference to the setting, not to the model directly.* </br>
*You only want to override this to be False if you are sure your model should always point towards the swapped-in model - for example, if it is a profile model designed specifically for your custom user model.* </br>
*Setting it to False does not mean you can reference a swappable model even if it is swapped out - False just means that the migrations made with this ForeignKey will always reference the exact model you specify (so it will fail hard if the user tries to run with a User model you don’t support, for example).* </br>
*If in doubt, leave it to its default of True.*

#### ManyToManyField
#### `class ManyToManyField(to, **options)`
*A many-to-many relationship. Requires a positional argument: the class to which the model is related, which works exactly the same as it does for ForeignKey, including recursive and lazy relationships.*</br>
*Related objects can be added, removed, or created with the field’s RelatedManager.*

##### Database Representation
*Behind the scenes, Django creates an intermediary join table to represent the many-to-many relationship. By default, this table name is generated using the name of the many-to-many field and the name of the table for the model that contains it. Since some databases don’t support table names above a certain length, these table names will be automatically truncated and a uniqueness hash will be used, e.g. author_books_9cdf. You can manually provide the name of the join table using the db_table option.*

**Arguments**
*ManyToManyField accepts an extra set of arguments – all optional – that control how the relationship functions.*

**ManyToManyField.related_name**
*Same as ForeignKey.related_name.*

**ManyToManyField.related_query_name**
*Same as ForeignKey.related_query_name.*

**ManyToManyField.limit_choices_to**
*Same as ForeignKey.limit_choices_to.* </br>
*limit_choices_to has no effect when used on a ManyToManyField with a custom intermediate table specified using the through parameter.*

**ManyToManyField.symmetrical**
*Only used in the definition of ManyToManyFields on self. Consider the following model:*
``` python
from django.db import models

class Person(models.Model):
    friends = models.ManyToManyField("self")
```
*When Django processes this model, it identifies that it has a ManyToManyField on itself, and as a result, it doesn’t add a person_set attribute to the Person class. Instead, the ManyToManyField is assumed to be symmetrical – that is, if I am your friend, then you are my friend.* </br>
*If you do not want symmetry in many-to-many relationships with self, set symmetrical to False. This will force Django to add the descriptor for the reverse relationship, allowing ManyToManyField relationships to be non-symmetrical.*

##### ManyToManyField.through
*Django will automatically generate a table to manage many-to-many relationships. However, if you want to manually specify the intermediary table, you can use the through option to specify the Django model that represents the intermediate table that you want to use.* </br>
*The most common use for this option is when you want to associate extra data with a many-to-many relationship.* </br>
*If you don’t specify an explicit through model, there is still an implicit through model class you can use to directly access the table created to hold the association. It has three fields to link the models.* </br>
*If the source and target models differ, the following fields are generated:* </br>
*id: the primary key of the relation.* </br>
*<containing_model>_id: the id of the model that declares the ManyToManyField.* </br>
*<other_model>_id: the id of the model that the ManyToManyField points to.* </br>
*If the ManyToManyField points from and to the same model, the following fields are generated:* </br>
*id: the primary key of the relation.*
*from_<model>_id: the id of the instance which points at the model (i.e. the source instance).* </br>
*to_<model>_id: the id of the instance to which the relationship points (i.e. the target model instance).* </br>
*This class can be used to query associated records for a given model instance like a normal model.* </br>

##### ManyToManyField.through_fields
*Only used when a custom intermediary model is specified. Django will normally determine which fields of the intermediary model to use in order to establish a many-to-many relationship automatically. However, consider the following models:*
``` python
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=50)

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(
        Person,
        through='Membership',
        through_fields=('group', 'person'),
    )

class Membership(models.Model):
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    inviter = models.ForeignKey(
        Person,
        on_delete=models.CASCADE,
        related_name="membership_invites",
    )
    invite_reason = models.CharField(max_length=64)
```
*Membership has two foreign keys to Person (person and inviter), which makes the relationship ambiguous and Django can’t know which one to use. In this case, you must explicitly specify which foreign keys Django should use using through_fields, as in the example above.* </br>
*through_fields accepts a 2-tuple ('field1', 'field2'), where field1 is the name of the foreign key to the model the ManyToManyField is defined on (group in this case), and field2 the name of the foreign key to the target model (person in this case).* </br>
*When you have more than one foreign key on an intermediary model to any (or even both) of the models participating in a many-to-many relationship, you must specify through_fields. This also applies to recursive relationships when an intermediary model is used and there are more than two foreign keys to the model, or you want to explicitly specify which two Django should use.* </br>
*Recursive relationships using an intermediary model are always defined as non-symmetrical – that is, with symmetrical=False – therefore, there is the concept of a “source” and a “target”. In that case 'field1' will be treated as the “source” of the relationship and 'field2' as the “target”.*

**ManyToManyField.db_table**
*The name of the table to create for storing the many-to-many data. If this is not provided, Django will assume a default name based upon the names of: the table for the model defining the relationship and the name of the field itself.*

**ManyToManyField.db_constraint**
*Controls whether or not constraints should be created in the database for the foreign keys in the intermediary table. The default is True, and that’s almost certainly what you want; setting this to False can be very bad for data integrity. That said, here are some scenarios where you might want to do this:*

*You have legacy data that is not valid.*
*You’re sharding your database.*
*It is an error to pass both db_constraint and through.*

**ManyToManyField.swappable**
*Controls the migration framework’s reaction if this ManyToManyField is pointing at a swappable model. If it is True - the default - then if the ManyToManyField is pointing at a model which matches the current value of settings.AUTH_USER_MODEL (or another swappable model setting) the relationship will be stored in the migration using a reference to the setting, not to the model directly.* </br>
*You only want to override this to be False if you are sure your model should always point towards the swapped-in model - for example, if it is a profile model designed specifically for your custom user model.* </br>
*If in doubt, leave it to its default of True.* </br>
*ManyToManyField does not support validators.* </br>
*null has no effect since there is no way to require a relationship at the database level.*

#### OneToOneField
#### `class OneToOneField(to, on_delete, parent_link=False, **options)`
*A one-to-one relationship. Conceptually, this is similar to a ForeignKey with unique=True, but the “reverse” side of the relation will directly return a single object.* </br>
*This is most useful as the primary key of a model which “extends” another model in some way; Multi-table inheritance is implemented by adding an implicit one-to-one relation from the child model to the parent model, for example.* </br>
*One positional argument is required: the class to which the model will be related. This works exactly the same as it does for ForeignKey, including all the options regarding recursive and lazy relationships.* </br>
*If you do not specify the related_name argument for the OneToOneField, Django will use the lowercase name of the current model as default value.* </br>

*With the following example:*
``` python
from django.conf import settings
from django.db import models

class MySpecialUser(models.Model):
    user = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
    )
    supervisor = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name='supervisor_of',
    )
```
*your resulting User model will have the following attributes:*
``` python
>>> user = User.objects.get(pk=1)
>>> hasattr(user, 'myspecialuser')
True
>>> hasattr(user, 'supervisor_of')
True
```
*A DoesNotExist exception is raised when accessing the reverse relationship if an entry in the related table doesn’t exist. For example, if a user doesn’t have a supervisor designated by MySpecialUser:*
``` python
>>> user.supervisor_of
Traceback (most recent call last):
 ```
*DoesNotExist: User matching query does not exist.*
*Additionally, OneToOneField accepts all of the extra arguments accepted by ForeignKey, plus one extra argument:*

**OneToOneField.parent_link**
*When True and used in a model which inherits from another concrete model, indicates that this field should be used as the link back to the parent class, rather than the extra OneToOneField which would normally be implicitly created by subclassing.*
### Field options
#### null
##### Field.null
*If True, Django will store empty values as NULL in the database. Default is False.* </br>
*Avoid using null on string-based fields such as CharField and TextField. If a string-based field has null=True, that means it has two possible values for “no data”: NULL, and the empty string. In most cases, it’s redundant to have two possible values for “no data;” the Django convention is to use the empty string, not NULL. One exception is when a CharField has both unique=True and blank=True set. In this situation, null=True is required to avoid unique constraint violations when saving multiple objects with blank values.* </br>
*For both string-based and non-string-based fields, you will also need to set blank=True if you wish to permit empty values in forms, as the null parameter only affects database storage (see blank).*
#### blank
##### Field.blank
*If True, the field is allowed to be blank. Default is False.* </br>
*Note that this is different than null. null is purely database-related, whereas blank is validation-related. If a field has blank=True, form validation will allow entry of an empty value. If a field has blank=False, the field will be required.*

#### choices
##### Field.choices
*A sequence consisting itself of iterables of exactly two items (e.g. [(A, B), (A, B) ...]) to use as choices for this field. If choices are given, they’re enforced by model validation and the default form widget will be a select box with these choices instead of the standard text field.* </br>
*The first element in each tuple is the actual value to be set on the model, and the second element is the human-readable name. For example:*
``` python
YEAR_IN_SCHOOL_CHOICES = [
    ('FR', 'Freshman'),
    ('SO', 'Sophomore'),
    ('JR', 'Junior'),
    ('SR', 'Senior'),
]
```
*Generally, it’s best to define choices inside a model class, and to define a suitably-named constant for each value:*
``` python
from django.db import models

class Student(models.Model):
    FRESHMAN = 'FR'
    SOPHOMORE = 'SO'
    JUNIOR = 'JR'
    SENIOR = 'SR'
    YEAR_IN_SCHOOL_CHOICES = [
        (FRESHMAN, 'Freshman'),
        (SOPHOMORE, 'Sophomore'),
        (JUNIOR, 'Junior'),
        (SENIOR, 'Senior'),
    ]
    year_in_school = models.CharField(
        max_length=2,
        choices=YEAR_IN_SCHOOL_CHOICES,
        default=FRESHMAN,
    )

    def is_upperclass(self):
        return self.year_in_school in (self.JUNIOR, self.SENIOR)
```
*Though you can define a choices list outside of a model class and then refer to it, defining the choices and names for each choice inside the model class keeps all of that information with the class that uses it, and makes the choices easy to reference (e.g, Student.SOPHOMORE will work anywhere that the Student model has been imported).* </br>
*You can also collect your available choices into named groups that can be used for organizational purposes:*
``` python
MEDIA_CHOICES = [
    ('Audio', (
            ('vinyl', 'Vinyl'),
            ('cd', 'CD'),
        )
    ),
    ('Video', (
            ('vhs', 'VHS Tape'),
            ('dvd', 'DVD'),
        )
    ),
    ('unknown', 'Unknown'),
]
```
*The first element in each tuple is the name to apply to the group. The second element is an iterable of 2-tuples, with each 2-tuple containing a value and a human-readable name for an option. Grouped options may be combined with ungrouped options within a single list (such as the unknown option in this example).* </br>
*For each model field that has choices set, Django will add a method to retrieve the human-readable name for the field’s current value. See get_FOO_display() in the database API documentation.* </br>
*Note that choices can be any sequence object – not necessarily a list or tuple. This lets you construct choices dynamically. But if you find yourself hacking choices to be dynamic, you’re probably better off using a proper database table with a ForeignKey. choices is meant for static data that doesn’t change much, if ever.*

#### db_column
##### `Field.db_column`
*The name of the database column to use for this field. If this isn’t given, Django will use the field’s name.* </br>
*If your database column name is an SQL reserved word, or contains characters that aren’t allowed in Python variable names – notably, the hyphen – that’s OK. Django quotes column and table names behind the scenes.*

#### db_index
##### `Field.db_index`
*If True, a database index will be created for this field.*

##### `db_tablespace`
#### Field.db_tablespace
*The name of the database tablespace to use for this field’s index, if this field is indexed. The default is the project’s DEFAULT_INDEX_TABLESPACE setting, if set, or the db_tablespace of the model, if any. If the backend doesn’t support tablespaces for indexes, this option is ignored.*

#### default
##### Field.default
*The default value for the field. This can be a value or a callable object. If callable it will be called every time a new object is created.* </br>
*The default can’t be a mutable object (model instance, list, set, etc.), as a reference to the same instance of that object would be used as the default value in all new model instances. Instead, wrap the desired default in a callable. For example, if you want to specify a default dict for JSONField, use a function:*
``` python
def contact_default():
    return {"email": "to1@example.com"}

contact_info = JSONField("ContactInfo", default=contact_default)
```
*lambdas can’t be used for field options like default because they can’t be serialized by migrations. See that documentation for other caveats.* </br>
*For fields like ForeignKey that map to model instances, defaults should be the value of the field they reference (pk unless to_field is set) instead of model instances.* </br>
*The default value is used when new model instances are created and a value isn’t provided for the field. When the field is a primary key, the default is also used when the field is set to None.*

#### editable
##### `Field.editable`
*If False, the field will not be displayed in the admin or any other ModelForm. They are also skipped during model validation. Default is True.*

#### error_messages
##### `Field.error_messages`
*The error_messages argument lets you override the default messages that the field will raise. Pass in a dictionary with keys matching the error messages you want to override.* </br>
*Error message keys include null, blank, invalid, invalid_choice, unique, and unique_for_date. Additional error message keys are specified for each field in the Field types section below.* </br>
*These error messages often don’t propagate to forms. See Considerations regarding model’s error_messages.*

#### help_text
##### `Field.help_text`
*Extra “help” text to be displayed with the form widget. It’s useful for documentation even if your field isn’t used on a form.* </br>
*Note that this value is not HTML-escaped in automatically-generated forms. This lets you include HTML in help_text if you so desire. For example:* </br>
`help_text="Please use the following format: <em>YYYY-MM-DD</em>."`
*Alternatively you can use plain text and django.utils.html.escape() to escape any HTML special characters. Ensure that you escape any help text that may come from untrusted users to avoid a cross-site scripting attack.*

#### primary_key
##### `Field.primary_key`
*If True, this field is the primary key for the model.* </br>
*If you don’t specify primary_key=True for any field in your model, Django will automatically add an AutoField to hold the primary key, so you don’t need to set primary_key=True on any of your fields unless you want to override the default primary-key behavior. For more, see Automatic primary key fields.* </br>
*primary_key=True implies null=False and unique=True. Only one primary key is allowed on an object.* </br>
*The primary key field is read-only. If you change the value of the primary key on an existing object and then save it, a new object will be created alongside the old one.*

#### unique
##### `Field.unique`
*If True, this field must be unique throughout the table.* </br>
*This is enforced at the database level and by model validation. If you try to save a model with a duplicate value in a unique field, a django.db.IntegrityError will be raised by the model’s save() method.* </br>
*This option is valid on all field types except ManyToManyField and OneToOneField.* </br>
*Note that when unique is True, you don’t need to specify db_index, because unique implies the creation of an index.*

#### unique_for_date
##### `Field.unique_for_date`
*Set this to the name of a DateField or DateTimeField to require that this field be unique for the value of the date field.* </br>
*For example, if you have a field title that has unique_for_date="pub_date", then Django wouldn’t allow the entry of two records with the same title and pub_date.* </br>
*Note that if you set this to point to a DateTimeField, only the date portion of the field will be considered. Besides, when USE_TZ is True, the check will be performed in the current time zone at the time the object gets saved.* </br>
*This is enforced by Model.validate_unique() during model validation but not at the database level. If any unique_for_date constraint involves fields that are not part of a ModelForm (for example, if one of the fields is listed in exclude or has editable=False), Model.validate_unique() will skip validation for that particular constraint.*

#### unique_for_month
##### `Field.unique_for_month`
*Like unique_for_date, but requires the field to be unique with respect to the month.*

#### unique_for_year
##### `Field.unique_for_year`
*Like unique_for_date and unique_for_month.*

#### verbose_name
##### `Field.verbose_name`
*A human-readable name for the field. If the verbose name isn’t given, Django will automatically create it using the field’s attribute name, converting underscores to spaces. See Verbose field names.*

#### validators
##### `Field.validators`
*A list of validators to run for this field. See the validators documentation for more information.*

#### Registering and fetching lookups
*Field implements the lookup registration API. The API can be used to customize which lookups are available for a field class, and how lookups are fetched from a field.*

## Model Meta options
*This document explains all the possible metadata options that you can give your model in its internal class Meta.*

### Available Meta options
#### abstract
**Options.abstract**
*If abstract = True, this model will be an abstract base class.*

#### app_label
**Options.app_label**
*If a model is defined outside of an application in INSTALLED_APPS, it must declare which app it belongs to:*
``` python
app_label = 'myapp'
```
*If you want to represent a model with the format app_label.object_name or app_label.model_name you can use model._meta.label or model._meta.label_lower respectively.*

#### base_manager_name
**Options.base_manager_name**
*The attribute name of the manager, for example, 'objects', to use for the model’s _base_manager.*

#### db_table
**Options.db_table**
*The name of the database table to use for the model:*

`db_table = 'music_album'`
</br>
*Table names*
*To save you time, Django automatically derives the name of the database table from the name of your model class and the app that contains it. A model’s database table name is constructed by joining the model’s “app label” – the name you used in manage.py startapp – to the model’s class name, with an underscore between them.* </br>
*For example, if you have an app bookstore (as created by manage.py startapp bookstore), a model defined as class Book will have a database table named bookstore_book.* </br>
*To override the database table name, use the db_table parameter in class Meta.*

#### default_manager_name
**Options.default_manager_name**
*The name of the manager to use for the model’s _default_manager.*

#### default_related_name
**Options.default_related_name**
*The name that will be used by default for the relation from a related object back to this one. The default is <model_name>_set.* </br>
*This option also sets related_query_name.* </br>
*As the reverse name for a field should be unique, be careful if you intend to subclass your model. To work around name collisions, part of the name should contain '%(app_label)s' and '%(model_name)s', which are replaced respectively by the name of the application the model is in, and the name of the model, both lowercased. See the paragraph on related names for abstract models.*

#### get_latest_by
**Options.get_latest_by**
*The name of a field or a list of field names in the model, typically DateField, DateTimeField, or IntegerField. This specifies the default field(s) to use in your model Manager’s latest() and earliest() methods.*
</br>
*Example:*
``` python
# Latest by ascending order_date.
get_latest_by = "order_date"

# Latest by priority descending, order_date ascending.
get_latest_by = ['-priority', 'order_date']
```
#### order_with_respect_to
**Options.order_with_respect_to**
*Makes this object orderable with respect to the given field, usually a ForeignKey. This can be used to make related objects orderable with respect to a parent object. For example, if an Answer relates to a Question object, and a question has more than one answer, and the order of answers matters, you’d do this:*
``` python
from django.db import models

class Question(models.Model):
    text = models.TextField()
    # ...

class Answer(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    # ...

    class Meta:
        order_with_respect_to = 'question'
```
*When order_with_respect_to is set, two additional methods are provided to retrieve and to set the order of the related objects: get_RELATED_order() and set_RELATED_order(), where RELATED is the lowercased model name. For example, assuming that a Question object has multiple related Answer objects, the list returned contains the primary keys of the related Answer objects:*
``` python
>>> question = Question.objects.get(id=1)
>>> question.get_answer_order()
[1, 2, 3]
```
*The order of a Question object’s related Answer objects can be set by passing in a list of Answer primary keys:*
``` python
>>> question.set_answer_order([3, 1, 2])
```
*The related objects also get two methods, get_next_in_order() and get_previous_in_order(), which can be used to access those objects in their proper order. Assuming the Answer objects are ordered by id:*
``` python
>>> answer = Answer.objects.get(id=2)
>>> answer.get_next_in_order()
<Answer: 3>
>>> answer.get_previous_in_order()
<Answer: 1>
```

#### ordering
**Options.ordering**
*The default ordering for the object, for use when obtaining lists of objects:* </br>
`ordering = ['-order_date']`
*This is a tuple or list of strings and/or query expressions. Each string is a field name with an optional “-” prefix, which indicates descending order. Fields without a leading “-” will be ordered ascending. Use the string “?” to order randomly.* </br>

*For example, to order by a pub_date field ascending, use this:* </br>
`ordering = ['pub_date']`
</br>
*To order by pub_date descending, use this:* </br>
`ordering = ['-pub_date']`
</br>
*To order by pub_date descending, then by author ascending, use this:* </br>
`ordering = ['-pub_date', 'author']`
</br>
*You can also use query expressions. To order by author ascending and make null values sort last, use this:*
``` python
from django.db.models import F

ordering = [F('author').asc(nulls_last=True)]
```

##### permissions
**Options.permissions**
*Extra permissions to enter into the permissions table when creating this object. Add, change, delete, and view permissions are automatically created for each model. This example specifies an extra permission, can_deliver_pizzas:*
</br>
``` python
permissions = [('can_deliver_pizzas', 'Can deliver pizzas')]
```
*This is a list or tuple of 2-tuples in the format (permission_code, human_readable_permission_name).*

##### default_permissions
**Options.default_permissions**
*Defaults to ('add', 'change', 'delete', 'view'). You may customize this list, for example, by setting this to an empty list if your app doesn’t require any of the default permissions. It must be specified on the model before the model is created by migrate in order to prevent any omitted permissions from being created.*
</br>
*Changed in Django 2.1:*
*The view permission was added.*

##### proxy
**Options.proxy**
*If proxy = True, a model which subclasses another model will be treated as a proxy model.*

##### required_db_features
**Options.required_db_features**
*List of database features that the current connection should have so that the model is considered during the migration phase. For example, if you set this list to `['gis_enabled']`, the model will only be synchronized on GIS-enabled databases. It’s also useful to skip some models when testing with several database backends. Avoid relations between models that may or may not be created as the ORM doesn’t handle this.*

##### required_db_vendor
**Options.required_db_vendor**
*Name of a supported database vendor that this model is specific to. Current built-in vendor names are: sqlite, postgresql, mysql, oracle. If this attribute is not empty and the current connection vendor doesn’t match it, the model will not be synchronized.*

##### select_on_save
**Options.select_on_save**
*Determines if Django will use the pre-1.6 `django.db.models.Model.save() algorithm`. The old algorithm uses SELECT to determine if there is an existing row to be updated. The new algorithm tries an UPDATE directly. In some rare cases the UPDATE of an existing row isn’t visible to Django. An example is the PostgreSQL ON UPDATE trigger which returns NULL. In such cases the new algorithm will end up doing an INSERT even when a row exists in the database.*
</br>
*Usually there is no need to set this attribute. The default is False.*

##### indexes
**Options.indexes**
*A list of indexes that you want to define on the model:*
``` python
from django.db import models

class Customer(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)

    class Meta:
        indexes = [
            models.Index(fields=['last_name', 'first_name']),
            models.Index(fields=['first_name'], name='first_name_idx'),
        ]
```

##### unique_together
**Options.unique_together**
*se UniqueConstraint with the constraints option instead.*
</br>
UniqueConstraint provides more functionality than unique_together. unique_together may be deprecated in the future.*
</br>
*Sets of field names that, taken together, must be unique:*
</br>
`unique_together = [['driver', 'restaurant']]`
*This is a list of lists that must be unique when considered together. It’s used in the Django admin and is enforced at the database level (i.e., the appropriate UNIQUE statements are included in the CREATE TABLE statement).*
</br>
*For convenience, unique_together can be a single list when dealing with a single set of fields:*
</br>
`unique_together = ['driver', 'restaurant']`
*A ManyToManyField cannot be included in unique_together. (It’s not clear what that would even mean!) If you need to validate uniqueness related to a ManyToManyField, try using a signal or an explicit through model.*
</br>
*The ValidationError raised during model validation when the constraint is violated has the unique_together error code.*

##### verbose_name
**Options.verbose_name**
*A human-readable name for the object, singular:*
</br>
`verbose_name = "pizza"`
</br>
*If this isn’t given, Django will use a munged version of the class name: CamelCase becomes camel case.*

##### verbose_name_plural
**Options.verbose_name_plural**
*The plural name for the object:*
</br>
`verbose_name_plural = "stories"`
</br>
*If this isn’t given, Django will use verbose_name + "s".*

#### Read-only Meta attributes
##### label
**Options.label**
*Representation of the object, returns app_label.object_name, e.g. 'polls.Question'.*

##### label_lower
**Options.label_lower**
*Representation of the model, returns app_label.model_name, e.g. 'polls.question'.*

### Model methods
*Define custom methods on a model to add custom “row-level” functionality to your objects. Whereas Manager methods are intended to do “table-wide” things, model methods should act on a particular model instance.* </br>
*This is a valuable technique for keeping business logic in one place – the model.* </br>
*For example, this model has a few custom methods:*
``` python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    birth_date = models.DateField()

    def baby_boomer_status(self):
        "Returns the person's baby-boomer status."
        import datetime
        if self.birth_date < datetime.date(1945, 8, 1):
            return "Pre-boomer"
        elif self.birth_date < datetime.date(1965, 1, 1):
            return "Baby boomer"
        else:
            return "Post-boomer"

    @property
    def full_name(self):
        "Returns the person's full name."
        return '%s %s' % (self.first_name, self.last_name)
```
*The last method in this example is a property.* </br>
*The model instance reference has a complete list of methods automatically given to each model. You can override most of these – see overriding predefined model methods, below – but there are a couple that you’ll almost always want to define:*

#### __str__()
**Model.__str__()**
*The` __str__()` method is called whenever you call str() on an object. Django uses str(obj) in a number of places. Most notably, to display an object in the Django admin site and as the value inserted into a template when it displays an object. Thus, you should always return a nice, human-readable representation of the model from the` __str__()` method.*
</br>
For example:
``` python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)

    def __str__(self):
        return '%s %s' % (self.first_name, self.last_name)
```

#### Creating objects
*To create a new instance of a model, just instantiate it like any other Python class:* </br>
**`class Model(**kwargs)`**
*The keyword arguments are simply the names of the fields you’ve defined on your model. Note that instantiating a model in no way touches your database; for that, you need to save().* </br>

*Note* </br>

*You may be tempted to customize the model by overriding the __init__ method. If you do so, however, take care not to change the calling signature as any change may prevent the model instance from being saved. Rather than overriding __init__, try using one of these approaches:* </br>

*Add a classmethod on the model class:*
``` python
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=100)

    @classmethod
    def create(cls, title):
        book = cls(title=title)
        # do something with the book
        return book

book = Book.create("Pride and Prejudice")
```
*Add a method on a custom manager (usually preferred):*
``` python
class BookManager(models.Manager):
    def create_book(self, title):
        book = self.create(title=title)
        # do something with the book
        return book

class Book(models.Model):
    title = models.CharField(max_length=100)

    objects = BookManager()

book = Book.objects.create_book("Pride and Prejudice")
```

#### Refreshing objects from database
*If you delete a field from a model instance, accessing it again reloads the value from the database:*
``` python
>>> obj = MyModel.objects.first()
>>> del obj.field
>>> obj.field  # Loads the field from the database
```
**`Model.refresh_from_db(using=None, fields=None)`**
*If you need to reload a model’s values from the database, you can use the refresh_from_db() method. When this method is called without arguments the following is done:* </br>

*All non-deferred fields of the model are updated to the values currently present in the database.* </br>
*Any cached relations are cleared from the reloaded instance.* </br>
*Only fields of the model are reloaded from the database. Other database-dependent values such as annotations aren’t reloaded. Any @cached_property attributes aren’t cleared either.* </br>

*The reloading happens from the database the instance was loaded from, or from the default database if the instance wasn’t loaded from the database. The using argument can be used to force the database used for reloading.* </br>
*It is possible to force the set of fields to be loaded by using the fields argument.* </br>
*For example, to test that an update() call resulted in the expected update, you could write a test similar to this:*

### Validating objects
There are three steps involved in validating a model:
</br>
Validate the model fields - Model.clean_fields() </br>
Validate the model as a whole - Model.clean() </br>
Validate the field uniqueness - Model.validate_unique() </br>
All three steps are performed when you call a model’s full_clean() method.
</br>
When you use a ModelForm, the call to is_valid() will perform these validation steps for all the fields that are included on the form. See the ModelForm documentation for more information. You should only need to call a model’s full_clean() method if you plan to handle validation errors yourself, or if you have excluded fields from the ModelForm that require validation.

**`Model.full_clean(exclude=None, validate_unique=True)`**
This method calls Model.clean_fields(), Model.clean(), and Model.validate_unique() (if validate_unique is True), in that order and raises a ValidationError that has a message_dict attribute containing errors from all three stages.
</br>
The optional exclude argument can be used to provide a list of field names that can be excluded from validation and cleaning. ModelForm uses this argument to exclude fields that aren’t present on your form from being validated since any errors raised could not be corrected by the user.
</br>
Note that full_clean() will not be called automatically when you call your model’s save() method. You’ll need to call it manually when you want to run one-step model validation for your own manually created models. For example:
</br>
``` python
from django.core.exceptions import ValidationError
try:
    article.full_clean()
except ValidationError as e:
    # Do something based on the errors contained in e.message_dict.
    # Display them to a user, or handle them programmatically.
    pass
```
The first step full_clean() performs is to clean each individual field.
</br>
**`Model.clean_fields(exclude=None)`**
This method will validate all fields on your model. The optional exclude argument lets you provide a list of field names to exclude from validation. It will raise a ValidationError if any fields fail validation.
</br>
The second step full_clean() performs is to call Model.clean(). This method should be overridden to perform custom validation on your model.

#### Model.clean()
This method should be used to provide custom model validation, and to modify attributes on your model if desired. For instance, you could use it to automatically provide a value for a field, or to do validation that requires access to more than a single field:
``` python
import datetime
from django.core.exceptions import ValidationError
from django.db import models
from django.utils.translation import gettext_lazy as _

class Article(models.Model):
    ...
    def clean(self):
        # Don't allow draft entries to have a pub_date.
        if self.status == 'draft' and self.pub_date is not None:
            raise ValidationError(_('Draft entries may not have a publication date.'))
        # Set the pub_date for published items if it hasn't been set already.
        if self.status == 'published' and self.pub_date is None:
            self.pub_date = datetime.date.today()
 ```
Note, however, that like Model.full_clean(), a model’s clean() method is not invoked when you call your model’s save() method.
</br>
In the above example, the ValidationError exception raised by Model.clean() was instantiated with a string, so it will be stored in a special error dictionary key, NON_FIELD_ERRORS. This key is used for errors that are tied to the entire model instead of to a specific field:
</br>
``` python
from django.core.exceptions import NON_FIELD_ERRORS, ValidationError
try:
    article.full_clean()
except ValidationError as e:
    non_field_errors = e.message_dict[NON_FIELD_ERRORS]
```
To assign exceptions to a specific field, instantiate the ValidationError with a dictionary, where the keys are the field names. We could update the previous example to assign the error to the pub_date field:
``` python
class Article(models.Model):
    ...
    def clean(self):
        # Don't allow draft entries to have a pub_date.
        if self.status == 'draft' and self.pub_date is not None:
            raise ValidationError({'pub_date': _('Draft entries may not have a publication date.')})
        ...
```
If you detect errors in multiple fields during Model.clean(), you can also pass a dictionary mapping field names to errors:
``` python
raise ValidationError({
    'title': ValidationError(_('Missing title.'), code='required'),
    'pub_date': ValidationError(_('Invalid date.'), code='invalid'),
})
```
Finally, full_clean() will check any unique constraints on your model.
</br>
How to raise field-specific validation errors if those fields don’t appear in a ModelForm
</br>
You can’t raise validation errors in Model.clean() for fields that don’t appear in a model form (a form may limit its fields using Meta.fields or Meta.exclude). Doing so will raise a ValueError because the validation error won’t be able to be associated with the excluded field.
</br>
To work around this dilemma, instead override Model.clean_fields() as it receives the list of fields that are excluded from validation. For example:
``` python
class Article(models.Model):
    ...
    def clean_fields(self, exclude=None):
        super().clean_fields(exclude=exclude)
        if self.status == 'draft' and self.pub_date is not None:
            if exclude and 'status' in exclude:
                raise ValidationError(
                    _('Draft entries may not have a publication date.')
                )
            else:
                raise ValidationError({
                    'status': _(
                        'Set status to draft if there is not a '
                        'publication date.'
                     ),
                })
```
**Model.validate_unique(exclude=None)**
This method is similar to clean_fields(), but validates all uniqueness constraints on your model instead of individual field values. The optional exclude argument allows you to provide a list of field names to exclude from validation. It will raise a ValidationError if any fields fail validation.
</br>
Note that if you provide an exclude argument to validate_unique(), any unique_together constraint involving one of the fields you provided will not be checked.

### Saving objects
To save an object back to the database, call save():
</br>
**Model.save(force_insert=False, force_update=False, using=DEFAULT_DB_ALIAS, update_fields=None)**
If you want customized saving behavior, you can override this save() method. See Overriding predefined model methods for more details.
</br>
The model save process also has some subtleties; see the sections below.
</br>
**Auto-incrementing primary keys**
</br>
If a model has an AutoField — an auto-incrementing primary key — then that auto-incremented value will be calculated and saved as an attribute on your object the first time you call save():
``` python
>>> b2 = Blog(name='Cheddar Talk', tagline='Thoughts on cheese.')
>>> b2.id     # Returns None, because b2 doesn't have an ID yet.
>>> b2.save()
>>> b2.id     # Returns the ID of your new object.
```
There’s no way to tell what the value of an ID will be before you call save(), because that value is calculated by your database, not by Django.
</br>
For convenience, each model has an AutoField named id by default unless you explicitly specify primary_key=True on a field in your model. See the documentation for AutoField for more details.
</br>
**The pk property**
**Model.pk**
</br>
Regardless of whether you define a primary key field yourself, or let Django supply one for you, each model will have a property called pk. It behaves like a normal attribute on the model, but is actually an alias for whichever attribute is the primary key field for the model. You can read and set this value, just as you would for any other attribute, and it will update the correct field in the model.
</br>
**Explicitly specifying auto-primary-key values**
</br>
If a model has an AutoField but you want to define a new object’s ID explicitly when saving, just define it explicitly before saving, rather than relying on the auto-assignment of the ID:
``` python
>>> b3 = Blog(id=3, name='Cheddar Talk', tagline='Thoughts on cheese.')
>>> b3.id     # Returns 3.
>>> b3.save()
>>> b3.id     # Returns 3.
```
If you assign auto-primary-key values manually, make sure not to use an already-existing primary-key value! If you create a new object with an explicit primary-key value that already exists in the database, Django will assume you’re changing the existing record rather than creating a new one.
</br>
Given the above 'Cheddar Talk' blog example, this example would override the previous record in the database:
``` python
b4 = Blog(id=3, name='Not Cheddar', tagline='Anything but cheese.')
b4.save()  # Overrides the previous blog with ID=3!
```
See How Django knows to UPDATE vs. INSERT, below, for the reason this happens.
</br>
Explicitly specifying auto-primary-key values is mostly useful for bulk-saving objects, when you’re confident you won’t have primary-key collision.
</br>
If you’re using PostgreSQL, the sequence associated with the primary key might need to be updated; see Manually-specifying values of auto-incrementing primary keys.
</br>
**What happens when you save?**
</br>
When you save an object, Django performs the following steps:
</br>
Emit a pre-save signal. The pre_save signal is sent, allowing any functions listening for that signal to do something.
</br>
Preprocess the data. Each field’s pre_save() method is called to perform any automated data modification that’s needed. For example, the date/time fields override pre_save() to implement auto_now_add and auto_now.
</br>
Prepare the data for the database. Each field’s get_db_prep_save() method is asked to provide its current value in a data type that can be written to the database.
</br>
Most fields don’t require data preparation. Simple data types, such as integers and strings, are ‘ready to write’ as a Python object. However, more complex data types often require some modification.
</br>
For example, DateField fields use a Python datetime object to store data. Databases don’t store datetime objects, so the field value must be converted into an ISO-compliant date string for insertion into the database.
</br>
Insert the data into the database. The preprocessed, prepared data is composed into an SQL statement for insertion into the database.
</br>
Emit a post-save signal. The post_save signal is sent, allowing any functions listening for that signal to do something.
</br>
**How Django knows to UPDATE vs. INSERT**
</br>
You may have noticed Django database objects use the same save() method for creating and changing objects. Django abstracts the need to use INSERT or UPDATE SQL statements. Specifically, when you call save(), Django follows this algorithm:
</br>
If the object’s primary key attribute is set to a value that evaluates to True (i.e., a value other than None or the empty string), Django executes an UPDATE. </br>
If the object’s primary key attribute is not set or if the UPDATE didn’t update anything (e.g. if primary key is set to a value that doesn’t exist in the database), Django executes an INSERT. </br>
The one gotcha here is that you should be careful not to specify a primary-key value explicitly when saving new objects, if you cannot guarantee the primary-key value is unused. For more on this nuance, see Explicitly specifying auto-primary-key values above and Forcing an INSERT or UPDATE below. </br>

In Django 1.5 and earlier, Django did a SELECT when the primary key attribute was set. If the SELECT found a row, then Django did an UPDATE, otherwise it did an INSERT. The old algorithm results in one more query in the UPDATE case. There are some rare cases where the database doesn’t report that a row was updated even if the database contains a row for the object’s primary key value. An example is the PostgreSQL ON UPDATE trigger which returns NULL. In such cases it is possible to revert to the old algorithm by setting the select_on_save option to True. </br>

**Forcing an INSERT or UPDATE**
In some rare circumstances, it’s necessary to be able to force the save() method to perform an SQL INSERT and not fall back to doing an UPDATE. Or vice-versa: update, if possible, but not insert a new row. In these cases you can pass the force_insert=True or force_update=True parameters to the save() method. Obviously, passing both parameters is an error: you cannot both insert and update at the same time! </br>

It should be very rare that you’ll need to use these parameters. Django will almost always do the right thing and trying to override that will lead to errors that are difficult to track down. This feature is for advanced use only. </br>

Using update_fields will force an update similarly to force_update. </br>

**Updating attributes based on existing fields**
Sometimes you’ll need to perform a simple arithmetic task on a field, such as incrementing or decrementing the current value. The obvious way to achieve this is to do something like:
``` python
>>> product = Product.objects.get(name='Venezuelan Beaver Cheese')
>>> product.number_sold += 1
>>> product.save()
```
If the old number_sold value retrieved from the database was 10, then the value of 11 will be written back to the database.
</br>
The process can be made robust, avoiding a race condition, as well as slightly faster by expressing the update relative to the original field value, rather than as an explicit assignment of a new value. Django provides F expressions for performing this kind of relative update. Using F expressions, the previous example is expressed as:
``` python
>>> from django.db.models import F
>>> product = Product.objects.get(name='Venezuelan Beaver Cheese')
>>> product.number_sold = F('number_sold') + 1
>>> product.save()
```
For more details, see the documentation on F expressions and their use in update queries.
</br>
**Specifying which fields to save**
</br>
If save() is passed a list of field names in keyword argument update_fields, only the fields named in that list will be updated. This may be desirable if you want to update just one or a few fields on an object. There will be a slight performance benefit from preventing all of the model fields from being updated in the database. For example:
``` python
product.name = 'Name changed again'
product.save(update_fields=['name'])
```
The update_fields argument can be any iterable containing strings. An empty update_fields iterable will skip the save. A value of None will perform an update on all fields.
</br>
Specifying update_fields will force an update.
</br>
When saving a model fetched through deferred model loading (only() or defer()) only the fields loaded from the DB will get updated. In effect there is an automatic update_fields in this case. If you assign or change any deferred field value, the field will be added to the updated fields.
</br>
### Deleting objects
**Model.delete(using=DEFAULT_DB_ALIAS, keep_parents=False)**
Issues an SQL DELETE for the object. This only deletes the object in the database; the Python instance will still exist and will still have data in its fields. This method returns the number of objects deleted and a dictionary with the number of deletions per object type.
</br>
For more details, including how to delete objects in bulk, see Deleting objects.
</br>
If you want customized deletion behavior, you can override the delete() method. See Overriding predefined model methods for more details.
</br>
Sometimes with multi-table inheritance you may want to delete only a child model’s data. Specifying keep_parents=True will keep the parent model’s data.

### __hash__()
#### Model.__hash__()
The __hash__() method is based on the instance’s primary key value. It is effectively hash(obj.pk). If the instance doesn’t have a primary key value then a TypeError will be raised (otherwise the __hash__() method would return different values before and after the instance is saved, but changing the __hash__() value of an instance is forbidden in Python.

### get_absolute_url()
#### Model.get_absolute_url()
Define a get_absolute_url() method to tell Django how to calculate the canonical URL for an object. To callers, this method should appear to return a string that can be used to refer to the object over HTTP.
</br>
For example:
``` python
def get_absolute_url(self):
    return "/people/%i/" % self.id
```
While this code is correct and simple, it may not be the most portable way to to write this kind of method. The reverse() function is usually the best approach.
</br>
For example:
``` python
def get_absolute_url(self):
    from django.urls import reverse
    return reverse('people.views.details', args=[str(self.id)])
```
One place Django uses get_absolute_url() is in the admin app. If an object defines this method, the object-editing page will have a “View on site” link that will jump you directly to the object’s public view, as given by get_absolute_url().
</br>
Similarly, a couple of other bits of Django, such as the syndication feed framework, use get_absolute_url() when it is defined. If it makes sense for your model’s instances to each have a unique URL, you should define get_absolute_url().
</br>
Warning
</br>
You should avoid building the URL from unvalidated user input, in order to reduce possibilities of link or redirect poisoning:
``` python
def get_absolute_url(self):
    return '/%s/' % self.name
```
If self.name is '/example.com' this returns '//example.com/' which, in turn, is a valid schema relative URL but not the expected '/%2Fexample.com/'.
</br>
It’s good practice to use get_absolute_url() in templates, instead of hard-coding your objects’ URLs. For example, this template code is bad:
``` python
<!-- BAD template code. Avoid! -->
<a href="/people/{{ object.id }}/">{{ object.name }}</a>
```
This template code is much better:
``` python
<a href="{{ object.get_absolute_url }}">{{ object.name }}</a>
```
The logic here is that if you change the URL structure of your objects, even for something simple such as correcting a spelling error, you don’t want to have to track down every place that the URL might be created. Specify it once, in get_absolute_url() and have all your other code call that one place.

#### Model.get_FOO_display()
For every field that has choices set, the object will have a get_FOO_display() method, where FOO is the name of the field. This method returns the “human-readable” value of the field.
</br>
For example:
``` python
from django.db import models

class Person(models.Model):
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    shirt_size = models.CharField(max_length=2, choices=SHIRT_SIZES)
>>> p = Person(name="Fred Flintstone", shirt_size="L")
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'
```
**`Model.get_next_by_FOO(**kwargs)`** </br>
**`Model.get_previous_by_FOO(**kwargs)`** </br>
For every DateField and DateTimeField that does not have null=True, the object will have get_next_by_FOO() and get_previous_by_FOO() methods, where FOO is the name of the field. This returns the next and previous object with respect to the date field, raising a DoesNotExist exception when appropriate.
</br>
Both of these methods will perform their queries using the default manager for the model. If you need to emulate filtering used by a custom manager, or want to perform one-off custom filtering, both methods also accept optional keyword arguments, which should be in the format described in Field lookups.
</br>
Note that in the case of identical date values, these methods will use the primary key as a tie-breaker. This guarantees that no records are skipped or duplicated. That also means you cannot use those methods on unsaved objects.

#### Other attributes
#### DoesNotExist
**`exception Model.DoesNotExist`**
</br>
This exception is raised by the ORM in a couple places, for example by QuerySet.get() when an object is not found for the given query parameters.
</br>
Django provides a DoesNotExist exception as an attribute of each model class to identify the class of object that could not be found and to allow you to catch a particular model class with try/except. The exception is a subclass of `django.core.exceptions.ObjectDoesNotExist.`
