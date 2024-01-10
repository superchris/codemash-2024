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

# Subjectivity warning
## Based on experience tho

---

# Web development compared to 5-10 years ago
- More complex
- Slower
- Less enjoyable
- Not noticeably better

---

# This talk is about..
- What happened
- Why
- How we can make things better

---

# First let's talk about how we got here
## A brief history of web development

---

# The dawn of time: 1993-1997
- NCSA Mosaic
- Lynx
- Netscape
- No dynamic anything

---

# CGI: the web app equivalent of stone tablets
- Perl, VB, PL/SQL, you name it!
- This was ok for little tiny things
- For large applications, not so much
- Code organization mostly spaghetti nightmare

---

# The browser wars: 1997-2005ish
- Netscape adds Java/Javascript
- Microsoft gets scared
- IE gets bundled into Windoze
- IE wins (95%+ by 2004)

---

# Server side MVC

![](web1.png)

---

# Rails
## IMO opinion the pinnacle of server side MVC
- Convention over configuration
- Really productive language
- State lives in the DB
- We could build apps really fast

---

# The AJAX era: 2005-2015
- XMLHttpRequest
- DHTML
- GMail sets the bar 2004
- IE stops innovating after it wins
  - standards, schmandards

---

# Meanwhile in developer land...
- The user experience got a lot better
- The developer experiences, not so much..
- We need to deal with client side code
- Browsers ain't gonna innovate, so DIY

---

# Getting the client under control

---

# Client side MVC
1. Let's apply same good ideas that worked server side in JS!
2. Let's build a JS MVC framework!
3. Ugghh this one got big and complicated...
4. Go to 1.

---

# All the MVCs

![](web2.png)

---

# Maybe the problem is multiple languages
## Let's just use one!
- Rails RJS (remote javascript)
- Server side JS frameworks
- Compile *insert language here* into JS

---

