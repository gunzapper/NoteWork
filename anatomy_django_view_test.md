# Anatomy of a Django's view test

* [Prerequisites](#prereq)
* [Test the url](#url)
* [Create a fake user](#fakeuser)
* [Menage the session](#session)
* [Request object ](#request)

## <a name="prereq"></a> Prerequisites

I use

```{python}
    import pytest
    from model_mommy import mommy
```

I also use [pytest-django](https://pytest-django.readthedocs.io/en/latest/)

## <a name="url"></a> Test the url

The first thing to test is if the url correspond to the right view function. We have to write a simple test.

```{python}
    from my_app import views
    from django.urls import resolve


    def test_url_resolve(live_server):
        """test if the view is mapped to correct url."""
        found = resolve("/my/url")
        assert found.func == views.my_view
```

`live_server` is a pytest-django fixture.

## <a name="fakeuser"></a> Create a fake user

We use to create a fixture in `conftest.py`

```{python}

    import pytest


    @pytest.fixture
    def fake_user(django_user_model):
        """create a fake user."""
        return mommy.make(django_user_model)  


    @pytest.fixture
    def fake_super_user(django_user_model):
        """create a fake super user."""
        return mommy.make(django_user_model, is_superuser=True)
```

The `django_user_model` is a fixture of django-pytest
that returns the User model in use in the project.
The `model_mommy` fabricator return an item of the giving model.

## <a name="session"></a> Manage the session

To manage the session the most simple thing is use a function.

```{python}
def set_session(request, client, user):
    """set session of request as client session."""
    session = client.session
    # You could set also variable stored in the session
    session["the_answer"] = 42
    request.session = session
```

* `request` is a request object
* `client` is a fake client dispose through a django-pytest's fixture
* `user` is a fake user


## <a name="request"></a> Request object

Usually request object need to be customized. With a fixture we can
do that.

```{python}
def _request(rf, client, fake_user):
    """Set request."""
    request = rf.get("/a/wrong/url", follow=True)
    set_session(request, client, fake_user)
    return request
```



