---
layout: post
title: BlocJams
feature-img: "img/sample_feature_img.png"
thumbnail-path: "img/blocjams_main.png"
short-description: BlocJams for iOS is awesome!

---
**Summary:**  This project focused on creating the front end of a music player (not unlike spotify or pandora) and was my first serious attempt working with javascript (eventually refactored to jQuery) and front-end development with more in depth uses of CSS and HTML than I’d previously attempted. (MOVE) with the full suite of volume and track position controls, track selections, with a few additional pages for selection, and a wide range of on-click and mouse-over/hover functions.

**Explanation:** BlocJams basically acts as a main skeleton of an online music player. It has a landing page and a collection page, but the meat of the application is the media player.  The landing page is mostly just style, with some popup icons and images, as well header links to the collection page. The collection page provides a listing of available albums and relevant descriptions (artist, album name, number of tracks, etc.) though in this basic version only as a singular repeated is shown. The player itself required substantially more functionality. It featured the album’s collection of songs (numbered) and allowed the user to scroll anywhere over a songs and have the play & pause button-icons replace the song’s number appropriately depending on the condition (more detail on this in the next section).

![blocjams_3](/img/blocjams_3.png){:class="img-responsive"}

Additionally the page also featured a “player-bar” which beyond displaying current track information, and providing forward & backward skip buttons, also used a time-duration “bar” which displayed the songs current position in real time (both in terms of x:xx minute/second numerical time formatting and also as percentage of the bar’s fill) and allowed players to move to move the “thumb” dot to select any point in the song to play from. Lastly, the bar also provided volume controls which used a similar “bar & draggable-thumb” system as the time-duration system.

![blocjams_4](/img/blocjams_4.png){:class="img-responsive"}

**Problem:** So not unlike [Bloccit], the scope of this project is quite large and contains a great deal more than visible here, (you can feel free to check it out at the [Github Page] for more detail), as well as being originally coded in vanilla javascript via DOM before being refactored into jQuery (for educational purposes). For brevity, I've focused on the two main challanging features of the project and their code in jQuery. They are as follows:

1.) **Setting the Play and Pause icons correctly for the player:**
*Here the challenge was to create a system in which the track listing visibly altered (as well as performed) to show play/pause icons based on the currently playing track as well as the users selection. For example, if no track was playing, mousing over any song should replace the selected track number with a clickable play icon. Once a track is played that icon should switch to a pause icon (until a new track is played), but still allow the same mousover-to-play effect on any other song. Additionally when a currently playing song is paused, the play button should revert to a pause button, but stay locked in this position as long this song is still loaded as the currently playing selection.*

2.) **Making the Player bar work:**
*This requires a number of different features. First, it would have a unique play pause button (which changed accordingly depending on the status of the current song -- see above) along with forward/backward skip buttons that would not only change the playing track, but also create the appropriate visual change from the play/pause/track-number icons above. Secondly, it featured a time-duration bar that measured the current songs position both in terms of minutes and seconds, as well as a ratio of filled to unfilled space in the bar. It also featured a movable "thumb" that a user could drag to spot-play any part of the song (to which it would also have to respond in terms of minute/seconds and percentage-filled.) Lastly, it included a volume bar that responded to a similar user interaction, both by adjusting the volume and the respective "fill."*

**Solution:**

To address the first challenge there are a number of issues to be considered. The first is allowing the mouseover functions to respond to each track anywhere on the player (i.e. the track number, the title, the song duration, etc.) To accomplish a few things have to be understood. The first is that the entire album is a javascript object as seen here, but more important to our coding, we can see how we build a collection of "rows" (each containing the relevant data about the track song name/length/etc):

