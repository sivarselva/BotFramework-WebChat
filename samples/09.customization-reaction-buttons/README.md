# Sample - Customize Web Chat with Reaction Buttons

This sample builds on top of the ideas expressed in sample [08.customization-user-highlighting](../08.customization-user-highlighting) and creates more involved middleware that will alert the bot to helpful and unhelpful messages via reaction buttons. This sample is implemented with React and makes changes that are based off of the [host with React sample](../03.a.host-with-react).

# Test out the hosted sample

-  [Try out MockBot](https://microsoft.github.io/BotFramework-WebChat/09.customization-reaction-buttons)

# How to run locally

-  Fork this repository
-  Navigate to `/Your-Local-WebChat/samples/09.customization-reaction-buttons` in command line
-  Run `npx serve` in the full-bundle directory
-  Browse to [http://localhost:5000/](http://localhost:5000/)

# Things to try out

-  Click the 👍👎 button next to activities from bot

# Code

> Jump to [completed code](#completed-code) to see the end-result `index.html`.

## Overview

We'll start by using the [host with React sample](../03.a.host-with-react) as our Web Chat React template.

In this sample, we will build a new React component that decorates the `activity` sent from the bot with two reaction buttons. Depending on which button is clicked, a new activity will be sent to the bot indicating which button the user has selected.

Let's start by building the React Component called `BotActivityDecorator`. It will have upvote and downvote buttons and render its children inside a `<div>` element. The bot activity will be rendered inside the inner container.

```jsx
const BotActivityDecorator = ({ children }) => {
   return (
      <div>
         <ul>
            <li>
               <button>👍</button>
            </li>
            <li>
               <button>👎</button>
            </li>
         </ul>
         <div>{children}</div>
      </div>
   );
};
```

Next, build our CSS and apply class names to our component.

```css
.botActivityDecorator {
   min-height: 60px;
   position: relative;
}

.botActivityDecorator .botActivityDecorator__content {
   padding-left: 40px;
}

.botActivityDecorator .botActivityDecorator__buttonBar {
   list-style-type: none;
   margin: 0 0 0 10px;
   padding: 0;
   position: absolute;
}

.botActivityDecorator .botActivityDecorator__buttonBar .botActivityDecorator__button {
   background: White;
   border: solid 1px #E6E6E6;
   margin-bottom: 2px;
   padding: 2px 5px 5px;
}
```

Then, apply the style sheet to our React component.

```diff
   const BotActivityDecorator = ({ children }) => {
      return (
-        <div>
+        <div className="botActivityDecorator">
-           <ul>
+           <ul className="botActivityDecorator__buttonBar">
               <li>
-                 <button>👍</button>
+                 <button className="botActivityDecorator__button">👍</button>
               </li>
               <li>
-                 <button>👎</button>
+                 <button className="botActivityDecorator__button">👎</button>
               </li>
            </ul>
-           <div>{children}</div>
+           <div className="botActivityDecorator__content">{children}</div>
         </div>
      );
   };
```

Then, add business logic to the component:

- When the upvote button is clicked, send a post back activity to the bot with the activity ID and `helpful` of `1`.
- When the downvote button is clicked, send a post back activity with `helpful` of `-1`.

The `sendPostBack` function will be retrieve from Web Chat hooks via `useSendPostback` function.

```diff
-  const { ReactWebChat } = window.WebChat;
+  const {
+     hooks: { usePostActivity },
+     ReactWebChat
+  } = window.WebChat;
+
+  const { useCallback } = window.React;

-  const BotActivityDecorator = ({ children }) => {
+  const BotActivityDecorator = ({ activityID, children }) => {
+     const postActivity = usePostActivity();
+
+     const handleDownvoteButton = useCallback(() => {
+        postActivity({
+           type: 'messageReaction',
+           reactionsAdded: [{ activityID, helpful: -1 }]
+        });
+     }, [activityID, postActivity]);
+
+     const handleUpvoteButton = useCallback(() => {
+        postActivity({
+           type: 'messageReaction',
+           reactionsAdded: [{ activityID, helpful: 1 }]
+        });
+     }, [activityID, postActivity]);

      return (
         <div className="botActivityDecorator">
            <ul className="botActivityDecorator__buttonBar">
               <li>
-                 <button className="botActivityDecorator__button">👍</button>
+                 <button className="botActivityDecorator__button" onClick={handleUpvoteButton}>👍</button>
               </li>
               <li>
-                 <button className="botActivityDecorator__button">👎</button>
+                 <button className="botActivityDecorator__button" onClick={handleDownvoteButton}>👎</button>
               </li>
            </ul>
            <div className="botActivityDecorator__content">{children}</div>
         </div>
      );
   };
```

Next let's build the `activityMiddleware` that will filter which activities are being rendered with the new component, `BotActivityDecorator`.

```jsx
const activityMiddleware = () => next => card => {
   if (card.activity.from.role === 'bot') {
      return children => (
         <BotActivityDecorator activityID={card.activity.id} key={card.activity.id}>
            {next(card)(children)}
         </BotActivityDecorator>
      );
   } else {
      return next(card);
   }
};
```

Make sure `activityMiddleware` is passed into the the Web Chat component, and that's it.

```diff
   window.ReactDOM.render(
      <ReactWebChat
+        activityMiddleware={activityMiddleware}
         directLine={window.WebChat.createDirectLine({ token })}
      />,
      document.getElementById('webchat')
   );
```

## Completed code

```diff
   <!DOCTYPE html>
   <html lang="en-US">

      <head>
         <title>Web Chat: Decorates bot activity with upvote and downvote buttons</title>
         <meta name="viewport" content="width=device-width, initial-scale=1.0" />
         <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
         <script src="https://unpkg.com/react@16.8.6/umd/react.development.js"></script>
         <script src="https://unpkg.com/react-dom@16.8.6/umd/react-dom.development.js"></script>
         <script src="https://unpkg.com/react-redux@7.1.0/dist/react-redux.min.js"></script>
         <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
         <style>
            html,
            body {
               height: 100%;
            }

            body {
               margin: 0;
            }

+           #webchat {
+              height: 100%;
+              width: 100%;
+           }
+
+           .botActivityDecorator {
+              min-height: 60px;
+              position: relative;
+           }
+
+           .botActivityDecorator .botActivityDecorator__content {
+              padding-left: 40px;
+           }
+
+           .botActivityDecorator .botActivityDecorator__buttonBar {
+              list-style-type: none;
+              margin: 0 0 0 10px;
+              padding: 0;
+              position: absolute;
+           }
+
+           .botActivityDecorator .botActivityDecorator__buttonBar .botActivityDecorator__button {
+              background: White;
+              border: solid 1px #E6E6E6;
+              margin-bottom: 2px;
+              padding: 2px 5px 5px;
+           }
         </style>
      </head>

      <body>
         <div id="webchat" role="main"></div>
         <script type="text/babel">
            (async function () {
               'use strict';

               const {
+                 hooks: { usePostActivity },
                  ReactWebChat
               } = window.WebChat;

+              const { useCallback } = window.React;

+              const BotActivityDecorator = ({ activityID, children }) => {
+                 const postActivity = usePostActivity();
+
+                 const handleDownvoteButton = useCallback(() => {
+                    postActivity({
+                       type: 'messageReaction',
+                       reactionsAdded: [{ activityID, helpful: -1 }]
+                    });
+                 }, [activityID, postActivity]);
+
+                 const handleUpvoteButton = useCallback(() => {
+                    postActivity({
+                       type: 'messageReaction',
+                       reactionsAdded: [{ activityID, helpful: 1 }]
+                    });
+                 }, [activityID, postActivity]);
+
+                 return (
+                    <div className="botActivityDecorator">
+                       <ul className="botActivityDecorator__buttonBar">
+                          <li>
+                             <button className="botActivityDecorator__button" onClick={handleUpvoteButton}>👍</button>
+                          </li>
+                          <li>
+                             <button className="botActivityDecorator__button" onClick={handleDownvoteButton}>👎</button>
+                          </li>
+                       </ul>
+                       <div className="botActivityDecorator__content">{children}</div>
+                    </div>
+                 );
+              };

               const res = await fetch('https://webchat-mockbot.azurewebsites.net/directline/token', { method: 'POST' });
               const { token } = await res.json();
+              const activityMiddleware = () => next => card => {
+                 if (card.activity.from.role === 'bot') {
+                    return children => (
+                       <BotActivityDecorator key={card.activity.id} activityID={card.activity.id}>
+                          {next(card)(children)}
+                       </BotActivityDecorator>
+                    );
+                 }

                  return next(card);
               };

               window.ReactDOM.render(
                  <ReactWebChat
+                    activityMiddleware={activityMiddleware}
                     directLine={window.WebChat.createDirectLine({ token })}
                  />,
                  document.getElementById('webchat')
               );

               document.querySelector('#webchat > *').focus();
            })().catch(err => console.error(err));
         </script>
      </body>

   </html>
```

# Further reading

[User highlighting bot](https://microsoft.github.io/BotFramework-WebChat/08.customization-user-highlighting) | [(User highlighting source code)](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/08.customization-user-highlighting)

[Card components bot](https://microsoft.github.io/BotFramework-WebChat/10.a.customization-card-components) | [(Card components source code)](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/10.a.customization-card-components)

## Full list of Web Chat hosted samples

View the list of [available Web Chat samples](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples)
