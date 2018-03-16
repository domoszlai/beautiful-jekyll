---
layout: post
title: Stateful event handling in PureScript 
subtitle: the big-canvas experiment
tags: [functional programming, purescript, bigcanvas]
show-avatar: true
description: PureScript is a Haskell like purely functional programming language that compiles directly to JavaScript. In this post I explain one way to handle async DOM events in a stateful way using coroutines.
excerpt: PureScript is a Haskell like purely functional programming language that compiles directly to JavaScript. In this post I explain one way to handle async DOM events in a stateful way using coroutines.
image: /img/purescript_logo.png
gh-repo: domoszlai/purescript-bigcanvas
gh-badge: [star, fork, follow] 
---

## Overview

In this post I'd like to share my experiments with PureScript. For one of my ongoing projects I needed to develop a JavaScript component,
a kind of [infinite canvas](https://en.wikipedia.org/wiki/Infinite_canvas), that enables to scroll and zoom in/out on arbitrary big 
drawings.

I wanted my app to run in a browser, but I did not want to develop JavaScript directly. As I'm pretty much into functional programming,
I looked for a *strict* functional language that directly compiles to JavaScript; I finally picked [PureScript](http://www.purescript.org/) to give a try.
There were other attractive alternatives, e.g. [Elm](http://elm-lang.org/). I mainly selected PureScript, because it is very similar to [Haskell](https://www.haskell.org/), and seemed that there is a reasonable sized community around it.

## The experiment

Before starting to work on a full-fledged library, I wanted a proof of concept. I know that IO can be a real pain with pure functional languages,
so the idea was to develop a simple app that handles some events in a stateful way. Finally, I chose a basic task with a canvas:

- catch `mousemove` events of the canvas and store the cursor position in the state
- in a [`requestAnimationFrame`](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) callback, take current cursor position from the state and draw the coordinates to the canvas

Simple enough, still covers most of the basic functionality I need for the library.

## PureScript

PureScript is a strict, purely functional programming language which compiles directly to JavaScript. Its syntax and type system is very similar to as of Haskell with some notable differences (as far as we are concerned here):

- Instead of an IO monad, PureScript handles [effects](https://github.com/purescript/documentation/blob/master/guides/Eff.md)
- PureScript has an extensive syntax for [records](https://github.com/purescript/documentation/blob/master/language/Records.md) and it implements [row polymorphism](https://leanpub.com/purescript/read#leanpub-auto-record-patterns-and-row-polymorphism)

Of course, there are also zillion of tiny, obscure differences between Haskell and PureScript what makes programming in it very challenging in the beginning :)

PureScript is easy to [install](https://github.com/purescript/documentation/blob/master/guides/Getting-Started.md) using [Node.js and npm](https://docs.npmjs.com/getting-started/installing-node), and it has many official and third-party modules available that can be easily installed by [bower](https://bower.io/).

PureScript apps looks similarly to Haskell ones, only the types seems a bit more involved, because of the effect system in use
and, because using the `forall` quantifier is mandatory:

```haskell
module Main where

import Prelude (Unit)

import Control.Monad.Eff (Eff)
import Control.Monad.Eff.Console (CONSOLE, log)

main :: forall e. Eff (console :: CONSOLE | e) Unit
main = log "Hello world"
```

## Implementation

In the following I try to explain the main idea behind the implementation, and I'll
skip most of the details, boilerplate code, imports, etc. The whole source code can be found in
the BigCanvas github repository at [https://github.com/domoszlai/purescript-bigcanvas](https://github.com/domoszlai/purescript-bigcanvas).

Finally, you will need a minimal piece of HTML code to run the sample:

```html
<html>
  <body>
    <canvas id="canvas"></canvas>
    <script type="text/javascript" src="bigcanvas.js"></script>
  </body>
</html>
```

For the compiling to browserified JavaScript, we will use the following command:

```sh
pulp browserify --optimise --to bigcanvas.js
```

### Reading the DOM

First of all, we need access to the HTML DOM to be able to attach event handlers to the canvas and 
use the [Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API). Looking up a DOM element
can be done by the `purescript-dom` module:

```haskell
main :: forall e. Eff (dom :: DOM, console :: CONSOLE | e) Unit
main = do
   documentType <- document =<< window
   -- returns type Maybe Element 
   mbCanvas <- getElementById (ElementId "canvas") 
                              (htmlDocumentToNonElementParentNode documentType)
   case mbCanvas of
      Nothing -> log "no canvas found"
      Just canvas -> do ...
```

Unfortunately, `purescript-dom` does not give access to the Canvas API. For that we need the `purescript-canvas` module, which
provides a different mechanism to look up a canvas element:
 
```haskell
main :: forall e. Eff (canvas :: CANVAS, console :: CONSOLE | e) Unit
main = do
   -- returns type Maybe CanvasElement 
   mbCanvas <- getCanvasElementById "canvas"
   case mbCanvas of
      Nothing -> log "no canvas found"
      Just canvas -> do ...
```

Because we need both, we already face a little bit of drama here. The easiest way I could find to solve this problem, involves 
`unsafeCoerce`. Not nice, but works perfectly...

```haskell
canvasToHTMLElement :: CanvasElement -> HTMLElement
canvasToHTMLElement = unsafeCoerce
```

### Having a state

The default way to handle effects are the [`Eff`](https://github.com/purescript/purescript-eff), and its asynchronous extension,
the [`Aff`](https://github.com/slamdata/purescript-aff) monads (we will use the `Aff` monad whenever it is possible, because it helps dealing with async events easier). Of course, none of these provides a way to maintain a pure state for the application, so we will try to use the good, old state monad transformer:

```haskell
type BigCanvasState = 
            { canvas  :: CanvasElement
            , context :: Context2D 
            , x       :: Int
            , y       :: Int 
            }
      
type BigCanvasT e = StateT BigCanvasState 
   (Aff (canvas :: CANVAS, dom :: DOM, console :: CONSOLE, avar :: AVAR | e))
```

### Handling DOM events

Next step, adding event handlers. Fortunately, `purescript-dom` enables it by 

```haskell
addEventListener :: forall eff. EventType -> EventListener (dom :: DOM | eff) 
   -> Boolean -> EventTarget -> Eff (dom :: DOM | eff) Unit
```

and

```haskell
requestAnimationFrame :: forall eff. Eff (dom :: DOM | eff) Unit -> Window 
   -> Eff (dom :: DOM | eff) RequestAnimationFrameId
```

But, wait a minute, they run in the `Eff` monad... That's too bad :(

We need to be very smart here...

What follows I will not give much explanation; to be able to fully understand you need to spend a couple of 
hours with [pursuit](https://pursuit.purescript.org/) anyway... The gist is that we create a kind of event loop,
utilizing [coroutines](https://www.schoolofhaskell.com/school/to-infinity-and-beyond/pick-of-the-week/coroutines-for-streaming/part-2-coroutines). 
The necessary functions can be found in module `purescript-aff-coroutines`. The event loop turns the async DOM events into a sequence of events of type `BigCanvasEvent` and passes it to the event handler function `onEvent`:

```haskell
onEvent ::  forall e. BigCanvasEvent -> BigCanvasT e Unit
onEvent event = ... -- explained later

data BigCanvasEvent 
        = EMouseDown Event
        | EMouseMove Event
        | EDraw 
```

The `setupEventLoop` function handles the `mousemove` and `mousedown` events of the canvas
and also generates an infinite stream of `EDraw` events by continuously asking for `AnimationFrame`s from the browser.

```haskell
setupEventLoop :: forall e. EventTarget -> BigCanvasT e Unit
setupEventLoop target = runProcess $ consumer `pullFrom` producer
   where
   producer :: Producer BigCanvasEvent (BigCanvasT e) Unit
   producer = produce' \emit -> do
      -- Listen to events 
      addEventListener (EventType "mousemove") 
              (eventListener (emit <<< Left <<< EMouseMove)) false target  
      addEventListener (EventType "mousedown") 
              (eventListener (emit <<< Left <<< EMouseDown)) false target 

      -- Generate an infinite stream of `EDraw` events
      launchAff_ (forever $ waitForAnimationFrame *> liftEff (emit (Left EDraw)))

   consumer :: Consumer BigCanvasEvent (BigCanvasT e) Unit
   consumer = forever $ lift <<< onEvent =<< await
```

Drawing to the canvas at the proper point of time is very crucial for the quality of animation (I mean panning, zooming).
Web browsers provide the [`requestAnimationFrame()`](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) method to register a call back which invoked just before the next repaint. It is a tricky call, though, as it calls back only once, so it must be requested continuously. This is done with the help
of the `waitForAnimationFrame` function:

```haskell
waitForAnimationFrame :: forall e. Aff (dom :: DOM, console :: CONSOLE | e) Unit
waitForAnimationFrame = 
   makeAff \emit -> do 
      win <- window
      _   <- requestAnimationFrame (emit $ Right unit) win
      pure mempty 
```

{% include note.html content="I'm not sure that it is actually implemented properly. The [documentation](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) says that the next `requestAnimationFrame()` should be called from the callback itself. It is currently called again after the callback returned. If you have a better idea to do that, please comment it below. " %}

### Handling BigCanvasEvents

The last step is finally straightforward. Save coordinates when mouse moves, display them when `EDraw` comes:

```haskell
onEvent ::  forall e. BigCanvasEvent -> BigCanvasT e Unit
onEvent (EMouseDown _) = pure unit -- do nothing for now

onEvent (EMouseMove e) =  do
    s <- get
    -- JavaScript does not make it easy to find the actual coordinates
    rect <- liftEff $ getBoundingClientRect (canvasToHTMLElement s.canvas)
    withMouseEvent e (\me -> modify \s -> s {
      x = clientX me - ceil rect.left, y = clientY me - ceil rect.top })

onEvent (EDraw) = do
    s <- get
    liftEff $ log "draw"
    -- OK, it should not be 100, 100 ...
    _ <- liftEff $ clearRect s.context {x: 0.0, y: 0.0, w: 100.0, h: 100.0}
    _ <- liftEff $ strokeText s.context (show s.x <> ", " <> show s.y) 10.0 10.0 
    pure unit
```

And that's all! If you are interested in the missing details, all the source code can be found
in the [BigCanvas github repository](https://github.com/domoszlai/purescript-bigcanvas).

I hope you found the post useful. Please comment below if you have any idea how to deal better with async events in a stateful way.