{% highlight javascript %}
var createSongRow = function(songNumber, songName, songLength) {
    var template =
        '<tr class="album-view-song-item">'
        +'  <td class="song-item-number" data-song-number="' + songNumber + '">' + songNumber + '</td>'
        +'  <td class="song-item-title">' + songName + '</td>'
        +'  <td class="song-tem-duration">' + filterTimeCode(songLength) + '</td>'
        +'</tr>'
        ;

        var $row = $(template);
{% endhighlight %}

Obviously this function continues and there's a lot more going on here, but we will explore that in more detail later, for now let's jump to the end of the file and look at how the code is called.

{% highlight javascript %}
var setCurrentAlbum = function(album) {
    currentAlbum = album;
    var $albumTitle = $('.album-view-title');
    var $albumArtist = $('.album-view-artist');
    var $albumReleaseInfo = $('.album-view-release-info');
    var $albumImage = $('.album-cover-art');
    var $albumSongList = $('.album-view-song-list');

    $albumTitle.text(album.title);
    $albumArtist.text(album.artist);
    $albumReleaseInfo.text(album.year + ' ' + album.label);
    $albumImage.attr('src', album.albumArtUrl);


    $albumSongList.empty();

    for (var i=0; i < album.songs.length; i++) {
        var $newRow = createSongRow(i + 1, album.songs[i].title, album.songs[i].duration);
        $albumSongList.append($newRow);
    }
};

{% endhighlight %}

Also at the very end of the createSongRow function we have these two lines of code that allow our click and mouseover functions operate on the whole song row.

{% highlight javascript %}
$row.find('.song-item-number').click(clickHandler);
$row.hover(onHover, offHover);

return $row;
{% endhighlight %}

Here we can see that we call both the click and hover jQuery functions, passing them our own clickHandler and on/offHover functions (discussed next) on the row object, before returning the row object at the end of the function. (Note that we call find on the click function using the song-item-number, because the click function is specific to the track in question.)

The next thing to examine is exactly how the "createSongRow" function continues to work and set up the play/pause functionality. We will start with the embedded function clickHandler:

{%highlight javascript %}
var clickHandler = function() {
        var songNumber = parseInt($(this).attr('data-song-number'));

        if (currentlyPlayingSongNumber !== null){
            var currentlyPlayingCell = getSongNumberCell(currentlyPlayingSongNumber);
            currentlyPlayingCell.html(currentlyPlayingSongNumber);
        }
        if (currentlyPlayingSongNumber !== songNumber){
            $(this).html(pauseButtonTemplate);
            setSong(songNumber);
            currentSoundFile.play();
            updateSeekBarWhileSongPlays();
            updatePlayerBarSong();

            var $volumeFill = $('.volume .fill');
            var $volumeThumb = $('.volume .thumb')
            $volumeFill.width(currentVolume + '%');
            $volumeThumb.css({left: currentVolume + '%'});

        } else if (currentlyPlayingSongNumber === songNumber){
            if (currentSoundFile.isPaused()){
                $(this).html(pauseButtonTemplate);
                $('.main-controls .play-pause').html(playerBarPauseButton);
                currentSoundFile.play();
                updateSeekBarWhileSongPlays();
            }
            else{
                $(this).html(playButtonTemplate);
                $('.main-controls .play-pause').html(playerBarPlayButton);
                currentSoundFile.pause();
            }
        }
    };
{% endhighlight %}

There's a lot to unpack in this function. Let's focus on the conditionals. First, we see if the currentlyPlayingSongNumber is not null (i.e. a song is playing) we set the "cell" (the space occupied by the number or play/pause icon) to the number. Remember that this function specifically handles when a song is clicked on, so in effect this states that when a song is playing and a new song is clicked it switches the icon of the formally playing song back to it's track number.

Next we deal with the situation of what to do with the track that is actually clicked. In our first conditional, if the song clicked is not the same as the song number, we do a number of things. First we set the icon to the pause button (because now that the song is playing the remaining option on that song is to pause it.) Next we set the current song (this is another function which handles a lot of a set up and information storage - which we will skip for brevity.) We then play the selected song, and then make a few updates to the player bar (which will be discussed later.)

However what if the song clicked *is* the song playing. Right below it, in the next conditional, we see as one might expect that we follow a similar set of procedures, however, there are two distinct cases to consider. In the first, (which one might forget,) a user might not only click a currently playing song to pause it, but also to *unpause* it. So within this conditional, we have sub-conditional. First we that if the current sound file is paused, we perform (*nearly*) identical steps to the previous case of playing a new song (a few changes are different to account for the fact that the time/position of the song remains the same, but that will be explored more in the player bar section.) Next, in our else case, we perform similar but basically opposite functions for when the current song is the song clicked, but it is **not** paused. It's a little simpler because we don't have to adjust any of the "currentSong" type of details, but we still replace the pause icon with a play icon, (recall that as you often see in music players the available icon is inverted from the current status because when a song is paused the available choice to be clicked is now "play" or vice versa,) and we also pause the physical sound file. (We also mess with the player bar, but again, we will cover that later.)

Next we need to examine how the various hovering functions work. Let's look at the following code:

{% highlight javascript %}
var onHover = function(event){
        var songNumberCell = $(this).find('.song-item-number');
        var songNumber = parseInt(songNumberCell.attr('data-song-number'));

        if (songNumber !== currentlyPlayingSongNumber) {
            songNumberCell.html(playButtonTemplate);
        }
    };

    var offHover = function(event){
        var songNumberCell = $(this).find('.song-item-number');
        var songNumber = parseInt(songNumberCell.attr('data-song-number'));

        if (songNumber !== currentlyPlayingSongNumber) {
        songNumberCell.html(songNumber);
        }
    };
{% endhighlight %}

So first, with the onHover function, when we hover over any part of the song row we first find the song in question based on it's number and use that to define the "cell" we want to alter. We also set a songNumber variable to be equivalent to the actual song's track position. Then if the songNumber is not equal to the currently playing song's number, we set the cell to equal the play button. In other words, if we hover over a song's row, and that song is not currently playing, we show the user the play button to let them know they can click it and play the song.

Next in the offHover function, we start with the same two lines of code, but here, instead of providing a play button, because we are leaving the song row, we replace set the cell back to the track's song number.

Now that we've explored some of the clicking/mouseover functionality. Let's talk about the player bar below. As discussed earlier, the player bar needs to have a play/pause functionality as well as track forward and back buttons. Again for the sake of brevity, I'm going to skip the play/pause code (which is very similar to the play/pause functionality above) and just focus on the nextSong function (previousSong is basically the same,) but again, feel free to check out the [Github Page] for more detail.

{% highlight javascript %}
var nextSong = function() {
    var getLastSongNumber = function(index){
        return index == 0 ? currentAlbum.songs.length : index;
    }

    var currentSongIndex = trackIndex(currentAlbum, currentSongFromAlbum);

    currentSongIndex++;

    if (currentSongIndex >= currentAlbum.songs.length){
        currentSongIndex = 0;
    }

    setSong(currentSongIndex + 1);
    currentSoundFile.play();
    updateSeekBarWhileSongPlays();
    updatePlayerBarSong();

    var lastSongNumber = getLastSongNumber(currentSongIndex);
    var $nextSongNumberCell = getSongNumberCell(currentlyPlayingSongNumber);
    var $lastSongNumberCell = getSongNumberCell(lastSongNumber)

    var $volumeFill = $('.volume .fill');
    var $volumeThumb = $('.volume .thumb')
    $volumeFill.width(currentVolume + '%');
    $volumeThumb.css({left: currentVolume + '%'});

    $nextSongNumberCell.html(pauseButtonTemplate);
    $lastSongNumberCell.html(lastSongNumber);

};
{% endhighlight %}

So the first thing we do in this block of code is get the previous song number by calling a function (again, not shown here cause this is getting *really* long.) Note that we return index zero (the first song) if the current song is equal to the length of the album (this way the next song is the first song and the next button wraps around.) Next using another function we get the index value of the current song and then increment it by one (again if the index is larger than the length of the album we reset the value to zero - the first track.) We then set the song, (note that this function uses actual track listing numbers rather than the array index so we need to increment this value by one to account for the 0/1 first tack shift,) and then we play the song, and do some stuff with the seek bars (will be discussed next.) We also update the visible data in the player bar using this function with jQuery to modify the html:

{% highlight javascript %}
var updatePlayerBarSong = function(){
    $('.currently-playing .song-name').text(currentSongFromAlbum.title);
    $('.currently-playing .artist-name').text(currentAlbum.artist);
    $('.currently-playing .artist-song-mobile').text(currentSongFromAlbum.title + " - " + currentAlbum.artist);
    $('.main-controls .play-pause').html(playerBarPauseButton);
    setTotalTimeInPlayerBar(filterTimeCode(currentSongFromAlbum.duration));
};
{% endhighlight %}

We then get the previous song number (so our application has this information in case our user wants to hit the track-back button.) We then set the nextSongNumberCell (which acts as the current song) via the currently playing song number and do the same respectively for the lastSongNumberCell. We then replace these cells with the pause icon and the appropriate song number respectively so that the above track listing retains the appropriate visual layout.

Now onto the seek bar, again here I will focus on the seek bar and not the volume bar because the volume controls are largely redundant code-wise (in relation to the seek bar) and actually simpler.

{% highlight javascript %}

var updateSeekPercentage = function($seekBar, seekBarFillRatio) {
    var offsetXPercent = seekBarFillRatio * 100;
    offsetXPercent = Math.max(0, offsetXPercent);
    offsetXPercent = Math.min(100, offsetXPercent);

    var percentageString = offsetXPercent + '%';
    $seekBar.find('.fill').width(percentageString);
    $seekBar.find('.thumb').css({left: percentageString});
};

{% endhighlight %}

First we see the function we used to update the percentage of filled bar. The function takes a specific seekBar element and a ratio of fill. We create a percent offset, and the set the min and max so the percentage cannot be more than 100 or less than 0 (these math operations look inverted here because the offset represents the amount filled) we convert the value into a string with the % symbol and pass it into the fill and thumb items and move them accordingly by modifying the html and css elements. Next let's see how this code is utilized.

{% highlight javascript %}

var setupSeekBars = function() {
    var $seekBars = $('.player-bar .seek-bar')
    $seekBars.click(function(event){
        var offsetX = event.pageX - $(this).offset().left;
        var barWidth = $(this).width();
        var seekBarFillRatio = offsetX / barWidth;

        if ($(this).parent().attr('class') == 'seek-control') {
            seek(seekBarFillRatio * currentSoundFile.getDuration());
        }

        else
        {
            setVolume(seekBarFillRatio * 100);
        }
        updateSeekPercentage($(this), seekBarFillRatio);

    });
{% endhighlight %}

What happens above? Well here is how we create the interaction where the user can move the thumb and the application will determine the % offset. We set the jQuery variable $seekBars to our css element related to either of the seek-bars in our player bar (either the track seek or the volume.) Next we use .click function and the jQuery property pageX to determine where on the page (within the seek bar obviously) the user has clicked. We subtract the left offset of the seek bar in relation to the whole page to determine the correct value. Then we divide this value by the width of the whole bar to determine the ratio we need to fill. We then have a quick conditional to determine which seek bar we are dealing with. If the parent class is the seek-control then we use seek function (a fairly basic function which sets the current time of the track) and uses the ratio against the length of the whole song to determine the tracks new playing position. Otherwise we can conclude that we want to change the volume and we follow a similar set of steps. Lastly we call our updateSeekPercentage function (seen previously) with the appropriate arguments to create the correct graphical fill.

So this sets the song position/volume (physically and graphically) when a user clicks somewhere in a seek-bar, but what about when someone wants to actually *drag* the thumb button? Well, as we can see, the function continues...

{% highlight javascript %}

$seekBars.find('.thumb').mousedown(function(event){
        var $seekBar = $(this).parent();

        $(document).bind('mousemove.thumb', function(event){
            var offsetX = event.pageX - $seekBar.offset().left;
            var barWidth = $seekBar.width();
            var seekBarFillRatio = offsetX / barWidth;

            if ($(this).parent().attr('class') == 'seek-control') {
                seek(seekBarFillRatio * currentSoundFile.getDuration());
            }

            else
            {
                setVolume(seekBarFillRatio * 100);
            }

            updateSeekPercentage($seekBar, seekBarFillRatio);
        });

        $(document).bind('mouseup.thumb', function(event){
            $(document).unbind('mousemove.thumb');
            $(document).unbind('mouseup.thumb');
        });  
{% endhighlight %}

Here we find the thumb and set the seek bar by finding its parent class. We then use bind to attach a mouse-move of the thumb to the whole document (incase the mouse leaves the actual seek-bar.) Then again we can see all the same steps shown in the previous block of code to determine the various ratios.

Lastly, let's look at how the seek-control updates continuously as the song plays.

{% highlight javascript %}

var updateSeekBarWhileSongPlays = function() {
    if (currentSoundFile) {
        currentSoundFile.bind('timeupdate', function(event){
            var seekBarFillRatio = this.getTime() / this.getDuration();
            var $seekBar = $('.seek-control .seek-bar');

            updateSeekPercentage($seekBar, seekBarFillRatio);  
            setCurrentTimeInPlayerBar(filterTimeCode(this.getTime()));
        });
    }
};

{% endhighlight %}

First we check if there is a currentSoundFile present, (i.e. that a song is playing.) We again use the .bind function with the time-update event (a custom event in the [Buzz] audio playback library.) We use Buzz's getTime and getDuration functions to calculate the ratio again, set the seek bar, and pass these values into our previous function to update the fill percentage as well as the time (details about parsing and displaying time have been omitted for brevity.)

**Results:** Again, this project is much larger in scope than discussed here, but a lot of the functionality has been outlined. Overall, though there are a few issues dragging the thumb while a song is actively playing, the project was largely successful. Here are a couple of screen shots of the main pages.


The landing page:
![blocjams_main](/img/blocjams_main.png){:class="img-responsive"}

The (*repetitive*, but nonetheless functional) album page:
![blocjams_1](/img/blocjams_1.png){:class="img-responsive"}

...and the full player page:
![blocjams_2](/img/blocjams_2.png){:class="img-responsive"}

Lastly let's look at two quick videos. The first displays the player's mouse-over functionality:

![blocjams_demo_1](/img/blocjams_demo_1.gif)

and here is the seek bar in action: 

![blocjams_demo_2](/img/blocjams_demo_2.gif)



**Conclusion:**

[Github Page]: https://github.com/americool/bloc-jams
[Bloccit]: /_portfoilo/1_bloccit.md
[buzz]: http://buzz.jaysalvat.com/
