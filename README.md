# fest
## Files backed up rESTfully

### Theory
This module is an API library designed to operate as an endpoint to provide file backup services.  It is meant work in user spaces, but with the files themselves as a shared resource.  The advantage of this is that common web frameworks can be shared resources among uploads even among different users.  The hash encoding for a file, SHA256, is unique enough to provide adequate security for files.  And, of course, once a file has been archived, that file will always be available to a user no matter where it is moved, or even if it is deleted and then re-added.

Each backup job is divided into a `manifest` and zero or more `files`.  The manifest must be provided (and is returned) in JSON format, and has a structure similar to the following example:

````
{
    "name": "my_backup",
    "timestamp": "1985-10-26T21:45:00",
    "base": "/var/www/edgiese.com/public"
    "omit": ["tmp/*", ".git/*"],
    "files": {
        "./index.php": {
            "hash": "1234567890ABCDEF1234567890ABCDEF12345678",
            "timestamp": "2015-04-15T00:00:00+0000",
            "is_available": false,
            "available_bytes": [[0, 5000], [10000, 12344]]
            "size":12345,
            "user":"iansltx",
            "group":"iansltx",
            "permissions":777
        },
        "./subdir/blah.php": {
            "hash": "1234567890ABCDEF1234567890ABCDEF12345678",
            "timestamp": "2015-04-15T00:00:00+0000",
            "is_available": true,
            "size": 24488,
            "user":"edgiese",
            "group":"apache",
            "permissions":644
        }
    },    "omit": ["tmp/*", ".git/*"],

    "directories": {
        "./subdir": {
            "timestamp": "2015-04-15T00:00:00+0000",
            "user":"edgiese",
            "group":"apache",
            "permissions":644
        }
    }
}
````
The manifest's role is to provide enough information to reproduce the files and directories that were backed up exactly.  It is up to the client side to make sure that the task has adequate permissions to restore the files with the same ownership and permissions that were saved.  Fest only stores contents with a file, not permission or directory location.

Note that directories are specified separately from files, and there is no data for the "contents" of directories saved.  The manifest is (mostly) untouched by fest, with the exception of `is_available` and `available_bytes` fields. The former is a boolean that is true only if the entire file has been successfully uploaded and hashed. The latter, if a file is partially uploaded, indicates the zero-indexed (size - 1 is the highest possible value) ranges of bytes that have been successfully uploaded to the service.

### Client Side Operation and Requirements

The `omit` array contains pathspecs that should be ignored.  The proper handling client side is to determine this spec based on the user's requirements, then ignore all files matching the path spec on backup, then ignore those directories on restore.

A client may elect to completely delete a target directory structure or attempt to reduce files downloaded by comparing the manifest with the target structure.

Database backups must be handled by reducing the database to one or more files, and restoring from those files.

### TODO:
1.  fest should have endpoint support to allow partial upload and download of files.  A parameter on the download API, as well as positive integers showing offsets, is the anticipated solution.
2.  User and Admin endpoints for managing accounts, as well as full oauth2 support.

## Add a manifest
````
POST /manifests
````


## Get lists of available manifests
````
GET /manifests
````

## Get a single manifest (names are namespaced per-user)
````
GET /manifests/<name>
````

## Determine whether partial file uploads are supported
````
OPTIONS /files
````
This returns the following header and body:
````
Server: ....
Allow: GET,HEAD,POST,OPTIONS,TRACE
Content-Type: Application/octet-binary
Accept-Ranges: bytes

204 NO CONTENT
````
The purpose of this method is to determine if this fest version supports partial uploads.  If the `Accept-Ranges` header includes `bytes` then it is possible, otherwise, not.

## Get a file
````
GET /files/a123b1234

Accept-Ranges: bytes
Content-Range: bytes 0-199/*

````
Note that if the fest API instance doesn't support ranges, then it will return a `416` status.  A fest API instance should support ranges for GET if it supports it for 


##Put a file
````
PUT /files/a123b1234

[Header]

Content-Range: bytes 21010-47021/47022

````
