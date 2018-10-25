# Adding filter arguments to queries

[Graphene Filtering Documentation](https://docs.graphene-python.org/projects/django/en/latest/filtering/)

## Install the filter module

```
pipenv install django-filter
```

## Filtering example

```python
from django.conf import settings
from graphene_django import DjangoObjectType
import graphene
from .models import Note

from graphene_django.filter import DjangoFilterConnectionField # <--

class NoteType(DjangoObjectType):

    class Meta:
        model = Note
        filter_fields = ['id', 'title', 'content']  # <--
        interfaces = (graphene.relay.Node,)

class Query(graphene.ObjectType):
    notes = DjangoFilterConnectionField(NoteType) # <--

schema = graphene.Schema(query=Query)
```

## More complex filtering

Similar to how Django `filter()` works.

```python
class NoteType(DjangoObjectType):

    class Meta:
        model = Note
        filter_fields = {
            'id': ['exact'],
            'title': ['exact', 'icontains', 'istartswith'],
            'content': ['exact', 'icontains'],
        }
        interfaces = (graphene.relay.Node,)
```

```graphql
query {
  # Note that fields names become camelcased
  notes(content_Icontains: "hello") {
    edges {
      node {
        title,
        content
      }
    }
  }
}
```
