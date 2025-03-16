# GraphQL

## Queries

At its simplest, GraphQL is about asking for specific fields on objects. The `useQuery` React hook is the primary API for executing queries in an Apollo application. To run a query within a React component, call useQuery and pass it a GraphQL query string. When your component renders, useQuery returns an object from Apollo Client that contains `loading`, `error`, and `data` properties you can use to render your UI.

```javascript
import { gql, useQuery } from "@apollo/client";

const GET_DOGS = gql`
  query GetDogs {
    dogs {
      id
      breed
    }
  }
`;
```

```jsx
function Dogs({ onDogSelected }) {
  const { loading, error, data } = useQuery(GET_DOGS);

  if (loading) return "Loading...";
  if (error) return `Error! ${error.message}`;

  return (
    <select name="dog" onChange={onDogSelected}>
      {data.dogs.map((dog) => (
        <option key={dog.id} value={dog.breed}>
          {dog.breed}
        </option>
      ))}
    </select>
  );
}
```

## Arguments

In GraphQL, every field and nested object can get its own set of arguments, making GraphQL a complete replacement for making multiple API fetches. You can even pass arguments into scalar fields, to implement data transformations once on the server, instead of on every client separately.

GraphQL:

```graphql
{
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}
```

Apollo:

```jsx
const GET_DOG_PHOTO = gql`
  query Dog($breed: String!) {
    dog(breed: $breed) {
      id
      displayImage
    }
  }
`;

function DogPhoto({ breed }) {
  const { loading, error, data } = useQuery(GET_DOG_PHOTO, {
    variables: { breed },
  });

  if (loading) return null;
  if (error) return `Error! ${error}`;

  return (
    <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
  );
}
```

## Aliases

Since the result object fields match the name of the field in the query but don’t include arguments, you can’t directly query for the same field with different arguments. That’s why you need aliases - they let you rename the result of a field to anything you want.

Query:

```graphql
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

Result:

```json
{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}
```

## Fragments

GraphQL includes reusable units called fragments. Fragments let you construct sets of fields, and then include them in queries where you need to.

GraphQL:

```graphql
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}
```

With a variable (with default value):

```graphql
query HeroComparison($first: Int = 3) {
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  friendsConnection(first: $first) {
    totalCount
    edges {
      node {
        name
      }
    }
  }
}
```

Apollo:

```javascript
import { gql } from "@apollo/client";

export const CORE_COMMENT_FIELDS = gql`
  fragment CoreCommentFields on Comment {
    id
    postedBy {
      username
      displayName
    }
    createdAt
    content
  }
`;
```

```javascript
import { gql } from "@apollo/client";
import { CORE_COMMENT_FIELDS } from "./fragments";

export const GET_POST_DETAILS = gql`
  ${CORE_COMMENT_FIELDS}
  query CommentsForPost($postId: ID!) {
    post(postId: $postId) {
      title
      body
      author
      comments {
        ...CoreCommentFields
      }
    }
  }
`;
```

#### Inline Fragments

If you are querying a field that returns an interface or a union type, you will need to use inline fragments to access data on the underlying concrete type.

```graphql
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
    ... on Human {
      height
    }
  }
}
```

Variables:

```graphql
{
  "ep": "JEDI"
}
```

Result:

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "primaryFunction": "Astromech"
    }
  }
}
```

In this query, the hero field returns the type Character, which might be either a Human or a Droid depending on the episode argument. In the direct selection, you can only ask for fields that exist on the Character interface, such as name.

To ask for a field on the concrete type, you need to use an inline fragment with a type condition. Because the first fragment is labeled as ... on Droid, the primaryFunction field will only be executed if the Character returned from hero is of the Droid type. Similarly for the height field for the Human type.

## Directives

A directive can be attached to a field or fragment inclusion, and can affect execution of the query in any way the server desires. The core GraphQL specification includes exactly two directives, which must be supported by any spec-compliant GraphQL server implementation:

