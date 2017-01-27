---
layout: post
title:  "React-Native-Maps: Using AirBnB's Open Source Map"
date:   2017-01-26 14:25:00
categories: react, react-native, maps
---

*If you're interested in background on my vendor project check out [my previous blog post].*   

So last post I talked about setting up navigation on my react-native app. The app, (which I never discuss in too much detail) is what I've loosely called a "vendor locator" which displays lists of vendors near the user and provides their name, description, and distance. Initially, the app only displayed this information in a list form as you can see here:

{: .center }
![vndr_screenshot](/img/navigation_vndr_screenshot.png)

...But like most apps that use or provide any sort of location or distance related data, I also wanted to be able to provide an actual real life map view.
<br/><br/>
Enter [react-native-maps] -- an open source tool provided by AirBnB. It provides a whole host of customizable features ranging from a basic map display, to displaying markers (or pins), or even overlaying polygonal graphics.  For the purposes of simplicity (*and the fact that this all I've implemented as of now*,) I'm going to focus on the first two features. Namely, getting the map up and running, and then implementing the ability to add pins to the map to represent our various vendors locations.


Let's start with just getting the map up and running. Obviously one can examine the github link provided above for some setup-details, but I generally found installing the dependencies for this fairly straightforward. (*Note: I'm currently only running this on IOS, it does seem a little trickier to get Android and Google Maps up and running, but for now we will focus on the IOS system and the default apple map tool*) Once you've got everything set up - let's take a look at the most basic implementation. To simply display a map, you'll really only need to do two things: To import the map view and to set the component with a few details...

{% highlight javascript %}
import MapView from 'react-native-maps';

<MapView
  style={{ flex: 1, height: 450 }}
  showsUserLocation
  initialRegion={{
    latitude: this.state.userLatitude,
    longitude: this.state.userLongitude,
    latitudeDelta: 0.019,
    longitudeDelta: 0.019,
  }}>
{% endhighlight %}

As you can see here, while there are a number of other potential props to add or alter, I've kept it fairly basic here. I've given it a basic style in order for it look decent on my screen, included the showsUserLocation prop, so that we'll get the little blue icon (whose position can be set in the IOS simulator,) and lastly I've set an initial region. Note that here I'm using some state specific properties I've set so that the user automatically gets a map related to their current location. (The delta represents the initial range of the map in either direction, for my purposes I keep the view rather tight.)
If you're interested in getting your user location you can use:

{% highlight javascript %}
navigator.geolocation.getCurrentPosition
{% endhighlight %}

Feel free to read more about react-native's [geolocation] if you're interested.

I should also clarify that the basic MapView code above not only will display a map and user location but also allows for all the dragging and zooming functionality we've come to expect out of all mobile-app maps.

{: .center }
![map1_gif](/img/basicmapview.gif){:height="700px" }

Next comes the question of how to implement our own markers to the map. React-Native-Maps sets it up so that inside our MapView component we can render separate Marker components as children like so:

{% highlight javascript %}

<MapView
  style={{ flex: 1, height: 450 }}
  showsUserLocation
  initialRegion={{
    latitude: (some number),
    longitude: (some number),
    latitudeDelta: (some number),
    longitudeDelta: (some number),
  }}>
    <MapView.Marker
      coordinate={{
        latitude: (some number),
        longitude: (some number),
      }}
      title=(some name)
      description=(some description)
    />
</MapView>

{% endhighlight %}

This, (*and yes again there are a bunch of other changeable props for this component as well*,) does technically work, and this code will allow us to render a marker anywhere on the map we like. But how often do we really want to hard code in a single (or several) markers into our map? Pretty much never, I'd say. So it's not a big surprise that AirBnB's documentation for this tool seems to only really discuss the idea of looping through a hypothetical set of data and rendering *several* markers based on whatever data we have at our disposal. Here's what I actually did:

{% highlight javascript %}
<MapView
  style={{ flex: 1, height: 450 }}
  showsUserLocation
  initialRegion={{
    latitude: this.state.userLatitude,
    longitude: this.state.userLongitude,
    latitudeDelta: 0.019,
    longitudeDelta: 0.019,
  }}>

  {this.state.data.map(vendor => (
    <MapView.Marker
      coordinate={{
        latitude: vendor.latitude,
        longitude: vendor.longitude,
      }}
      title={vendor.vndrName}
      description={vendor.description}
    />

  ))}
</MapView>
{% endhighlight %}

Now again, there's a lot more to this than I've included here, and as always I'll provide a link to github at the end if you want to explore more. In short though: As I mentioned earlier I'm using geolocation to set the lat/long states in the constructor. Next we can see were using .map to move through my "data" state and create marker locations, names, and descriptions based on each objects properties. (For the record, "data" is fetched earlier from firebase and provides an array of vendor objects which have the various properties shown above.)

While the end goal of this project's functionality is quite different we now have a system where we can add vendors to the database (*which may be discussed in a different post*), provide their longitude and latitude, and not only will they show up on a list view, but also will appear as markers on the map. See below:

{: .center }
![map2_gif](/img/mapviewmarkerdemo.gif){:height="700px" }

Perhaps in the next post I'll discuss working with firebase, or perhaps even some of the other logic in this app. Please feel free to [contact me] if you have any questions, or checkout the github for [this project].

[contact me]: /contact/
[this project]: https://github.com/americool/vndr_ideas/blob/master/src/ViewMap.js
[my previous blog post]: /React-Native-Navigation/
[react-native-maps]: https://github.com/airbnb/react-native-maps
[geolocation]: https://facebook.github.io/react-native/docs/geolocation.html
