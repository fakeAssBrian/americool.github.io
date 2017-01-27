---
layout: post
title:  "React-Native: Intro and Getting Basic Navigation Working"
date:   2017-01-06 14:25:00
categories: react, react-native, Navigation
---

*Note: Most of my writing up until this point has been a bunch of unorganized, inconsistent, nonsense. However, rather than try to worry about backlogging and trying to recall all the different things I've worked on up until this point, I figure it make sense to just jump in head first with what's happening now.*

  In the last year or so I've been learning to code (*primarily focused on web-development*) and one thing that's always notably alluded me, or at least seemed like a large demanding goal outside my skill-set is mobile-app-development. This has been endlessly frustrating, in no small part because a large swath of current projects, (*not to mention every idea that I have*), usually requires (or at least eventually needs) some sort of mobile support. The challenge of learning a fundamentally different language (or two, given that development for IOS and Android were considered separate for a time,) seemed completely overwhelming. That was, until I learned about React, and more specifically React-Native. I won't get carried away with explaining what either of them are - as I assume most people know more than me, (and you know if you don't - google,) but the important thing to note is that in addition to being cross-platform, React-Native is basically still just good old-fashioned JavaScript.

  In this post I mostly want to focus on the first issue that proved to be a  challenge for me, which is navigation (*i.e. being able to switch to different pages/screens/scenes/tiles/whatever-have-you within the mobile app rather than have a singular page/display*), but first a couple introductory notes:

  React and React-Native, while not incredibly difficult, are not super easy to setup either. If you are totally new to both I would suggest starting with [This React Tutorial]. It's a little out of date and missing some ES6-fancy-pants-new-JS conventions, but it's a good way to get an idea of how the library works and how to get everything set up...plus it's free. After that I'd also suggest [This Udemy React-Native Course], it's fairly long, and though not free, it's usually on sale for like $10-$15. It's how I learned a lot of react-native and how I eventually solved the challenge of getting my App to properly navigate.

  Now let's jump ahead and imagine you've got a basic static React-Native app running on your IOS simulator. Here's an example of mine:

  {: .center }
  ![vndr_screenshot](/img/navigation_vndr_screenshot.png)

  I'm working on what I'd call a "Vendor Locator", but we can loosely think of it as an app that will display a list of items with descriptions and their respective distance. Note this data is currently hard-coded into firebase, and isn't actually pulling real locations for the time being. We've got a ListView, a decent wrapper/card layout, and a few buttons. But let's say I want these buttons to actually take the user places. Now what? Well I first looked at [React-Native's Navigation Documentation]. Now, while I suspect this in itself might present a solution for a more experienced developer, for someone like me who is less experienced, also *really* hates combing through documentation, learns better by functional examples, and tends to be a little ADD, this can leave you feeling confused and at the bottom of the rabbit hole of self doubt and the imposter syndrome. Luckily, that Udemy course I linked to above has a pretty practical solution. As the author of the course notes, there really isn't a clear or fundamental nav system for react-native, but he offers a nice walk through for one approach, and I've been pretty happy with it so far. The downside I suppose is that he doesn't discuss it till near the end of the course. So I'll offer the basic bones of it here:

  First you need to npm install [react-native-router-flux]. This is the tool I used to make it happen. As I understand it, it's cross-platform, and I found it fairly simple, logical, and straight forward. Once you've got it running you really only need to modify a few files to make everything work. First in your App.js (or index.whatever.js - my App file just points at this,) you want to replace all your code with something like this.

  {% highlight javascript %}
  import React from 'react';
  import Router from './Router';


    const App = () => {
      return (
        <Router />
      );
    };

  export default App;
  {% endhighlight %}

  I would argue you don't really need anything else, (react router flux provides a pretty decent default header,) just make sure the various views you want to display/navigate between are separate component files.  Next you'll want to actually make the router file, here's what mine looks like:

  {% highlight javascript %}

  import React from 'react';
  import { Scene, Router } from 'react-native-router-flux';
  import VndrList from './VndrList';
  import MapView from './MapView';
  import Form from './Form';

  const RouterComponent = () => {
    return (
      <Router>
        <Scene key="list">
          <Scene key="VndrList" component={VndrList} title="List View" />
        </Scene>

        <Scene key="map">
          <Scene key="MapView" component={MapView} title="Map View" />
        </Scene>

        <Scene key="form">
          <Scene key="Form" component={Form} title="Add Vendor" />
        </Scene>

      </Router>
    );
  };

  export default RouterComponent;

  {% endhighlight %}

  Okay, there are a few things to note here. As we can see we create a router component and start our return with a <Router> wrapper. Ignore the first <Scene key> tag for a moment (*I'll explain in a second*) and let's look at the more detailed Scene key nested below it. This is the important part. We pass three things in here, the first (key) references the name we use to define the path when we push the user over to that screen. The component points to one of the imported component views above, stating what to render once the user is pushed to a new screen. Lastly the title appears in the default header provided by the navigator. I should also note, that while by default, the initial or landing "page" is defined in a top down order, you can pass an 'initial' property to any of these scenes to make it the default view/landing.

  Now, what about those wrapping Scene key elements that don't have a component or title options?  Well these are here because, by default if a user navigates from one scene to the next inside the router component. The navigator will offer a "back button"  on the left side of the header that the user can click to go back to the previous scene. There a number of instances where we wouldn't want this functionality (*say after entering data and submitting a form*). Wrapping the Scene Keys in outer Scene Key components prevents this. (Though it should be noted that if we nested two sibling components inside an outer scene key, the 'back-button' behavior would still continue between them.) Now there's really one last step to make it all go. We have to call a function to push the user to a new scene. Basically, anywhere we want to do this we need two steps, First at the top we import 'Actions':

  {% highlight javascript %}
    import { Actions } from 'react-native-router-flux';
  {% endhighlight %}

  ...then we simply call it:

  {% highlight javascript %}
    <Button onPress={() => Actions.list()}> List View </Button>
  {% endhighlight %}

  One important thing to note here though is I'm calling Actions.list() rather than Actions.VndrList(). If you check the longer code snippet above, you'll note that this references the outer Scene Key wrapper rather than the inner one.

  And **VOILA NAVIGATION**

  {: .center }
  ![nav_gif](/img/nav_recording.gif)

  It really is *that* simple.  


  [This React Tutorial]: https://online.reacttraining.com/p/reactjsfundamentals
  [This Udemy React-Native Course]: https://www.udemy.com/the-complete-react-native-and-redux-course/
  [React-Native's Navigation Documentation]: https://facebook.github.io/react-native/docs/navigation.html
  [react-native-router-flux]: https://github.com/aksonov/react-native-router-flux