- `@include(if: Boolean)` Only include this field in the result if the argument is true.

- `@skip(if: Boolean)` Skip this field if the argument is true.
  Directives can be useful to get out of situations where you otherwise would need to do string manipulation to add and remove fields in your query.

GraphQL:

```graphql
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}
```

Apollo Directives:

### @client

The `@client` directive allows you to resolve client-only data alongside your server data. These fields are not sent to the GraphQL server

```graphql
query LaunchDetails($launchId: ID!) {
  launch(id: $launchId) {
    site
    rocket {
      type
      # resolved locally on the client,
      # removed from the request to the server
      description @client
    }
  }
}
```

### @connection

The `@connection` directive allows you to specify a custom cache key for paginated results

```graphql
query Feed($offset: Int, $limit: Int) {
  feed(offset: $offset, limit: $limit) @connection(key: "feed") {
    ...FeedFields
  }
}
```

### @defer

This directive enables your queries to receive data for specific fields incrementally, instead of receiving all field data at the same time. This is helpful whenever some fields in a query take much longer to resolve than others.

```graphql
query PersonQuery($personId: ID!) {
  person(id: $personId) {
    # Basic fields (fast)
    id
    firstName
    lastName

    # Friend fields (slower)
    ... @defer {
      friends {
        id
      }
    }
  }
}
```

### @export

If your GraphQL query uses variables, the local-only fields of that query can provide the values of those variables.

```javascript
const GET_CURRENT_AUTHOR_POST_COUNT = gql`
  query CurrentAuthorPostCount($authorId: Int!) {
    currentAuthorId @client @export(as: "authorId")
    postCount(authorId: $authorId)
  }
`;
```

In the query above, the result of the local-only field `currentAuthorId` is used as the value of the `$authorId` variable that's passed to `postCount`.

### @nonreactive

The `@nonreactive` directive can be used to mark query fields or fragment spreads and is used to indicate that changes to the data contained within the subtrees marked `@nonreactive` should not trigger rerendering. This allows parent components to fetch data to be rendered by their children without rerendering themselves when the data corresponding with fields marked as `@nonreactive` change.

```javascript
const TrailFragment = gql`
  fragment TrailFragment on Trail {
    name
    status
  }
`;

const ALL_TRAILS = gql`
  query allTrails {
    allTrails {
      id
      ...TrailFragment @nonreactive
    }
  }
  ${TrailFragment}
`;

function App() {
  const { data, loading } = useQuery(ALL_TRAILS);
  return (
    <main>
      <h2>Ski Trails</h2>
      <ul>
        {data?.trails.map((trail) => (
          <Trail key={trail.id} id={trail.id} />
        ))}
      </ul>
    </main>
  );
}
```

The `Trail` component renders a trail's name and status and allows the user to execute a mutation to toggle the status of the trail between `"OPEN"` and `"CLOSED"`:

```jsx
const Trail = ({ id }) => {
  const [updateTrail] = useMutation(UPDATE_TRAIL);
  const { data } = useFragment({
    fragment: TrailFragment,
    from: {
      __typename: "Trail",
      id,
    },
  });
  return (
    <li key={id}>
      {data.name} - {data.status}
      <input
        checked={data.status === "OPEN" ? true : false}
        type="checkbox"
        onChange={(e) => {
          updateTrail({
            variables: {
              trailId: id,
              status: e.target.checked ? "OPEN" : "CLOSED",
            },
          });
        }}
      />
    </li>
  );
};
```

Notice that the `Trail` component isn't receiving the entire `trail` object via props, only the `id` which is used along with the fragment document to create a live binding for each `trail` item in the cache. This allows each `Trail` component to react to the cache updates for a single trail independently. Updates to a trail's `status` will not cause the parent `App` component to rerender since the `@nonreactive` directive is applied to the `TrailFragment` spread, a fragment that includes the `status` field.

## Mutations

