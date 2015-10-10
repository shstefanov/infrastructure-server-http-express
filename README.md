This is metapackage, containing needed tools for building http-pages structure for application that uses infrastructure framework (https://github.com/shstefanov/infrastructure)

Installation
============

    npm install https://github.com/shstefanov/infrastructure-server-pages-express.git

Configuration
=============

    {
      "path": "pages",
      "engines": [
        "infrastructure-server-engine-mongodb",
        "infrastructure-server-engine-express"
      ],
      "loaders": ["infrastructure-server-pages-express/loader"],
      "libs": {
        "Page" :              "infrastructure-server-pages-express/Page",
        "Api" :               "infrastructure-server-pages-express/Api"
      },

      "config": {

        "session": {
          "secret":             "app.session",
          "collection":         "sessions",
          "clear_interval":     60,
          "cookie": {
            "maxAge":           86400000
          },
          "autoReconnect":      true,
          "stringify":          false,
          "resave":             true,
          "saveUninitialized":  true
        },

        "views":{
          "path":        "templates",
          "view_engine": "jade",
          "cache":       true
        },

        "mongodb": {
          "host":            "localhost",
          "port":            27017,
          "db":              "orbits",
          "auto_reconnect":  true,
          "options":         { "nativeParser": false }
        },

        "http": {
          "port":    3000,
          "favicon": "public/fav.ico",
          "morgan":  ":method :url :status :response-time ms - :res[content-length]",
          "static": {
            "/public":   "public",
            "/locales":  "locales",
            "/vendor/bootstrap":    "node_modules/bootstrap/dist",
            "/vendor/font-awesome": "node_modules/font-awesome/"
          } 
        }

      }

    }

- "path": points to folder (based on project rootDir) where your structure files are located
- "engines": "infrastructure-server-engine-express" (https://github.com/shstefanov/infrastructure-server-engine-express) is required, "infrastructure-server-engine-mongodb" is optional. Keep in mind, that mongodb engine should be initialize first if used.
- "loaders": the loader for the pages in this structure. This structure should be named "pages".
- "config": Express engine requires "http" configuration as shown above. The only required option is "port". Other options are optional and will activate specific middlewares. "favicon" will activate "serve-favicon" middleware, "morgan" will actiavte "morgan" logger that will log using specified pattern, "static" will activate express.static using key as http path, value as fs path.If mongodb engine is added to engines and config contains configurations for "mongodb" and "session". MongoStore will be activated to keep the sessions in specific mongodb database. "views" is separate config object and tells the engine what view-engine to use, where tempates are located (based on rootDir) and option to use cache. For now, only "jade" is supported.


Usage
=====

In your pages structure directory you need to place files that looks like

    module.exports = function(){
      var env = this;
      return env.lib.Page.extend("HomePage", {
        root:          "/dashboard",         // Root path that will be prependen to matching url
        template:      "dashboard",          // The template to be rendered (this will point to templates folder -> dashboard.jade)
        title:         "Dashboard - Orbits", // page title, also it can be function
        // title: function(req, res){ return "Page title"; }

        meta: { // metatags
          "name": "content"
        },

        config: {
          // some config that will be passed to the template (will be stringified)
        },

        // Styles urls that will be provided to template to be rendered
        styles: [
          "/vendor/bootstrap/css/bootstrap.min.css"
        ],

        // Javascripts urls that will be provided to template to be rendered
        javascripts: [],

        // This handler acts as middleware that will be executed before any of matched urls handlers below
        pre: function(req, res, next){
          if( !req.session.logget ) return res.redirect("/auth");
          next();
        },

        // This will match GET request for "/dashboard*""
        // Available methods: GET, POST, PUT, DELETE, ALL
        "GET *" : function(req, res, next){
          // For every request, res.data empty object will be attached;
          // this object will collect any data and finally will be passed to the template
          // render method can be forced by this.render(req, res)
          next();
        },

        // This handler acts as middleware that will be executed after any of matched urls handlers above
        after: function(req, res, next){
          this.render(req, res);
        }

      });
    };

The data, attached to response object, but some properties will e handled and changed before rendering

    {
      "cache": false,    // This property will be always overriden - the value of config.views.cache will be set
      "meta": {},        // Will extend metatags defined in the page class
      "styles": [],      // Styles that will be added to styles, defined in page class
      "javascripts": [], // Styles that will be added to javascripts, defined in page class
      "config": {},      // This will extend config, defined in page class
      "title":           // If provided, will be used instead of title defined in page class
      "page":  page      // page instance itself, in case it holds helpers functions or some dinamic data
      // Everithing else you can attach to res.data in handlers
    }

