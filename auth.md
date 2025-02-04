# **Authentication**

Authentication for the Top Villas API follows a **JWT (JSON Web Token) Security Token flow**. To interact with the API, you must **obtain a token** using your credentials and include it in the **Authorization header** of every request.

## **Authentication Flow**  

Authentication is based on the **JWT (JSON Web Token)** security scheme.

### 1. Obtain a JWT Token

- Send your credentials to the authentication endpoint.  
- Receive two tokens in response:  
  - A **JWT access token**, which is **short-lived** and must be included in all API requests.  
  - A **refresh token**, which is **long-lived** and can be used to obtain new access tokens.  

### 2. Include the Access Token in API Requests

- Attach the **access token** in the `Authorization` header using the `Bearer` scheme:  

  ```http
  Authorization: Bearer your_access_token
  ```

- Do not persist the access token—store it only in memory and discard it once expired.

### 3. Refresh the Access Token Before Expiration

- Before the access token expires, use the **refresh token** to obtain a new one.  
- The refresh token is **persistent** and can be securely stored for long-term authentication.  

#### When Refreshing

1. Submit the **refresh token** to the refresh endpoint.  
2. Receive a **new access token** (and possibly a new refresh token).
3. **Discard the old access token** and use the new one for authentication

**NOTE:** when your refresh token expires you simply need to login again.

## Obtain a JWT Token

Request a new JWT token.

### Login Endpoint

```bash
GET /api/auth/login
```

### Login Request

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "LoginRequest",
  "type": "object",
  "properties": {
    "username": {
      "type": "string",
      "description": "The username or email associated with the API account."
    },
    "password": {
      "type": "string",
      "description": "The password for the API account."
    }
  },
  "required": ["username", "password"]
}
```

```bash
curl -X POST https://{api_host}/api/auth/login \
     -H "Content-Type: application/json" \
     -d '{
           "username": "your_username",
           "password": "your_password"
         }'
```

### Login Response

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "LoginResponse",
  "type": "object",
  "properties": {
    "token": {
      "type": "string",
      "description": "The JWT access token to be used for authenticated requests."
    },
    "refresh_token": {
      "type": "string",
      "description": "A refresh token that can be used to obtain a new access token when it expires."
    },
    "expires_in": {
      "type": "integer",
      "description": "The time in seconds until the access token expires."
    }
  },
  "required": ["token", "refresh_token", "expires_in"]
}

```

```bash
{
  "token": "your_jwt_token",
  "refresh_token": "your_jwt_refresh_token"
  "expires_in": 3600
}
```

## Making an Authenticated API Request

Once you have the JWT, include it in the Authorization header as a Bearer token:

### Authenticated Request

```bash
curl -X GET https://{api_host}/api/protected/resource \
     -H "Authorization: Bearer your_jwt_token"
```

## Refreshing the JWT Token

Tokens have an expiration time, so you need to refresh them before they expire (or you can login again).

### Refresh Endpoint

```bash
GET /api/auth/refresh
```

### Refresh Request

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "RefreshTokenRequest",
  "type": "object",
  "properties": {
    "refresh_token": {
      "type": "string",
      "description": "The refresh token obtained during login, used to get a new access token."
    }
  },
  "required": ["refresh_token"]
}

```

```bash
curl -X POST https://{api_host}/api/auth/refresh \
     -H "Authorization: Bearer your_jwt_refresh_token"
```

### Refresh Response

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "RefreshTokenResponse",
  "type": "object",
  "properties": {
    "token": {
      "type": "string",
      "description": "The new JWT access token to be used for authenticated requests."
    },
    "refresh_token": {
      "type": "string",
      "description": "A new refresh token, if issued."
    },
    "expires_in": {
      "type": "integer",
      "description": "The time in seconds until the new access token expires."
    }
  },
  "required": ["token", "expires_in"],
  "anyOf": [
    { "required": ["refresh_token"] }
  ]
}
```

```bash
{
  "token": "your_new_jwt_token",
  "refresh_token": "your_new_refresh_token",
  "expires_in": 3600
}
```