In REST, any request might end up causing some side-effects on the server, but by convention it’s suggested that one doesn’t use GET requests to modify data. GraphQL is similar - technically any query could be implemented to cause a data write. However, it’s useful to establish a convention that any operations that cause writes should be sent explicitly via a mutation.

Just like in queries, if the mutation field returns an object type, you can ask for nested fields. This can be useful for fetching the new state of an object after an update.

GraphQL:

```graphql
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

Apollo:

```javascript
import { gql, useMutation } from "@apollo/client";

// Define mutation
const INCREMENT_COUNTER = gql`
  # Increments a back-end counter and gets its resulting value
  mutation IncrementCounter {
    currentValue
  }
`;

function MyComponent() {
  // Pass mutation to useMutation
  const [mutateFunction, { data, loading, error }] =
    useMutation(INCREMENT_COUNTER);
}
```

With options/variables:

```javascript
const [addTodo, { data, loading, error }] = useMutation(ADD_TODO, {
  variables: {
    type: "placeholder",
    someOtherVariable: 1234,
  },
});
```

## Meta fields

Given that there are some situations where you don’t know what type you’ll get back from the GraphQL service, you need some way to determine how to handle that data on the client. GraphQL allows you to request `__typename`, a meta field, at any point in a query to get the name of the object type at that point.

```graphql
{
  search(text: "an") {
    __typename
    ... on Human {
      name
    }
    ... on Droid {
      name
    }
    ... on Starship {
      name
    }
  }
}
```

Result:

```json
{
  "data": {
    "search": [
      {
        "__typename": "Human",
        "name": "Han Solo"
      },
      {
        "__typename": "Human",
        "name": "Leia Organa"
      },
      {
        "__typename": "Starship",
        "name": "TIE Advanced x1"
      }
    ]
  }
}
```

In the above query, search returns a union type that can be one of three options. It would be impossible to tell apart the different types from the client without the \_\_typename field.

## Schemas and Types

The GraphQL query language is basically about selecting fields on objects.

### Object types and fields

The most basic components of a GraphQL schema are object types, which just represent a kind of object you can fetch from your service, and what fields it has.

```graphql
type Character {
  name: String!
  appearsIn: [Episode!]!
}
```

- `Character` is a GraphQL Object Type, meaning it’s a type with some fields. Most of the types in your schema will be object types.
- `name` and `appearsIn` are fields on the Character type. That means that name and appearsIn are the only fields that can appear in any part of a GraphQL query that operates on the Character type.
- `String` is one of the built-in scalar types - these are types that resolve to a single scalar object, and can’t have sub-selections in the query. We’ll go over scalar types more later.
- `String!` means that the field is non-nullable, meaning that the GraphQL service promises to always give you a value when you query this field. In the type language, we’ll represent those with an exclamation mark.
- `[Episode!]!` represents an array of Episode objects. Since it is also non-nullable, you can always expect an array (with zero or more items) when you query the appearsIn field. And since Episode! is also non-nullable, you can always expect every item of the array to be an Episode object.

### Arguments

All arguments are named. Unlike languages like JavaScript and Python where functions take a list of ordered arguments, all arguments in GraphQL are passed by name specifically.

```graphql
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```

In this case, the length field has one defined argument, `unit`.

Arguments can be either required or optional. When an argument is optional, we can define a default value - if the `unit` argument is not passed, it will be set to `METER` by default.

### The Query and Mutation types

Most types in your schema will just be normal object types, but there are two types that are special within a schema:

```graphql
schema {
  query: Query
  mutation: Mutation
}
```

Every GraphQL service has a `query` type and may or may not have a `mutation` type. These types are the same as a regular object type, but they are special because they define the entry point of every GraphQL query. So if you see a query that looks like:

```graphql
query {
  hero {
    name
  }
  droid(id: "2000") {
    name
  }
}
```

Result:

```json
{
  "data": {
    "hero": {
      "name": "R2-D2"
    },
    "droid": {
      "name": "C-3PO"
    }
  }
}
```

That means that the GraphQL service needs to have a Query type with `hero` and `droid` fields:

```graphql
type Query {
  hero(episode: Episode): Character
  droid(id: ID!): Droid
}
```

Mutations work in a similar way - you define fields on the `Mutation` type, and those are available as the root mutation fields you can call in your query.

It’s important to remember that other than the special status of being the "entry point" into the schema, the `Query` and `Mutation` types are the same as any other GraphQL object type, and their fields work exactly the same way.

### Scalar types

A GraphQL object type has a name and fields, but at some point those fields have to resolve to some concrete data. That’s where the scalar types come in: they represent the leaves of the query.

GraphQL comes with a set of default scalar types out of the box:

- `Int`: A signed 32‐bit integer.
- `Float`: A signed double-precision floating-point value.
- `String`: A UTF‐8 character sequence.
- `Boolean`: true or false.
- `ID`: The `ID` scalar type represents a unique identifier, often used to refetch an object or as the key for a cache. The `ID` type is serialized in the same way as a String; however, defining it as an `ID` signifies that it is not intended to be human‐readable.

In most GraphQL service implementations, there is also a way to specify custom scalar types. For example, we could define a Date type:

```graphql
scalar Date
```

Then it’s up to our implementation to define how that type should be serialized, deserialized, and validated.

### Enumeration types

Also called Enums, enumeration types are a special kind of scalar that is restricted to a particular set of allowed values. This allows you to:

- Validate that any arguments of this type are one of the allowed values
- Communicate through the type system that a field will always be one of a finite set of values

Here’s what an enum definition might look like in the GraphQL schema language:

```graphql
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

