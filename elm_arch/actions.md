# Actions

We created an `update` function.

```elm
update : () -> Model -> Model
update _ model =
  { model | count = model.count + 1 }
```

As is, this `update` function can work with only one input signal i.e. `Mouse.clicks` which produces the expected `()` input.

But what we really want is for `update` to be able to handle different actions in our application. For this we will introduce the concept of __actions__ in Elm. We will use __Algebraic data types (ADT)__ for this.

```elm
type Action
  = NoOp
  | Increase
```

Here we have an `Action` that can be either be `NoOp` (Do nothing) or `Increase` (Increase the counter).

## Adding actions

Let's refactor the application to use actions:

```elm
module Main (..) where

import Html
import Mouse


type alias Model =
  { count : Int }


type Action
  = NoOp
  | Increase


initialModel : Model
initialModel =
  { count = 0 }


view : Model -> Html.Html
view model =
  Html.text (toString model.count)


update : Action -> Model -> Model
update action model =
  case action of
    Increase ->
      { model | count = model.count + 1 }

    NoOp ->
      model


actionSignal : Signal.Signal Action
actionSignal =
  Signal.map (\_ -> Increase) Mouse.clicks


modelSignal : Signal.Signal Model
modelSignal =
  Signal.foldp update initialModel actionSignal


main : Signal.Signal Html.Html
main =
  Signal.map view modelSignal
```

<https://github.com/sporto/elm-tutorial-assets/blob/master/code/elm_arch/Actions.elm>

#### actions

We added `type Action ...` as described before.

#### update

```elm
update : Action -> Model -> Model
update action model =
  case action of
    Increase ->
      { model | count = model.count + 1 }

    NoOp ->
      model
```

The `update` function now takes an `Action` as first argument, instead of `()`.

We added a `case` to deal with different actions. This case checks for the type of action and acts accordingly.

The Increase case returns a new model with it's count property incremented, same as before.

The NoOp case just returns the same model.

### actionSignal

```elm
actionSignal : Signal.Signal Action
actionSignal =
  Signal.map (\_ -> Increase) Mouse.clicks
```

Here we introduce a new signal as an intermediate step before the `modelSignal`. This signal listens for mouse clicks and maps them to an action, in this case `Increase`.

### modelSignal

```elm
modelSignal : Signal.Signal Model
modelSignal =
  Signal.foldp update initialModel actionSignal
```

The model signal now takes the `actionSignal` as input and calls update with the action.

## More actions

To understand the beauty of this, imagine what will happen if you have several actions that can happen in your application e.g. mouse clicks, key up, key downs, etc. By converting everything to __actions__ our application can handle all these cases.

Here is an application that responds to both mouse clicks and key presses:

```elm
module Main (..) where

import Html
import Mouse
import Keyboard


type alias Model =
  { count : Int }


type Action
  = NoOp
  | MouseClick
  | KeyPress


initialModel : Model
initialModel =
  { count = 0 }


view : Model -> Html.Html
view model =
  Html.text (toString model.count)


update : Action -> Model -> Model
update action model =
  case action of
    MouseClick ->
      { model | count = model.count + 1 }

    KeyPress ->
      { model | count = model.count - 1 }

    NoOp ->
      model


mouseClickSignal : Signal.Signal Action
mouseClickSignal =
  Signal.map (\_ -> MouseClick) Mouse.clicks


keyPressSignal : Signal.Signal Action
keyPressSignal =
  Signal.map (\_ -> KeyPress) Keyboard.presses


actionSignal : Signal.Signal Action
actionSignal =
  Signal.merge mouseClickSignal keyPressSignal


modelSignal : Signal.Signal Model
modelSignal =
  Signal.foldp update initialModel actionSignal


main : Signal.Signal Html.Html
main =
  Signal.map view modelSignal
```

<https://github.com/sporto/elm-tutorial-assets/blob/master/code/elm_arch/ActionsMultiple.elm>

Every mouse click increases the count, every key press decreases the count. Note how we __merge__ the signals (mouseClickSignal and keyPressSignal) into one, as they both are of type `Signal.Signal Action` our folp can handle them.

## Sending values

You can send a payload with your actions:

```elm
module Main (..) where

import Html
import Mouse


type alias Model =
  { count : Int }


type Action
  = NoOp
  | MouseClick Int


initialModel : Model
initialModel =
  { count = 0 }


view : Model -> Html.Html
view model =
  Html.text (toString model.count)


update : Action -> Model -> Model
update action model =
  case action of
    MouseClick amount ->
      { model | count = model.count + amount }

    NoOp ->
      model


mouseClickSignal : Signal.Signal Action
mouseClickSignal =
  Signal.map (\_ -> MouseClick 2) Mouse.clicks


modelSignal : Signal.Signal Model
modelSignal =
  Signal.foldp update initialModel mouseClickSignal


main : Signal.Signal Html.Html
main =
  Signal.map view modelSignal
```

<https://github.com/sporto/elm-tutorial-assets/blob/master/code/elm_arch/ActionsWithPayload.elm>

Note the new __actions__ declaration:

```
MouseClick Int
```

This is saying that the `MouseClick` action should be accompanied with an integer.

Then in mouseClickSignal:

```elm
mouseClickSignal =
  Signal.map (\_ -> MouseClick 2) Mouse.clicks
```

We map mouse clicks to the `MouseClick` action with `2` as its payload.

Finally, `update` uses __pattern matching__ in the case expression to extract the payload:

```elm
case action of
  MouseClick amount ->
    { model | count = model.count + amount }
```
