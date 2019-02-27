### Search users

Search for other users in the platform and retrieve their public key for key sharing.

| Type   | Value|
|--------|--------------------------------------------------------------------------------------|
| URL    | /user/search|
| Method | POST|
| Input  | {<br>&nbsp;"email": "enceeper@example.com"<br>}|
| Output | {<br>&nbsp;"sharePubKey": "hex pub key"<br>}|

### Create key share

### Accept key share

### Delete key share

Deletes a request to share a slot.

| Type   | Value|
|--------|-|
| URL    | /user/keys/shares/{shareId}|
| Method | DELETE|
| Input  | -|
| Output | -|
