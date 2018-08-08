# GraphQL: Mutations

Mutations are how we change information in the database. In this example, we'll add a mutation to create a new `PersonalNote`.

## Example Mutation Query

```graphql
mutation {
  createPersonalNote(title:"test 5", content:"content 5") {
    note {
      title
      content
    }
    ok
  }
}
```

## Adding the Mutation

In `schema.py` where we already defined our query, we'll now define a _mutation_
that allows us to create a new record. Add it below where we defined
`PersonalNote`.


```python
class CreatePersonalNote(graphene.Mutation):
    """Handle the incoming data to create the new PersonalNote."""

    class Arguments:
        # Input attributes for the mutation
        title = graphene.String()
        content = graphene.String()

    ok = graphene.Boolean()
    note = graphene.Field(PersonalNote)

    def mutate(self, info, title, content):
        """Create the new PersonalNote."""
        new_user = info.context.user

        if new_user.is_anonymous:
            new_ok = False
            return CreatePersonalNote(ok=new_ok)
        else:
            new_note = PersonalNote(title=title, content=content, user=new_user)
            new_ok = True
            new_note.save()
            return CreatePersonalNote(note=new_note, ok=new_ok)

class Mutation(graphene.ObjectType):
    create_note = CreatePersonalNote.Field()

# Add a schema and attach the mutation, as well as the query
schema = graphene.Schema(query=Query, mutation=Mutation)
```

Run the example query up top. What happens? What happens if you leave off the `title` field?