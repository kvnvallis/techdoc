Programming Languages
=====================

Notes and documentation on python and other languages

[TOC]


Django
------

### (Don't) Delete your migrations

__NOTE__: I thought this fixed my problem, but afterward I realized that no new changes to the models were being detected by `makemigrations`. So, migrations are still broken, but differently now.

__FIX__: If you're going to delete all your migrations, you will then have to recreate your initial migrations for the affected app, and use `--fake-initial` to apply those migrations to existing tables.

    python manage.py makemigrations appname
    python manage.py migrate --fake-initial

__The part you shouldn't do__:

Upon deleting a model and trying to run `migrate`, I kept getting an exception about a field in the (now missing) model. This may have happened because I never successfully migrated the model before I deleted it. Regardless, the solution was to delete my migrations.

1. Delete migrations folder at `./appname/migrations/`
2. `python manage.py dbshell` then `DELETE FROM django_migrations WHERE app = 'appname'`


### Always redirect a POST request

The view function below gets the POST data, which contains the name of an object defined in your django models. It then gets the object using that name.

    from django.http import HttpResponseRedirect
    from django.urls import reverse

    def myview(request):
        objectname = request.POST.get('postname')
        object = MyModel.objects.get(name=objectname)
        return HttpResponseRedirect(reverse('appname:index'))

If you DON'T return the redirect, you'll see that the django dev server attempts to get the object TWICE, and fails the second time because the POST dict comes up empty. This happens whether you run `python manage.py runserver --noreload` with the *noreload* flag or without it. 

With default logging, you should also see a green POST 302 code in the console, instead, of two POST 200 messages. 

The django tutorial lets you know that returning a redirect is important for when the user hits the back button, potentially resending POST data, but it does not mention this odd behavior from the dev server.


Python
------

### Pretty-print json to stdout

Printing readable json is important when working with the `requests` module and json APIs. When you need to examine a long ugly response from an API, output it like this:

    import json
    # variable r is the response from the API
    print(json.dumps(r.json(), indent=4, sort_keys=True))

If you read the response with `r.content` or `r.text` you should use `json.loads()` before `json.dumps()`.

    p = json.loads(r.content)
    print(json.dumps(...))


### `__file__` variable only works on import ###

This line will print nothing if you run the script directly.

__File: test.py__

    from os.path import dirname
    print(dirname(__file__))

But if your import the script from another file, it will print the directory of *test.py*.

__File: main.py__

    import test

__Shell:__

    $ python main.py
    /home/user/projects/testapp

