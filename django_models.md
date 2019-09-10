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













