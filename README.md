## EventBroker API
Provides a general purpose [Backbone](http://documentcloud.github.com/backbone/ "Title") Event Broker implementation based on the Backbone [Events API](http://documentcloud.github.com/backbone/#Events "Title").

Run the <a href="http://htmlpreview.github.com/?https://github.com/efeminella/backbone-eventbroker/blob/master/spec/spec-runner.html" target="_blank">Specs</a>
 / <a href="http://htmlpreview.github.com/?https://github.com/efeminella/backbone-eventbroker/blob/master/examples/basic/index.html" target="_blank">Example</a>

The `EventBroker` can be used directly to serve as a centralized event management mechanism for an entire application. Namespaced brokers can also be created in order to provide context specific brokers within an application.

### Basic Usage
The `EventBroker` can be used directly to publish and subscribe to events of interest:

``` javascript
var Users = Backbone.Collection.extend{{
    initialize: function(){
      // subscribe to an event ...
      this.listenTo(Backbone.EventBroker, 'users:add', this.add);
    },
    add: function(user) {
      console.log(user.get('id'));
    }
};

var UserEditor = Backbone.View.extend({
     el: '#user-editor',
     initialize: function(broker){
        this.$userId = this.$('.user-id');
     },
     add: function() {
       // publish an event ...
       Backbone.EventBroker.trigger('users:add', new User({
           'id': this.$userId().val()
           //other values ...
       }));
    }
};
// ...
```

### Creating namespaced EventBrokers
The `EventBroker` API can be used to create and retrieve any number of specific namespaced `EventBrokers`. A namespaced `EventBroker` ensures that all events are published and subscribed against a specific namespace.

Namespaced `EventBrokers` are retrieved via `Backbone.EventBroker.get([namespace])`. If an `EventBroker` has not been created for the given namespace, it will be created and returned. All subsequent retrievals will return the same `EventBroker` instance for the specified namespace; i.e. only one unique `EventBroker` is created per namespace.

``` javascript
var Users = Backbone.Collection.extend{{
    // use the 'users' broker
    usersBroker: Backbone.EventBroker.get('users'),
    initialize: function(broker){
      this.listenTo(this.usersBroker, 'add', this.add);
    },
    add: function(user) {
      console.log(user.get('id'));
    }
};

var UserEditor = Backbone.View.extend({
    el: '#user-editor',
    events: {
        'click .add-user' : 'addUser'
      , 'click .add-role' : 'addRole'
    },
    // use the 'users' broker
    usersBroker: Backbone.EventBroker.get('users'),
    // also use the 'roles' broker
    rolesBroker: Backbone.EventBroker.get('roles'),

    addUser: function(evt) {
      // publish an event to the usersBroker
      this.usersBroker.trigger('add', new User({
           'id': this.$('.user-id').val()
           //other values ...
      }));
    },
    addRole: function(evt) {
      // publish an event to the rolesBroker
      this.rolesBroker.trigger('add', new Role({
           'type': this.$('.user-id').val()
      }));
    }
};
```

Since namespaced `EventBrokers` ensure events are only piped thru the `EventBroker` of the given namespace, it is not necessary to prefix event names with the specific namespace to which they belong. While this can simplify implementation code, you can still prefix event names to aid in readability if desired.

``` javascript
var Users = Backbone.Collection.extend{{
    // use the 'users' broker
    usersBroker: Backbone.EventBroker.get('users'),

    initialize: function(broker){
      // prefix the namespace if desired
      this.listenTo(this.usersBroker, 'users:add', this.add);
    },
    add: function(user) {
      console.log(user.get('id'));
    }
};

var UserEditor = Backbone.View.extend({
    el: '#user-editor',
    events: {
        'click .add-user' : 'addUser'
      , 'click .add-role' : 'addRole'
    },
    // use the 'users' broker
    usersBroker: Backbone.EventBroker.get('users'),
    // also use the 'roles' broker
    rolesBroker: Backbone.EventBroker.get('roles'),

    addUser: function(evt) {
      // publish an event to the usersBroker
      this.usersBroker.trigger('users:add', new User({
           'id': this.$('.user-id').val()
           //other values ...
      }));
    },
    addRole: function(evt) {
      // publish an event to the rolesBroker
      this.rolesBroker.trigger('roles:add', new Role({
           'type': this.$('.user-id').val()
      }));
    }
};
```

### Registering Interests
Modules can register events of interest with an `EventBroker` via the default '[on](http://documentcloud.github.com/backbone/#Events-on "Title")' method or the `register` method. The `register` method allows for registering multiple event/callback mappings declaratively for a given context in a manner similar to that of the [events hash](http://documentcloud.github.com/backbone/#View-extend "Title") in a Backbone.View.

``` javascript
// Register event/callbacks based on a hash and associated context
var Users = Backbone.Collection.extend({
    initialize: function() {
      Backbone.EventBroker.register({
        'user:select'   : 'select'
      , 'user:deselect' : 'deselect'
      , 'user:edit'     : 'edit'
      , 'user:update'   : 'update'
      , 'user:remove'   : 'remove'
      }, this );
    },
    select: function() { ... },
    deselect: function() { ... },
    edit: function() { ... },
    update: function() { ... },
    remove: function() { ... }
});
```

Alternatively, modules can define an "interests" property which provides specific event/callback mappings, allowing for declarative registration with an `EventBroker`:

``` javascript
// Register event/callbacks based on a hash and associated context
var Users = Backbone.Collection.extend({
    // defines events of interest and their corresponding callbacks
    interests: {
        'user:select'   : 'select'
      , 'user:deselect' : 'deselect'
      , 'user:edit'     : 'edit'
      , 'user:update'   : 'update'
      , 'user:remove'   : 'remove'
    },
    initialize: function() {
      // register this object with the EventBroker
      Backbone.EventBroker.register(this);
    },
    select: function() { ... },
    deselect: function() { ... },
    edit: function() { ... },
    update: function() { ... },
    remove: function() { ... }
});
```
Objects can also implement their "interests" as a function which returns an object of specific event/callback mappings, allowing for runtime configurations of interests:

``` javascript
// Register event/callbacks based on a hash and associated context
var Users = Backbone.Collection.extend({
    // defines events of interest and their corresponding callbacks
    this.interests: function(){
        return _.extend({
            'user:select'   : 'select'
          , 'user:deselect' : 'deselect'
        }, ( this.isAdmin() ? {
            'user:edit'     : 'edit'
          , 'user:update'   : 'update'
          , 'user:remove'   : 'remove'
        } : {} ));
    },
    initialize: function() {
      // register this object with the EventBroker
      Backbone.EventBroker.register(this);
    },
    select: function() { ... },
    deselect: function() { ... },
    edit: function() { ... },
    update: function() { ... },
    remove: function() { ... }
});
```

As of version 1.0.0, if a given callback is not a function, the EventBroker will throw an exception, similar to declaratively mapping an event callback in a Backbone.View.

Modules can use different namespaced `EventBrokers` for different things...

``` javascript
// Register event/callbacks with different EventBrokers...
var CartView = Backbone.View.extend({
    // reference the 'items' EventBroker...
    itemsBroker: Backbone.EventBroker.get('items'),

    // reference the 'inventory' EventBroker...
    inventoryBroker: Backbone.EventBroker.get('inventory'),

    initialize: function() {
      // register events/callbacks with 'items' EventBroker...
      this.itemsBroker.register({
        'add'      : 'add'
      , 'update'   : 'update'
      , 'remove'   : 'remove'
      }, this );
      // register events/callbacks with 'inventory' EventBroker...
      this.inventoryBroker.register({
        'select'   : 'select'
      , 'deselect' : 'deselect'
      , 'edit'     : 'edit'
      }, this );
    },
    add: function() { ... },
    update: function() { ... },
    remove: function() { ... },
    select: function() { ... },
    deselect: function() { ... },
    edit: function() { ... }
});
```

### Determining if an EventBroker has been created
To test if an `EventBroker` has been created for a given `namespace`, invoke the `has` method:

``` javascript
// determines if an event broker for the given namespace exists
var broker = Backbone.EventBroker;
broker.get('roles'); // returns the 'roles' EventBroker
broker.has('roles'); //true
broker.has('users'); //false
```


### Destroying an EventBroker
To destroy an existing `EventBroker` for a given `namespace`, invoke the `destroy` method:

``` javascript
// deletes the event broker for the given namespace
var broker = Backbone.EventBroker;
broker.get('permissions');
broker.destroy('permissions'); // returns the 'permissions' EventBroker
broker.has('permissions'); //false
```


### Destroying all EventBrokers
To destroy all existing `EventBrokers`, invoke the `destroy` method with no arguments:

``` javascript
// deletes the event broker for the given namespace
var broker = Backbone.EventBroker;
broker.get('permissions'); // returns the 'permissions' EventBroker
broker.get('users'); // returns the 'users' EventBroker
broker.get('roles'); // returns the 'roles' EventBroker
broker.destroy();

broker.has('permissions' ); //false
broker.has('users'); //false
broker.has('roles'); //false
```

### Undefined and null Events
As of version 1.1.1, invoking EventBroker.trigger with null or undefined will result in an error being thrown. This can help mitigate tracking down unhandled events due to erroneously undefined event names being referenced.

``` javascript
var Events = {
    LOGIN: 'login'
};
// Events.LOGIN is defined, thus the event is successfully triggered
Backbone.EventBroker.trigger(Events.LOGIN); // triggers

// Events.AUTHENTICATION is undefined, thus an exception is thrown
Backbone.EventBroker.trigger(Events.AUTHENTICATION); // error thrown
```
