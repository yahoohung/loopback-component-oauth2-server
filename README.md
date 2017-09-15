# loopback-component-oauth2-server

This fork was tested on Loopback version 3.x.

The LoopBack oAuth 2.0 component provides full integration between [OAuth 2.0](http://tools.ietf.org/html/rfc6749)
and [LoopBack](http://loopback.io). It enables LoopBack applications to function
as an oAuth 2.0 provider to authenticate and authorize client applications and/or
resource owners (i.e. users) to access protected API endpoints. This fork allows 
you implement your own login logic.

The oAuth 2.0 protocol implementation is based on [oauth2orize](https://github.com/jaredhanson/oauth2orize)
and [passport](http://passportjs.org/). 

See [LoopBack Documentation - OAuth 2.0 Component](http://loopback.io/doc/en/lb3/OAuth-2.0.html) for more information.

## Install

Install the component as usual:

```
$ npm i loopback-component-oauth2-server -S
```

## Config example

server/boot/oauth.js

```js
module.exports = function(app) {
    var router = app.loopback.Router();
    var passport = require('passport');
    var CustomStrategy = require('passport-custom');
    var oauth2 = require('loopback-component-oauth2-server')

    var options = {
        // custom user model
        userModel: 'user',
        applicationModel: 'OAuthClientApplication',
        // -------------------------------------
        // Resource Server properties
        // -------------------------------------
        resourceServer: true,

        // used by modelBuilder, loopback-component-oauth2-server/models/index.js
        // Data source for oAuth2 metadata persistence
        dataSource: app.dataSources.db,

        // -------------------------------------
        // Authorization Server properties
        // -------------------------------------
        authorizationServer: true,

        // path to mount the authorization endpoint
        authorizePath: '/oauth/authorize',

        // path to mount the token endpoint
        tokenPath: '/oauth/token',

        // backend api does not host the login page
        loginPage: '/oauth/login',
        loginPath: '/oauth/login',
        loginFailPage: '/oauth/login?fail',

        // grant types that should be enabled
        supportedGrantTypes: [
            'implicit',
            'jwt',
            'clientCredentials',
            'authorizationCode',
            'refreshToken',
            'resourceOwnerPasswordCredentials'
        ]
    }
    oauth2.oAuth2Provider(
        app,
        options
    )

    router.get('/', function(req, res) {
        res.render('index');
    });

    router.get('/oauth/login', function(req, res) {
        res.render('login', {
            loginFailed: req.query && req.query.fail == '' ? true : false
        });
    });

    passport.use('loopback-oauth2-local', new CustomStrategy(
        function(req, callback) {
            // Do your custom user finding logic here, or set to false based on req object
            // verify user name, password 

            // findOrCreate local user 

            // return user
            callback(null, user);
        }
    ));

    app.use(router);
};
```

model-config.json
```JSON
    "user": {
        "dataSource": "db",
        "public": true,
        "options": {
            "remoting": {
                "sharedMethods": {
                    "*": false,
                    "logout": true
                }
            }
        }
    },
    "OAuthAccessToken": {
        "dataSource": "db",
        "public": false,
        "relations": {
            "application": {
                "type": "belongsTo",
                "model": "OAuthClientApplication",
                "foreignKey": "appId"
            },
            "user": {
                "type": "belongsTo",
                "model": "user",
                "foreignKey": "userId"
            }
        },
        "options": {
            "remoting": {
                "sharedMethods": {
                    "*": false
                }
            }
        }
    },
    "OAuthAuthorizationCode": {
        "dataSource": "db",
        "public": false,
        "options": {
            "remoting": {
                "sharedMethods": {
                    "*": false
                }
            }
        }
    },
    "OAuthClientApplication": {
        "dataSource": "db",
        "public": false,
        "options": {
            "remoting": {
                "sharedMethods": {
                    "*": false
                }
            }
        }
    },
    "OAuthPermission": {
        "dataSource": "db",
        "public": false,
        "options": {
            "remoting": {
                "sharedMethods": {
                    "*": false
                }
            }
        }
    },
    "OAuthScopeMapping": {
        "dataSource": "db",
        "public": false,
        "options": {
            "remoting": {
                "sharedMethods": {
                    "*": false
                }
            }
        }
    },
    "OAuthScope": {
        "dataSource": "db",
        "public": false,
        "options": {
            "remoting": {
                "sharedMethods": {
                    "*": false
                }
            }
        }
    },
```

common/models/user.json
```JSON
    "relations": {
        "accessTokens": {
            "type": "hasMany",
            "model": "OAuthAccessToken",
            "foreignKey": "userId",
            "options": {
                "disableInclude": true
            }
        }
    },
```
The app instance will be used to set up middleware and routes. The data source
provides persistence for the oAuth 2.0 metadata models.

For more information, see [OAuth 2.0](http://loopback.io/doc/en/lb3/OAuth-2.0.html) LoopBack component official documentation.
