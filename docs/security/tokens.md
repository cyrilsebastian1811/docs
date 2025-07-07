# __:simple-jsonwebtokens:{.lg .top} Tokens__

In the context of authentication and authorization, tokens are a security measure used to verify the identity of users and control their access to resources.

Tokens can be class classified based on their purpose:

1. [Authentication tokens](#authentication-tokens)
2. [Authorization tokens](#authorization-tokens)

Or, based on data it carries:

1. [Opaque tokens](#opaque-tokens)
2. [Structured tokens](#structured-tokens)


## Authentication tokens

These are used to verify a user's identity. After a user logs in with their credentials (like a username and password), the server generates a unique token for the user. This token is then sent back to the user's client (like a web browser), and the client includes the token in subsequent requests to the server. The server checks the token to verify the user's identity without needing the user to re-enter their credentials.


## Authorization tokens

They are used to control a user's access to resources. After a user's identity is verified, the server generates a token that includes information about the user's permissions. This token is included in requests to access resources, and the server checks the token to determine whether the user has permission to access the requested resources.


## Opaque tokens

Opaque tokens, aka __Bearer token__, are a type of token used in authentication and authorization that contain no information about the user or authorization context in their content. They are simply a reference to the server-side data, where the actual user details and permissions are stored.

The term __"opaque"__ is used because from the client's perspective, the token is just a string of characters with no inherent meaning or structure. The token's value is only meaningful to the server that issued it and can interpret it.

Here's an example of an opaque token: `Authorization: Bearer a9f0e61a-7001-47a5-bb2a-2ad2c21f1589`. As you can see, it's just a string of characters with no inherent meaning or structure.

!!! warning

    - Token can only be validated by the resource server. Hence not scalable
    - Often used in scenarios where tokens need to be stored securely and not exposed to clients or transmitted over the network. They are typically used in OAuth 2.0 as reference tokens.


## Structured tokens

Structured tokens, aka JSON Web Tokens (JWTs), are a type of token that contain information about the user or authorization context in their content. Unlike opaque tokens, structured tokens are self-contained and can be validated and decoded by clients or other parties without needing to contact the server that issued them.

!!! info

    - Structured tokens are signed by authorization server(with it's private key). Which can be varified by clients(with authorization servers public key).
    - This circumvents unnecessary calls to authorization server for validation. Hence scalable


