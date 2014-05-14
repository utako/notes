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

1. in `index.jst.ejs`:

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
          },

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


## w7d2 - Intro to BACKBONE


### MODEL

#### Model methods to know:

* initialize
* get
* set
* id (database) vs cid (client-side only, prefixed with a 'c')
* escape
* save
* fetch: takes option hash and merges it with a default ajax hash
* url
* model

#### `.url()`

1. When `.url()` is called, it first looks at t.url()
  '/api/todos/1'
2. Todo way (set in model). It looks at t.urlRoot
  '/api/todos'
3. Constructs the url using the url Root and the model.
4. Comments way (set in collection): t.collection.url()


#### .models

  Underscore provides a bunch of extra shortcut methods to make dealing with models of collections easier.


### ROUTER

* The router creates the "multi-page" illusion to the "one-page" property of the Backbone app.
You can pass the routes a function, but instead, it's better to pass it the name of the function you want it to call.

        routes: {
          '': 'index',
          'examples/new': 'new',
          'examples/:id': 'show',
          'examples/:id/edit': 'edit'
        },
        index: function() {},
        new: function() {},
        show: function() {},
        edit: function() {}

`View.prototype.render()` fills up the `$el`. You want the thing you're always changing in your one-pager to be your `$rootEl`.

        new Router() {
          $rootEl: $("#main")
        }
This is too tightly coupled. To loosen this up, declare your stuff elsewhere

        this.$rootEl.html(view.render().$el)

* `.el` and  `$el`: These will point to a newly created <div> if not already referenced. You can otherwise set `.el` and `$el` to a specific element, or you can specify a tagName to set it to something other than a div. Try to just rely on the default div it gives you.

You can have as many router as you want, as long as there are no conflicting route.

### listenTo

In the views initialize,

          this.listenTo(this.todo, 'add', this.render)


### Events

Use an events hash instead of manually binding events in the initialize of the view.

### listenTo vs Events

Events: Interactions with the client
listento: Interactions with the model

## Backbone Video Notes

### 12: HTML Injection Attacks and XSS, nested Backbone views, form for associated object, markdown, keyup


#### HTML Injection Attacks

Example of an attack:

        TodoComment.create!(todo_id: 1, content: "<script>alert('hello')</script>")
* Comes from the fact that we are injecting the text of the comment title directly into the template:

        comment.get("content")
Can avoid this by escaping the HTML. We don't want to put this directly into the DOM.

##### `escape()`
Browser won't misinterpret injected HTML and will escape those in the template. It will literally put <script>alert('hello')</script> instead of the HTML entity.

#### Creating a Form to Create New Comments

1. Create new view `comments_new.js`

        window.Collection.Views.CommentsNew = Backbown.View.extend({
          template: JST["comments/new"]  
        });
2. Make a template: `new.jst.ejs`

        <form>
          <textarea></textarea>
          <input type="submit">
        </form>
3.

        window.Collection.Views.CommentsNew = Backbown.View.extend({
          template: JST["comments/new"]  
          render: function() {
            var renderedContent = this.template();
            this.$el.html(renderedContent);
            return this;
          }
        });
4. Add this view to our `todo_show.js` (janky way)

        <h3><%= todo.get("title") %></h3>

        <ul>
          <% todo.comments().each(function (comment) { %>
          <li><%= comment.escape("content") %></li>
          <% }) %>
        </ul>

        <div class="comment-new"></div>
        <a href="#/">Back to Index</a>
5. Add to our view:

        window.Todo.Views.CommentsNew = Backbown.View.extend({
          template: JST["comments/new"]  
          render: function() {
            var renderedContent = this.template();
            this.$el.html(renderedContent);
            var commentNewView = new Todo.Views.CommentsNew();
            this.$(".comment-new").html(commentNewView.render().$el)
            return this;
          }
        });
6. To make it actually do something upon submit, you add it to the events hash:

        window.Todo.Views.CommentsNew = Backbown.View.extend({

          events: {
            "submit form": "submit"
          },

          template: JST["comments/new"],

          render: function() {
            var renderedContent = this.template();
            this.$el.html(renderedContent);
            var commentNewView = new Todo.Views.CommentsNew();
            this.$(".comment-new").html(commentNewView.render().$el)
            return this;
          },

          submit: function(event) {
            event.preventDefault();
            var comment = new Todo.Models.Comment({
              todo_id: "...",
              content: "..."
            })
          }

        });
