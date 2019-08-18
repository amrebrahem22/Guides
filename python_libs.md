``` python
import sys
>>> print(sys.path) # to print all the system variables
# Output : ['', 'C:\\Users\\Amr\\AppData\\Local\\Programs\\Python\\Python36\\Lib\\idlelib', 'C:\\Users\\Amr\\AppData\\Local\\Programs\\Python\\Python36\\python36.zip', 'C:\\Users\\Amr\\AppData\\Local\\Programs\\Python\\Python36\\DLLs', 'C:\\Users\\Amr\\AppData\\Local\\Programs\\Python\\Python36\\lib', 'C:\\Users\\Amr\\AppData\\Local\\Programs\\Python\\Python36', 'C:\\Users\\Amr\\AppData\\Roaming\\Python\\Python36\\site-packages', 'C:\\Users\\Amr\\AppData\\Local\\Programs\\Python\\Python36\\lib\\site-packages']
'''
the fisrt is the directory where i run the current script
and the second is the listed dirctory in the python path environment directory
'''
sys.path.append('/your/other/file/path/') # to add a new variable path
>>> # that's to add another path to you system to import it to your file
>>> print(sys.path)
['', 'C:\\Users\\Amr\\AppData\\Local\\Programs\\Python\\Python36\\Lib\\idlelib', 'C:\\Users\\Amr\\AppData\\Local\\Programs\\Python\\Python36\\python36.zip', 'C:\\Users\\Amr\\AppData\\Local\\Programs\\Python\\Python36\\DLLs', 'C:\\Users\\Amr\\AppData\\Local\\Programs\\Python\\Python36\\lib', 'C:\\Users\\Amr\\AppData\\Local\\Programs\\Python\\Python36', 'C:\\Users\\Amr\\AppData\\Roaming\\Python\\Python36\\site-packages', 'C:\\Users\\Amr\\AppData\\Local\\Programs\\Python\\Python36\\lib\\site-packages', '/your/other/file/path/']

============================

import random
>>> courses = ['math', 'bio', 'science', 'history']
>>> print(random.choice(courses))
# Output : bio
>>> print(random.choice(courses))
# Output : history

=============================

import datetime
>>> import calendar
>>> today = datetime.date.today()
>>> print(today)
2019-08-19
>>> print(calendar.isleap(2019)) # to check if it leap year
False

==============================

>>> import os
>>> print(os.getcwd())
# output : C:\Users\Amr\AppData\Local\Programs\Python\Python36
print(os.__file__)
# output: C:\Users\Amr\AppData\Local\Programs\Python\Python36\lib\os.py

===============================

import antigravity # this will open in a new window in the browser
```
