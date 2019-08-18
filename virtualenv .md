## Pip
*`pip search requests` to search on package*</br>
``` python
# Example
>>> pip search pympler
Pympler (0.7)  - A development tool to measure, monitor and analyze the memory
                 behavior of Python objects.
```
*`pip install requests` to install on package*</br>
*`pip uninstall requests` to uninstall on package*</br>
*`pip list` to list all installed packages*</br>
``` python
>>> pip list --outdated # to get the outdated packages
Django (1.11.5) - Latest: 2.2.4 [wheel]
pip (9.0.1) - Latest: 19.2.2 [wheel]
pytz (2019.1) - Latest: 2019.2 [wheel]
setuptools (28.8.0) - Latest: 41.1.0 [wheel]
stripe (2.32.1) - Latest: 2.35.0 [wheel]
virtualenv (16.6.1) - Latest: 16.7.3 [wheel]
```
*`pip install -U pympler` to upgrade the outdated package and the uppercase -U starnds for Upgrade* </br>
*`pip freeze` to list the installed packages* </br>
*`pip freeze > requirements.txt` to create a file with a list of all the installed packages* </br>
*`cat requirements.txt` to show what's inside it* </br>
*`pip install -r requirement.txt` to install the requirements* </br>
## Virtualenv
*`pip install virtualenv` to install the virtualenv* </br>
*`virtualenv -p pythonPath ProjectName` to create a new virtualenv for project* </br>
*`.\Scripts\activate.` to activate the virtualenv for project* </br>
``` python
>>> which python # to print which python i use
# output : /c/Users/Amr/Desktop/30days/scrape/Scripts/python
```
``` python
>>> which pip
# output : /c/Users/Amr/Desktop/30days/scrape/Scripts/pip
```
*`deactivate` to deactivate the virtualenv* </br>
*`rm -rf project_env_name` to remove the virtualenv* </br>
