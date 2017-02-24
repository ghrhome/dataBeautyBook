# Flux For Stupid People

**TL;DR**As a stupid person, this is what I wish someone had told me when I struggled learning Flux. It's not straightforward, not well documented, and has many moving parts.

This is a follow-up to[ReactJS For Stupid People](http://blog.andrewray.me/reactjs-for-stupid-people/).

## Should I Use Flux? {#shouldiuseflux}

If your application deals with**dynamic data**then**yes**, you should probably use Flux.

If your application is just**static views**that don't share state, and you never save nor update data, then**no**, Flux won't give you any benefit.

## Why Flux? {#whyflux}

Humor me for a moment, because Flux is a moderately complicated idea. Why add complexity?

90% of iOS applications are**data in a table view.**The iOS toolkit has well defined views and data models that make application development easy.

On the Front End™ \(HTML, Javascript, CSS\), we don't even have that. Instead**we have a big problem:**No one knows how to structure a Front End™ application. I've worked in this industry for years, and "best practices" are never taught. Instead, "libraries" are taught. jQuery? Angular? Backbone? The real problem, data flow, still eludes us.

## What is Flux? {#whatisflux}

**Flux**is a word made up to describe "one way" data flow with very specific events and listeners. There's no official Flux library, but you'll need the[Flux Dispatcher](https://github.com/facebook/flux/blob/master/src/Dispatcher.js), and any Javascript[event library](http://notes.jetienne.com/2011/03/22/microeventjs.html).

The[official documentation](http://facebook.github.io/flux/docs/overview.html)is written like someone's stream of conciousness, and is a bad place to start. Once you get Flux in your head, though, it can help fill in the gaps.

Don't try to compare Flux to Model View Controller \(MVC\) architecture. Drawing parallels will only be confusing.

Let's dive in! I will**explain concepts in order**and build on them one at a time.

## 1. Your Views "Dispatch" "Actions" {#1yourviewsdispatchactions}

A "dispatcher" is essentially an event system with some extra rules added. It broadcasts events and registers callbacks. There is only ever**one, global dispatcher**. You should use the Facebook[Dispatcher Library](https://github.com/facebook/flux/blob/master/src/Dispatcher.js). It's very easy to instantiate:

```
var AppDispatcher = new Dispatcher();  
```

Let's say your application has a "new" button that adds an item to a list.

```
<button onClick={ this.createNewItem }>New Item</button>  
```

What happens on click? Your view**dispatches a very specific "action"**, with the action name and new item data:

```
createNewItem: function( evt ) {

    AppDispatcher.dispatch({
        actionName: 'new-item',
        newItem: { name: 'Marco' } // example data
    });

}
```

---

## 2. Your "Store" Responds to Dispatched Actions {#2yourstorerespondstodispatchedactions}

Like Flux, "store" is just a word Facebook made up. For our application, we need a specific collection of logic and data for the list. This describes our store. We'll call it a ListStore.

A store is a singleton, meaning you probably shouldn't declare it with`new`. There's only one instance of each store across your application:

```
/ Single object representing list data and logic
var ListStore = {

    // Actual collection of model data
    items: []

};
```

Your store then **responds to the dispatched action:**

```
var ListStore = …

// Tell the dispatcher we want to listen for *any*
// dispatched events
AppDispatcher.register( function( payload ) {

    switch( payload.actionName ) {

        // Do we know how to handle this action?
        case 'new-item':

            // We get to mutate data!
            ListStore.items.push( payload.newItem );
            break;

    }

}); 
```

This is traditionally how Flux handles dispatch callbacks. Each payload contains an action name and data. A switch statement determines if the store should respond to the action, and mutates data if it knows how to handle that action type.

![](http://blog.andrewray.me/content/images/2014/Oct/key.png#inline-block)Key Concept:**A store is not a model. A store**_**contains**_**models.**

![](http://blog.andrewray.me/content/images/2014/Oct/key.png#inline-block)Key concept: A store is the only thing in your application that knows how to update data.**This is the most important part of Flux.**The action we dispatched doesn't know how to add or remove items.

If, for example, a different part of your application needed to keep track of some images and their metadata, you'd make another store, and call it ImageStore. A store**represents a single "domain"**of your application. If your application is large, the domains will probably be obvious to you already. If your application is small, you probably only need one store. Generally, there is one store for one model type.

**Only your stores**are allowed to register dispatcher callbacks! Your views should never call`AppDispatcher.register`. The dispatcher only exists to send messages from views to stores. Your views will respond to a different kind of event.

## 3. Your Store Emits a "Change" Event {#3yourstoreemitsachangeevent}

We're almost there! Now that your data is definitely changed, we need to tell the world.

Your store emits an**event**, but**not using the dispatcher.**This is confusing, but it's the Flux way. Let's give our store the ability to trigger events. If you're using[MicroEvent.js](http://notes.jetienne.com/2011/03/22/microeventjs.html)this is easy:

```
MicroEvent.mixin( ListStore );  
```

Then let's trigger our change event:

```
case 'new-item':

            ListStore.items.push( payload.newItem );

            // Tell the world we changed!
            ListStore.trigger( 'change' );

            break;

```

![](http://blog.andrewray.me/content/images/2014/Oct/key.png#inline-block)Key Concept: We don't pass the newest item when we`trigger`. Our views only care that _something _changed. Let's keep following the data to understand why.

All Flux tells us how to do is manage data flow. It**doesn't answer**these questions:

* How do you load and save data to the server?
* How do you handle communication between components with no common parent?
* What events library should I use? Does it matter?
* Why hasn't Facebook released all of this as a library?
* Should I use a model layer like Backbone as the model in my store?

The answer to all of these questions is:**have fun!**

## PS: Don't Use forceUpdate {#psdontuseforceupdate}

I used`forceUpdate`for simplicty's sake. The correct way for your component to read store data is to**copy data into state**and read from`this.state`in the`render`function. You can see how this works in the[TodoMVC example](https://github.com/facebook/flux/tree/master/examples/flux-todomvc).

When the component first loads,[store data is copied](https://github.com/facebook/flux/blob/master/examples/flux-todomvc/js/components/TodoApp.react.js#L34)into`state`. When the store updates,[the data is re-copied](https://github.com/facebook/flux/blob/master/examples/flux-todomvc/js/components/TodoApp.react.js#L65)in its entirety. This is better, because internally,`forceUpdate`is synchronous, while`setState`is more efficient.

## That's It! {#thatsit}

For additional resources, check out the[Example Flux Application](https://github.com/facebook/flux/tree/master/examples/flux-todomvc)provided by Facebook. Hopefully after this article, the files in the`js/`folder will be easier to understand.

The[Flux Documentation](http://facebook.github.io/flux/docs/overview.html)contains some helpful nuggets, buried deep in layers of poorly accessible writing.

For an example of organizing components with Flux, see the follow up[ReactJS Controller View Pattern](http://blog.andrewray.me/the-reactjs-controller-view-pattern/).

