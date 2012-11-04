# WinJS Library for Facebook

With facebook-winjs-sdk you can now easily write the same code and share between your windows 8 apps (winjs) and the client ([Facebook Javascript SDK](https://developers.facebook.com/docs/reference/javascript/)).

facebook-winjs-sdk is a fork of [facebook-node-sdk](https://github.com/Thuzi/facebook-node-sdk/) designed to work in both WinJS applications and NodeJS.

# Sample

You can find the sample of facebook-winjs-sdk at https://github.com/Thuzi/facebook-winjs-sdk-sample.

# Installing facebook-winjs-sdk

```
Install-Package FBWinJS
```

```html
<script src="/js/fb.js"></script>
```

facebook-winjs-sdk is exposed as a global variable FBWinJS in order to avoid conflict with the Facebook Javascript SDK.

You can set `FBWinJS` as `FB` variable for portability.

```js
// global variable alias
window.FB = FBWinJS;
```

```js
// local variable aslias
var FB = FBWinJS;

// or

(function(FB){
    
})(FBWinJS);
```

**Following examples assumes `FB` as `FBWinJS`.**

## Graph Api

### Get

```js
FB.api('4', function (res) {
  if(!res || res.error) {
   console.log(!res ? 'error occurred' : res.error);
   return;
  }
  console.log(res.id);
  console.log(res.name);
});
```

__Passing Parameters__

```js
FB.api('4', { fields: ['id', 'name'] }, function (res) {
  if(!res || res.error) {
    console.log(!res ? 'error occurred' : res.error);
    return;
  }
  console.log(res.id);
  console.log(res.name);
});
```

### Post

```js
FB.setAccessToken('access_token');

var body = 'My first post using facebook-node-sdk';
FB.api('me/feed', 'post', { message: body}, function (res) {
  if(!res || res.error) {
    console.log(!res ? 'error occurred' : res.error);
    return;
  }
  console.log('Post Id: ' + res.id);
});
```

### Delete

```js
FB.setAccessToken('access_token');

var postId = '1234567890';
FB.api(postId, 'delete', function (res) {
  if(!res || res.error) {
    console.log(!res ? 'error occurred' : res.error);
    return;
  }
  console.log('Post was deleted');
});
```

## Facebook Query Language (FQL)

### Query

```js
FB.setAccessToken('access_token');

FB.api('fql', { q: 'SELECT uid FROM user WHERE uid=me()' }, function (res) {
  if(!res || res.error) {
    console.log(!res ? 'error occurred' : res.error);
    return;
  }
  console.log(res.data);
});
```

### Multi-query

```js
FB.setAccessToken('access_token');

FB.api('fql', { q: [
  'SELECT uid FROM user WHERE uid=me()',
  'SELECT name FROM user WHERE uid=me()'
] }, function(res) {
  if(!res || res.error) {
    console.log(!res ? 'error occurred' : res.error);
    return;
  }
  console.log(res.data[0].fql_result_set);
  console.log(res.data[1].fql_result_set);
});
```

### Named Multi-query

```js
FB.setAccessToken('access_token');

FB.api('fql', { q : {
  id: 'SELECT uid FROM user WHERE uid=me()',
  name: 'SELECT name FROM user WHERE uid IN (SELECT uid FROM #id)'
} }, function(res) {
  if(!res || res.error) {
    console.log(!res ? 'error occurred' : res.error);
    return;
  }
  console.log(res.data[0].fql_result_set);
  console.log(res.data[1].fql_result_set);
});
```

## Batch Requests

```js
FB.setAccessToken('access_token');

var extractEtag;
FB.api('', 'post', { 
    batch: [
        { method: 'get', relative_url: '4' },
        { method: 'get', relative_url: 'me/friends?limit=50' },
        { method: 'get', relative_url: 'fql?q=' + encodeURIComponent('SELECT uid FROM user WHERE uid=me()' ) }, /* fql */
        { method: 'get', relative_url: 'fql?q=' + encodeURIComponent(JSON.stringify([
                    'SELECT uid FROM user WHERE uid=me()',
                    'SELECT name FROM user WHERE uid=me()'
                ])) }, /* fql multi-query */
        { method: 'get', relative_url: 'fql?q=' + encodeURIComponent(JSON.stringify({
                    id: 'SELECT uid FROM user WHERE uid=me()',
                    name: 'SELECT name FROM user WHERE uid IN (SELECT uid FROM #id)'
                })) }, /* named fql multi-query */
        { method: 'get', relative_url: '4', headers: { 'If-None-Match': '"7de572574f2a822b65ecd9eb8acef8f476e983e1"' } }, /* etags */
        { method: 'get', relative_url: 'me/friends?limit=1', name: 'one-friend' /* , omit_response_on_success: false */ },
        { method: 'get', relative_url: '{result=one-friend:$.data.0.id}/feed?limit=5'}
    ]
}, function(res) {
    var res0, res1, res2, res3, res4, res5, res6, res7,
        etag1;

    if(!res || res.error) {
        console.log(!res ? 'error occurred' : res.error);
        return;
    }

    res0 = JSON.parse(res[0].body);
    res1 = JSON.parse(res[1].body);
    res2 = JSON.parse(res[2].body);
    res3 = JSON.parse(res[3].body);
    res4 = JSON.parse(res[4].body);
    res5 = res[5].code === 304 ? undefined : JSON.parse(res[5].body);   // special case for not-modified responses
                                                                        // set res5 as undefined if response wasn't modified.
    res6 = res[6] === null ? null : JSON.parse(res[6].body);
    res7 = res6 === null ? JSON.parse(res[7].body) : undefined; // set result as undefined if previous dependency failed

    if(res0.error) {
        console.log(res0.error);
    } else {
        console.log('Hi ' + res0.name);
        etag1 = extractETag(res[0]); // use this etag when making the second request.
        console.log(etag1);
    }

    if(res1.error) {
        console.log(res1.error);
    } else {
        console.log(res1);
    }

    if(res2.error) {
        console.log(res2.error);
    } else {
        console.log(res2.data);
    }

    if(res3.error) {
        console.log(res3.error);
    } else {
        console.log(res3.data[0].fql_result_set);
        console.log(res3.data[1].fql_result_set);
    }

    if(res4.error) {
        console.log(res4.error);
    } else {
        console.log(res4.data[0].fql_result_set);
        console.log(res4.data[0].fql_result_set);
    }

    // check if there are any new updates
    if(typeof res5 !== "undefined") {
        // make sure there was no error
        if(res5.error) {
            console.log(error);
        } else {
            console.log('new update available');
            console.log(res5);
        }
    }
    else {
        console.log('no updates');
    }

    // check if dependency executed successfully    
    if(res[6] === null) {
        // then check if the result it self doesn't have any errors.
        if(res7.error) {
            console.log(res7.error);
        } else {
            console.log(res7);
        }
    } else {
        console.log(res6.error);
    }
});

extractETag = function(res) {
    var etag, header, headerIndex;
    for(headerIndex in res.headers) {
        header = res.headers[headerIndex];
        if(header.name === 'ETag') {
            etag = header.value;
        }
    }
    return etag;
};
```
### Post

```js
FB.setAccessToken('access_token');

var message = 'Hi from facebook-winjs-sdk';
FB.api('', 'post', {
    batch: [
        { method: 'post', relative_url: 'me/feed', body:'message=' + encodeURIComponent(message) }
    ]
}, function (res) {
    var res0;

    if(!res || res.error) {
        console.log(!res ? 'error occurred' : res.error);
        return;
    }

    res0 = JSON.parse(res[0].body);

    if(res0.error) {
        console.log(res0.error);
    } else {
        console.log('Post Id: ' + res0.id);
    }
});
```

## OAuth Requests

*This is a non-standard behavior and does not work in the official client side FB JS SDK.*

facebook-winjs-sdk is capable of handling oauth requests which return non-json responses. You can use it by calling `api` method.

### Get facebook application access token

```javascript
FB.api('oauth/access_token', {
    client_id: 'app_id',
    client_secret: 'app_secret',
    grant_type: 'client_credentials'
}, function (res) {
    if(!res || res.error) {
        console.log(!res ? 'error occurred' : res.error);
        return;
    }
    
    var accessToken = res.access_token;
});
```

### Exchange code for access token

```javascript
FB.api('oauth/access_token', {
    client_id: 'app_id',
    client_secret: 'app_secret',
    redirect_uri: 'http://yoururl.com/callback',
    code: 'code'
}, function (res) {
    if(!res || res.error) {
        console.log(!res ? 'error occurred' : res.error);
        return;
    }

    var accessToken = res.access_token;
    var expires = res.expires ? res.expires : 0;
});
```

You can safely extract the code from the url using the `url` module. Always make sure to handle invalid oauth callback as
well as error.

```javascript
var url = require('url');

var urlToParse = 'http://yoururl.com/callback?code=.....#_=_';
var result = url.parse(urlToParse, true);
if(result.query.error) {
    if(result.query.error_description) {
        console.log(result.query.error_description);
    } else {
        console.log(result.query.error);
    }
    return;
} else if (!result.query.code) {
    console.log('not a oauth callback');
    return;
}

var code = result.query.code;
```

It is easier to set the `response_type=token` for WinJS and avoid the exchange of code for access token.

```javascript
// For an introduction to the Page Control template, see the following documentation:
// http://go.microsoft.com/fwlink/?LinkId=232511
(function () {
    "use strict";

    WinJS.UI.Pages.define("/pages/login/login.html", {
        // This function is called whenever a user navigates to this page. It
        // populates the page elements with the app's data.
        ready: function (element, options) {
            WinJS.Utilities.query('#login').listen('click', loginClicked, false);
        }
    });

    function loginClicked(e) {
        e.preventDefault();
        
        var redirectUri = 'https://www.facebook.com/connect/login_success.html',
            loginUrl = 'https://www.facebook.com/dialog/oauth'
                + '?response_type=token'
                + '&display=popup'
                + '&scope=' + encodeURIComponent('user_about_me,publish_stream,read_stream')
                + '&redirect_uri=' + encodeURIComponent(redirectUri)
                + '&client_id=' + FB.options('appId');

        try {
            
            Windows.Security.Authentication.Web.WebAuthenticationBroker.authenticateAsync(
                Windows.Security.Authentication.Web.WebAuthenticationBroker.default,
                new Windows.Foundation.Uri(loginUrl),
                new Windows.Foundation.Uri(redirectUri))
                .then(function success(result) {
                    if(result.responseStatus == 2) {
                        console.log('error: ' + result.responseerrordetail);
                        return;
                    }

                    var parser = document.createElement('a');
                    parser.href = result.responseData;

                    var qs = extractQuerystring(parser.hash.substr(1).split('&'));
                    
                    if (qs.error) {
                        // most likely user clicked don't allow
                        console.log('error: ' + qs.error + ' : ' + qs.error_description);
                        return;
                    }

                    // we now have the access token,
                    
                    // set it as the default access token.
                    FB.setAccessToken(qs.access_token);
                    
                    // save it in local storage so can access it later
                    localStorage.setItem('fb_access_token', FB.getAccessToken());

                    // now navigate to home page
                    WinJS.Navigation.navigate('/pages/home/home.html');

                }, function error(err) {
                    console.log('Error Number: ' + err.number);
                    console.log('Error Message: ' + err.message);
                });
            
        } catch(e) {
            // error launching web auth
            console.log(e);
        } 
    }

    function extractQuerystring(a) {
        if (a == "") return {};
        var b = {};
        for (var i = 0; i < a.length; ++i) {
            var p = a[i].split('=');
            if (p.length != 2) continue;
            b[p[0]] = decodeURIComponent(p[1].replace(/\+/g, " "));
        }
        return b;
    }
    
})();
```

### Extend expiry time of the access token

```javascript
FB.api('oauth/access_token', {
    client_id: 'client_id',
    client_secret: 'client_secret',
    grant_type: 'fb_exchange_token',
    fb_exchange_token: 'existing_access_token'
}, function (res) {
    if(!res || res.error) {
        console.log(!res ? 'error occurred' : res.error);
        return;
    }
    
    var accessToken = res.access_token;
    var expires = res.expires ? res.expires : 0;
});
```

## Legacy REST Api

__Although Legacy REST Api is supported by facebook-winjs-sdk, it is highly discouraged to be used, as Facebook is in the process of deprecating the Legacy REST Api.__

### Get

```javascript
FB.api({ method: 'users.getInfo', uids: ['4'], fields: ['uid', 'name'] }, function (res) {
    if(!res || res.error_msg) {
        console.log(!res ? 'error occurred' : res.error_msg);
        return;
    }

    console.log('User Id: ' + res[0].uid);
    console.log('Name: ' + res[0].name);
});
```

### Post

```javascript
FB.setAccessToken('access_token');

var message = 'Hi from facebook-node-sdk';
FB.api({ method: 'stream.publish', message: message }, function (res) {
    if(!res || res.error_msg) {
        console.log(!res ? 'error occurred' : res.error_msg);
        return;
    }
    
    console.log(res);
});
```
### Delete

```javascript
FB.setAccessToken('access_token');

var postId = '.....';
FB.api({ method: 'stream.remove', post_id: postId }, function (res) {
    if(!res || res.error_msg) {
        console.log(!res ? 'error occurred' : res.error_msg);
        return;
    }
    
    console.log(res);
});
```

## Access Tokens

### setAccessToken
*This is a non-standard api and does not exist in the official client side FB JS SDK.*

```js
FB.setAccessToken('access_token');
```

If you want to use the api compaitible with FB JS SDK, pass `access_token` as parameter.

```js
FB.api('me', { fields: ['id', 'name'], access_token: 'access_token' }, function (res) {
    console.log(res);
}
```

### getAccessToken
*Unlike `setAccessToken` this is a standard api and exists in FB JS SDK.*

```js
FB.setAccessToken('access_token');
var accessToken = FB.getAccessToken();
```
## Configuration options

### options

*This is a non-standard api and does not exist in the official client side FB JS SDK.*

When this method is called with no parameters it will return all of the current options.

```js
var options = FB.options();
```

When this method is called with a string it will return the value of the option if exists, null if it does not.

```js
var timeout = FB.options('timeout');
```

When this method is called with an object it will merge the object onto the previous options object.
```js
FB.options({accessToken: 'abc'}); //equivalent to calling setAccessToken('abc')
FB.options({timeout: 1000, accessToken: 'XYZ'}); //will set timeout and accessToken options
var timeout = FB.options('timeout'); //will get a timeout of 1000
var accessToken = FB.options('accessToken'); //will get the accessToken of 'XYZ'
```

The existing options are:
* `'accessToken'` string representing the facebook accessToken to be used for requests. This is the same option that is updated by the `setAccessToken` and `getAccessToken` methods.
* `'appSecret'` string representing the facebook application secret.
* `'timeout'` integer number of milliseconds to wait for a response. Requests that have not received a response in *X* ms. If set to null or 0 no timeout will exist. On timeout an error object will be returned to the api callback with the error code of `'ETIMEDOUT'` (example below).

`'scope'` and `'redirect_uri'` have been whitelisted in options for convenience. These value will not be automatically
added when using any of the sdk apis unlike the above options. These are whitelisted so you can use it to pass values
using the same `FB` object.

### version

*This is a non-standard api and does not exist in the official client side FB JS SDK.*

Gets the string representation of the facebook-winjs-sdk library version.

```js
var version = FB.version;
```

## Parsing Signed Request

### parseSignedRequest

*This is a non-standard api and does not exist in the official client side FB JS SDK. Currently supported only in NodeJS.*

```js
var signedRequestValue = 'signed_request_value';
var appSecret = 'app_secret';

var signedRequest  = FB.parseSignedRequest(signedRequestValue, appSecret);
if(signedRequest) {
    var accessToken = signedRequest.oauth_token;
    var userId = signedRequest.user_id;
    var userCountry = signedRequest.user.country;
}
```

*Note: parseSignedRequest will return undefined if validation fails. Always remember to check the result of parseSignedRequest before accessing the result.*

If you already set the appSeceret in options, you can ignore the second parameter when calling parseSignedRequest. If you do pass the second parameter it will use the appSecret passed in parameter instead of using appSecret from options.

If appSecret is absent, parseSignedRequest will throw an error.

```js
FB.options({ 'appSecret': 'app_secret'});

var signedRequestValue = 'signed_request_value';

var signedRequest  = FB.parseSignedRequest(signedRequestValue);
if(signedRequest) {
    var accessToken = signedRequest.oauth_token;
    var userId = signedRequest.user_id;
    var userCountry = signedRequest.user.country;
}
```

## Error handling

*Note: facebook is not consistent with their error format, and different systems can fail causing different error formats*

Some examples of various error codes you can check for:
* `'ECONNRESET'` - connection reset by peer
* `'ETIMEDOUT'` - connection timed out
* `'ESOCKETTIMEDOUT'` - socket timed out
* `'UNKNOWN'` - unknown error occured. (most likely there is not internet connection)


```js
FB.options({timeout: 1, accessToken: 'access_token'});

FB.api('/me', function (res) {
    if(res && res.error) {
        if(res.error.code === 'ETIMEDOUT') {
            console.log('request timeout');
        }
        else {
            console.log('error', res.error);
        }
    }
    else {
        console.log(res);
    }
});
```
