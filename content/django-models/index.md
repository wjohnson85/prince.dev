---
title: 'Building Your First Django Models'
date: '2020-02-26'
slug: 'django-models'
tags: ['django', 'python', 'models']
---

A lot of applications nowadays require having some sort of database. This allows developers to build better experiences as information about a user's interactions can persist even if they decide to leave the page.

In Django, our application communicates to our database through a tool called an ORM, Object Relational Mapper. Developers can describe the data and their relationships in the system using Python without directly writing the code to communicate within the database, such as SQL.

In this post, we'll cover:

- Building a Model
- Creating data in our database
- Describing relationships between data

If you want to follow along, feel free to [clone down the project](https://github.com/maxcell/django-model-example). You can also see the [differences](https://github.com/maxcell/django-model-example/compare/final) or just the [end result after this post](https://github.com/maxcell/django-model-example/tree/final).

## Building a Model

[Models](https://docs.djangoproject.com/en/3.0/topics/db/models/) are a human-readable way of describing what the data is and can do. They follow the paradigm of Object Oriented Programming where our models will contain fields and behaviors of our data. Models act as our source of truth, from inside our application, about data within our database.

For our learning, we'll build the structure for a flashcard app. Our MVP will have cards with text on the front and back, and bundle cards together. This surfaces two models for us:

1. `CardBundle`
2. `Card`

In our `CardBundle`, we could have a title and a description. That way in the user interface, one would be able to understand what kind of `Card`s could be inside. Let's write our `CardBundle` model:

```python
# django-model-example/flashcards/models.py
from django.db import models

class CardBundle(models.Model):
	title = models.CharField(max_length=30)
	description = models.CharField(max_length=500)
```

Dissecting this code:

- We import our `model` package from inside of `django.db`. This package allows us to declare fields and relationships.
- We write a class for our `CardBundle`. It will be a child of the `models.Model` class, meaning it will inherit properties from `models.Model`.
- Lastly, we declare two fields on our model, `title` and `description`. For any column in our database, we would write it as a data attribute within that class. `models.CharField` represents the data type we want to be stored, along with any arguments that can be used as validations like `max_length`.

There we have our first model! Even though we've created this model, we have to make sure that the database knows about it.

### Adding our model to the database

We will need to wire up some settings in our project. **It is important to do these steps**, otherwise nothing will be connected together and simply won't work. We will need to connect our application to the project.

Let's add our `flashcards` app to the listed `INSTALLED_APPS` array found within our settings:

```python{7}
# Inside `django-model-example/flashcard_project/settings.py`

# ...
# Application definition

INSTALLED_APPS = [
    'flashcards.apps.FlashcardsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

Now, Django is aware that we have an application we created. To get a model on the database will require two things: 1) creating the changes we want to make and then 2) running the changes on the database. We need to tell Django to make a migration, a file that specifies the changes we want to make to the database, on our `flashcards` app:

```shell
# In your terminal
python manage.py makemigrations flashcards

# Output
Migrations for 'flashcards':
  flashcards/migrations/0001_initial.py
    - Create model CardBundle
```

You should notice a file under `flashcards/migrations/0001_initial.py` listing the changes we want to make to our database. Since this is our first one, it gets the designation as our `initial` migration. These files are imperative to communicating how the database was modified over the course of time!

Then finally, we need to perform the migration:

```shell
# In your terminal
python manage.py migrate

# Output
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, flashcards, sessions
Running migrations:
	# ... all other project migrations
  Applying flashcards.0001_initial... OK
```

Now we're ready to go to create some data!

## Creating Data inside of Our Database

For now, we're going to create data within the Terminal that way we can better understand what is happening within our application code. We can access our database through a command in our Terminal: `python manage.py shell`. This will create an interactive Python shell where we can talk to our database.

```bash
# In your terminal
python manage.py shell

# Output
(InteractiveConsole)
>>>
```

Here we'll create a few `CardBundle`s. You'll need to import the class into this shell before you can create anything.

```python
>>> from flashcards.models import CardBundle
>>> CardBundle.objects.create(title="Django Models", description="Helpful tips for Django Models")
<CardBundle: CardBundle object (1)>
>>> CardBundle.objects.create(title="Tagalog", description="Helpful phrases for new language")
<CardBundle: CardBundle object (2)>
```

You'll notice we're using `CardBundle.objects`. This is known as the `Manager`. It is the interface for how Django communicates to the database. Through it, we can `create` an object, `get` an object, and much more. We'll pass in the values that we want for `title` and `description` and it will create and save these into rows inside of our database!

## Describing Relationships Between Data

Now that we have our `CardBundles`, we should build out our cards! A `Card` has a `front_text` and a `back_text` for our user. To start, our model will look like this:

```python{8-10}
# django-model-example/flashcards/models.py
from django.db import models

class CardBundle(models.Model):
	title = models.CharField(max_length=30)
	description = models.CharField(max_length=500)

class Card(models.Model):
	front_text = models.CharField(max_length=500)
	back_text = models.CharField(max_length=500)
```

However in our code now, we have no connection between a `Card` and a `CardBundle`. For every bundle, there could be many `Card`s but each `Card` will only ever live inside of a single `CardBundle`. We need to establish the relationship between a `CardBundle` and a `Card`. In our `Card` model we will need to say, "This card belongs to a specific `CardBundle`". We do this through a `ForeignKey`, which can make our [Many-to-One Relationship](https://docs.djangoproject.com/en/3.0/topics/db/examples/many_to_one/):

```python{11}
# django-model-example/flashcards/models.py
from django.db import models

class CardBundle(models.Model):
	title = models.CharField(max_length=30)
	description = models.CharField(max_length=500)

class Card(models.Model):
	front_text = models.CharField(max_length=500)
	back_text = models.CharField(max_length=500)
	bundle = models.ForeignKey(CardBundle, on_delete=models.CASCADE)
```

Every time we look at a `Card` now, we'll be able to tell what `CardBundle` it is associated to. It also says that whenever the foreign object (`CardBundle`) is deleted, it should also be deleted.

With our model made, we can make and perform the migration!

```shell
# In the terminal
python manage.py makemigrations flashcards
# ... Output
python manage.py migrate
# ... Output
```

If we open up our `python manage.py shell` again, we can make some `Card`s:

```python
(InteractiveConsole)
>>> from flashcards.models import CardBundle,Card
>>> models_bundle = CardBundle.objects.get(title="Django Models")
>>> Card.objects.create(
... front_text="A field that only contains characters",
... back_text="django.models.CharField",
... bundle=models_bundle)
<Card: Card object (1)>
>>> Card.objects.create(
... front_text="Declaring a max_length of 10 on a CharField",
... back_text="models.CharField(max_length=10)",
... bundle=models_bundle
... )
<Card: Card object (2)>
```

Breaking this down:

- We import our models again. Notice how you can import multiple through the same package!
- Then, we find one of our bundles using `CardBundle.objects.get()`. This can take any attribute on our model and find the first thing that matches. We store the result of that into a variable called `models_bundle` so we can later use it for creating our `Card`s.
- To wrap up, we create two `Card`s we want in our bundle. Each one of the models receives `models_bundle` as the value to `bundle`. This is because the Django ORM is expecting something with a `CardBundle` type.

With that, we've made all the associations and set the base of our application up.

## Conclusion

Models are a friendly way to interface with data and not have to leave the comfort of your language. This means we can spend more time focused on building the rest of the application and making all the stuff that users enjoy! Continue your journey with Django reading through some other [fields on Models in the Django Documentation](https://docs.djangoproject.com/en/3.0/ref/models/fields/#module-django.db.models.fields) and try building your own models! You can also read about different ways to [get data through queries](https://docs.djangoproject.com/en/3.0/topics/db/queries/#retrieving-all-objects).

P.S. Shoutout to a few folks who helped with this blog post!

- [Ryan](https://twitter.com/ryancharris), [Zac](https://twitter.com/thatzacdavis), and Jeremy for proofreading!
