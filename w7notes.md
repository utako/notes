# Week 7 - Client Side Manipulation!

## w7d1 - Client Side Manipulation WITHOUT BACKBONE

Some trivia: The person who wrote underscore also wrote Backbone.

### Jeff's Fighter Demo - Adding a Fighter Client Side

1. Add `fighter.js` file to `assets/models/`
2. Make the Fighter class.

        (function(root) {
          var Gym = root.Gym = (root.Gym || {});
          var Fighter = Gym.Fighter = function(attributes) {
            this.attributes = attributes;
          }
        })(this);

3. In `application.js`, be sure to `require` the appropriate trees and underscore library.
4. Use `_.extend` to define methods on the Fighter prototype in `fighter.js`:

        _.extend(Fighter.prototype, {

        // Getter of attributes:

          get: function(key) {
            return this.attributes[key];
          },

        // Setter:

          set: function (key, value) {
            this.attributes[key] = value;
          },

        // The comma is a comma and not a semicolon because it's a Javascript object.
        // Upon create, post the attributes:

          create: function() {
            var that = this;
            $.ajax({
              method: 'POST',
              url: 'api/fighters',
              data: this.attributes,
              success: function(response) {
                alert("Hell's bells it worked");
                _.extend(that.attributes, response);
              }
            });
          }
        });

5. Running this and trying to create a fighter from the client (running `f.create()` from the console) will give you a 400 error (bad request). It's missing a parameter. It should be nested inside "fighter." So, we fix:

        create: function() {
          var that = this;
          $.ajax({
            method: 'POST',
            url: 'api/fighters',
            data: {fighter: this.attributes},
            success: function(response) {
              alert("Hell's bells it worked");
              _.extend(that.attributes, response);
            }
          });
        }

6. And hell's bells, it works!

### Jeff's Fighter Demo - Client-side Fighter.all

1. Let's extend the class:

        _.extend(Fighter,{
          all: [],
        });

2. Upon creation of a fighter, we want to add it to the Fighter.all array.

        create: function() {
          var that = this;
          $.ajax({
            method: 'POST',
            url: 'api/fighters',
            data: {fighter: this.attributes},
            success: function(response) {
              alert("Hell's bells it worked");
              _.extend(that.attributes, response);
              Fighter.all.push(that)
            }
          });
        }

3. This will work temporarily, but it won't save the fighters in Fighter.all upon refresh. :-(
4. So we do this:

        _.extend(Fighter, {
          all: [],
          fetch: function(methodToCall) {
            $.ajax({
              method: 'GET',
              url: 'api/fighters',
              success: function(response) {
                Fighter.all = [];
                response.forEach(function(fighterObject){
                  var newFighter = new Gym.Fighter(fighterObject);
                  Fighter.all.push(newFighter);
                });
                methodToCall();
              }
            });
          }
        });

Note: `forEach` will not let you break.

### Jeff's Fighter Demo - Adding a View for Index

1. Make folder called "views" within `assets/javascripts/` and make a new file called `index.js`. Don't forget to include this directory in `application.js`.

        (function(root) {
          var Gym = root.Gym = (root.Gym || {});
          var Index = Gym.Index = function() {
            this.$el = $('gym-stuff');
          }
        })(this);

2. Make a new div in `index.html`:

        <div id="gym stuff"></div>

3. Then, let's make a render method by extending on the Index prototype.
        _.extend(Index.prototype, {
          render: function() {
            this.$el.html('yay it worked');
          }
        });

4. var v = new Gym.Index(); v.render(); will render "yay it worked" on the page. Now we need a template.

5. Create the template at `app/assets/templates/fighters/index.jst.ejs` and add it to the Index initialize. Let's also change the `render` method.

        (function(root) {
          var Gym = root.Gym = (root.Gym || {});
          var Index = Gym.Index = function() {
            this.$el = $('gym-stuff');
            this.template = JST['fighters/index'];
          }
        })(this);

        _.extend(Index.prototype, {
          render: function() {
            var renderedTemplate = this.template();
            this.$el.html(renderedTemplate);
          }
        });
