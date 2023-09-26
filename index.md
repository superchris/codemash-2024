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
- Long-time (old) Web App Developer
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
- Things got out of control rather quickly....

---

# On the second day there was MVC
- Finally we could organize our code
- A place for everything
- A little clunky perhaps
- Java, .NET

---

# On the third day DHH created Rails
- Convention over configuration
- Really productive language
- We could go faster

---

# And it was good..

---

# And then AJAX happened

---

# And it was terrible...
- Wads of JQuery all over the place
- Then the cycle of JS frameworks began

---

# The cycle
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
- Turns multiple languages isn't problem
- Which means isomorphism isn't the answer

---

# The actual problem: We are building distributed systems
## And building distributed systems is hard...

---

# But y tho?
- Shared mutable state is hard
- Across the network boundary it is almost impossible
- For emphasis, see CORBA

---

# So let's talk about state...

---

# We have a great design pattern for managing state
- We just need to describe it
- Redux, Elm, GenServers
- It keeps emerging..

---

# Event State Reducers
- A function which takes
  - Event (w/payload)
  - Current state
- And returns:
  - A new state

---

# Interesting points
- State is immutable
- ???

---

# How we can we apply this to web app development?
### What are the pieces and where should they live?

---

# Events
- Our browser is great at producing these
- Seems like they need to originate client side
- It would be nice to use plain ole DOM events

---

# State
- We'd *really* like a single source of truth
- This makes us want to originate it server side

---

# Reducers
- They kinda need to run where the state is
- This means server side

---

# LiveState
- Dispatch events
- Subscribe to state
- Client code is simple
  - render state and dispatch events
- Server code is simple
  - reducer functions

---

# Example time
## Let's make a comment section
- We want to render existing comments
- We want to submit new ones
- We want to see them come in without a refresh

---

# How does this work?
- Elixir and Erlang
- OTP: 25 years of distributed computing lessons
- Extremely light-weight processes to manage state
- High availabity, concurrent

---

![link to slides](loop-rocket.gif)
- slides: https://github.com/launchscout/elixirconf2023-beyond-liveview
- live_elements: https://github.com/launchscout/live_elements
- live_state elixir library: https://github.com/launchscout/live_state
- phx-live-state client npm: https://github.com/launchscout/live-state

---
