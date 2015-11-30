# Byte API Objects

This documents describes the primary types of objects returned by the Byte API.

**Note About Summary Fields:**
Summary fields are a limited set of fields returned when the object is included inside another object.
For example, when a user object is included inside a comment object, only the summary fields for that user are returned.
In some API endpoints the caller can specify whether they want summary results or full results.

## Post

A post is a single Byte.
The contents used to display the Byte are in the field named `package`, which contains a JSON string that conforms to the BFF guidelines.
The other fields contain metadata about the Byte.

#### Fields

| Field | Type | Summary | Description |
| :---: | :---: | :---: |:--- |
| `id` | String | Yes | Unique identifier for the Byte. |
| `name` | String | Yes | Unique name for the Byte. Contains no spaces. |
| `caption` | String | Yes |  Freeform description for the Byte. Can include unicode. |
| `author` | User Object | Yes | Author of this Byte. Not included for incognito Bytes. |
| `previewUrl` | String | Yes | URL to a resizable thumbnail image for the Byte. |
| `thumbnailSrc` | String | Yes | URL to a non-resizable thumbnail image for the Byte. |
| `musicSrc` | String |  |  Url to an MP3 file of the music for the Byte, if it has music. |
| `package` | String |  |  JSON string containing the BFF for this Byte. |
| `comments` | Array |  |  Array of comments on the Byte. |
| `ownedByUser` | Boolean |  |  Whether the currently logged in user owns this Byte. Only included for authed requests. |
| `profile` | Boolean | Yes |  Whether this Byte is the featured Byte for the author's profile. |
| `stickersCount` | Integer | Yes |  Number of sticker comments on this Byte. |
| `commentsCount` | Integer | Yes |  Number of text commetns on this Byte. |
| `lookCount` | Integer | Yes |  Number of looks on this Byte. |
| `updated` | Integer | Yes |  Millisecond-based Unix timestamp for when this Byte was updated. |
| `updatedString` | String | Yes |  Millisecond-based Unix timestamp for when this Byte was updated. Useful in 32 bit environments where the integer overflows. |
| `created` | Integer | Yes |  Millisecond-based Unix timestamp for when this Byte was created. |
| `createdString` | String | Yes |  Millisecond-based Unix timestamp for when this Byte was created. Useful in 32 bit environments where the integer overflows. |


#### Sample Object

```json
    {
        "id": "<postId>",
        "previewUrl": "<previewUrl>",
        "comments": [
            {
                "author": null,
                "point": {
                    "y": 297.04230000000001,
                    "x": 192.88130000000001
                },
                "sticker": "StickerThumbsUp",
                "created": 1439879220892,
                "createdString": "1439879220892",
                "postId": "<postId>",
                "type": "sticker",
                "id": "<commentId>"
            }
        ],
        "thumbnailSrc": "<thumbnailSrc>",
        "musicSrc": "<musicSrc>",
        "hidden": false,
        "updated": 1441050296279,
        "authorType": "ByteUser",
        "stickersCount": 11,
        "lookCount": 284,
        "createdString": "1439504831994",
        "name": "<postName>",
        "package": {"<package>"},
        "ownedByUser": true,
        "created": 1439504831994,
        "caption": "<caption>",
        "updatedString": "1441050296279",
        "commentsCount": 1
    }
```


## User

Most API's have users and ours does too!

When included in another object only summary fields are included (`id`, `username`, `profilePhoto`).
Not all users have a username. Those that don't will return an empty string as their username.


#### Fields

| Field | Type | Summary | Description |
| :---: | :---: | :---: | :--- |
| `id` | String | Yes | Unique identifier for this user. |
| `username` | String | Yes | Unique name for this user. Empty string if user has not claimed a username. |
| `profilePhoto` | String | Yes | URL to a resizable thumbnail image for the user's profile photo. |
| `created` | Integer |  | Millisecond-based Unix timestamp for when this user was created. |
| `createdString` | String |  | Millisecond-based Unix timestamp for when this user was created. Useful in 32 bit environments where the integer overflows. |



#### Sample Object

```json
{
    "id": "<postId>",
    "username": "<username>",
    "profilePhoto": "<profilePhoto>",
    "created": 1439504831994,
    "createdString": "1439504831994"
}
```


## Comment

Byte comments are unique because they have a position, specified as X and Y coordinates.
Byte comments can be stickers or regular text comments. Stickers are chosen from the list of Byte stickers.
For the list of stickers see the documentation of the comment creation endpoint.



#### Fields

| Field | Type | Description |
| :---: | :---: | :--- |
| `id` | String | Unique identifier for this comment. |
| `postId` | String | Unique identifier for the post this comment belongs to. |
| `author` | User | Author of this comment. Note that older comments may not have an author. |
| `point` | Dictionary | Location of this comment as a dictionary containing x and y keys with integer values.  |
| `type` | String | `sticker` or `text`. |
| `sticker` | String | Name of the sticker used for this comment. Only for sticker-type comments. |
| `body` | String | Text for this comment. Only for text-type comments. |
| `created` | Integer | Millisecond-based Unix timestamp for when this comment was created. |
| `createdString` | String | Millisecond-based Unix timestamp for when this comment was created. Useful in 32 bit environments where the integer overflows. |

#### Sample Objects

Text comment:
```json
{
    "body": "Looks good!",
    "author": "<user>",
    "point": {
        "y": 261.92649999999998,
        "x": 219.45599999999999
    },
    "created": 1446681611448,
    "createdString": "1446681611448",
    "postId": "xyz123",
    "type": "text",
    "id": "abc987"
}
```

Sticker comment:
```json
{
    "sticker": "StickerFrog",
    "author": "<user>",
    "point": {
        "y": 211.93649999999998,
        "x": 218.32599999999999
    },
    "created": 1446681611448,
    "createdString": "1446681611448",
    "postId": "xyz123",
    "type": "sticker",
    "id": "dof832"
}
```
