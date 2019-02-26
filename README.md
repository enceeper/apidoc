# What is Enceeper?

[Enceeper](https://www.enceeper.com/) is a service used to store and organize users’ passwords and secrets. It provides high level of security since users’ information is not directly stored to the service.

The [Enceeper App](https://github.com/enceeper/enceeper) is using the service in the following way:

- All data are first encrypted locally and then transmitted over the network.
- Moreover, a user has the option to "share" those secrets with other users of the service by adding slots to selected entries.
- Finally, any third party can use the API of the service to enhance the core functionality.

To this end we have developed two solutions:

- [enceeper-phpconf](https://github.com/enceeper/enceeper-phpconf) to store configuration information in Enceeper and have it delivered to a PHP application
- [enceeper-boot](https://github.com/enceeper/enceeper-boot) for unattended booting a GNU/Linux distro via Initramfs utilizing Enceeper

# API documentation

In this document we describe the available API calls, the input they receive and the output they produce.

The current API version is: **1.0.0** and the base URL of the service is: [https://www.enceeper.com/api/v1](https://www.enceeper.com/api/v1)

The API uses [HTTP response codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) to indicate success or failure of requests. Specifically, codes in the 2xx range indicate success, codes in the 4xx range indicate an error that resulted from the provided arguments (e.g. instead of an integer a string is provided) and 5xx error codes indicate an internal Enceeper error.

> There is only one error that can be handled in a special way, the 403 error. This error is generated when the auth token has expired and the user needs to re-authenticate. The client can then check for this error and re-authenticate the user without requiring user interaction.

All inputs and outputs follow the JSON format and based on whether the outcome of an API call was sucessfull or not we have the following response templates:

Sucess response:
```
{
  "outcome": "OK",
  "result": {
    "data": {
      "one": "...",
      "two": "..."
    }
  },
  "errorMessage": null
}
```

> If an API call does not return any data then the **result** key above will be null.

Failed response:
```
{
  "outcome":      "NOK",
  "result":       null,
  "errorMessage": "API method not found"
}
```

## API calls

The summary of the API calls is:

| Title                                               | Method | URL                               |
|-----------------------------------------------------|--------|-----------------------------------|
| [Test call](#test-call)                             | GET    | /                                 |
| [User registration](#user-registration)             | POST   | /user                             |
| [Edit user](#edit-user)*                            | PUT    | /user                             |
| [Delete user](#delete-user)*                        | DELETE | /user                             |
| [Initiate auth procedure](#initiate-auth-procedure) | POST   | /user/challenge                   |
| [Authenticate user](#authenticate-user)             | POST   | /user/login                       |
| [Get specific key](#get-specific-key)               | GET    | /user/slots/{identifier}          |
| [Check for key approval](#check-for-key-approval)   | GET    | /user/slots/check/{ref}           |
| [Get account keys](#get-account-keys)*              | GET    | /user/keys                        |
| [Create new key](#create-new-key)*                  | POST   | /user/keys                        |
| [Edit key](#edit-key)*                              | PUT    | /user/keys/{keyId}                |
| [Delete key](#delete-key)*                          | DELETE | /user/keys/{keyId}                |
| [Create new slot](#create-new-slot)*                | POST   | /user/keys/{keyId}/slots          |
| [Edit slot](#edit-slot)*                            | PUT    | /user/keys/{keyId}/slots/{slotId} |
| [Delete slot](#delete-slot)*                        | DELETE | /user/keys/{keyId}/slots/{slotId} |
| [Search users](#search-users)*                      | POST   | /user/search                      |
| [Create key share](#create-key-share)*              | POST   | /user/keys/{keyId}/share          |
| [Accept key share](#accept-key-share)*              | POST   | /user/keys/shares/{shareId}       |
| [Delete key share](#delete-key-share)*              | DELETE | /user/keys/shares/{shareId}       |

> All API calls marked with * require the use of the **X-Enceeper-Auth** HTTP header with a valid authentication token retrieved from the **Authenticate user** API call.

### Test call

Test API call to verify network connectivity and check that the Enceeper service is working as expected.

| Type   | Value|
|--------|-|
| URL    | /|
| Method | GET|
| Input  | -|
| Output | -|

### User registration

Create a new user in the Enceeper service. The following constrains are in place for the provided JSON:

- the **email** key is required and must be a valid email address
- the **auth.srp6a.salt** key is required
- the **auth.srp6a.verifier** key is required
- all the **auth.srp6a.xxx** keys are reserved and must not be used
- all the **auth.enceeper.xxx** keys are reserved and must not be used
- the **auth.keys.pub** key is required and must contain the public key of the user to facilitate key sharing
- You can place anything inside the **auth** object and it will be stored by the Enceeper service
- The overall size of the **auth** object must not exceed the 12Kbytes

| Type   | Value|
|--------|-|
| URL    | /user|
| Method | POST|
| Input  | {<br>&nbsp;"email": "enceeper@example.com",<br>&nbsp;"auth": {<br>&nbsp;&nbsp;"srp6a": {<br>&nbsp;&nbsp;&nbsp;"salt": "hex salt",<br>&nbsp;&nbsp;&nbsp;"verifier": "hex verifier"<br>&nbsp;&nbsp;},<br>&nbsp;&nbsp;...<br>&nbsp;&nbsp;"keys": {<br>&nbsp;&nbsp;&nbsp;"pub": "the public key of the user used in key sharing",<br>&nbsp;&nbsp;&nbsp;...<br>&nbsp;&nbsp;},<br>&nbsp;&nbsp;...<br>&nbsp;}<br>}|
| Output | -|

> The Enceeper service is utilizing the [SRP6a protocol](http://srp.stanford.edu/design.html) for user registration and authentication. In the future we may provide additional protocols (i.e. SPAKE2).

### Edit user

Update user details. For the **auth** object the same constrains are in place as described above in the **User registration** section.

| Type   | Value|
|--------|-|
| URL    | /user|
| Method | PUT|
| Input  | {<br>&nbsp;"auth": {<br>&nbsp;&nbsp;"srp6a": {<br>&nbsp;&nbsp;&nbsp;"salt": "hex salt",<br>&nbsp;&nbsp;&nbsp;"verifier": "hex verifier"<br>&nbsp;&nbsp;},<br>&nbsp;&nbsp;...<br>&nbsp;&nbsp;"keys": {<br>&nbsp;&nbsp;&nbsp;"pub": "the public key of the user used in key sharing",<br>&nbsp;&nbsp;&nbsp;...<br>&nbsp;&nbsp;},<br>&nbsp;&nbsp;...<br>&nbsp;}<br>}|
| Output | -|

### Delete user

| Type   | Value|
|--------|-|
| URL    | /user|
| Method | DELETE|
| Input  | -|
| Output | -|

### Initiate auth procedure

The client will provide the user email and the Enceeper service will bootstrap the SRP6a protocol. The **ref** must be used in the **Authenticate user** API call below, in order to restore the information created in this procedure.

| Type   | Value|
|--------|--------------------------------------------------------------------------------------|
| URL    | /user/challenge|
| Method | POST|
| Input  | {<br>&nbsp;"email": "enceeper@example.com"<br>}|
| Output | {<br>&nbsp;"srp6a": {<br>&nbsp;&nbsp;"B": "hex B value",<br>&nbsp;&nbsp;"salt": "hex salt",<br>&nbsp;&nbsp;"ref": "string ref"<br>&nbsp;}<br>}|

### Authenticate user

This API completes the proof of the SRP6a protocol with the provided information:

- The **ref** taken from the above procedure
- The **A** and **M1** SRP6a protocol values

and if sucessfull it will provide back the following details:

- The server proof **srp6a.M2** to be checked by the client
- The **enceeper.authToken** to be used for susequent API calls in the **X-Enceeper-Auth** HTTP header
- The **enceeper.plan** object outlining the plan details of the user account
- The **auth** details provided to the Enceeper service during registration

In the current implementation of the [Enceeper App](https://github.com/enceeper/enceeper) and the [Enceeper JS library](https://github.com/enceeper/enceeper-jslib) the following additional JSON keys are utilized inside the **auth** object:

- The **scrypt.salt** contains the salt for the scrypt algorithm
- The **keys.kek** contains the Key Encryption Key (itself is encrypted)
- The **keys.pub** contains the public key of the user for key sharing (plaintext)
- The **keys.prv** contains the private key of the user for key sharing (encrypted)

| Type   | Value|
|--------|-|
| URL    | /user/login|
| Method | POST|
| Input  | {<br>&nbsp;"srp6a": {<br>&nbsp;&nbsp;"A": "hex A value",<br>&nbsp;&nbsp;"M1": "client proof",<br>&nbsp;&nbsp;"ref": "string ref"<br>&nbsp;}<br>}|
| Output | {<br>&nbsp;"srp6a": {<br>&nbsp;&nbsp;"M2": "server proof"<br>&nbsp;},<br>&nbsp;"enceeper": {<br>&nbsp;&nbsp;"authToken": "the auth token"<br>&nbsp;&nbsp;"plan": { the plan details }<br>&nbsp;},<br>&nbsp;...<br>}|

### Get specific key

Get the key details of the provided **identifier** (for details see **Get account keys**). The owner of the slot can choose one of the following settings:

- Give the slot immediately to the caller
- Give the slot to the caller, but notify me by email that this slot was used
- Do not provide the slot and wait for manual approval by the owner

So the response can be one of the following:

- For the first two scenarios the **slot**, **meta** and **value** keys are populated
- For the last scenario the **ref** and **ttl** keys are populated

And the user must call the **Check for key approval** with the provided **ref** in order to get the key details. The **ttl** is the number of seconds the **ref** will be valid, before it expires.

| Type   | Value|
|--------|-|
| URL    | /user/slots/{identifier}|
| Method | GET|
| Input  | -|
| Output | {<br>&nbsp;"ref": "string ref",<br>&nbsp;"ttl": time in seconds that the ref expires,<br>&nbsp;"slot": "string encrypted slot",<br>&nbsp;"meta": "string encrypted meta",<br>&nbsp;"value": "string encrypted value"<br>}|

### Check for key approval

Check if the provided **ref** has been approved or not. If it has not been approved an error is returned (HTTP status code 428). If the request was approved by the slot owner the **slot**, **meta** and **value** keys are populated. For details on the **ref** parameter read the **Get specific key** API call above.

| Type   | Value|
|--------|-|
| URL    | /user/slots/check/{ref}|
| Method | GET|
| Input  | -|
| Output | {<br>&nbsp;"slot": "string encrypted slot",<br>&nbsp;"meta": "string encrypted meta",<br>&nbsp;"value": "string encrypted value"<br>}|

### Get account keys

### Create new key

### Edit key

### Delete key

### Create new slot

### Edit slot

### Delete slot

### Search users

### Create key share

### Accept key share

### Delete key share

## Rate limiting

