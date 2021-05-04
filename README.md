На этом уроке мы встретимся с Django 3.2 фреймом который нам позволяет легко создавать сайты используя python!

https://docs.djangoproject.com/en/3.2/intro/tutorial01/

Документация:

https://docs.djangoproject.com/en/3.2/

Начнём с создания виртуального пространства для работы с проектом!
```shell
pipenv install django
```
Далее можем зайти в виртуальное пространство
```shell
pipenv shell
```
Стоит убедиться что у нас не пустой джанго
```shell
python -m django --version
```
далее можем создать наш проект!
```shell
django-admin <project_name>
```
Это наша стандартная комманда для работы с Django. она создаст джанго проект в папке с указаным именем!
Стоит ознакомиться с файлами!
```shell
python manage.py migrate 
```
создаёт всё нужное для сервера согласно settings.py файлу!
```shell
python manage.py runserver
```
Запускает наш дев сервер.
# MODEL
```shell
python manage.py createsuperuser
```
Позволит нам создать админ аккаунт для входа в панель на сервере!

Войди в панель на `http://127.0.0.1:8000/admin` для примера!
## Компоненты
```shell
python manage.py startapp <app_name>
```
Позволяет нам создавать новые компоненты нашего сайта.  Посмотрите компонент папку и зайдите в models.py файл.
```python
from django.db import models

# Create your models here.
class Product(models.Model):
    title = models.TextField()
    description = models.TextField()
    price = models.TextField()
```
Компоненты легко могут хранить информацию в базе данных. После создания компонента и модели можно добавить компонент к основному проекту через `settings.py` файл.
```python
#Компоненты называются apps
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    #own
    'products',
]
```
Не забываем оставить запятую после имени компонента в списке!

Запоминаем ещё раз 2 важные комманды
```shell
python manage.py makemigrations
python manage.py migrate
```
`makemigrations` подготавливает модели и компоненты а `migrate` создаёт нужные таблици в базе! Мы должны запускать эти две команды каждый раз когда мы меняем файл `models.py`

Сейчас посмотрим на `admin.py` файл где мы импортируем нашу модель.
```python
from django.contrib import admin

# Register your models here.
from .models import Product
# Регистрируем на админ панели наши продукты
admin.site.register(Product)
```
Сейчас наша модель доступна панели администрации сайта.  Но мы так-же можем добавлять информацию пользуясь `python shell`!
```shell
python manage.py shell
```
Эта команда запускает python shell который имеет доступ ко всем модулям внутри сервера Django
```python
>>> from products.models import Product
>>> Product.objects.all()
```
покажет все обьекты в базе.
```python
>>> Product.objects.create(title='Product2',description='description2',price='100')
```
Перемены будут отражены на сайте и в `objects.all()` функции.
## Переменные модели
В моделях компонента мы можем иметь разные типы переменных!  Если нас не устраивают переменные мы можем начать с удаления файлов в `migrations` папке и базы данных `db.splite3`.

https://docs.djangoproject.com/en/3.2/ref/models/fields/#field-types

В документации ссылка на типы полей которые мы можем использовать с моделями!  Подберите подходящие поля.

## Перемена Модели
Мы можем добавить новое поле к уже существующей модели.  Например:
```python
    featured = models.BooleanField()
```
При запуске `makemigrations` мы получим ошибку.
```commandline
You are trying to add a non-nullable field 'featured' to product without a default; we can't do that (the database needs something to populate existing rows).
Please select a fix:
 1) Provide a one-off default now (will be set on all existing rows with a null value for this column)
 2) Quit, and let me add a default in models.py
```
Так как у нас уже есть значения в базе данных нам в классе модели придётся предоставить стандартные значения для всех прошлых объектов!
```python
   featured = models.BooleanField(default=False) # даёт стандартное значение
    # или
   featured = models.BooleanField(null=True)
```
Так-же мы можем иметь значения которые не обязательны без стандартных значений с помощью `blank` переменной:
```python
    summary = models.TextField(blank=True) #показывает нужно ли значение при заполнении формы
    summary = models.TextField(null=True) #показывает нужно ли значение в базу данных
```
После `makemigrations` и `migrate` обратите внимание на то что `summary` переменная уже не выделена и может быть сохранена пустой.
# VIEW 
## Виды Views & URLs
Как изменить домашнюю страницу? Создадим компонент `pages`
```shell
python manage.py startapp pages
```
добавим новый компонент к сайту в `settings.py` файле
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    #own
    'products',
    'pages',
]
```
Теперь обращаем внимание на файл `views.py` это где мы будем работать с визуальными обьектами наших сайтов.
```python
from django.shortcuts import render

