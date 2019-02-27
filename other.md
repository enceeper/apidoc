### Test call

Test API call to verify network connectivity and check that the Enceeper service is working as expected.

| Type   | Value|
|--------|-|
| URL    | /|
| Method | GET|
| Input  | -|
| Output | -|

### Get specific key

Get the key details of the provided **identifier** (for details on the identifier see [**Get account keys**](keys.md#get-account-keys)). The owner of the slot can choose one of the following settings:

- Give the slot immediately to the caller
- Give the slot to the caller, but notify me by email that this slot was used
- Do not provide the slot and wait for manual approval by the owner

So the response can be one of the following:

- For the first two scenarios the **slot**, **meta** and **value** keys are populated
- For the last scenario the **ref** and **ttl** keys are populated

In the latter case the user must call the **Check for key approval** (see below) with the provided **ref** in order to get the key details. The **ttl** is the number of seconds the **ref** will be valid, before it expires.

| Type   | Value|
|--------|-|
| URL    | /user/slots/{identifier}|
| Method | GET|
| Input  | -|
| Output | {<br>&nbsp;"ref": "string ref",<br>&nbsp;"ttl": time in seconds that the ref expires,<br>&nbsp;"slot": "string encrypted slot",<br>&nbsp;"meta": "string encrypted meta",<br>&nbsp;"value": "string encrypted value"<br>}|

### Check for key approval

Check if the provided **ref** has been approved or not. If it has not been approved an error is returned (HTTP status code 428). The client should not abort in this case, but retry in a few seconds and keep re-trying as long as the error remains the same (428). If the request was approved by the slot owner the **slot**, **meta** and **value** keys are populated. For details on the **ref** parameter read the **Get specific key** API call above.

| Type   | Value|
|--------|-|
| URL    | /user/slots/check/{ref}|
| Method | GET|
| Input  | -|
| Output | {<br>&nbsp;"slot": "string encrypted slot",<br>&nbsp;"meta": "string encrypted meta",<br>&nbsp;"value": "string encrypted value"<br>}|
