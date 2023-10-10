---
marp: true
style: |

  section h1 {
    color: #6042BC;
  }

  section code {
    background-color: #e0e0ff;
  }

  footer {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    height: 100px;
  }

  footer img {
    position: absolute;
    width: 120px;
    right: 20px;
    top: 0;

  }
  section #title-slide-logo {
    margin-left: -60px;
  }
---

# Beyond Request/Response
### Why and how we should change the way we build web applications
Chris Nelson
@superchris
![h:200](full-color.png#title-slide-logo)

---

<!-- footer: ![](full-color.png) -->
# Who am I?
- 25+ year Web App Developer
- Co-Founder of Launch Scout
- Creator of LiveState

---

# We need to change the way we build web applications
### This is a big claim, right?

---

# Where are we?
- Is your experience of web app development better than it was 5 years ago?
- Are you building better apps faster?
- Why is there a new JS framework every week?
- Would this happen if we had something that most people were happy with?

---

# First let's talk about how we got here

---

# In the beginning there was CGI
- Scripts that printed out HTML
- Perl, VB, PL/SQL, you name it!
- This was ok for little tiny things
- For large applications, not so much

---

# Reasons
- HTTP is stateless, but applications are not
- Code organization was pretty random :(
- Perl is a write only language ;)
- DB stored procs that generated HTML
  - Need I say more?

---

# On the second day there was MVC
- Finally we could organize our code
- A place for everything
- Java, .NET

---

# Server side MVC

![](web1.png)

---

# On the third day DHH created Rails
- Convention over configuration
- Really productive language
- State lives in the DB
- We could build apps really fast

---

# And it was good..
## We were pretty productive at building web apps

---

# And then AJAX happened
- The user experience got a lot better
- The developer experiences, not so much..
- Previous server side MVC was no longer viable

---

# Two approaches

---

# Client side MVC
1. Let's apply same good ideas that worked server side in JS!
2. Let's build a JS MVC framwork!
3. Ugghh this one got big and complicated...
4. Go to 1.

---

# I know, we'll just use one language!
- Rails RJS (remote javascript)
- Server side JS framworks
- Compile *insert language here* into JS

---

# How'd that turn out?
- Mostly, not that great :(
- Turns multiple languages isn't the most important problem
- Client MVC and server MVC means 2 applications to keep in sync

---

# What's the actual problem ?
- HTTP is stateless, our applications have state
- With server-side MVC we had a place for our state
- Now it lives in (at least) two places..

---

# Maybe diagram here?

---

# Congratulations! We are building distributed systems
## And building distributed systems is hard...
- Shared mutable state is hard
- Across the network boundary it is almost impossible
- For emphasis, see CORBA

---

# This is where we've been for about 10 years
### *Le sigh*

---

# Let's talk about some good ideas!

---

# Let's talk about managing state...

---

## We see the same Design Pattern again and again
- Redux
- Elm
- GenServers
- LiveView
- It keeps emerging..

---

# The core idea
- A function which takes
  - Event (w/payload)
  - Current state
- And returns:
  - A new state

---

# It needs a name!
- Functional Reactive Programming?
- Maybe, but kinda not

---

# My proposal: Event State Reducers
## Spread the word!

---

# Example reducer
```js
addTodo({item}, todoList) {
  return todoList.concat(item);
}
```

---

# Another good idea: "dumb" components
- React
- EmberJS (actions up, data down)
- LiveView functional components

---

# Keeping components simple
- render data
  - often passed in as props
- dispatch events
- Step 3: Profit!

---

# Todo list element
```ts
@customElement('todo-list')
export class TodoList extends LitElement {

  @property({type: Array})
  todos: string[] = [];

  render() {
    return html`
      <ul>
        ${this.todos?.map(todo => html`<li>${todo}</li>`)}
      </ul>
    `;
  }
}
```

---
# Todo form
```ts
@customElement('todo-list')
export class TodoForm extends LitElement {

  @query('input[name="todo"]')
  todoInput: HTMLInputElement;

  render() {
    return html`
    <form>
      <input name="todo"/>
      <button @click=${this.addTodo}>Add item</button>
    </form>
    `;
  }

  addTodo(e) {
    e.preventDefault();
    this.dispatchEvent(new CustomEvent('add-todo', {detail: this.todoInput.value}));
  }
}
```

---
# Using em on a [web page](todo-website.html)
```html
<html>
  <head>
    <script type="module" src="http://localhost:4000/assets/app.js"></script>
  </head>
  <body>
    <h1>Todo List</h1>
    <todo-list todos='["Buy Milk", "Speak at Momentum"]'></todo-list>
    <todo-form></todo-form>
  </body>
</html>
```
---

# But how do we actually add todos?
- We could make an API
  - REST
  - GraphQL
- Is there a simpler way?

---

# What if we put these ideas together?
## On the client:
- subcribe to changes from server
- render state
- dispatch events
## On the server:
- receive events
- reducer functions compute new state
- push state changes to clients

---

# Diagram
![](event_reducers.png)

---

# LiveState
- An implementation of this pattern
- Not the first, not the only
- Client code javascript npm
- Server side Elixir library

---

# Finishing our Todo List
```ts
@customElement('todo-list')
@liveState({
  url: 'ws://localhost:4000/live_state',
  topic: 'todo_list',
  provide: {scope: window, name: 'todos'},
  properties: ['todos']
})
export class TodoList extends LitElement {
...
```
```ts
@customElement('todo-form')
@liveState({
  context: 'todos',
  events: {send: ['add-todo']}
})
export class TodoForm extends LitElement {
...
```
---
# Server side reducer
```elixir
defmodule SimpifiedCommentsWeb.TodoListChannel do
  @moduledoc false

  use LiveState.Channel, web_module: SimpifiedCommentsWeb

  @impl true
  def init(_channel, _params, _socket) do
    {:ok, %{todos: ["Buy Milk", "Speak at Momentum"]}}
  end

  @impl true
  def handle_event("add-todo", item, %{todos: todos} = state) do
    {:noreply, Map.put(state, :todos, [item | todos])}
  end

end

```
---

# How does this even work?
- `CommentSectionElement` makes WebSocket bidirectional connection during `connectCallback`
- `add-comment` event listeners are added to push events over the WS connection
- state updates arrive over WS connection
- listeners for state change events update the `comments` property
- `Lit` re-renders on prop changes

---

# That sounds like a lot of work!
- Nope, thanks to Phoenix and Elixir :)
- Phoenix Channels
  - An thin abstraction over WebSockets
- Erlang/OTP: 25 years of distributed computing lessons
- Extremely light-weight processes to manage state
- High availabity, concurrent

---

# Some observations
- Bi-directional
- Serve it from anywhere (include file://)
- No longer request/response
  - Message passing
  - PubSub
- Real time is essentially free!
  - Events can come from other sources
  - Computing state and notifying clients is the same

---
## Let's make a comment section
- We want to render existing comments
- We want to submit new ones

---
## Client code:
```ts
import { LitElement, html } from "lit";
import { customElement, property, query, state } from "lit/decorators.js";
import { liveState, liveStateConfig, liveStateProperty } from 'phx-live-state';

type Comment = {
  author: string;
  text: string;
}

@customElement('comments-section')
@liveState({
  url: 'ws://localhost:4000/live_state',
  topic: 'comments:all',
  events: {send: ['add-comment']}
})
export class CommentsSectionElement extends LitElement {

  @state()
  @liveStateProperty()
  comments: Comment[] = [];

  @query('form')
  form: HTMLFormElement;

  ...
}
```
---
## More client code:
```ts
  render() {
    return html`
      <dl>
        ${this.comments.map((comment) => html`
          <dt>${comment.author}</dt>
          <dd>${comment.text}</dd>
        `)}
      </dl>
      <form @submit=${this.addComment}>
        <div>
          <label>Author</label>
          <input name="author" />
        </div>
        <div>
          <label>Comment</label>
          <input name="text" />
        </div>
        <button>Add comment</button>
      </form>
    `;
  }

  addComment(e: SubmitEvent) {
    e.preventDefault();
    const comment = Object.fromEntries(new FormData(this.form));
    this.dispatchEvent(new CustomEvent('add-comment', {detail: comment}));
    this.form!.reset();
  }

```
---
# Server code:
```elixir
defmodule SimpifiedCommentsWeb.CommentsChannel do
  @moduledoc false

  use LiveState.Channel, web_module: SimpifiedCommentsWeb

  @impl true
  def init(_channel, _params, _socket) do
    {:ok, %{comments: []}}
  end

  @impl true
  def handle_event("add-comment", comment, %{comments: comments} = state) do
    {:noreply, Map.put(state, :comments, [comment | comments])}
  end

end
```

---

# [Demo](./wobsite.html)
### Page html
```html
<html>
  <head>
    <script type="module" src="http://localhost:4000/assets/app.js"></script>
  </head>
  <body>
    <h1>Pretend website</h1>

    Comments below...
    
    <comments-section url="ws://localhost:4000/live_state"></comments-section>
  </body>
</html>
```
---

# Lets make our comments real-time
- The scope of channel state is connection
- We need to notify other connected users
- Phoenix PubSub to the rescue!

---

# Show me teh codez
```elixir
defmodule SimpifiedCommentsWeb.CommentsChannel do
  @moduledoc false

  use LiveState.Channel, web_module: SimpifiedCommentsWeb
  alias Phoenix.PubSub

  @impl true
  def init(_channel, _params, _socket) do
    PubSub.subscribe(SimpifiedComments.PubSub, "comments")
    {:ok, %{comments: []}}
  end

  @impl true
  def handle_event("add-comment", comment, state) do
    PubSub.broadcast(SimpifiedComments.PubSub, "comments", {:add_comment, comment})
    {:noreply, state}
  end

  @impl true
  def handle_message({:add_comment, comment}, %{comments: comments} = state) do
    {:noreply, Map.put(state, :comments, [comment | comments])}
  end

end

```
---
# [Let's see it](wobsite.html)
---

# How about a more interesting example?
## Let's go shopping!
- We have static website (plain old html)
- We want to add *interactive* ecommerce

---

# What do we need?
- An add item button
- A cart
- And they need to share state

---

# Launch cart demo

---

# Other implementations
- Phoenix LiveView
 - the original
 - Elixir on client and server
 - Lots of JS to make it work, but you don't have to touch it :)

---
# LiveView client
```html
<dl>
  <%= for comment <- @comments do %>
    <dt><%= comment["author"] %></dt>
    <dd><%= comment["text"] %></dd>
  <% end %>
</dl>
<form phx-submit="add_comment">
  <div>
    <label>Author</label>
    <input name="author" />
  </div>
  <div>
    <label>Comment</label>
    <input name="text" />
  </div>
  <button>Add comment</button>
</form>
```
---
# LiveView server
```elixir
defmodule SimpifiedCommentsWeb.CommentsLive do
  use SimpifiedCommentsWeb, :live_view

  def mount(_parms, _session, socket) do
    {:ok, socket |> assign(:comments, [])}
  end

  def handle_event("add_comment", comment, %{assigns: %{comments: comments}} = socket) do
    {:noreply, socket |> assign(:comments, [comment | comments])}
  end
end
```

---

# Other other implementations
- LiveViewJS
- Hotwire (kinda?)

---

# But is this actually viable?
- Heck yes
  - We've been building LiveView apps for a couple years
  - LiveState is newer but starting to catch on
  - Our dev experience is radically improved
  - We're hitting estimates at a rate we haven't since Rails
- Not just us
  - [Cars.com](https://cars.com)
  - [LiveRoom.app](https://liveroom.app)
  - [PatientReach360](https://patientreach360.com)

---