# Create your views here.
```
Здесь мы можем добавлять виды делая из них функции которые возвращают `HTML` на пример
```python
def home_view():
    return "<h1>Hello World</h1>" # string of HTML code
```
Но это обычная функция не использующая `Django`.  Мы можем вернуть полноценную страницу используя `HtmlResponce` модуль в `Django`:
```python
from django.http import HttpResponse
from django.shortcuts import render

# Create your views here.
def home_view(*args,**kwargs):
    return HttpResponse("<h1>Hello World</h1>")
```
Это не меняет факта что наша домашняя страница всё ещё рисует ракету.  Чтобы поменять конкретную страницу нам нужно воспользоваться `urls.py` как нашим стандартным раутером.
```python
"""first URL Configuration

The `urlpatterns` list routes URLs to views. For more information please see:
    https://docs.djangoproject.com/en/3.2/topics/http/urls/
Examples:
Function views
    1. Add an import:  from my_app import views
    2. Add a URL to urlpatterns:  path('', views.home, name='home')
Class-based views
    1. Add an import:  from other_app.views import Home
    2. Add a URL to urlpatterns:  path('', Home.as_view(), name='home')
Including another URLconf
    1. Import the include() function: from django.urls import include, path
    2. Add a URL to urlpatterns:  path('blog/', include('blog.urls'))
"""
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('admin/', admin.site.urls),
]

```
Нам тут даётся подсказка как работать с этим файлом в большом коментарии сверху.  Сначала импортируем наши `views.py` следующим образом:
```python
from pages import views
```
И добавить `URL` адресс в `urlpatterns` список используя `path` функцию которая преобразует местоположение вида в абсолютную функцию понятную `Django`.
```python
urlpatterns = [
    path('', views.home_view, name='home'),
    path('admin/', admin.site.urls),
]
```
Первое значение это `URL` адрес на сайте, второе это вызываемая функция для рисования страницы.  Это не лучший способ импортировать виды так как разные компоненты пользуються стандартно генерированым файлом `views.py` лучше импортировать сразу нужный вид из файла!
```python
from pages.views import home_view

urlpatterns = [
    path('',home_view,name='home'),
    path('admin/', admin.site.urls),
]
```
Сейчас мы можем ближе рассмотреть что-же приходит когда мы обновляем ту или иную страницу в `view` функцию. 
```python
def home_view(*args,**kwargs):
    print(args)
    print(kwargs)
    return HttpResponse("<h1>Hello World</h1>")
```
При попытке распечатки `args` и `kwargs` мы видим что есть одина позиционная переменная `<WSGIRequest: GET '/'>` и `HEAD` это два запроса которые нам даёт `Django` при запросе и загрузке страницы!
```shell
(<WSGIRequest: GET '/'>,)
(<WSGIRequest: HEAD '/'>,)
[04/May/2021 06:59:55] "GET / HTTP/1.1" 200 20
[04/May/2021 06:59:55] "HEAD / HTTP/1.1" 200 20
```
Так-же можно обратить внимание на то что сервер автоматически перезапускается с каждым изменением файла.

Мы можем вывести позиционный аргумент в консоль. Стандарт названия его `request`.
```python
def home_view(request, *args,**kwargs):
    print(request)
    print(dir(request))
    print(request.user)
    return HttpResponse("<h1>Hello World</h1>")
```
Хэд запрос возвращает анонима а гет уже моего пользователя на сервере!

Возврат `H1` элемента по себе не является ни чем особенным.  Поэтому разучим `Django Templates`
# Django Templating
Для работы со страницами джэнго у нас в `view.py` файле есть `render` функция которая берёт 3 аргумента. 
```python
render(request,HtmlTemplate,kwargs)
```
Можем это применить следуюцим образом:
```python
def home_view(request):
    return  render(request,'home.html')
