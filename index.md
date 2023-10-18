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
chris@launchscout.com
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

# All the MVCs

![](web2.png)

---

# What's the actual problem ?
- HTTP is stateless, our applications have state
- With server-side MVC we had a place for our state
- Now it lives in (at least) two places..
  - And it's our job to keep it in sync(ish)

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
## A functional design pattern for managing state
- Reducer functions which take
  - Event (w/payload)
  - Current state
- And return:
  - A new state

---

# Todo list reducer
* Add item event
  ```js
  todoReducer({name: "Add item", item: "Get Milk"}, [])
  ```
* Returns new state: ```["Get Milk"]```
* Add a second item
  ```js
  todoReducer({name: "Add item", item: "Speak at Momentum"}, ["Get Milk"])
  ```
* Returns new state: ```["Get Milk", "Speak at Momentum"]```
---

# Things we like
- Easy to understand
- State is immutable
- Well suited to functional languages

---

# It needs a name!
- Functional Reactive Programming?
- Maybe, but kinda not

---

# My proposal: Event State Reducers
## Spread the word!

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

# `<todo-list>` element
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
# <todo-form>
```ts
@customElement('todo-form')
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

# But how do we actually add todos?
- We could make an API
  - REST
  - GraphQL
- Is there a simpler way?

---

# What if we put these ideas together?
## On the client:
- render state
- dispatch events
- subcribe to changes from server
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

# Putting it all [together](todo-website.html)
```html
<html>
  <head>
    <script type="module" src="http://localhost:4000/assets/app.js"></script>
  </head>
  <body>
    <h1>Todo List</h1>
    <todo-list></todo-list>
    <todo-form></todo-form>
  </body>
</html>
```
---

# How does this even work?
- `<todo-list>` and `<todo-form>` make WebSocket bidirectional connection during `connectCallback`
- `add-todo` event listeners are added to push events over the WS connection
- state updates arrive over WS connection
- listeners for state change events update the `todos` property in `<todo-list>`
- `Lit` re-renders on prop changes

---

# That sounds like a lot of work!
- Nope, thanks to Phoenix and Elixir :)
- Phoenix Channels
  - An thin abstraction over WebSockets
- Erlang/OTP: 25 years of distributed computing lessons
- Extremely light-weight processes to manage state
  - Each connection has their own process and state
- High availabity, concurrent

---

# Things we like
- Bi-directional
- Serve HTML from anywhere (include file://)
- No longer request/response
  - Event oriented
  - PubSub
- Real time is essentially free!
  - Events can come from other sources
  - Computing state and notifying clients is the same

---

# Let's chat!

---
# <lets-chat>
```ts
import { LitElement, html } from "lit";
import { customElement, property, query, state } from "lit/decorators.js";
import { liveState, liveStateConfig, liveStateProperty } from 'phx-live-state';

type Message = {
  author: string;
  text: string;
}

@customElement('lets-chat')
@liveState({
  topic: 'chat:all',
  events: {send: ['add-message']}
})
export class LetsChatElement extends LitElement {

  @liveStateConfig('url')
  @property()
  url: string = '';

  @state()
  @liveStateProperty()
  messages: Message[] = [];

  @query('form')
  form: HTMLFormElement;
```

---
# <lets-chat> cont'd
```ts
  render() {
    return html`
      <dl>
        ${this.messages.map((message) => html`
          <dt>${message.author}</dt>
          <dd>${message.text}</dd>
        `)}
      </dl>
      <form @submit=${this.addMessage}>
        <div>
          <label>Author</label>
          <input name="author" />
        </div>
        <div>
          <label>Message</label>
          <input name="text" />
        </div>
        <button>Add message</button>
      </form>
    `;
  }

  addMessage(e: SubmitEvent) {
    e.preventDefault();
    const message = Object.fromEntries(new FormData(this.form));
    this.dispatchEvent(new CustomEvent('add-message', {detail: message}));
    this.form!.reset();
  }
}
```

---

# Server code:
```elixir
defmodule SimpifiedCommentsWeb.ChatChannel do
  @moduledoc false

  use LiveState.Channel, web_module: SimpifiedCommentsWeb
  alias Phoenix.PubSub

  @impl true
  def init(_channel, _params, _socket) do
    PubSub.subscribe(SimpifiedComments.PubSub, "messages")
    {:ok, %{messages: []}}
  end

  @impl true
  def handle_event("add-message", message, state) do
    PubSub.broadcast(SimpifiedComments.PubSub, "messages", {:add_message, message})
    {:noreply, state}
  end

  @impl true
  def handle_message({:add_message, message}, %{messages: messages} = state) do
    {:noreply, Map.put(state, :messages, [message | messages])}
  end

end

```

---

# [Demo](./chat.html)
### Page html
```html
<html>
  <head>
    <script type="module" src="http://localhost:4000/assets/app.js"></script>
  </head>
  <body>
    <h1>Pretend website</h1>

    Let's chat...
    
    <lets-chat url="ws://localhost:4000/live_state"></lets-chat>
  </body>
</html>
```

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
- use-live-state React hook

---

# But is this actually viable?
- Heck yes
- We've been building LiveView apps for a couple years
- LiveState is newer but starting to catch on
- Our dev experience is radically improved
- We're hitting estimates at a rate we haven't since Rails

---

# Production examples
- [Launch Elements](https://launch-cart-dev.fly.dev/)
  - [Demo](tiny-store.html)
- [LiveRoom.app](https://liveroom.app)
- [PatientReach360](https://patientreach360.com)

---

# Future things
- WebAssembly!
  - Until fairly recently, not super practical
  - Things like Extism and WebAssembly Components eliminate significant hurdles
  - Writing event handlers in the language of your choice is now practical

---

# Thanks!!

---