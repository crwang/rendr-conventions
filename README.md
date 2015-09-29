rendr-conventions
=================

[![Join the chat at https://gitter.im/crwang/rendr-conventions](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/crwang/rendr-conventions?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Conventions we are using for working with apps developed on the Rendr Framework

# Introduction

Rendr has been loosely modeled after Rails, in terms of having standard conventions.  The directory structures combines thet Rails convention with Backbone JS.

# Project Structure

    ├─┬ app
    │ ├── collections
    │ ├── controllers
    │ ├── lib
    │ ├── models
    │ ├── templates
    │ ├── views
    │ │
    │ ├── app.js
    │ ├── router.js
    │ └── routes.js
    │ 
    ├─┬ assets
    │ ├── stylesheets
    │ └── vendor
    ├── config
    ├── node_modules
    ├─┬ public
    │ └── images
    ├─┬ server
    │ └─┬ middleware
    │   └── index.js
    ├─┬ test
    │ └─┬ app
    │   ├── collections
    │   ├── models
    │   └── views
    │
    ├── Gruntfile.js
    ├── package.json
    ├── Readme.md

## app

### Structure Overview

    ├── collections
    ├── controllers
    ├── lib
    ├── models
    ├── templates
    ├── views
    │
    ├── app.js
    ├── router.js
    └── routes.js

#### collections

##### Description

The collections folder is used to house the Rendr collections, based on backbone js.

Collections are ordered sets of models.
http://backbonejs.org/#Collection

Rendr collections typically require a model and extend the base collection (app/collections/base.js).  

##### Convention

Rendr collections should be named similar to the Rails convention of how they treat models and resources.  

The collection filename should be snake-cased (underscored) and pluralized.

The class name for the collection however should be cap-cased.

##### Example - app/collections/companies.js

    var Company = require('../models/company')
      , Base = require('./base');

    module.exports = Base.extend({
      model: Company,
      url: '/companies'
    });
    module.exports.id = 'Companies';


#### controllers

##### Description

Controllers are for responding to routes.

Controllers do not exist in Backbone js, but the convention follows the rails naming convention for routes.  

##### Convention

Rendr controller filenames should be snake-cased and pluralized and followed by an _controller.js

#####  Actions

Actions in Rendr are most advantageous as HTTP GET calls.

Actions map to views which will display the "page" based on the routes.

http://guides.rubyonrails.org/action_controller_overview.html

Conventional Actions:

*index*

Similar to Rails convention, the index action on a resource-based controller, should display a collection of models.

Url: http://example.com/employees

*show*

Similar to Rails convention, the show action on a resource-based controller, should display a single model.

Url: http://example.com/employees/1

*new*

Similar to Rails convention, the new action on a resource-based controller, should display a form for filling to create a new record/instance of a resource.

Url: http://example.com/employees/new

*edit*

Similar to Rails convention, the edit action on a resource-based controller, should display a form for filling to edit an existing record/instance of a resource.

Url: http://example.com/employees/1/edit

Note: These need to be defined in the app/routes.js file.

An example app/routes.js file would be:

    module.exports = function(match) {
      match('',                             'home#index');
      match('companies',                    'companies#index');
      match('companies/:id',                'companies#show');
      match('companies/new',                'companies#new');
      match('companies/:id/edit',           'companies#edit');
    };


##### Example - app/controllers/companies_controller.js

Below is an example of 

    module.exports = {
        // Show the list of companies
        index: function(params, callback) {
            var spec = {
                  collection: {collection: 'Companies', params: params}
                };
            };
            this.app.fetch(spec, function(err, result) {
              callback(err, result);
            });
        },

        // Show the current company.
        // Also, show related data - the company's employees.
        show: function(params, callback) {
            var spec = {
                model: {
                    model: 'Company',
                    params: params
                },
                employees: {
                    collection: 'Employees',
                    params: {
                        company: params.id
                    }
                }
            };
            this.app.fetch(spec, function(err, result) {
                callback(err, result);
            });
        }
    };


#### lib

#### models

##### Description

The models folder is used to house the Rendr models, based on backbone js.

Models are the core data of the application.
http://backbonejs.org/#Model

Rendr models extend the base model (app/models/base.js).  

##### Convention

Rendr models should be named similar to the Rails convention of how they treat models and resources.  

The model filename should be snake-cased (underscored) and singular.

The class name for the model however should be cap-cased.

##### Example - app/models/company.js

    var Base = require('./base');

    module.exports = Base.extend({
      url: '/companies/:id' // The url used for syncing the model.
    });

    module.exports.id = 'Company';

Note that Rendr expects the primary key of the model to be id as is the standard in Rails Active Record.  If the id is not 'id', then it should be set with the idAttribute inside the Base.extend.

Example of id field not being named id:

    module.exports = Base.extend({
      url: '/companies/:id',
      idAttribute: 'companyId' // custom id.
    });


#### templates

##### Description

The templates folder is used to house the Rendr templates/markup, the default being handlebars js (http://handlebarsjs.com).

Rendr templates are modeled after backbone templates.

Here, I will only include information on handlebars since that is what we are using.

##### Convention

Rendr templates should be in a folder under templates, with the folder name being the pluralized version of the resource.  

For example, if we have templates for our resource company, then there should be an app/templates/companies folder.

The template filenames should be snake-cased (underscored) and named after either the action of the controller OR after the view it is associated with.

The template names we use follows the Rails action controller standard, plus our own standard for non-controller based, partial views.

Controller-based:
- index.hbs
- show.hbs
- new.hbs
- edit.hbs

Non-Controller-based:

Non-Controller-based views and templates are used as partials and are meant for re-usability across the entire application.  Action-based templates and views are purposed for a route and the more traditional "page" concept.

Non-Controller based templates are meant for partials.

- list.hbs 
    + Similar to index.hbs, but instead of representing a page, it more closely resembles a re-usable partial template.
- item.hbs
    + Similar to show.hbs, but instead of representing a page, it more closely resembles a re-usable partial template.
- form.hbs
    + Similar to edit.hbs, but instead of representing a page, it more closely resembles a re-usable partial template.

*A note about Controller-based vs Non-Controller-based templates and views:*

Rendr does not make a distinction in the templates and views, but I have found that the distinction between a controller-based view and a re-usable partial view is strong enough to warrant a separation.  Templates especially will be different if they are to represent a list of a resource or a the main page of a resource.  Things like headers and other auxiliary information may be found on a page that are not found in a partial.


##### Example - app/templates/companies/index.hbs

A template for a page with a list of companies.

    <h1>Companies</h1>
    <div class="media-list">
        {{#each _collection.models}}
            {{view "companies/item" model=this}}
        {{/each}}
    </div>

##### Example - app/templates/companies/list.hbs

A template for a list of companies.

    <div class="media-list">
        {{#each _collection.models}}
            {{view "companies/item" model=this}}
        {{/each}}
    </div>


##### Example - app/templates/companies/show.hbs

A template for a page of a particular company.

We may want to show employees and other information as well.

    <header>
        <h1>{{name}}</h1>
        <p>
        {{description}}
        </p>
    </header>
    <section>
        <h2>Employees</h2>
        {{#each employees.models}}
            {{view "employees/item" model=this}}
        {{/each}}
    </section>
    <section>
        <h2>Other Related Company Information</h2>
    </section>


##### Example - app/templates/companies/item.hbs

A template for a company item.

    <div class="media media-company">
        <a class="pull-left" href="#">
            {{#if imageUrl}}
                <img class="media-object" src="{{imageUrl}}" alt="pic">
            {{/if}}
            {{^imageUrl}}
                <img class="media-object" src="http://placehold.it/100x100" alt="pic">
            {{/imageUrl}}
        </a>
        <div class="media-body">
            <h4 class="media-heading">{{name}}</h4>
            <p>
            {{description}}
            </p>
            <button class="btn btn-default">Show Details</button>
        </div>
    </div>

#### views

##### Description

The views folder is used to house the Rendr views, based on backbone views.

On Github, there is some documentation for Rendr's Views: https://github.com/rendrjs/rendr#baseview 

##### Convention

Rendr views should be in a folder under views, with the folder name being the pluralized version of the resource.  

For example, if we have views for our resource "company", then there should be an app/views/companies folder.

The view filenames should be snake-cased (underscored) and named after either the action of the controller OR a name of a partial.

The view names we use follows the Rails action controller standard, plus our own standard for non-controller based, partial views.

Controller-based:
- index.js
- show.js
- new.js
- edit.js

Non-Controller based:

Non-Controller-based views and templates are used as partials and are meant for re-usability across the entire application.  Controller-based templates and views are purposed for a route and the more traditional "page" concept.

Non-Controller based views are meant for partials.

- list.js 
    + Similar to index.js, but instead of representing a page, it more closely resembles a re-usable partial view.
- item.js
    + Similar to show.js, but instead of representing a page, it more closely resembles a re-usable partial view.
- form.js
    + Similar to edit.js, but instead of representing a page, it more closely resembles a re-usable partial view.

*A note about Controller-based vs Non-Controller-based templates and views:*

Rendr does not make a distinction in the templates and views, but I have found that the distinction between a controller-based view and a re-usable partial view is strong enough to warrant a separation.  Views especially will be different if they are to represent a list of a resource or a the main page of a resource.  Things like headers and other auxiliary information may be found on a page that are not found in a partial.  Additionally, page-based views (controller views) are more likely to have other partial views included, where as partial views (non-controller) are more likely to thin. 

##### Example - app/views/employees/index.js

    var BaseView = require('../base');

    module.exports = BaseView.extend({
      className: 'employees_index_view'
    });
    module.exports.id = 'employees/index';

##### Example - app/views/employees/item.js

    var BaseView = require('../base');

    module.exports = BaseView.extend({
        className: 'employees_item_view',

        // We have post render for dom and other stuff
        postRender: function() {
            this.model.on('change', this.render, this);
        },

        // It's backbone, so we have events!
        events: {
            'click button': 'buttonClicked'
        },
        buttonClicked: function(e) {
            alert('We have an attached model! ' + this.model.get('id'));
        }          
    });
    module.exports.id = 'employees/item';

## assets

## config

## node_modules

## public

## server

## test


# Running the app

From the command-line of your application, type:

    grunt server


# Routes

## Overview

Routes are defined in the /app/routes

An example is below:

    match('account/login','account#login');

The first argument matches the url.  The second parameter is the [controller]#[action].  So `account/login` would route to account_controller's login method.

As of late 2014, no helpers exist to help with abstracting out the routes as exists in Rails.  Therefore, in our links, we are hard-coding the path

## Query parameters

Reference: https://github.com/rendrjs/rendr/issues/89

Also, if we need to ignore query parameters, then we can add the query portion to the route to enable it to be ignored and to be handled correctly on the client/server side.

    match('categories',           'home#index');
    match('categories?*qs',       'home#index');


## Leading slash for link urls

eg.

    <a href="/account/login">Login</a>

Please note that the / is necessary currently for the client-side routing to work correctly.

Bad (no leading / )

    <a href="account/login">Login</a>


# Coding Tips

These are current coding tips.  Because Rendr is in flux, I'm going to datestamp these to try to prevent errors as it changes.

## Retrieving Data Dynamically (2014-06-16) (data pulled from API)

    fetchEmployees: function(callback) {
        var sessionData = this.app.get('session');
        var spec = [{
            collection: 'Employees',
            params: {
                organization_id: 1,
                access_token: sessionData.access_token
            }
        }];
        this.app.fetch(spec, function(err, result) {
            if (result) {
                var collection = null;
                try {
                    collection = result[0];
                } catch (error) {
                }
                callback(err, collection);
            }
        });
    },

## Adding Views with Data Dynamically (2014-06-16)

    fetchEmployees: function(callback) {
        var sessionData = this.app.get('session');
        var spec = [{
            collection: 'Employees',
            params: {
                organization_id: 1,
                access_token: sessionData.access_token
            }
        }];
        this.app.fetch(spec, function(err, result) {
            if (result) {
                var collection = null;
                try {
                    collection = result[0];
                } catch (error) {
                }
                callback(err, collection);
            }
        });

        // This is another way but asking it not to read from cache which is the default
        /*
        this.app.fetch(spec, { readFromCache: false }, function(err, result) {
            if (result) {
                var collection = null;
                try {
                    collection = result[0];
                } catch (error) {
                }
                callback(err, collection);
            }
        });
        */
    },

    someFunctionToDoTagInputForEmployees: function() {        
        var TagInputView = BaseView.getView('taginput/form');
        this.fetchEmployees(function(err, data) {
                      
            // Taginput
            _this.tagInputView = new TagInputView({
                app: _this.app,
                minKeywordLength: 0,
                collection: data,
                onSelect: function(model) {
                    console.log(model);
                }
            });    
            $('#someElem').html(_this.tagInputView.render().el);
        });    
    }


## Adding Views Dynamically (data exists)

This is an example of adding an employee view to an employee list view.

Note that we already have the employee model and are just creating the view dynamically on the client-side.

This code would be located in **views/employees/list.js**
    
        var itemViews = this.getItemViews();
        // Then, we add a new view.
    
        // Get the View
        var EmployeeItemView = BaseView.getView('employees/item'); 
    
        // Instantiate the new view.
        var view = new EmployeeItemView({
            app: this.app,
            model: employee
        });
    
        // Add the model to our collection.
        this.collection.add(employee);
    
        // Add the view to our child views.  Don't use the itemViews because it's a different reference.
        this.childViews.push(view);
    
        // Add the view to the list.
        this.$('div.media-list').append(view.render().el);
    

## Adding a Model to a Collection Dynamically and then re-rendering the List view. (2014-06-04)

Because we are using web sockets for part of our app, we will received json data that we will need to convert into a model and then have reflected on the UI.

The flow of the code is this:

1. Receive JSON
2. Convert it to a model
3. Add the model to the collection
4. Re-render the view.

Here's the way I've gotten it to work:

    // Some view
    ...
    postRender: function() {
        var _this = this;
        // 1. Receive the new member data
        channel.bind('new_member', function(json) {
            // 2. Convert it into a model.
            // Note that we passi n the app
            var newTeamMember = new TeamMember(json, {
                app: _this.app // Make sure to add the app.
            });
        newTeamMember.store(); // And store this model in the store since we are persisting it.
        // 3. Add the model ot the collection
        _this.collectionMembers.add(newTeamMember);
    }

    inSomeOtherCodeThatIsFiredByAButtonClick: function() {    
        var TeamMemberListView = BaseView.getView('team_members/list');
        var currMemberListView = new TeamMemberListView({
            app: _this.app,
            collection: this.collectionMembers
        });                    
        // 4. Then render with the latest data.
        $(this.el).find('#some-element').html(currMemberListView.render().el);
    }


## Accessing Subviews (2014-05-29)

In the way we have been coding, we have also needed to access subviews from within our views, especially client-side.  Following are some examples of how we are accomplishing this:

### Accessing List Views

This is a basic function declaration in the parent view.

    getItemViews: function() {
        var views = this.getChildViewsByName('view_name_here');
        return views;
    },

This is basic function declaration in a view with a subview of 'employees/list'.  In this example, if we want to access the subview from the parent view, we can do it this way.  

**Note: if there are more than one subview with the same view name, we cannot use this approach.**

This code could be located in **views/company/index.js**

    getEmployeesListView: function() {
        var views = this.getChildViewsByName('employees/list');
        return views;
    },


### Accessing Other Kinds of Views

Here is an example of getting a single sub-view that is not a list.  Notice the array access:

This code could be located in **views/employees/item.js**

    getTeamsActionBarView: function() {
        var views = this.getChildViewsByName('employees/action_bar');
        if (views && views.length) {
            return views[0];
        }
        return null;
    },


### Using Templates inside Views/Controllers

    var template = this.app.templateAdapter.getTemplate('users/show'),
        html = template(data);


### Sending events to the app

## Nested JSON Data

[Converting Nested Data to Models or Collections](https://github.com/crwang/rendr-conventions/wiki/Converting-nested-JSON-data-into-models-or-collections)

## Passing Vales through Templates

[Example of how to pass values through templates](https://github.com/crwang/rendr-conventions/wiki/Passing-values-into-views-through-a-template)

# Config

## Introduction
The config example https://github.com/rendrjs/rendr-examples/blob/master/01_config shows a way to set up config using yaml files.

It is look for a NODE_ENV setting to determine which config file to use, because it uses the config module: https://www.npmjs.org/package/config.

For more information on setting this on heroku, the link for node is here: https://devcenter.heroku.com/articles/nodejs-support

To find the config setting on the command line:
    
    echo $NODE_ENV

To set the env config setting on the Mac from the command line:

    export NODE_ENV=production

To check the env config setting on the Mac from the command line:

    env

or if you have a ton of enviroment variables and want to filter:

    env | grep NODE

The config then can be access through dot notation, but is only apparently available in the express portion of the app.  The typical use case would be to access the config in /index.js.

Please note that indentation is important in yml files.

## Setting up the config

There should be at least 3 files in /config: default.yml, development.yml, production.yml

Here are some examples:

default.yml

    server:
      port: 3030
    api:
      default:
        host: localhost:3000
        protocol: http
    oauth:
      client_id: xxxxxxxxx
      client_secret: xxxxxxxxx
    session:
      secret: A secret
    pusher:
      app_key: aaaaa

development.yml

    server:
      port: 80
    api:
      default:
        host: dev-app.herokuapp.com
        protocol: http
    oauth:
      client_id: xxxxxxxxx
      client_secret: xxxxxxxxx
    session:
      secret: A secret for dev
    pusher:
      app_key: bbbbb

production.yml

    server:
      port: 80
    api:
      default:
        host: productionApp.com
        protocol: https
    oauth:
      client_id: xxxxxxxxx
      client_secret: xxxxxxxxx
    session:
      secret: A secret for production
    pusher:
      app_key: ccccc

**When changing the config, you will need to rerun grunt server for the changes to take effect**

If you want to avoid this, you need to modify the /gruntfile.js and add this code under your watch to force a rebuild.

    grunt.initConfig({
        ...
        watch: {
            ...
            config: {
                files: 'config/**/*.yml',
                tasks: ['browserify'],
                options: {
                    interrupt: true
                }
            },
            ...
        ...
    });

## Using Config in /index.js

    var express = require('express'),
        config = require('config'),
        ...
        var dataAdapterConfig = {
            'default': {
                host: config.api.default.host,
                protocol: config.api.default.protocol
            }
        };
        ...

## Using Config in Views, Models, Controllers

Then, if the config data is needed within views, models, or controllers, we should set the data into the app.

In /index.js

Add this code 

    server.configure(function(rendrExpressApp) {
        ...
        // Set the config into the app
        rendrExpressApp.use(function(req, res, next) {
            var app = req.rendrApp;
            app.set('config', config);
            next();
        });
        ...
    }

Then in your views, models, and controllers you can acces the config as

Eg, some views/employees/index.js

    var config = this.app.get('config');
    var pusherAppKey = config.pusher.app_key;

# Writing Middleware

# Cookies

# Session Data

# Request flow

## Client

syncer (client-side) --> Backbone.sync --> (HTTP) --> apiProxy --> dataAdapter --> (HTTP) --> backend

The syncer calls the client Sync which makes the rest call, proxied through the api proxy and the data adapter.  The response returns in the syncer's clientSync call and returned through Backbone.sync.

## Server
syncer --> dataAdapter --> (HTTP) --> backend

The syncer (rendr/shared/syncer) calls the data adapter to make the request.

Then when the request returns, the syncer abstracts the success and passes through the data.

*From the Rendr code documentation:*

The syncer is a collection of instance methods that are mixed into the prototypes of `BaseModel` and `BaseCollection`. The purpose is to encapsulate shared logic for fetching data from the API.

# Testing

To run the tests, make sure to first install the karma command-line client:

    npm install -g karma-cli

or 

    sudo npm install -g karma-cli

Then, you can run the tests:

    npm test


# Notes from Rendr Docs, but with some additions

## Rendr Options

### Server Config

#### Example

    var config = {
      dataAdapterConfig: {
        'default': {
          host: 'api.github.com',
          protocol: 'https'
        }
      },
      apiPath: '/api',
      appData: { myAttr: 'value'},
      dataAdapter: myDataAdapterInstance,
      apiProxy: myApiProxy,
      defaultEngine: 'js',
      entryPath: process.cwd() + '/myapp'
      errorHandler: function (err, req, res, next){},
      notFoundHandler: function (req, res, next){},
      viewsPath: "/app/views",
    };
    rendr.createServer(config);