- Mostly, not that great :(
- Turns multiple languages isn't the most important problem
- Client and server code have different concerns
- Client MVC and server MVC means 2 applications to keep in sync

---

# What makes things so complicated?
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

# And all that client side code...
- Client side build tools
- Dependency management
- Compilation and transpilation

---

# The modern era: 2015 - present
- Competitive browsers makes standards matter again
- Browser makers are more cooperative than ever before
- Browser innovation has absolutely exloded
  - to the point where developers are not keeping up

--

# A few highlights
- Web components (custom HTML elements)
- Websockets
- Javascript maturation
- Webassembly

---

# An embarrassment of riches
- It's impossible to keep up
- Browsers are innovating faster then developers
- New ideas based are not yet mainstream
- Displacement is uncomfortable

---

# We are mostly still coding like its 2015
- Frameworks that reinvent rather than leverage standards
- Transpiling to obsolete JS versions
- Complex builds
- Oscillations between SPA vs server rendered

---

# But it's doesn't have to be this way...

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

# Event/State Reducers
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
- Simple and predictable
- Easy to test
- Well suited to async

---

# Another good idea: "dumb" clients
- React
- EmberJS (actions up, data down)
- LiveView functional components

---

# Clients are dumb if...
- render data
  - often passed in as props
- dispatch events
- That's it!

---

# An example: let's make a comment section

---

## `<comments-section>` element
```ts
@customElement('comments-section')
export class CommentsSectionElement extends LitElement {

  @state()
  comments: string[] = [];

  render() {
    return html`
      <ul>
        ${this.comments.map((comment) => html`<li>${comment}</li>`)}
      </ul>
      <form @submit=${this.addComment}>
        <div>
          <label>Comment</label>
          <input name="comment" />
        </div>
        <button>Add comment</button>
      </form>
    `;
  }

  addComment(e: SubmitEvent) {
    e.preventDefault();
    this.dispatchEvent(new CustomEvent('add-comment', {detail: {comment: this.commentInput.value}}));
    this.commentInput!.value = '';
  }
}
```
---

# But how do we actually add comments?
- We could make an API
  - REST
  - GraphQL
- Is there a simpler way?

---

# Websockets
- Allows bidirectional communication from client to server
- What could that give us?

---

# Putting the good ideas together...

---
![](event_reducers.png)

## Client
- render state
- sends events
- receive state changes
## Server
- receive events
- reducer functions compute new state
- send state changes

---

# LiveState
- An implementation of this pattern
- Not the first, not the only
- Client code javascript npm
- Server side Elixir library

---

# Finishing our Comment Section
```ts
@customElement('comments-section')
@liveState({
  topic: 'comments:all',
  url: 'ws://localhost:4000/live_state',
  events: {send: ['add-comment']}
})
export class CommentsSectionElement extends LitElement {

  @state()
  @liveStateProperty()
  comments: string[] = [];
...
```
---
# Server side reducer
```elixir
defmodule SimpifiedCommentsWeb.CommentsChannel do
  use LiveState.Channel, web_module: SimpifiedCommentsWeb

  def init(_channel, _params, _socket) do
    {:ok, %{comments: []}}
  end

  def handle_event("add-comment", %{"comment" => comment}, %{comments: comments} = state) do
    {:noreply, Map.put(state, :comments, [comment | comments])}
  end

end
```
---

# Putting it all [together](wobsite.html)
```html
<html>
  <head>
    <script type="module" src="http://localhost:4000/assets/app.js"></script>
  </head>
  <body>
    <h1>Pretend website</h1>

    Comments below...
    
    <comments-section></comments-section>
  </body>
</html>
```

---

# How does this even work?
- `<comments-section>` makes WebSocket bidirectional connection during `connectCallback`
- `add-comment` event listeners are added to push events over the WS connection
- state updates arrive over WS connection
- listeners for state change events update the `comments` property in `<comments-section>`
- `Lit` re-renders on prop changes

---

# Why Elixir?
- Erlang/OTP: 25 years of distributed computing learning baked in
- Extremely light-weight processes to manage state
  - Each connection has their own process and state
- High availabity, concurrent
- Phoenix Channels
  - An thin abstraction over WebSockets

---

# Why is this better?
- Higher level of abstraction
- From request/response
- To events and state
- State lives on the server
  - Not shared
  - Immutable

---

# More goodies
- Bi-directional
- Serve HTML from anywhere
- Real time is essentially free!
  - Events can come from other sources that user interacation
  - Computing state and notifying clients is the same

---

# Real-time comments!

---
## Just sprinkle in some [PubSub...](wobsite.html)
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
  def handle_event("add-comment", %{"comment" => comment}, state) do
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

# So that's cool but I want more control!
- I want to control the HTML that renders my comments
- What if I could put the template to render the comments right in my HTML?
- What if I didn't have to make a custom element to do it?

---

# What do we need?
- We've had the `template` element
- We haven't had a way to add dynamic bits
- W3C specs that aim to solve this:
  - Template instantiation
  - DOM Parts

---

# No the specs are not final but...
- Multiple polyfill implementations exist
- They are largely interoperable

---

# Introducing `<live-template>`
- Connects a template to a Livestate
- Renders state
- Dispatches events
- Repeat...

---

# An interesting aside..
- We don't need a build tool to use `live-template`
- import maps are :fire:
- let your browser resolve and fetch dependencies
- jspm makes it ridonkulously easy

---

# Let's write some code

---

# Other other implementations
- LiveView (Elixir)
- LiveViewJS
- Hotwire (kinda?)
- LiveSvelte

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
- [Cars.com](https://cars.com)

---

# Questions to ask yourself
- Do I need a frameork?
  - Could my browser do this instead?
- Does my state need to be in two places?
- Could I keep things simpler?
- Is there a test for this?

---

# Bonus round!:
- WebAssembly!
- Until fairly recently, not super practical
  - Calling WebAssembly modules with anything other than numbers was a nightmare
- Things like Extism and WebAssembly Components eliminate this hurdle
- Writing event handlers in the language of your choice is now possible!

---

## Livestate [todo list](http://localhost:4004) reducer in Javascript (compiled to wasm)
```js
import { wrap } from "./wrap";

export const init = wrap(function() {
  return { todos: ["Hello", "WASM"]};
});

export const addTodo = wrap(function({ todo }, { todos }) {
  return { todos: [`${todo} from WASM!`, ...todos]};
});

```

---

# Thanks!

---