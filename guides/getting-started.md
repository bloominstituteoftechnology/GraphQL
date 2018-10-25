# GraphQL: Experimentation and Django back end

This is analogous to what we did with REST, but we'll be doing it with GraphQL instead.

## Resources
* Experimentation with GraphQL queries: https://www.howtographql.com/
* Tons of GraphQL info: https://graphql.github.io/swapi-graphql/
* React front ends with GraphQL: https://www.howtographql.com/react-apollo/0-introduction/


## Experimentation with the graphiql Interface

[Click here to get to the demo page.](https://graphql.github.io/swapi-graphql/)

Enter some queries in the left panel.

Show all the links to the nodes on the graph:

```graphql
{
  allStarships
}
```

Or we could get all the starships themselves:

```graphql
{
  allStarships {
    starships
  }
}
```

Or get the name and ID:

```graphql
{
  allStarships {
    starships {
      id
      name
    }
  }
}
```

Mess around with this. With GraphQL you just specify the keys you want.

Click the `Docs` tab in the upper right. Explore the metadata and see what you
can look up.

## Installing the graphene GraphQL Library for Django

Get into the shell.

```
pipenv shell
```

We're going to install Django's GraphQL lib, `graphene_django`.

```
pipenv install graphene_django
```

What next? Add to `INSTALLED_APPS`.

`djorg/settings.py`:

```python
    'graphene_django',
```

## Define a GraphQL Schema

Next, we need to define the schema that describes the data.

```python
from graphene_django import DjangoObjectType
import graphene
from .models import PersonalNote

class PersonalNoteType(DjangoObjectType):
    """Describe which model we want to expose through GraphQL."""
    class Meta:
        model = PersonalNote

        # Describe the data as a node in a graph for GraphQL
        interfaces = (graphene.relay.Node, )

class Query(graphene.ObjectType):
    """Describe which records we want to show."""
    personalnotes = graphene.List(PersonalNoteType)

    def resolve_personalnotes(self, info):
        """Decide what notes to return."""
        pass # TODO
```

Note the analogies between this and what we did to set up REST in `notes/api.py`.

Now let's finish `resolve_notes()` in `Query`:

```python
from django.conf import settings
from graphene_django import DjangoObjectType
import graphene
from .models import PersonalNote

class PersonalNoteType(DjangoObjectType):
    """Describe which model we want to expose through GraphQL."""
    class Meta:
        model = PersonalNote

        # Describing the data as a node in a graph for GraphQL
        interfaces = (graphene.relay.Node, )

class Query(graphene.ObjectType):
    """Describe which records we want to show."""
    personalnotes = graphene.List(PersonalNoteType)

    def resolve_personalnotes(self, info):
        """Decide what notes to return."""
        user = info.context.user  # Find this with the debugger

        if user.is_anonymous:
            return PersonalNoteModel.objects.none()
        else:
            return PersonalNoteModel.objects.filter(user=user)

# Add a schema and attach to the query
schema = graphene.Schema(query=Query)
```

Lastly, at the very bottom, we had to tell graphene that schema query behavior
is based on the `Query` class we just defined.

## Configure graphene and set the URL endpoint

Now what do we have to do to get it going?

So far we've
* Installed graphene
* Wrote this schema describing the data

Let's configure in djorg/settings.py:

```python
GRAPHENE = {
    'SCHEMA': 'notes.schema.schema'  # dir.file.varname
}
```

Need to make an endpoint URL. Just a single one for all the queries.

`django/urls.py`:

```python
from graphene_django.views import GraphQLView
```

```python
    # in urlpatterns:
    path('graphql/', GraphQLView.as_view(graphiql=True)),
```

graphiql ("graphy-QL") is the UI we were experimenting with initially. It comes
built-in with graphene!

Don't have to specify the schema here since we did that in the settings.

```
./manage.py runserver to see if it works.
```

Hit the `/graphql/` endpoint to see the graphiql interface.

Check out the Docs tab and see all the information about our Notes. Try a query:

```graphql
{
  personalnotes
}
```

More complex:

```graphql
{
  personalnotes {
    title
    content
    lastModified
  }
}
```

There we go!

You can do things at this point to set up more advanced filters, not include
fields in the output, etc.

You can decide to write a front end to either REST or GraphQL.