```
но при попытке загрузить сайт у нас выходит проблема `TemplateDoesNotExist` и нам желательно держать наши образцы(templates) в одной папке так-что давайте создадим `templates` папку в `root` папке там где распологается `manage.py`.  Но даже при создании файла в папке экземпляров `Django` не сможет найти этот файл. нам нужно изменить `TEMPLATES` отдел в `settings.py` чтобы добавить эту папку в список папок проекта.
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
И конкретнее нам нужно добавить `templates` папку в `DIRS` список.
```python
        'DIRS': ['/Users/Mishka1012/Desktop/Python Grodno IT Academy/Lesson 12/first/templates'],
```
так делать не надо так как это будет работать только на вашем компьютере! Вместо этого нужно воспользоваться `Path` переменной встроенной в питон!
```python
BASE_DIR.joinpath('templates')
```
это сделает путь к папке независимым от системы и это возможно благодаря двум линиям ранее в файле:
```python
from pathlib import Path

# Build paths inside the project like this: BASE_DIR / 'subdir'.
BASE_DIR = Path(__file__).resolve().parent.parent
```
## Basics
Чтоже теперь мы имеем html файл но это не крутая часть. В этот файл мы можем писать django python прямо в файле для примера:
```html
    <h1>Welcome home</h1>
    {{ request.user }}
    <p>This is a template</p>
```
Обращаем внимание на `{{ request.user }}` внутри скобок вписан python код!  Пойдём дальше и подумаем что на сайтах есть генеральная форма презентации страниц которая присутствует на каждой странице типа меню и footerа для этого мы можем импользовать базовый файл с `{% block content %}{% endblock %}` как это может выглядеть:
```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title>This is a page title</title>
  </head>
  <body>
    {% block content %}
        replace me
    {% endblock %}
  </body>
</html>
```
а в файле `home.html` можем продлить фунционал этого файла используя:
```html
{% extends 'base.html' %}
{% block content %}
 <h1>Welcome home {{ request.user }}</h1>
 <p>This is a template</p>
{% endblock %}
```
и мы можем иметь шаблонную наследовательность!
### Include template tag!
мы можем создовать разные элементы страниц в своих файлах например меню можно создать в `nav.html` и включить его в базовый файл используя `{% include 'nav.html' %}`
```html
<!-- nav.html -->
<nav>
  <ul>
    <li>Brand</li>
    <li>Contact</li>
    <li>Home</li>
  </ul>
</nav>
```
и в `base.html` можем добавить это:
```html
{% include 'nav.html' %}
```
и теперь можем отдельно работать над навигацией в nav файле!
### Template context
теперь мы уже можем манипулировать многое на нашем сайте!  
Поговорим больше о контексте и получении информации в наши страницы через контекст.
```python
# views.py
def home_view(request):
    my_context = {
        'title': "This is a home page",
        'number':123,
        'my_list': [i for i in range(5)],
    }
    return  render(request,'home.html',context=my_context)
```
контекст может хранить значения любого типа!  В `home.html` мы можем пользоваться контекстными переменными следующим образом:
```html
 <p>{{ title }}</p>
```
пример для печатки загаловка!  
### Cycles Циклы
Мы так-же можем иметь цыклы в примерах страниц.
```html
<ul>
{% for comment in my_list %}
 <li>{{ forloop.counter }} - {{ comment }}</li>
{% endfor %}
</ul>
```
### Conditions Условия
Условия в templates
```html
{% if condition %}
<!-- do stuff here -->
{% elif condition2 %}
<!-- if condition2 is true -->
{% else %}
<!-- if condition is false -->
{% endif %}
```

https://docs.djangoproject.com/en/3.2/ref/templates/builtins/

Всё что мы сейчас видели в скобках с процентами это встроеные таги для джанго.
## Прорисовка базы даннх
вспоминаем наш способ быстрой загрузки данных:
```shell
python manage.py shell
```
```python
>>> from products.models import Product
>>> Product.objects.get(id=1)
>>> obj = Product.objects.get(id=1)
>>> print(dir(obj))
>>> Product.objects.all()
```

ОСТАНОВКА 2.00