6. We can pass in arguments to the template like in Ruby Rails:

        (function(root) {
          var Gym = root.Gym = (root.Gym || {});
          var Index = Gym.Index = function() {
            this.$el = $('gym-stuff');
            this.template = JST['fighters/index'];
          }
        })(this);

        _.extend(Index.prototype, {
          render: function() {
            var renderedTemplate = this.template({
              fighters: Gym.Fighter.all
            });
            this.$el.html(renderedTemplate);
          }
        });

7. In `index.jst.ejs`:

        <ul id="fighter-list">
          <% fighters.forEach(function(fighter) { %>
            <li><%= fighter.get('name') + " : " + "fighter.get('skill') %></li>
          <% }); %>
        </ul>
Difference between `get` and `fetch`: `get` is an instance method, `fetch` is a class method. `fetch` will look at all of the fighters, `get` will look at an instance of a class.
8. Before this works, we must first `fetch` in `index.html.erb`. Remember to add a callback so we make render wait for the fetching:

        <h1>Welcome to Jeff's Gym!</h1>
        <div id="gym-stuff"></div>
        <script type="text/javascript" charset="utf-8">
          Gym.Fighter.fetch(function() {
            var indexView = new Gym.Index();
            indexView.render();
          });
        </script>

### Jeff's Fighter Demo - Creating a Form

1. in `index.jst.ejs`
        <ul id="fighter-list">
          <% fighters.forEach(function(fighter) { %>
            <li><%= fighter.get('name') + " : " + "fighter.get('skill') %></li>
          <% }); %>
        </ul>

        <form id="fighter-form">
          <label for="fighter_name">Name</label>
          <input type="text name="fighter[name]" id="fighter_name" value = "" />
          <label for="fighter_skill">Skill</label>
          <input type="text name="fighter[skill]" id="fighter_skil" value = "" />
          <button type="submit">Create New Fighter</button>
        </form>
2. Adding an event (remember to include `jquery.serializejson.js` in `application.js`):
        _.extend(Index.prototype, {
          render: function() {
            var renderedTemplate = this.template({
              fighters: Gym.Fighter.all
            });
            this.$el.html(renderedTemplate);
            this.$el.submit('#fighter-form', this.handleFighterCreate.bind(this));
          },

          handleFighterCreate: function(event) {
            event.preventDefault();
          }
        });
3. In the console, we can now use jQuery to add a fighter by hand.
4. Now we add it to `handleFighterCreate` and be sure to remember that the data are nested under 'fighter'. We create the fighter, and post it to the server, clear the field:
        handleFighterCreate: function(event) {
          event.preventDefault();
          var $form = $(event.target);
          var fighterData = $form.serializeJSON();
          var newFighter = new Gym.Fighter(fighterData.fighter);
          $form.find('input').val('');
          newFighter.create();
        }
5. The machinery for registering (a) callback(s) to an event - we do this by instantiating an events hash where the event title points to a callback:

    In `fighter.js`:
        _.extend(Fighter, {
            ...
            events: {},
            on: function(eventTitle, callback) {
              this.events[eventTitle] = this.events[eventTitle] || [];
              this.events[eventTitle].push(callback);
            }
            trigger: function(eventTitle) {
              var callbacks = this.events[eventTitle];
              var arg = arguments[1];
              callbacks = callbacks || [];
              callbacks.forEach(function(callback) {
                callback(arg)
            });
          },
          ...
        })
6. Then, we register the event:

        create: function() {
          var that = this;
          $.ajax({
            method: 'POST',
            url: 'api/fighters',
            data: {fighter: this.attributes},
            success: function(response) {
              alert("Hell's bells it worked");
              _.extend(that.attributes, response);
              Fighter.all.push(that);
              Fighter.trigger('enterTheDojo');
            }
          });
        }
7. Now we add it to the views:

        (function(root) {
          var Gym = root.Gym = (root.Gym || {});
          var Index = Gym.Index = function() {
            this.$el = $('gym-stuff');
            this.template = JST['fighters/index'];
            Gym.Fighter.on('enterTheDojo', this.render.bind(this));
          }
        })(this);
