Programming
===========

[TOC]


Git
---

### Rebase two unrelated histories

You have two branches, *feature* and *main*, where you want all the changes in main to be brought into the history of feature. The branches are unrelated (common was created as an orphan), and feature has a long commit history; So if you checkout feature and run `git rebase common`, you will have to deal with many conflicts as git replays the entire history of feature. 

My solution was to squash all commits on feature into one, so that I only have to deal with conflicts on the most recent changes. This obviously only works if you don't care about preserving the commit history. 

    git checkout feature
    git rebase --root -i

Then edit the todo so that the oldest commit is reworded and the rest are squashed (`s` or `f`) up to the latest commit.

    1 r 01cc5a08 Removes open div
    2 s a2b6eecf Restores old fonts
    3 s 603479ff Cleans left out div
    4 s 5afdbc33 Update: show logo on landing page 

Or if you're running the newest version of git (2.31.1 for me), you can pass in the oldest commit to `rebase` and squash everything in the todo. Earlier versions of git will complain about there being "no previous commit" if you try this, because it expects you to `pick` at least one.

    $ git rebase -i 01cc5a08

    1 f 01cc5a08 Removes open div
    2 f a2b6eecf Restores old fonts
    3 f 603479ff Cleans left out div
    4 f 5afdbc33 Update: show logo on landing page 

When you have just one commit on feature, you can then easily rebase onto common (modifying the history of feature, but leaving the history of common alone.)

    git checkout feature
    git rebase common


Django
------

## Templates

You tried to point to a template and got a `TemplateDoesNotExist` error. The key here is in `settings.py`.

    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [],
            'APP_DIRS': True,

`APP_DIRS: True` tells django to look in `yourapp/templates/` for template files. You can add other dirs in the DIRS list, but there is no need. 

If you followed the django tutorial's example your templates are currently in `yourapp/templates/yourapp/`, so you would point to them like this:

    {% include 'yourapp/header.html' %}

Here's why the seemingly redundant _yourapp_ folder (Source: <https://docs.djangoproject.com/en/3.2/intro/tutorial03/#write-views-that-actually-do-something>)

> Now we might be able to get away with putting our
> templates directly in polls/templates (rather than
> creating another polls subdirectory), but it would
> actually be a bad idea. Django will choose the first
> template it finds whose name matches, and if you had a
> template with the same name in a different application,
> Django would be unable to distinguish between them. We
> need to be able to point Django at the right one, and the
> best way to ensure this is by namespacing them. That is,
> by putting those templates inside another directory named
> for the application itself.


## Automate checking for migrations

You can write a script to check if migrations exist for any change made to the models. Use the `--check` flag with `showmigrations` to make manage.py exit with a nonzero status when there is a change in the models and migrations have not yet been created. When the models and migrations are in sync, that's considered a success (exit 0). 

    if python manage.py makemigrations --dry-run --check; then 
        echo nothing to do
    else
        echo there are migrations to make
    fi


### Passing arguments between views

Django is designed to accept arguments to a view that are included within the URL. Generic views know to accept arguments by default, but you still have to define them elsewhere.

In the django tutorial's "polls" app, the question id is passed to the detail view from within the index template. Because the variable is defined as `pk` in the urls file, the detail view accepts it as an argument. 

    # file: urls.py
    path('<int:pk>/', views.DetailView.as_view(), name='detail') 

    # file: index.html
    <a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a>

    # file: views.py
    class DetailView(generic.DetailView):
        model = Question
        template_name = 'polls/detail.html'

If you don't use a generic view, you can write your own views function; In which case you can name the variable whatever you want, but you have to define the parameter within your function.

    # file: urls.py
    path('<int:question_id>/', views.detail, name='detail') 

    # file: views.py
    def detail(request, question_id):
        question = Question.objects.get(pk=question_id)
        return render(request, 'polls/detail.html', {'question': question})


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

### Patch a method to return a mock object

When using a patch decorator with *side_effect* as a mock object, you must remember to enclose the value of *side_effect* in square brackets. If you don't, you might receive an unhelpful error like this one:

    AssertionError: <Mock name='Mocked response from the tvdb[31 chars]280'> != {'links': {'first': 1, 'last': 2, 'next':[141077 chars] 0}]} 

In hindsight, it's clear that what's happening here is *side_effect* returns what the mock object returns, instead of returning the mock object itself. Type `Mock()` in a python terminal and you'll see what I mean.

To make the method return a mock object, the patch decorator can use *side_effect=[response_mocked]* or *return_value=response_mocked*. The benefit of using *side_effect* is you can specify other effects in the list that will be returned for each subsequent call to the method. Here we are patching the *get* method of the *requests* module.

- `@patch.object(tvfile.requests, 'get', return_value=response_mock)`
- `@patch.object(tvfile.requests, 'get', side_effect=[response_mock])`

Since you have to define *response_mock* before assigning it to *side_effect*, you won't be able to use the above examples unless *response_mock* is defined outside the scope of your test function (before it, in the test class). If *response_mock* is defined inside your test function, you can patch the method like so:

    @patch('tvfile.requests')
    def test_get_episodes_success(self, mock_requests):
        """get_episodes returns an unmodified response object"""
        response_mock = Mock(name='Mocked response from the tvdb api')
        response_mock.status_code = 200
        response_mock.json.return_value = send_episodes('1')
        mock_requests.get.side_effect = [response_mock]


### Pretty-print json to stdout

Printing readable json is important when working with the `requests` module and json APIs. When you need to examine a long ugly response from an API, output it like this:

    import json
    # variable r is the response from the API
    print(json.dumps(r.json(), indent=4, sort_keys=True))

If you read the response with `r.content` or `r.text` you should use `json.loads()` before `json.dumps()`.

    p = json.loads(r.content)
    print(json.dumps(...))

To print out json in bash, use *json.tool*.

    python -m json.tool tests/data/seinfeld-episodes-page-1.json


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

