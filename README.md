rendr-conventions
=================

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

Rendr does not make a distinction in the templates and views, but I have found that the distinction between a controller-based view and a re-usable partial view is strong enough to warrant a separation.  Templates especially will be different if they are to represent a list of a resource or a the main page of a resource.  Things like headers and other auxilliary information may be found on a page that are not found in a partial.


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

Rendr does not make a distinction in the templates and views, but I have found that the distinction between a controller-based view and a re-usable partial view is strong enough to warrant a separation.  Views especially will be different if they are to represent a list of a resource or a the main page of a resource.  Things like headers and other auxilliary information may be found on a page that are not found in a partial.  Additionally, page-based views (controller views) are more likely to have other partial views included, where as partial views (non-controller) are more likely to thin. 

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

    

