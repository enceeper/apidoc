### Search users

Search for other users in the platform and retrieve their public key to initiate sharing.

| Type   | Value|
|--------|--------------------------------------------------------------------------------------|
| URL    | /user/search|
| Method | POST|
| Input  | {<br>&nbsp;"email": "enceeper@example.com"<br>}|
| Output | {<br>&nbsp;"sharePubKey": "hex pub key"<br>}|

### Create key share

Utilizing the public key of the recepient (see above API call) the client must create an encrypted **slot** and provide it to the Enceeper service along with the recepient's **email**. The Enceeper service facilitates key sharing, but is agnostic of the crypto algorithms used, so no validation is performed on the slot contents other than being a string.

The response of the API call is the **share** object that was created, for details see the [**Get account keys**](keys.md#get-account-keys) API call.

| Type   | Value|
|--------|-|
| URL    | /user/keys/{keyId}/share|
| Method | POST|
| Input  | {<br>&nbsp;"email": "enceeper@example.com",<br>&nbsp;"slot": "the encrypted slot details"<br>}|
| Output | {<br>&nbsp;"share": the share object created<br>}|

### Accept key share

Provided the encrypted slot details the share request is accepted by the Enceeper service and both parties can now access the same key.

The response of the API call is the **key** object that was accepted, for details see the [**Get account keys**](keys.md#get-account-keys) API call.

| Type   | Value|
|--------|-|
| URL    | /user/keys/shares/{shareId}|
| Method | POST|
| Input  | {<br>&nbsp;"slot": "the encrypted slot details"<br>}|
| Output | {<br>&nbsp;"key": the key object that was accepted<br>}|

### Delete key share

Deletes a request to share a slot.

| Type   | Value|
|--------|-|
| URL    | /user/keys/shares/{shareId}|
| Method | DELETE|
| Input  | -|
| Output | -|
