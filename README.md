### hapi-auth-jwt

[**hapi**](https://github.com/japijs/hapi) JSON Web Token (JWT) authentication plugin

JSON Web Token authentication requires verifying a signed token. The `'jwt'` scheme takes the following options:

### Hapi < 17
Use `hapi-auth-jwt < 5` for `hapi < 16`

### Hapi >= 17
Use `hapi-auth-jwt >= 5` for `hapi >= 17`

- `key` - (required) The private key the token was signed with.
- `validateFunc` - (optional) validation and user lookup function with the signature `function(token, callback)` where:
    - `token` - the verified and decoded jwt token
    - `callback` - a callback function with the signature `function(err, isValid, credentials)` where:
        - `err` - an internal error.
        - `isValid` - `true` if the token was valid otherwise `false`.
        - `credentials` - a credentials object passed back to the application in `request.auth.credentials`. Typically, `credentials` are only
          included when `isValid` is `true`, but there are cases when the application needs to know who tried to authenticate even when it fails
          (e.g. with authentication mode `'try'`).
- `audience` (optional): string or array of strings of valid values for the `aud` field.
- `issuer` (optional): string or array of strings of valid values for the `iss` field.
- `algorithms` (optional): List of strings with the names of the allowed algorithms. For instance, `["HS256", "RS256"]`.
- `subject` (optional): string of valid values for the `sub` field

See the example folder for an executable example.

```javascript

const Hapi = require('hapi');
const jwt = require('jsonwebtoken');
const server = new Hapi.Server({ port: 8080 });


const accounts = {
    123: {
        id: 123,
        user: 'john',
        fullName: 'John Doe',
        scope: ['a', 'b']
    }
};


const privateKey = 'BbZJjyoXAdr8BUZuiKKARWimKfrSmQ6fv8kZ7OFfc';

// Use this token to build your request with the 'Authorization' header.
// Ex:
//     Authorization: Bearer <token>
const token = jwt.sign({ accountId: 123 }, privateKey);


const validate = async function (decodedToken, extraInfo) {

    var credentials = accounts[decodedToken.accountId] || {};

    if (!credentials) {
        return { isValid: false };
    }

    return {
      isValid: true,
      credentials
    }
};


await server.register(require('hapi-auth-jwt'));

server.auth.strategy('token', 'jwt', {
    key: privateKey,
    validateFunc: validate
});

server.route({
    method: 'GET',
    path: '/',
    config: {
        auth: 'token'
    }
});

    // With scope requirements
server.route({
    method: 'GET',
    path: '/withScope',
    config: {
        auth: {
            strategy: 'token',
            scope: ['a']
        }
    }
});


await server.start();

```

You can specify audience, issuer, algorithms and/or subject as well:

```javascript
server.auth.strategy('token', 'jwt', {
    key: privateKey,
    validateFunc: validate,
    audience: 'http://myapi/protected',
    issuer: 'http://issuer',
    algorithms: ['RS256'],
    subject: 'myRequiredSubject'
});
```
