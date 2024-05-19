# Programming

[TOC]

## Javascript ##

### Run script when page is ready

I like to put script tags at the top of the page instead of the footer, but then you have to make sure your script runs after the page is loaded. Wrap your entire script in one of the following:

1. `window.onload = function() {}`

    Replaces load event with your script, which means no other scripts can use the load event. It waits for all resources to finish loading, including images and scripts. 
    
2. `window.addEventListener('load', function() {})`

    Same as assigning *window.onload* but lets multiple scripts attach to the load event. 

3. `document.addEventListener('DOMContentLoaded', () => {})`

    Instead of waiting for all resources to load, only wait for the html tags (or more accurately, the DOM). This option is usually the best.


## SHELL

*POSIX-compliant unless otherwise stated*

### Loop through find output

`find -exec` has its limits. Sometimes you want to run more complex code on the output of find. You can do this the bash way or the posix way. Both pipe find output into the next command. Make sure any subsequent commands that expect user input do not take input from stdin. The examples below tell `rm -i` to take input from `/dev/tty` instead. Note that these examples are not complex, so `find -exec rm -i -- {} \;` would be more suitable.

__BASH:__

    # IFS= preserves leading and trailing whitespace
    # -r preserves backslashes
    # -d '' interprets null line endings from -print0
    find ./ -type f -print0 |
        while IFS= read -r -d '' file; do
            rm -i -- "$file" < /dev/tty
        done

__POSIX:__

    # -0 interprets null line endings from -print0
    # -I {} prevents spaces being collapsed or trimmed
    find ./ -type f -print0 | xargs -0 -I {} sh -c '
        for file in "{}"; do
            echo $0
            rm -i -- "$file" < /dev/tty
        done
    ' sh
    # 'sh' fills param $0 for the sh command, so that it doesn't consume any
    # lines from the find output. Any string will work as a placeholder.


### Check if argument is a function

You can call a function by name as the first argument to a script, but it's a good idea to check if the function exists, otherwise your script will run any command that you pass as the first arg. The case statement checks if the output of *type* matches `*function*`. AFAIK you have to use a case statement (not an if) to match glob patterns.

    # Run script args as commands if first arg is a function
    case "$(type -- "$1" 2>/dev/null)" in
        *is\ a\ function*)
            "$@"
            ;;
        *)
            echo "$1" is not a function
            ;;
    esac


## C / C++

### Compile and run a standalone windows exe in arch linux

Install MinGW, then compile a cpp file and statically link the c and c++ standard libraries, along with _libwinpthread-1.dll_ (so they are all included in the exe). 
 
    sudo pacman -S mingw-w64-gcc
    x86_64-w64-mingw32-g++ -static-libgcc -static-libstdc++ -static -lwinpthread -o hello.exe hello.cpp
    wine hello.exe


## Raylib

### config flags

Config flags that can be set with `SetConfigFlags()` are not documented. You can look at the source instead: <https://github.com/raysan5/raylib/blob/master/examples/core/core_window_flags.c>

Set them like this:

    SetConfigFlags(FLAG_WINDOW_RESIZABLE);
    InitWindow(320, 240, "example");
    SetTargetFPS(60);


## Java

### JUnit Tests

The following commands will compile a java file containing junit tests and imports, then run the tests. 

    $ javac -d ./compile -classpath junit-platform-console-standalone-1.9.1.jar -sourcepath ./ TestCard.java
    $ java -jar junit-platform-console-standalone-1.9.1.jar --classpath ./compile --scan-classpath

The standalone jar can be downloaded here <https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone/>


## React Native

### onPress property requires anonymous function

For buttons or other pressables, there is an `onPress` property that lets you call a function when the button is pressed. However, if your handler function accepts an argument, then you will need to wrap the call to the handler function in an anonymous arrow function.

    // handler function
    const handleAddFighterSubmit = (fighter) => {
      setAddFighterName(fighter)
    } 

    // onPress callback
    <Button
      title="Submit"
      onPress={() => {
        props.onAddFighterSubmit(name)
      }}
    />

If the handler doesn't accept any arguments, you can call it directly.

    const handleLogSubmitPress = () => {
      console.log("Pressed Log Submit button")
    }

    <Button
      title="Log Submit"
      onPress={handleLogSubmitPress}
    />

    // console stdout
    // LOG  Pressed Log Submit button

The `onPress` property doesn't know to pass your argument to the handler function, and if you say `onPress={props.onAddFighterSubmit(name)` then you're calling the function at render time, instead of letting the Button component call your function when the the button is pressed. Compare this to TextInput's `onChangeText` property, that knows to pass the value of the text-input field as an argument.