This means that wherever we use the type Episode in our schema, we expect it to be exactly one of `NEWHOPE`, `EMPIRE`, or `JEDI`.

### Lists and Non-Null

When you use types in other parts of the schema, or in your query variable declarations, you can apply additional type modifiers that affect validation of those values. Let’s look at an example:

```graphql
type Character {
  name: String!
  appearsIn: [Episode]!
}
```

Here, we’re using a `String` type and marking it as Non-Null by adding an exclamation mark, `!` after the type name. This means that our server always expects to return a non-null value for this field, and if it ends up getting a null value that will actually trigger a GraphQL execution error, letting the client know that something has gone wrong.

The Non-Null type modifier can also be used when defining arguments for a field, which will cause the GraphQL server to return a validation error if a null value is passed as that argument, whether in the GraphQL string or in the variables.

Lists work in a similar way: We can use a type modifier to mark a type as a `List`, which indicates that this field will return an array of that type. In the schema language, this is denoted by wrapping the type in square brackets, `[` and `]`. It works the same for arguments, where the validation step will expect an array for that value.

The Non-Null and List modifiers can be combined. For example, you can have a List of Non-Null Strings:

```graphql
myField: [String!]
```

This means that the list itself can be null, but it can’t have any null members.

### Interfaces

Like many type systems, GraphQL supports interfaces. An Interface is an abstract type that includes a certain set of fields that a type must include to implement the interface.

For example, you could have an interface `Character` that represents any character in the Star Wars trilogy:

```graphql
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}
```

This means that any type that implements `Character` needs to have these exact fields, with these arguments and return types.

For example, here are some types that might implement Character:

```graphql
type Human implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  starships: [Starship]
  totalCredits: Int
}

type Droid implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  primaryFunction: String
}
```

Interfaces are useful when you want to return an object or set of objects, but those might be of several different types.

To ask for a field on a specific object type, you need to use an inline fragment:

```graphql
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
  }
}
```

### Union types

Union types share similarities with interfaces; however, they lack the ability to define any shared fields among the constituent types.

```graphql
union SearchResult = Human | Droid | Starship
```

