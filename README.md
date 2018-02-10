# Websocket Notes

> A real time note taking application using Websockets and Vue.js

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```

## Demo

http://rfstaging.aaahollister.webfactional.com/websocket-notes/

## Questions

### What was your reasoning for implementing the solution they way you have?
Given the time available, I went for a pared down approach avoiding Vuex and RxJS. Rather than using RxJS Observables from the start, I wanted to build out the logic and make initial progress towards the core requirements. I haven't used Observables much, nor Websockets really, so I wanted to tackle one challenge at a time and, knowing that I would be learning Websockets as I go, I chose to leave RxJS for the end if time allows.

To start I set up a basic Vue app using vue-cli and a browserify template, and set up a websocket connection to test out the different events and start getting messages broadcasting to connected clients. I didn't quite get anything going with your provided socket server address, so I switched over to using echo.websocket.org for testing. Once I had more of the app set up, I hosted the provided socket in an Express server on Heroku here: ws://vast-brushlands-58657.herokuapp.com/

You can find the code here: https://github.com/ahollister/simple-socket

I've tried to keep as much logic as possible outside of lower level components and in the App component. This is where messages are sent to the socket server and where they're handled, with the App component acting as a kind of controller for the rest of the app. This allows us to build more simple, reusable components, an example of this would be the MessageBox component, which takes some props and emits an event to the App component based on user input.

With every request sent to the socket server, I'm including an object with an action property and whatever other data is required for that action to be interpreted when received. When this is broadcast to clients, the onmessage event triggers a function which manipulates state and runs other code according to the action and data received, sometimes that might mean merging everybody's user lists together, or adding a note another user just created to our notes list and localStorage.

### Which part of your solution would you consider improving on if you had the time? Why?

I would have thought more about whether to fold the EditBox functionality into the MessageBox component since they're almost the same. I think there are more potential components in the App template, perhaps a multipurpose list component that could handle both the User and Notes lists, styling determined by props.

With more time, I'd definitely be up for re-constructing the app to use a pattern of subscribing to Observables. Having read more about them since starting this challenge it does look like this is pretty much a perfect use case, (probably why you suggested RxJS I guess!). I'm probably going to go on and do this at some point soon since it seems like a good opportunity to learn more about RxJS, if you want I can send you the updated code when I've implemented it.

Also if I were to continue building this project, especially if I were planning on adding any more features, I'd consider delegating the state management that's happening in App to a Vuex store and simplifying the App component. There are also some patterns that re-occur I'd pull into utility functions, a lot of creating objects to stringify and send to the socket for example.

I'd implement a more realistic system for user management. We'd need a system for users to sign up properly, (so they can have real accounts), and we'd need to implement an authentication system for the login.

With regards to your Backend challenge, I'm not totally sure how I'd implement a history as in-memory storage inside the Websocket server as you suggested, but to achieve a similar goal I'd probably look at offloading the data to a JSON NoSQL database, or Elasticsearch type service. Recently I've been looking into GraphQL which could potentially be suitable too.


### Which part of your solution are you most pleased with? Why?

I'm pretty happy with the data flow, App passes props to children, they pass events back up and trigger various state mutations and Websocket messages which, when received, trigger further state changes. Even though I like this pattern, I'm slightly frustrated by the fact that some of the messages can be handled in a couple of lines directly inside the onmessage function, but others need to be passed off to more long-winded methods.

### How would you manage users going offline and online?

Well I think there are some options here, we could:

1. Log users out when they disconnect
2. Allow users to continue viewing and adding notes while offline, but disable editing, since this would be the largest portion to implement.
3. Allow users to continue using the app offline if they're already logged in, and save all the changes they make while offline to localStorage, which can then be merged with everyone else's data when a connection is available. To me, this would be the ideal solution, and the most work to implement.

Of course you could start with option 1, and iterate towards option 3. All three options would require more thought on the UI and how best to inform the user of their connection status and available actions.

### How would you manage allowing users to continue editing and adding notes when in offline mode and then syncing when they come back online?

Again a couple of possibilities I can think of here:

1. We could update localStorage with new notes and edits, then send a single request to merge all the clients' notes together when the user comes back online
2. We could store a history of the socket messages that should have been sent, and send them when a connection is available again.

### How would you manage conflicts between edits made by different users to the same note?

1. Lock a note that is currently being edited. For offline users, disable editing existing messages.
2. Allow multiple users to edit the same note at the same time, but inform all users apart from the first their edit will create a new note. Create a version of the note for each edit that isn't the initial edit. To do this we'd need to track when other users are editing notes, so we'd broadcast the data about which note is being edited via the Websocket.
3. Allow multiple users to edit the same note at the same time, and while they are editing, broadcast their edits with every keystroke (with a delay of ~300ms) to the other users. With this data being shared we can start working towards having multiple users editing the same note at the same time and seeing real-time updates with other users' edits, similar to how Google Docs allows multi-user editing.
