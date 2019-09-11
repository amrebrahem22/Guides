## Models
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
# For DateTimeField: default=timezone.now - 
from django.utils.timezone.now()
``` </br>
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
** </br>
** </br>

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
``
*If you are using the default FileSystemStorage, the string value will be appended to your MEDIA_ROOT path to form the location on the local filesystem where uploaded files will be stored. If you are using a different storage, check that storage’s documentation to see how it handles upload_to.* </br>
*upload_to may also be a callable, such as a function. This will be called to obtain the upload path, including the filename. This callable must accept two arguments and return a Unix-style path (with forward slashes) to be passed along to the storage system. The two arguments are:* </br>
* **instance : ** An instance of the model where the FileField is defined. More specifically, this is the particular instance where the current file is being attached. </br>
In most cases, this object will not have been saved to the database yet, so if it uses the default AutoField, it might not yet have a value for its primary key field.* </br>
* **filename : ** The filename that was originally given to the file. This may or may not be taken into account when determining the final destination path.* </br>
*For Example: * </br>
``` python
def user_directory_path(instance, filename):
    # file will be uploaded to MEDIA_ROOT/user_<id>/<filename>
    return 'user_{0}/{1}'.format(instance.user.id, filename)

class MyModel(models.Model):
    upload = models.FileField(upload_to=user_directory_path)
``` </br>
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