Wherever we return a `SearchResult` type in our schema, we might get a `Human`, a `Droid`, or a `Starship`. Note that members of a union type need to be concrete object types; you can’t create a union type out of interfaces or other unions.

In this case, if you query a field that returns the `SearchResult` union type, you need to use an inline fragment to be able to query any fields at all:

```graphql
{
  search(text: "an") {
    __typename
    ... on Human {
      name
      height
    }
    ... on Droid {
      name
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

Since Human and Droid share a common interface (`Character`), you can query their common fields in one place rather than having to repeat the same fields across multiple types:

```graphql
{
  search(text: "an") {
    __typename
    ... on Character {
      name
    }
    ... on Human {
      height
    }
    ... on Droid {
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

Note that name is still specified on `Starship` because otherwise it wouldn’t show up in the results given that `Starship` is not a `Character`!

### Input types

You can pass complex objects as arguments to a field. This is particularly valuable in the case of mutations, where you might want to pass in a whole object to be created. In the GraphQL schema language, input types look exactly the same as regular object types, but with the keyword `input` instead of `type`:

```graphql
input ReviewInput {
  stars: Int!
  commentary: String
}
```

Use:

```graphql
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

Variables:

```graphql
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

The fields on an input object type can themselves refer to input object types, but you can’t mix input and output types in your schema. Input object types also can’t have arguments on their fields.

## Execution

After being validated, a GraphQL query is executed by a GraphQL server which returns a result that mirrors the shape of the requested query, typically as JSON. GraphQL cannot execute a query without a type system.

### Root fields & resolvers

At the top level of every GraphQL server is a type that represents all of the possible entry points into the GraphQL API, it’s often called the Root type or the Query type.

In this example, our Query type provides a field called `human` which accepts the argument `id`. The resolver function for this field likely accesses a database and then constructs and returns a `Human` object.

```graphql
Query: {
  human(obj, args, context, info) {
    return context.db.loadHumanByID(args.id).then(
      userData => new Human(userData)
    )
  }
}
```

Resolver functions receive four arguments:

- `obj` The previous object, which for a field on the root Query type is often not used.
- `args` The arguments provided to the field in the GraphQL query.
- `context` A value which is provided to every resolver and holds important contextual information like the currently logged in user, or access to a database.
- `info` A value which holds field-specific information relevant to the current query as well as the schema details

### Asynchronous resolvers

```javascript
human(obj, args, context, info) {
  return context.db.loadHumanByID(args.id).then(
    userData => new Human(userData)
  )
}
```

The `context` is used to provide access to a database which is used to load the data for a user by the `id` provided as an argument in the GraphQL query. Since loading from a database is an asynchronous operation, this returns a `Promise`. In JavaScript, Promises are used to work with asynchronous values, but the same concept exists in many languages, often called Futures, Tasks or Deferred. When the database returns, we can construct and return a new `Human` object.

Notice that while the resolver function needs to be aware of Promises, the GraphQL query does not. It simply expects the `human` field to return something which it can then ask the `name` of. During execution, GraphQL will wait for Promises, Futures, and Tasks to complete before continuing and will do so with optimal concurrency.

### Trivial resolvers

```javascript
Human: {
  name(obj, args, context, info) {
    return obj.name
  }
}
```

Resolving the name in this case is very straight-forward. The name resolver function is called and the `obj` argument is the `new Human` object returned from the previous field. In this case, we expect that Human object to have a `name` property which we can read and return directly.

In fact, many GraphQL libraries will let you omit resolvers this simple and will just assume that if a resolver isn’t provided for a field, that a property of the same name should be read and returned.

### Scalar coercion

The type system knows what to expect and will convert the values returned by a resolver function into something that upholds the API contract.

While the `name` field is being resolved, the `appearsIn` and `starships` fields can be resolved concurrently. The `appearsIn` field could also have a trivial resolver, but let’s take a closer look:

```javascript
Human: {
  appearsIn(obj) {
    return obj.appearsIn // returns [ 4, 5, 6 ]
  }
}
```

Notice that our type system claims `appearsIn` will return Enum values with known values, however this function is returning numbers! Indeed if we look up at the result we’ll see that the appropriate Enum values are being returned.

### List resolvers

```javascript
Human: {
  starships(obj, args, context, info) {
    return obj.starshipIDs.map(
      id => context.db.loadStarshipByID(id).then(
        shipData => new Starship(shipData)
      )
    )
  }
}
```

The resolver for this field is not just returning a Promise, it’s returning a list of Promises. The `Human` object had a list of ids of the `Starships` they piloted, but we need to go load all of those ids to get real `Starship` objects.

GraphQL will wait for all of these Promises concurrently before continuing, and when left with a list of objects, it will concurrently continue yet again to load the `name` field on each of these items.

## Introspection

It’s often useful to ask a GraphQL schema for information about what queries it supports. GraphQL allows us to do so using the introspection system!

We designed the type system, so we know what types are available, but if we didn’t, we can ask GraphQL, by querying the `__schema` field, always available on the root type of a Query. Let’s do so now, and ask what types are available.

```graphql
{
  __schema {
    types {
      name
    }
  }
}
```

Resulting types from example:

- `Query`, `Character`, `Human`, `Episode`, `Droid` - These are the ones that we defined in our type system.
- `String`, `Boolean` - These are built-in scalars that the type system provided.
- `__Schema`, `__Type`, `__TypeKind`, `__Field`, `__InputValue`, `__EnumValue`, `__Directive` - These all are preceded with a double underscore, indicating that they are part of the introspection system.

Finding out a type:

```graphql
{
  __type(name: "Droid") {
    name
    kind
  }
}
```

Result:

```json
{
  "data": {
    "__type": {
      "name": "Droid",
      "kind": "OBJECT"
    }
  }
}
```

`kind` returns a `__TypeKind` enum, one of whose values is `OBJECT`. If we asked about `Character` instead we’d find that it is an interface:

```graphql
{
  __type(name: "Character") {
    name
    kind
  }
}
```

Result:

```json
{
  "data": {
    "__type": {
      "name": "Character",
      "kind": "INTERFACE"
    }
  }
}
```

Another example:

```graphql
{
  __type(name: "Droid") {
    name
    fields {
      name
      type {
        name
        kind
        ofType {
          name
          kind
        }
      }
    }
  }
}
```

Result:

```json
{
  "data": {
    "__type": {
      "name": "Droid",
      "fields": [
        {
          "name": "id",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "ID",
              "kind": "SCALAR"
            }
          }
        },
        {
          "name": "name",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "String",
              "kind": "SCALAR"
            }
          }
        },
        {
          "name": "friends",
          "type": {
            "name": null,
            "kind": "LIST",
            "ofType": {
              "name": "Character",
              "kind": "INTERFACE"
            }
          }
        },
        {
          "name": "friendsConnection",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "FriendsConnection",
              "kind": "OBJECT"
            }
          }
        },
        {
          "name": "appearsIn",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": null,
              "kind": "LIST"
            }
          }
        },
        {
          "name": "primaryFunction",
          "type": {
            "name": "String",
            "kind": "SCALAR",
            "ofType": null
          }
        }
      ]
    }
```

Getting documentation:

```graphql
{
  __type(name: "Droid") {
    name
    description
  }
}
```

## Best Practices

Available [here](https://graphql.org/learn/best-practices/) covering HTTP, JSON (with GZIP), Versioning, Nullability, Pagination, Server-side Batching & Caching.

# Javscript Example

Taken from [Graphql Crash Course](https://github.com/iamshaunjp/graphql-crash-course) using Apollo.

## \_db.js

```javascript
let games = [
  { id: "1", title: "Zelda, Tears of the Kingdom", platform: ["Switch"] },
  { id: "2", title: "Final Fantasy 7 Remake", platform: ["PS5", "Xbox"] },
  { id: "3", title: "Elden Ring", platform: ["PS5", "Xbox", "PC"] },
  { id: "4", title: "Mario Kart", platform: ["Switch"] },
  { id: "5", title: "Pokemon Scarlet", platform: ["PS5", "Xbox", "PC"] },
];

let authors = [
  { id: "1", name: "mario", verified: true },
  { id: "2", name: "yoshi", verified: false },
  { id: "3", name: "peach", verified: true },
];

let reviews = [
  { id: "1", rating: 9, content: "lorem ipsum", author_id: "1", game_id: "2" },
  { id: "2", rating: 10, content: "lorem ipsum", author_id: "2", game_id: "1" },
  { id: "3", rating: 7, content: "lorem ipsum", author_id: "3", game_id: "3" },
  { id: "4", rating: 5, content: "lorem ipsum", author_id: "2", game_id: "4" },
  { id: "5", rating: 8, content: "lorem ipsum", author_id: "2", game_id: "5" },
  { id: "6", rating: 7, content: "lorem ipsum", author_id: "1", game_id: "2" },
  { id: "7", rating: 10, content: "lorem ipsum", author_id: "3", game_id: "1" },
];

export default { games, authors, reviews };
```

## schema.js

```javascript
export const typeDefs = `#graphql
  type Game {
    id: ID!
    title: String!
    platform: [String!]!
    reviews: [Review!]
  }
  type Review {
    id: ID!
    rating: Int!
    content: String!
    author: Author!
    game: Game!
  }
  type Author {
    id: ID!
    name: String!
    verified: Boolean!
    reviews: [Review!]
  }
  type Query {
    games: [Game]
    game(id: ID!): Game
    reviews: [Review]
    review(id: ID!): Review
    authors: [Author]
    author(id: ID!): Author
  }
  type Mutation {
    addGame(game: AddGameInput!): Game
    deleteGame(id: ID!): [Game]
    updateGame(id: ID!, edits: EditGameInput): Game
  }
  input AddGameInput {
    title: String!,
    platform: [String!]!
  }
  input EditGameInput {
    title: String,
    platform: [String!]
  }
`;
```

## index.js

```javascript
import { ApolloServer } from "@apollo/server";
import { startStandaloneServer } from "@apollo/server/standalone";

// data
import db from "./_db.js";

// types
import { typeDefs } from "./schema.js";

// resolvers
const resolvers = {
  Query: {
    games() {
      return db.games;
    },
    game(_, args) {
      return db.games.find((game) => game.id === args.id);
    },
    authors() {
      return db.authors;
    },
    author(_, args) {
      return db.authors.find((author) => author.id === args.id);
    },
    reviews() {
      return db.reviews;
    },
    review(_, args) {
      return db.reviews.find((review) => review.id === args.id);
    },
  },
  Game: {
    reviews(parent) {
      return db.reviews.filter((r) => r.game_id === parent.id);
    },
  },
  Review: {
    author(parent) {
      return db.authors.find((a) => a.id === parent.author_id);
    },
    game(parent) {
      return db.games.find((g) => g.id === parent.game_id);
    },
  },
  Author: {
    reviews(parent) {
      return db.reviews.filter((r) => r.author_id === parent.id);
    },
  },
  Mutation: {
    addGame(_, args) {
      let game = {
        ...args.game,
        id: Math.floor(Math.random() * 10000).toString(),
      };
      db.games.push(game);

      return game;
    },
    deleteGame(_, args) {
      db.games = db.games.filter((g) => g.id !== args.id);

      return db.games;
    },
    updateGame(_, args) {
      db.games = db.games.map((g) => {
        if (g.id === args.id) {
          return { ...g, ...args.edits };
        }

        return g;
      });

      return db.games.find((g) => g.id === args.id);
    },
  },
};

// server setup
const server = new ApolloServer({
  typeDefs,
  resolvers,
});

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
});

console.log(`Server ready at: ${url}`);
```