### onSubmitEditing property of TextInput

The TextInput component has two properties that allow you to get user input, `onChangeText` and `onSubmitEditing`. The former saves text as it is typed or deleted, and the latter saves text when the user hits return on the keyboard. 

onSubmitEditing seems like the obvious choice for submitting traditional forms, but there is no documented way to retrieve the user input, or to create a submit button (hitting enter is the only way). A stackoverflow post shows how to retrieve user input for a single field, by accessing `nativeEvent.text`:

    onSubmitEditing={(name) => {
      setName(name.nativeEvent.text)

But I haven't found a way to create a button that triggers a keyboard event for the Enter key. So for now, keep using onChangeText to save the text in real time. 
  
### Text style property

The style option of the Text component takes a javascript object (dictionary) with key/value pairs that define style properties (like fontStyle, fontWeight, or fontSize). 

    <Text style={
      { fontSize: 30, fontWeight: 'bold' }
    }>
      Text goes here
    </Text>

But it can also take a list of objects. This way if you want to combine multiple style objects, you don't have to build a new object out of them. Remember the initial pair of curly braces is there to allow javascript execution.

    <Text style={
      [
        { fontSize: 30, fontWeight: 'bold' },
        { fontStyle: 'italics' }
      ]
    }>

Passing a list of objects is useful when you have stylesheets defined.

    <Text style={
      [
        styles.text,
        { fontStyle: 'italics' }
      ]
    }>

    const styles = StyleSheet.create({
      text: {
        fontSize: 30,
        fontWeight: 'bold',
      },
    })


## JavaScript

### Cycle through a list and repeat

Show the first item in the list. When you click the button, change the index to show the next item. Instead of just incrementing *i*, set it to equal the remainder of *(i + 1)* divided by *the length of the list*. 

    i = (i + 1) % mylist.length

The following code will:

1. Return a remainder equal to the dividend *i + 1*, 
2. until *i + 1* is equal to *animals.length* (i.e. 4 % 4), 
3. at which point a remainder of 0 is returned.

---

    const animals = ['giraffe', 'baboon', 'hummingbird', 'caribou']
    let i = 0
    $('.animal-name').html(animals[i])

    $('button.cycle').click( () => {
        i = (i + 1) % animals.length
        $('.animal-name').html(animals[i])
    })

Even if you manually set *i* to equal an integer outside the range of the list, the equation will only return values within the acceptable range (in this case, [0-3]), because the remainder can never be greater than or equal to the divisor.

    >>> i = 4
    >>> (i + 1) % 4
    1


## Django

- The double-underscore WHERE filters are called [Field Lookups](https://docs.djangoproject.com/en/stable/ref/models/querysets/#field-lookups) and all of them are listed in the QuerySet API reference.
- __manage.py__ is a wrapper script for the __django-admin__ command.


### Create a super user

If this isn't a brand new project, first check if a super user already exists.

    $ python manage.py shell
    >>> from django.contrib.auth.models import User
    >>> User.objects.all()
    <QuerySet []>

An empty queryset means there are no users at all. If users do exist, you can filter for superuser.

    >>> User.objects.filter(is_superuser=True)
    <QuerySet [<User: admin>]>

When you're sure a superuser needs to be created, use this command:

    $ python manage.py createsuperuser

Now login with the admin account at `localhost:8000/admin`.


### Template Paths

You tried to point to a template and got a `TemplateDoesNotExist` error. The key here is in `settings.py`.

    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [],
            'APP_DIRS': True,

`APP_DIRS: True` tells django to look in `yourapp/templates/` for template files. You can add other dirs in the DIRS list, but there is no need. 

If you followed the django tutorial's example your templates are currently in `yourapp/templates/yourapp/`, so you would point to them (from another template) like this:

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

__The part you shouldn't do:__

Upon deleting a model and trying to run `migrate`, I kept getting an exception about a field in the (now missing) model. This may have happened because I never successfully migrated the model before I deleted it. Regardless, the solution was to delete my migrations.

1. Delete migrations folder at `./appname/migrations/`
2. `python manage.py dbshell` then `DELETE FROM django_migrations WHERE app = 'appname'`

__When it's okay to delete migrations:___

You haven't migrated them yet, or the migration failed. If migration 0006 failed, feel free to delete it and all migrations after 0006 (0007, 0008, etc.)


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



## Python


### Use pathlib instead of os.path.join

    >>> from pathlib import Path
    >>> Path('/tmp').resolve()
    PosixPath('/tmp')
    >>> p = Path('/tmp').resolve()
    >>> p / 'media'
    PosixPath('/tmp/media')


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

