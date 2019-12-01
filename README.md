# What is Enceeper?

[Enceeper](https://www.enceeper.com/) is a service used to store and organize users’ passwords and secrets. It provides high level of security since users’ information is stored to the service using [end-to-end encryption](https://en.wikipedia.org/wiki/End-to-end_encryption).

The [Enceeper App](https://github.com/enceeper/enceeper) is using the service in the following way:

- All data are first encrypted locally and then transmitted over the network.
- The password of the user is never transmitted, but is used locally to compute a proof-of-knowledge.
- Moreover, a user has the option to "share" those secrets with other users of the service by adding slots to selected entries.
- Finally, any third party can use the API of the service to enhance the core functionality.

To this end we have developed two solutions:

- [enceeper-phpconf](https://github.com/enceeper/enceeper-phpconf) to store configuration information in Enceeper and have it delivered to a PHP application
- [enceeper-boot](https://github.com/enceeper/enceeper-boot) for unattended booting a GNU/Linux distro via Initramfs utilizing Enceeper

# API documentation

In this document we describe the available API calls, the input they receive and the output they produce.

The current API version is: **1.1.0** and the base URL of the service is: [https://api.enceeper.com/api/v1/](https://api.enceeper.com/api/v1/)

The API uses [HTTP response codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) to indicate success or failure of requests. Specifically, codes in the 2xx range indicate success, codes in the 4xx range indicate an error that resulted from the provided arguments (e.g. instead of an integer a string is provided) and 5xx error codes indicate an internal Enceeper error.

> There is only one error that can be handled in a special way, the 403 error. This error is generated when the auth token has expired and the user needs to re-authenticate. The client can check for this error and re-authenticate the user without requiring any interaction.

All inputs and outputs follow the JSON format, and based on whether the outcome of an API call was sucessfull or not we have the following response templates:

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
  "enceeper": {
    "app": {
      "version": "1.0.0"
    },
    "api": {
      "deprecated": null
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
  "enceeper": {
    "app": {
      "version": "1.0.0"
    },
    "api": {
      "deprecated": null
    }
  },
  "errorMessage": "API method not found"
}
```

The **enceeper.app.version** key contains the latest version of the [Enceeper App](https://github.com/enceeper/enceeper). The **enceeper.api.deprecated** key will be null if the API version being used is fully supported and contain a message if the API version will reach it's end-of-life. At that point the message will contain the necessary details.

## API calls

The summary of the API calls is:

| Title                                                       | Method | URL                               |
|-------------------------------------------------------------|--------|-----------------------------------|
| [Test call](other.md#test-call)                             | GET    | /                                 |
| [User registration](user.md#user-registration)              | POST   | /user                             |
| [Initiate auth procedure](user.md#initiate-auth-procedure)  | POST   | /user/challenge                   |
| [Authenticate user](user.md#authenticate-user)              | POST   | /user/login                       |
| [Get specific key](other.md#get-specific-key)               | GET    | /user/slots/{identifier}          |
| [Check for key approval](other.md#check-for-key-approval)   | GET    | /user/slots/check/{ref}           |
| [Edit user](user.md#edit-user)*                             | PUT    | /user                             |
| [Retrieve web auth token](user.md#retrieve-web-auth-token)* | GET    | /user/webauth                     |
| [Delete user](user.md#delete-user)*                         | DELETE | /user                             |
| [Get account keys](keys.md#get-account-keys)*               | GET    | /user/keys                        |
| [Create new key](keys.md#create-new-key)*                   | POST   | /user/keys                        |
| [Edit key](keys.md#edit-key)*                               | PUT    | /user/keys/{keyId}                |
| [Delete key](keys.md#delete-key)*                           | DELETE | /user/keys/{keyId}                |
| [Create new slot](keys.md#create-new-slot)*                 | POST   | /user/keys/{keyId}/slots          |
| [Edit slot](keys.md#edit-slot)*                             | PUT    | /user/keys/{keyId}/slots/{slotId} |
| [Delete slot](keys.md#delete-slot)*                         | DELETE | /user/keys/{keyId}/slots/{slotId} |
| [Search users](share.md#search-users)*                      | POST   | /user/search                      |
| [Create key share](share.md#create-key-share)*              | POST   | /user/keys/{keyId}/share          |
| [Accept key share](share.md#accept-key-share)*              | POST   | /user/keys/shares/{shareId}       |
| [Delete key share](share.md#delete-key-share)*              | DELETE | /user/keys/shares/{shareId}       |

> All API calls marked with * require the use of the **X-Enceeper-Auth** HTTP header with a valid authentication token retrieved from the **Authenticate user** API call.

## Rate limiting

To prevent our API from being overwhelmed by too many requests, Enceeper imposes rate limits. The rate limit is IP based for unauthenticated API calls (the first five on the table above) and user based for authenticated API calls (all the remaining API calls).

When you make an API request, you will receive a set of rate-limiting headers to help your application respond appropriately. The headers are:

- X-RateLimit-Limit: the total number of requests you are allowed to make
- X-RateLimit-Remaining: the number of requests you have left to use
- X-RateLimit-Reset: a Unix epoch timestamp of when your limit is reset (future time)

If this limit is exceeded, an error is returned: HTTP 429 (Too Many Requests).

# Enceeper plans

For details on the available plans visit: [www.enceeper.com](https://www.enceeper.com/)

The service imposes limits based on the user plan for the following items:
- Requests per hour (see **rate limiting** above)
- The number of total keys
- The number of slots per key
- A limit on the size of the key
- If key sharing is allowed
- If the user can enable to be notified when a slot is used
- If the user can enable slot approval (approve slot before use)

The **Authenticate user** API call returns the current user plan in the following format:

```
{
  "name": "Default",
  "reqs_per_hour": 1000,
  "total_keys": 10,
  "slots_per_key": 2,
  "key_size_bytes": 1024,
  "key_sharing": false,
  "slot_notify_on_use": true,
  "slot_approve_use": false,
  "deprecated": false
}
```
