---
layout: site
title: App Navigation with a State Machine
permalink: app-navigation-with-state-machine
contributors: [marco_piccolino]
level: intermediate
---
[DRAFT] Level: {{page.level}}

*First published: 2015-09-23;* *Contributors:* {% for contributor in page.contributors %} -- {{ site.authors[contributor].name }} {% endfor %}

**Problem:** you find it hard to make changes to your app which affect navigation structure; you cannot describe your app's navigation structure easily.

**Solution:** a [Declarative State Machine](http://doc.qt.io/qt-5/qmlstatemachine.html) helps keeping app structure manageable and explainable.

Credit for this technique goes to [Marius Bugge Monsen](https://github.com/mariusbu) from [Cutehacks](http://cutehacks.com). You can find his slides for a C++ implementation [here](http://www.slideshare.net/mariusbu/the-anatomy-of-real-world-apps).
The implementation described in this tutorial is QML-based.

##Your app is currently a mess
If you are confortable with your app's current structure, this tutorial is not needed.
If, however, you have difficulties describing such structure in the first place, or the way you have implemented it, it's definitely time to nail it.  

You might have found that for your specific use case, a [StackView](http://doc.qt.io/qt-5/qml-qtquick-controls-stackview.html#using-stackview-in-an-application) is not well-suited, as your *pages* can be navigated in a very non-linear fashion.
Even when a StackView works well, you might still want to have a better grasp of the relationship between your app's pages, and between pages and content.
So what can you do?

### A digital touristic guide

Suppose you are implementing a touristic guide with the following components:

* a home page (`Home`) with a carousel of content highlights, a menu listing available content sections organized by topic (e.g. *sports*, *taste* etc.), and a utility button that opens an overlay dialog (e.g. `Credits`).
* a selector page (`Subsections`) where you can select a type of content (say *events* and *places*) for each of the topic in `Home`'s menu.
* a content viewer (`Viewer`), which is made of a textual view (`Textual`) and a map (`Map`).
The textual view can be either a list (`List`) or a card (`Card`) depending on whether the content is an overview of several content items or rather the details of a specific content item. The map displays two different layers with several content items at once (`ListLayer`) or just one (`CardLayer`) based on the same criterion.

You can thus devise the following QML components, and divide them based on whether they represent pages, panes or blocks (or any other naming convention that you follow in your design system. Because you *do* follow one, don't you?)

<pre>
/blocks/Card.qml
/blocks/CardLayer.qml
/blocks/List.qml
/blocks/ListLayer.qml
/panes/Credits.qml
/panes/Map.qml
/panes/Textual.qml
/pages/Home.qml
/pages/Subsections.qml
/pages/Viewer.qml
</pre>

## Define your app's navigation structure

Now you have to figure out the way these components will be accessed from each other, i.e. lay down your app's navigation structure.
By doing this and following the description above, you'll find out that from `Home` you can:

* open the `Credits` dialog
* go to the `Subsections` page from the sections menu
* go to the `Viewer` page from the highlights carousel. The `Viewer`'s `Textual` pane should display the `Card` view with details about a specific content item, while the `Map` should display the `CardLayer` which shows a single content item.

Similarly, from the `Subsections` page you can:

* go to the `Viewer`, and this time it should display an overview, i.e. a `List` of content items, and a `ListLayer` in the map.

For the time being, we will ignore any additional navigation path (including returning to `Home`) to focus on how to model the aforementioned page relationships.

## Model the navigation structure

You can think of your application as being at any given time in one out of several different *states*, each one keeping track of the current *active page*, together with the page's internal *active components*. If you have different data sources that can show up on the same page, these might as well be modeled as different states (think *active model*).
Your app's navigation structure should then capture how these states are connected to each other, i.e. whether a specific state should take to another state or not.

Looking at the paths defined above, you might decide to have:

* a `homeState`, which in turn will have two substates: `homePlainState` and `homeCreditsState`
* a `subsectionsState`
* a `viewerState`, which in turn will have two substates: `overviewState` and `detailsState`

From the `HomeState` one can enter the `SubsectionsState` via `Home`'s sections' menu, as well as enter the `DetailsState` via the highlights carousel.
From the `SubsectionsState` one can enter the `OverviewState` via a menu in `Subsections`.

Keeping your states separate from your views is a valuable thing, as when you change anything graphic in your app for aesthetic, performance or functional reasons, your navigation structure won't be affected. You might think of this abstraction as having routers or controllers in an MVC-style pattern.

## Enter the State Machine

The declarative state machine (short **DSM**) is the mechanism that allows you to express the aforementioned *state graph* in QML.
It does this by virtue of mainly three components: [StateMachine](http://doc.qt.io/qt-5/qml-qtqml-statemachine-statemachine.html), [State](http://doc.qt.io/qt-5/qml-qtqml-statemachine-state.html) and [SignalTransition](http://doc.qt.io/qt-5/qml-qtqml-statemachine-signaltransition.html). While the former two are pretty self-explanatory, the latter just allows the definition of the signal in your application which triggers a state transition. There are a few more useful components in the [QtQml.StateMachine module](http://doc.qt.io/qt-5/qmlstatemachine.html) that are worth looking at, but you won't need them here.

As a note, be careful not to confuse `State` from QtQtml.StateMachine with `State` from QtQuick. You might want to always namespace the QtQml.StateMachine module as `DSM`.

The above mentioned states can be expressed with a StateMachine as follows:

<pre>
import QtQml.StateMachine as DSM

DSM.StateMachine {
  DSM.State {
    id: homeState
    initialState: homePlainState
    DSM.SignalTransition {targetState: subsectionsState; signal: home.sectionsItemClicked}
    DSM.SignalTransition {targetState: detailsState; signal: home.carouselItemClicked}
    DSM.State {
      id: homePlainState
      DSM.SignalTransition {targetState: homeCreditsState; signal: home.openCreditsClicked}
    }
    DSM.State {
      id: homeCreditsState
      DSM.SignalTransition {targetState: homePlainState; signal: home.closeCreditsClicked}
    }
  }
  DSM.State {
    id: subsectionsState
    DSM.SignalTransition {targetState: overviewState; signal: subsections.menuItemClicked}
  }
  DSM.State {
    id: viewerState
    initialState: overviewState
    DSM.State {
      id: overviewState
    }
    DSM.State {
      id: detailsState
    }
  }
}
</pre>

## Wiring the State Machine to your pages

Your navigation structure is now in place, but you need to define what happens with the views (and possibly the data) when states are entered and/or exited. This can be achieved by using two methods of `State`: `onEntered()` and `onExited()`.

For example, you might want to create all your pages and components statically in your `main.qml`, and show/hide them, or move them out of sight, when their related states are entered/exited. In order to do that, you need to associate a state to a page (e.g. `homeState` to `home`), or to a page with a specific internal state (e.g. `detailsState` to `viewer` with its `Card` and `CardLayer` components active).
There are many ways to accomplish this. One of them consists in:

1. defining one or more properties in your page that will be changed when entering and exiting its associated DSM `State`. In their turn, these properties will then govern all visual/data changes to the page by using QtQuick's `State` or some other mechanism.
2. extending the `State` component to hold a reference to its associated page

Here is a simple example. You will first create a page template component (`PageTpl`) with an `isActive` property:

<pre>
Rectangle {
    id: root
    property bool isActive: false
    visible: false
    states: [
        State {
            name: "active"
            when: root.isActive
            PropertyChanges {
                target:root
                visible:true
            }
        }
    ]
}
</pre>

You will then extend the `State` component into a `PageState` to hold the reference to an item of type `PageTpl`, and to modify its `isActive` property when entering and exiting the state:
<pre>
DSM.State {
    id: root
    property PageTpl page
    onEntered: {page.isActive = true}
    onExited: {page.isActive = false}
}
</pre>

Thus, if your `home` and `subsections` pages are an instance of `PageTpl`, you can revisit their entries in the state machine as follows:

<pre>
PageState {
  id: homeState
  initialState: homePlainState
  DSM.SignalTransition {targetState: subsectionsState; signal: home.sectionsItemClicked}
...
PageState {
  id: subsectionsState
  DSM.SignalTransition {targetState: overviewState; signal: subsections.menuItemClicked}
}
</pre>

If your `home` and `subsections` pages extend PageTpl, when you trigger `home.sectionsItemClicked` via a button or some other mechanism, the `home` will be hidden and `subsections` will be shown (needless to say, make sure you give them a size!).

The same concept can be applied to substates (e.g. `overviewState`). You might either extend them as `PageSubState` and wire their properties to a specific property in the `PageTpl` which governs showing/hiding related page components. Since DSM substates inherit behaviour from their parent states (in this case, the `PageStates`), the correct page will be displayed automatically.

## Wrapping it up

By using DSM meaningfully, you should be able to keep a cohesive view about your app's structure and navigation, and perhaps add deep-linking features and more even for scenarios where a `StackView` seems not enough.

If you have any suggestions for improvements on the above technique, come join the discussion at [QtMob/Slack](https://qtmob.slack.com) (ask for an [invite](mailto:marco.a.piccolino@gmail.com) first).