7. Addressing the `todo_id` in `new.jst.ejs`:

        <form>
        <input type= "hidden" name="coment[todo_id]" value="<%= todo.escape("id")%>">
          <textarea name="comment[content]"></textarea>
          <input type="submit">
        </form>
8. Write an initialize method to get where your todo comes from, pass in comment params to the render method.

        window.Todo.Views.CommentsNew = Backbown.View.extend({
          initialize: function(options) {
            this.todo = options.todo;
          },
          events: {
            "submit form": "submit"
          },

          template: JST["comments/new"],

          render: function() {
            var renderedContent = this.template();
            this.$el.html(renderedContent);
            var commentNewView = new Todo.Views.CommentsNew({
              todo: this.model
              });
            this.$(".comment-new").html(commentNewView.render().$el)
            return this;
          },

          submit: function(event) {
            var view = this;
            event.preventDefault();
            var params = $(event.currentTarget).serializeJSON()["comment"];
            var comment = new Todo.Models.Comment(params);
          comment.save({}, {
            suess: function() { view.todo.comments().add(comment)}
            })

        });
9. Still won't automatically upload, so we have to go back and in the show page, we have to both listen to the sync event, but also listen to a "sync add."
10. If we want to let users inject HTML into the content and limit what you let them inject, then you use markdown!

#### Using Markdown for Injection

1. require the `marked` library.
2. In addition to escaping the content, compile the element into markdown HTML by:

        <li><%= marked(comment.escape("content"))%></li>
3. Then, users can type things into the textarea using markdown.
4. Then in the CommentsNew view, we add this to the events hash:

        event: {
          "submit form" : "submit",
          "keyup textarea": "handleKeyup"
        }
5. To render the preview every time someone types a key, we write handleKeyup and renderPreview methods. Be sure to make a preview `<div>` on the page.

        handleKeyup: function() {
          console.log(this.$("textarea").val())
        },
        renderPreview: function() {
          var content = this.$(textarea).val();
          this.$(".preview").html(marked(content));
        }
6. Unfortunately, we still need to escape this content. We can use underscore!

        renderPreview: function() {
          var content = this.$(textarea).val();
          var previewContent = marked(_.escape(content));
          this.$(".preview").html(previewContent);
        }

### 13: ZOMBIE VIEWS, listenTo, swapping router, View#remove

#### Old Way vs New Way

Zombie Views: views whose elements are no longer on the page, but where a call to render will still render the content "invisibly."

* Old Way: `$("body").html(newView.render().$el)`: Puts the new element of the view into the html. Removes existing contents of the body.
* New Way: swap 'em out! Don't instantiating views over and over again. Very common method to write on the router.

        _swapView: function(view) {
          if (this.currentView) {
            this.currentView.stopListening();
          }
          this.currentView = view;
          $("body").html(view.render().$el)
        }
`stopListening()` will eliminate the connection between events and the zombie view. This will make events faster and reduce memory consumption. As long as there's a listener on the zombie view, it can't be garbage collected.

#### `View#remove`

* `remove` removes the view element from the dom and stops it from listening. It's a nice method because any extra work we have to do beyond stopping listening is something we can write in a remove subclass. We can use this instead of `this.currentView.stopListening`.

### 16: Editing Comments, Opening Views

* Fancy-edit: being able to double click something to edit it.

1. Extend comments show view.

        events: {
          "click button.destroy": "destroy",
          "dblclick div.content": "beginEditing"
        }
2. And in the comments show template, add a div for the content.

        <div class"content"><%= marked(comment.escape("content")) %></div>
3. Back in the show view, introduce a property in the open property:

        initialize: function(options) {
          this.open = false;
        }

        beginEditing: function() {
          this.open = true;
        },
4. Make an `edit.jst.ejs`

        <li>
          <form class="comment">
            <input type="submit">
          </form>
        </li>
5. Now we make some logic in the template:

        template: function() {
          return this.open ? JST["comments/edit"] : JST["comments/show"];
        },
6. Be sure to add "render" to the beginEditing function.
7. Closing of the subview:

        endEditing: function(event) {
          this.open = false;
          this.render();
        }
8. Add it to the events:

        events: {
          "click button.destroy": "destroy",
          "dblclick div.content": "beginEditing",
          "submit form.comment": "endEditing"
        },
9. Update the template:

        <li>
          <form class="comment">
            <textarea class="comment_content"><%= comment.escape("content") %></textarea>
            <input type="submit">
          </form>
        </li>
10. Upon ending the editing, we need to pull the content out of the form, in addition to rendering.

        ndEditing: function(event) {
          this.open = false;
          var content = this.$(textarea.comment_content").val();
          this.model.save({ content: content} )
          this.render();
        }
11. This produces a "can't mass assign protected attributes" error. It's because the response uploads all of the fields, so we need to use `delete` in the model:

        toJSON: function() {
          var json = Backbone.Model.prototype.toJSON.call(this);
          delete json.id;
          delete json.created_at;
          delete json.updated_at;
          return json;
        }
12. New problem: if you have two comment edit textareas open and submit one, we close one of the other comment textareas. This will be solved by doing something...

### w7d3 - Creating and Rendering Comments on the Todo Item

#### On writing `getOrFetch()`
* In `App.Collections.Todos`:

        getOrFetch: function(id) {
          var todo;
          todo = this.get(id);
            if (todo) {
              return todo;
            }
            else {
              todo = new App.Models.Todo({ id: id });
              todo.fetch();
              return todo;
            }
        }
This starts fetching the model and returns before it has been fully fetched. This is ok, because it will wait for the data to render by doing the following:
* In `App.View.TodosShow`:

        initialize: function() {
          this.listenTo(this.model, "sync", this.render);
        };
`listenTo()` will bind the object to the method called.

#### Step 1: Fetching the comments

* In `App.Collections.TodoComments`
```javascript
  initialize: function () {
    this.listenTo(this.model, "sync", this.render);

    this.comments = new App.Collections.TodoComments([], {
      todo: this.model;
    })

    this.comments.fetch();
    this.listenTo(this.comments, "sync", this.render)
  }
```
Don't want to instantiate it every single time you want to get the comments for the todo. Instead of this, we want to go to the Todo model.
* In `App.Models.Todo`:
```javascript
  comments: function() {
    if (!this._comments) {
      this._comments = new App.Collections.TodoComments([], {
      todo: this;
      })
    }
    return this._comments;
  }
```
* And in the view:
```javascript
  render: function() {
    var renderedContent = this.template({
      todo: this.model,
      comments: this.model.comments()
    })
    this.$el.html(renderedContent);
    return this;
  }
```

#### Step 2:

* In App.Views.CommentsNew
```javascript
  template: [JST["comments/new"]],

  render: function() {
    var htmlContent = this.template({
      todo: this.model
    });

  }
```
* The Template:

  ```html
  <h3>New Comment!</h3>
  <form>
    </label>
      Content
      <textarea name="comment[content]"></textarea>
    </label>
    <input type="submit" value="Create Coment!">
  </form>
```
* In the TodoShow, we add a new div.

  ```html
  <div class="new-comment"></div>
```
* In `App.Views.TodoShow`, we make `this.newCommentView` in the initialize function, and also add it to `render()`.

* To add a `todo_id` to the comment:
```html
  <input type="hidden"  name="comment[todo_id]" value="<%= todo.escape("id")%>">
```

* Add it to the events hash in `App.Views.CommentNew`
```javascript
  events: {
    "submit form": "createComment"
  },

  createComment: function(event) {
    event.preventDefault();
    var $form = $(event.currentTarget);
    var comment = new App.Models.Comment($form.serializeJSON().comments);
    comment.save({}, {
      success: function() {
        view.model.comments().add(comment);
      }
    })
  }
```
* If we want to group API requests:
  1. `include` will automatically nest in associated items
```ruby
  def show
    @todo = Todo.find(params[:id])
    render json: :todo, include: :comments
  end
```
  2. This gives us an ugly response, so we `#parse()` it! We delete the resp.comments from the response, but we use the given comments to set `this.comments`. Neat. We have to ask it to keep parsing the comments, tho.
```javascript
  parse: function(resp) {
    if (resp.comments) {
      this.comments().set(resp.comments, { parse: true});
      delete resp.comments;
    }
    return resp;
  },
```

### On `#delegateEvents()` Magic:

For when you have a nested view! The idea here is at one point, I'm creating a `this.commentsNewView`. Whenever Backbone creates a new view, it creates an element that creates the body of the view. At one point, we're passing this element into the view. If we pass $el back into the view, we have to remind the DOM to start listening to events.

The first time we run render, it's going to run the template and stick $el into the template at the specified div location.

The second time it renders, it's goign to reset $el by reevaluating the template, clear out the contents of the show's $el and rebuilds it with the new content and stick it back into the $el. When you do that, we have to reattach the listeners by using `.delegateEvents()`.
