### Get account keys

This API call will retrieve all the keys available in the user account and also all pending share requests (either outgoing or incoming).

In each key object the following information is available:
- **key_id**: the key Id
- **value**: the encrypted value of the password, encrypted with a random key
- **meta**: the encrypted meta of the password, encrypted with the same random key
- **status**: 0 means enabled and 1 means disabled
- **shared**: indicator if this a shared key
- **created_on**: a Unix epoch timestamp of when the key was created
- **updated_on**: a Unix epoch timestamp of when the key was last updated
- **slots**: an array of access slots that hold the encrypted random key mentioned above
- **slots[x].slot_id**: the slot Id
- **slots[x].value**: the encrypted random key that was used for encrypting the meta and value above. Based on the slot type this random key will be encrypted either by the KEK of a user or a key derived from scrypt using an access password specific for this slot (or something else as it depents on the caller!)
- **slots[x].notify**: 0 means no action, 1 means report usage and 2 means wait for approval (applicable only for the **Get specific key** API call)
- **slots[x].status**: 0 means enabled and 1 means disabled
- **slots[x].identifier**: a unique identifier to access this slot via the **Get specific key** API call (not applicable on shared keys)
- **slots[x].shared**: indicator if this a shared slot
- **slots[x].created_on**: a Unix epoch timestamp of when the slot was created
- **slots[x].updated_on**: a Unix epoch timestamp of when the slot was last updated

In each share object the following information is available:
- **share_id**: the share id
- **key_id**: the key id this share was created for
- **from**: the email of the sender
- **to**: the email of the recepient
- **pub**: the public key of the sender
- **slot**: the encrypted slot to be used when accepting this share
- **status**: "new" for newly created shares and "old" for older share requests

| Type   | Value|
|--------|-|
| URL    | /user/keys|
| Method | GET|
| Input  | -|
| Output | {<br>&nbsp;"keys": [ an array with key objects ],<br>&nbsp;"shares": [ an array with share objects ]<br>}|

### Create new key

This API call creates a new key. While the Enceeper service is agnostic of the contents of the **slot**, **meta** and **value** JSON values the basic idea for all clients utilizing the Enceeper service is the following:

- The user provided information (the actual password and meta information) for this key are encrypted with a random generated encryption key
- The random encryption key is in turn encrypted using the Key Encryption Key (KEK) of the user
- Utilizing a KEK simplifies updating the master user password (no need to iterate over the keys on user master password updates)
- Also using slots as mentioned above reveals only this key to third parties (in case of sharing or utilizing the **Get specific key** API call)

| Type   | Value|
|--------|-|
| URL    | /user/keys|
| Method | POST|
| Input  | {<br>&nbsp;"slot": "the encrypted slot details",<br>&nbsp;"meta": "the encrypted meta details",<br>&nbsp;"value": "the encrypted value details"<br>}|
| Output | {<br>&nbsp;"key": the key object<br>}|

### Edit key

Update the key details. The **meta** and **value** entries are optional and can be emmited if only updating the **status**. Check the key object outlined above in the **Get account keys** section for valid values of the **status** flag.

| Type   | Value|
|--------|-|
| URL    | /user/keys/{keyId}|
| Method | PUT|
| Input  | {<br>&nbsp;"status": 0,<br>&nbsp;"meta": "the encrypted meta details",<br>&nbsp;"value": "the encrypted value details"<br>}|
| Output | {<br>&nbsp;"key": the key object<br>}|

### Delete key

| Type   | Value|
|--------|-|
| URL    | /user/keys/{keyId}|
| Method | DELETE|
| Input  | -|
| Output | -|

### Create new slot

Create a new slot for accessing the key. The **value** contains the encrypted random key of the key and check the key object outlined above in the **Get account keys** section for valid values of the **notify** flag.

| Type   | Value|
|--------|-|
| URL    | /user/keys/{keyId}/slots|
| Method | POST|
| Input  | {<br>&nbsp;"notify": 0,<br>&nbsp;"value": "the encrypted value details"<br>}|
| Output | {<br>&nbsp;"key": the key object<br>}|

### Edit slot

Update the slot. The **value** entry is optional and can be emmited if only updating the **status** and **notify** flags. Check the key object outlined above in the **Get account keys** section for valid values of the **status** and **notify** flags.

| Type   | Value|
|--------|-|
| URL    | /user/keys/{keyId}/slots/{slotId}|
| Method | PUT|
| Input  | {<br>&nbsp;"status": 0,<br>&nbsp;"notify": 0,<br>&nbsp;"value": "the encrypted value details"<br>}|
| Output | {<br>&nbsp;"key": the key object<br>}|

### Delete slot

| Type   | Value|
|--------|-|
| URL    | /user/keys/{keyId}/slots/{slotId}|
| Method | DELETE|
| Input  | -|
| Output | -|
