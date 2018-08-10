# GraphQL: Mutations

Mutations are how we change information in the database. In this example, we'll add a mutation to create a new `PersonalNote`.

## Example Mutation Query

```graphql
mutation {
  createPersonalNote(title:"test 5", content:"content 5") {
    personalnote {
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

    class Arguments:
        title = graphene.String()
        content = graphene.String()

    personalnote = graphene.Field(PersonalNote)
    ok = graphene.Boolean()
    status = graphene.String()

    def mutate(self, info, title, content):
        user = info.context.user

        if user.is_anonymous:
            return CreatePersonalNote(ok=False, status="Must be logged in!")
        else:
            new_note = PersonalNoteModel(title=title, content=content, user=user)
            new_note.save()
            return CreatePersonalNote(personalnote=new_note, ok=True, status="ok")

class Mutation(graphene.ObjectType):
    create_personal_note = CreatePersonalNote.Field()

schema = graphene.Schema(query=Query, mutation=Mutation)
```

Run the example query up top. What happens? What happens if you leave off the `title` field? What happens if we're logged in as an anonymous user?
