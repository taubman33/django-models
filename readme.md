[![General Assembly Logo](https://camo.githubusercontent.com/1a91b05b8f4d44b5bbfb83abac2b0996d8e26c92/687474703a2f2f692e696d6775722e636f6d2f6b6538555354712e706e67)](https://generalassemb.ly/education/web-development-immersive)

## SEIR 712 Sept 16, 2021

# Django Models and Migrations

This class will get you acquainted with the basics of building an application
with Django. We'll start by discussing models and migrations and follow up in a
second class to discuss views and templates. Once we do, we'll have covered
everything you need to build web applications in Django.

## Prerequisites

- Python
- Another web framework like Express

## Objectives

By the end of this, developers should be able to:

- Create a new Django application with Postgres as the default database
- Use `manage.py` commands to create, edit, update and seed a database
- Write models using Django and use them to modify the database tables.
- Look at Django's ORM.

## Introduction

In this lesson, we will be focusing on the many features that Django provide us
to set up and maintain our database and models.

## We Do: Set Up a Django Application & Virtual Environment
See [installation instructions.](https://git.generalassemb.ly/sei-712/django-installation/blob/master/README.md)

To start your Django server:
1. Navigate into your `/tunr_django` folder.
1. Run `pipenv shell` to activate your virtual environment.
1. Run `python3 manage.py runserver`.
1. Navigate to `localhost:8000`.

## Models (10 min / 0:40)

Let's start working with some data. In Django, we will write out models. Models
represent the data layer of our application. We store that data in our database.

First, lets create a Python class that inherits from the Django built in
`models.Model` class. Let's also define the fields within it. We do so like
this:

```python
# tunr/models.py
class Artist(models.Model):
    name = models.CharField(max_length=100)
    nationality = models.CharField(max_length=100)
    photo_url = models.TextField()
```

Here, we are defining three fields (which will be represented as columns in our
database): `name`, `photo_url` and `nationality`. `name` and `nationality` are
character fields which means that we must add an upper limit to how many
characters are in that database field. The `photo_url` will have unlimited
length. The full listing of the available fields are
[here](https://docs.djangoproject.com/en/2.1/ref/models/fields/).

Let's also add the magic method `__str__`. This method defines what an instance
of the model will show up as by default. It will be really helpful for debugging
in the future!

```python
class Artist(models.Model):
    name = models.CharField(max_length=100)
    nationality = models.CharField(max_length=100)
    photo_url = models.TextField()

    def __str__(self):
        return self.name
```

This is a brief example of how to write models in Django. The
[documentation](https://docs.djangoproject.com/en/2.1/topics/db/models/) is
fantastic and there are a number of
[built in field types](https://docs.djangoproject.com/en/2.1/ref/models/fields/#model-field-types)
that you can use for making very detailed models.

## Migrations (10 min / 0:50)

In the SQL class, we talked about how schema is enforced on the database side
when we use SQL databases. But here we are writing our schema on the Python
side! We have to translate that code into the schema for our database. We will
do so using migrations. In some frameworks, you have to write your migrations
yourself, but in Django the framework writes them for us!

In order to migrate this model to the database, we will run two commands. The
first is:

```bash
$ python3 manage.py makemigrations
```

This will generate a migration file that gets all of its data from the code in
the `models.py` file. Go ahead and open it up - it should be in
`tunr/migrations/0001_initial.py`.

Every time you make changes to your models, run `makemigrations` again.

You should **NEVER** edit the migration files manually, as Django automatically takes care
of generating these migration files based on our model changes. Instead, edit the models
files and let django figure out what to generate from them by running
`makemigrations` again.

You **should** commit the migration files into git, however. They are crucial
for other people who want to run their own app.

When you've made all the changes you think you need, go ahead and run:

```bash
$ python3 manage.py migrate
```

This will commit the migration to the database.

If you open up `psql`:
```
$ psql
```

and connect to the `tunr` database:
```
\c tunr
```
you'll see all the tables have now been created!

This is quite different than mongoDB, where the databases and collections get
created automatically as soon as you insert data into them.

<details>
<summary>What is a migration?</summary>
A set of changes/modifications intended for a database. They can be anything that makes a permanent change - creating columns, creating tables, changing properties, etc.
</details>

## Break (10 min / 1:00)

### Foreign Keys (10 min / 1:10)

Let's also start filling out the Song model. We will define the class and then
add a foreign key. We do so like this:

```python
class Song(models.Model):
    artist = models.ForeignKey(Artist, on_delete=models.CASCADE, related_name='songs')
```

A foreign key is a field or column in one table that uniquely identifies a row
of another table. In this case, `Song` will contain a column called `artist`
that contains the ID of the associated artist. We don't have to define `id` in
the model, django and psql add them for us.

The `related_name` refers to how the model will be referred to in relation to
its parent -- you will see this in use later on. `on_delete` specifies how we
want the models to act when their parent is deleted. By using cascade, related
children will be deleted.

<details>
    <summary>What kind of relationship is implied by giving the Song table the foreign key from the artist table?</summary>

    1 -> Many
    Artist -> Song
    An artist can have many songs

</details>

What needs to happen now that we made a change to the model file?

```bash
python3 manage.py makemigrations
```

Check out the migrations folder. You should see something like `0002_song.py`.

Python automatically sequences the migration files and tries to give a
description for them - in this case, we added a song model, so it gives it the
name `song`.

Now run:

```
python3 manage.py migrate
```

And notice that it's all updated!

You can read more about migrations
[in the Migrations section of the documentation](https://docs.djangoproject.com/en/2.1/topics/migrations/)

If you want to see which migrations have been run already, use the command
`python3 manage.py showmigrations`.

### Admin Console (10 min / 1:20)

Before we get too far, let's also create a superuser for our app. Django has
authentication (and authorization) right out of the box, so you don't have to
write it yourself or add a plugin.

In the terminal, run:

```bash
$ python3 manage.py createsuperuser
```

Then fill in the information in the boxes that pop up!

So far in this class, we have used seed files to add initial data to our
databases. We can also do that in Django
([see this article](https://docs.djangoproject.com/en/2.1/howto/initial-data/)),
but let's try something a little bit different instead.

Django has an admin dashboard built in, which gives us full CRUD functionality
straight out of the box.

Let's set it up! In `tunr/admin.py`, add the following code:

```python
from django.contrib import admin
from .models import Artist

admin.site.register(Artist)
```

**Now! Bear Witness To the Awesomeness of Django!!!**

Run your server again, then navigate to `localhost:8000/admin`. You can login
and get a full admin view where you have CRUD functionality for your model!

Create two Artists here using the interface.

### You Do: Finish the Song model (10 minutes / 1:30)

- Add `title`, `album` and `preview_url` fields, then create and run the
  migrations.
- Register your Song model like you did with Artist.
- Finally create three songs using the admin site.

<details>
<summary>Solution: Modify Song Model</summary>

```python
class Song(models.Model):
    artist = models.ForeignKey(Artist, on_delete=models.CASCADE, related_name='songs')
    title = models.CharField(max_length=100, default='no song title')
    album = models.CharField(max_length=100, default='no album title')
    preview_url = models.CharField(max_length=200, null=True)

    def __str__(self):
        return self.title
```

</details>

<details>
<summary>Solution: Modify admin.py</summary>

```python
from django.contrib import admin
from .models import Artist, Song
admin.site.register(Artist)
admin.site.register(Song)
```

</details>

<details>
<summary>Solution: create migration</summary>

```bash
python3 manage.py makemigrations
```

</details>

<details>
<summary>Solution: run migration</summary>

```bash
python3 manage.py migrate
```

</details>

## Django Extensions (10 min / 1:40)

Django Extensions adds additional debugging functionality to Django. We would
**highly** recommend using it to make coding easier!
[Link](https://github.com/django-extensions/django-extensions).

To set it up:

```
$ pipenv install django-extensions
```

Add `django_extensions` to your `INSTALLED_APPS` list:

```py
# tunr_django/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'tunr',
    'django_extensions'
]
```

You can now run `python3 manage.py shell_plus` to get to a python shell.

**BONUS** install `ipython` because it's a much nicer interface

```
pipenv install ipython
```

Now you can enter it:

```
python3 manage.py shell_plus --ipython
```

Note all the imports that happen! This allows us to use many common features
that django provides, without having to import them ourselves. Super neato.

## Django's ORM (30 minutes / 2:10)

<details>
<summary>We've used an ODM before, what does it do for us? Now we will use an ORM, how is it different?</summary>
Object-Relational Mapping VS Object Document Mapping
</details>

Django has an ORM, similar to Mongoose in Express. Let's look at a few queries.

In the django shell we just installed above, run these commands to explore the
models and ORM:

```python
# Select all of the artist objects in the database
Artist.objects.all()

# Select All Objects and Print All Values
Artist.objects.all().values_list()

# Select All Objects and Print Specific Value
Artist.objects.all().values_list('nationality')

# Get the artist with the id of 1 (can also do pk here which stands for primary key)
Artist.objects.get(id=1)

# Get the artist with the name "Kanye West", if there are two Kanye's this will error!
Artist.objects.get(name="Kanye West")

# Get all the Artists who are from the USA
Artist.objects.filter(nationality="USA")

# Store an artist in a variable for later access:
p = Artist.objects.get(name="Kanye West")

# Now you can look up the artist's songs:
p.songs.all()
p.songs.all().values_list()

# Create an Artist with the following attributes, then save, commiting it to the database
kanye = Artist(name="Kane West", photo_url="test.com", nationality="USA")
kanye.save()

# Oops, we misspelled Kanye's name! Let's change it and then commit to the DB
kanye.name = "Kanye West"
kanye.save()

# Let's add a song to the artist
song = Song(title="Ultralight Beam", album="The Life of Pablo", preview_url="test.com", artist=kanye)
song.save()

# Delete the song
song.delete()
```

These are some simple ones, but if you like Django, know that you can do some
really cool things with it's ORM. For example:

```python
# This will return all Artists who's name starts with an A
Artist.objects.filter(name__startswith="A")

# This will return all the songs with the id's 1 and 2, excluding all those equal to or greater than 3
Song.objects.exclude(artist_id__gte=3)
```

There's a whole bunch of neat stuff we can do with the `shell_plus` extension.
Check the docs out:
https://django-extensions.readthedocs.io/en/latest/shell_plus.html

Django does have a shell built in by default:

```
$ python3 manage.py shell
```

It can do everything we do in the shell_plus, but doesn't automatically import
our models, which is a little annoying.

If we wanted to import manually, we'd type this into the shell:

```python
from .models import Artist, Song

Artist.objects.all()
```



## Additional Resources

- [Django Docs: Models](https://docs.djangoproject.com/en/2.0/topics/db/models/)
- [Django Docs: Models & Databases](https://docs.djangoproject.com/en/2.0/topics/db/)
- [How to Create Django Models](https://www.digitalocean.com/community/tutorials/how-to-create-django-models)
- [Django Docs: Migrations](https://docs.djangoproject.com/en/2.0/topics/migrations/)
- [Django Docs: Writing Database Migrations](https://docs.djangoproject.com/en/2.0/howto/writing-migrations/)
- [Django Docs: Providing initial data for models](https://docs.djangoproject.com/en/2.1/howto/initial-data/)
- [Django Extensions](https://github.com/django-extensions/django-extensions)

## [License](LICENSE)

1. All content is licensed under a CC­BY­NC­SA 4.0 license.
1. All software code is licensed under GNU GPLv3. For commercial use or
   alternative licensing, please contact legal@ga.co.
