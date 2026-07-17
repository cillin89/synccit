# Account API Docs

The Account API is called on account.php. This is located at [http://api.synccit.com/account.php](http://api.synccit.com/account.php). If not using synccit.com, this should be shown on devices page.

Basic idea. This API manages the account itself: creating an account, exchanging a username and password for a login code, and adding or removing the device auth codes used by the standard API. The link syncing calls live on api.php and are documented in the main README.


The API includes 2 variables. The API version and revision. The version is only changed when major changes to the API occur and will break older uses of it. The revision is for smaller changes. This usually means adding features or small changes that don't break any older use of the API.

To determine the version and revision of the API being used, check the headers sent by account.php. `curl -I http://api.synccit.com/account.php` gives me `X-API: 1` and `X-Revision: 1`. To see how revisions change, you can check [the account.php history](https://github.com/drakeapps/synccit/commits/master/api/account.php). 

**Login code and passwords**

For added security, instead of using the account password for each call, it uses an login code. These are created and returned by doing a login call. This is similar to the auth code of the standard API, but is longer and the user never interacts with it.

## Variables

* **`username`**
 * synccit username
* **`login`**
 * login code returned by login command
 * Like auth code in standard API, but much longer and user never interacts with it
 * Note: not needed for create and login calls
* **`dev`**
 * Your developer name
 * Name you want to appear as, such as synccit-userscript or iReddit
* **`devauth`**
 * developer authentication
 * Note: Not implemented yet. This will allow you to ensure only you can use your developer name
* **`mode`**
 * Action you're taking.
  * `create` - create new account
  * `login` - check username/password and get login code
  * `delete` - delete auth code 
  * `history` - returns last 20 links visited
  * `devices` - returns list of devices with auth codes
  * `addauth` - add new device/auth code
* **`api`**
 * API version you're using (not required)
 * Current version is `1`

### Devices (returned) variables

* **`auth`**
 * The auth code of the device
 * Usually a 6 character string
* **`device`**
 * Device name
 * The device name the user entered while setting up the auth codes
* **`created`**
 * Unix timestamp of when the device was added

### History (returned) read variables

* **`id`**
 * Reddit link id (see above)
* **`lastvisit`**
 * Unix time stamp of when link was last visited
 * Defaults to `0` if link has never been visited
* **`comments`**
 * Number of comments read
 * Defaults to `0` if comments have never been viewed
* **`commentvisit`**
 * Unix time stamp of when comments were last viewed
 * Defaults to `0` if comments have never been viewed

### Create account variables (JSON/XML)

* **`password`**
 * Password for account
* **`email`**
 * Email to be associated with account (create account only)
 * Not required


## JSON

JSON data is sent of POST variable `data`

The GET or POST variable `type` should be json (though not required)

### Example JSON create account call

    {
        "username"  : "newuser",
        "password"  : "thebestpasswordever",
        "dev"       : "synccit demo",
        "email"     : "newuser@synccit.com",
        "mode"      : "create"
    }

Creates new user with username `newuser` and password `thebestpasswordever`. And an email of `newuser@synccit.com`, though email is not required. Mode is `create`

**Returns**


Success

    {
        "success"   : "account created"
    }

Error

    {
        "error"     : "ERROR_CODE"
    }

***
### Example JSON add authorization call

    {
        "username"  : "newuser",
        "login"     : "8sk3js93ks0dk2ms91xkq0amz73hdy4b",
        "dev"       : "synccit demo",
        "device"    : "developer API device",
        "mode"      : "addauth"
    }

Creates a new auth code for the user `newuser`, authenticated with the login code from a `login` call. Device name is `developer API device`. Mode is `addauth`

**Returns**

Success

    {
        "success"   : "device key added",
        "device"    : "developer API device",
        "auth"      : "409ssj"
    }

New device key added with auth code of `409ssj`. Device name is also returned back

Error

    {
        "error" : "ERROR_CODE"
    }

## XML

XML data is sent on POST variable data.

The GET or POST variable `type` has to be set to xml or, as of API revision 10, the first 4 charaters of POST data are `<?xml`

Only `create` and `addauth` are available over XML. The `login`, `delete`, `history` and `devices` modes are JSON only.

### Example XML create account call

    <?xml version="1.0"?>
    <synccit>
        <username>newuser</username>
        <password>thebestpasswordever</password>
        <dev>synccit demo</dev>
        <email>newuser@synccit.com</email>
        <mode>create</mode>
    </synccit>

Creates the user `newuser` with a password of `thebestpasswordever`. And an email of `newuser@synccit.com`, though email is not required. Mode is `create`

**Returns**

Success

    <?xml version="1.0"?>
    <synccit>
        <success>account created</success>
    </synccit>

Error

    <?xml version="1.0"?>
    <synccit>
        <error>ERROR_CODE</error>
    </synccit>

***
### Example XML add authorization 

    <?xml version="1.0"?>
    <synccit>
        <username>newuser</username>
        <login>8sk3js93ks0dk2ms91xkq0amz73hdy4b</login>
        <dev>synccit demo</dev>
        <device>developer API device</device>
        <mode>addauth</mode>
    </synccit>

Creates a new auth code for the user `newuser`, authenticated with the login code from a `login` call. Device name is `developer API device`. Mode is `addauth`

**Returns**

Success

    <?xml version="1.0"?>
    <synccit>
        <success>device key added</success>
        <device>developer API device</device>
        <auth>303b09</auth>
    </synccit>

Returns auth code under `auth`. Use this for future API calls for this user. 

Error

    <?xml version="1.0"?>
    <synccit>
        <error>ERROR_CODE</error>
    </synccit>

## Error Codes

* `no post data`
 * No post data sent or at least none that we know what to do with
* `not authorized`
 * Username and auth code combination doesn't work
* `database error`
 * Error executing query. Likely something on our end
* `username or password wrong`
 * That username and password combination isn't valid

**Create account errors**

* `email not valid`
 * Not valid email given. Only checks it '@' exists
* `username needs to be at least 3 characters long`
* `password needs to be at least 6 characters long`
* `username must consist of letters, numbers, or underscores`
* `username already exists`