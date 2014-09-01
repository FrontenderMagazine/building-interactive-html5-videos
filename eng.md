# Building Interactive HTML5 Videos

The [HTML5][1] `<video>` element makes embedding videos into your site as easy
as embedding images. And since all major browsers support `<video>` since 2011,
it’s also the most reliable way to get your moving pictures seen by people.

A more recent addition to the HTML5 family is the `<track>` element. It’s a sub-
element of `<video>`, intended to make the video timeline more accessible. Its
main use case is adding closed captions. These captions are loaded from a
separate text file (a [WebVTT][2] file) and printed over the bottom of the video
display. Ian Devlin has written [an excellent article][3] on the subject.

Beyond captions though, the `<track>` element can be used for any kind of
interaction with the video timeline. This article explores 3 examples: chapter
markers, preview thumbnails, and a timeline search. By the end, you will have
sufficient understanding of the `<track>` element and its scripting API to build
your own interactive video experiences.

## Chapter Markers

Let’s start with an example made popular by DVD disks: chapter markers. These
allow viewers to quickly jump to a specific section. It’s especially useful for
longer movies like Sintel:

<iframe src="https://s3.amazonaws.com/demo.jwplayer.com/text-tracks/chapters.html" frameborder="0" height="350" width="495"></iframe>

The chapter markers in this example reside in an [external VTT file][4] and are
loaded on the page through a `<track>` element with a kind of **chapters. The
track is set to load by default:

    <video width="480" height="204" poster="assets/sintel.jpg" controls>
      <source src="assets/sintel.mp4" type="video/mp4">
      <track src="assets/chapters.vtt" kind="chapters" default>
    </video>

Next, we use JavaScript to load the cues of the text track, format them, and
print them in a controlbar below the video. Note we have to wait until the
external VTT file is loaded:

    track.addEventListener('load',function() {
        var c = video.textTracks[0].cues;
        for (var i=0; i<c.length; i++) {
          var s = document.createElement("span");
          s.innerHTML = c[i].text;
          s.setAttribute('data-start',c[i].startTime);
          s.addEventListener("click",seek);
          controlbar.appendChild(s);
        }
    });

In above code block, we’re adding 2 properties to the list entries to hook up
interactivity. First, we set a `data` attribute to store the start position of
the chapter, and second we add a click handler for an external seek function.
This function will jump the video to the start position. If the video is not
(yet) playing, we’ll make that so:

    function seek() {
      video.currentTime = this.getAttribute('data-start');
      if(video.paused){ video.play(); }
    };

That’s it! You now have a visual chapter menu for your video, powered by a VTT
track. Note the [actual live Chapter Markers example][5] has a little bit more
logic than described, e.g. to toggle playback of the video on click, to update
the controlbar with the video position, and to add some CSS styling.

## Preview Thumbnails

This second example shows a cool feature made popular by Hulu and Netflix:
preview thumbnails. When mousing over the controlbar (or dragging on mobile), a
small preview of the position you’re about to seek to is displayed:

<iframe src="https://s3.amazonaws.com/demo.jwplayer.com/text-tracks/thumbs.html" frameborder="0" height="350" width="495"></iframe>

This example is also powered by an [external VTT file][6], loaded in a metadata
track. Instead of texts, the cues in this VTT file contain links to a [separate
JPG image][7]. Each cue could link to a separate image, but in this case we
opted to use a single JPG sprite - to keep latency low and management easy. The
cues link to the correct section of the sprite by using [Media Fragment
URIs][8].Example:

    http://example.com/assets/thumbs.jpg?xywh=0,0,160,90

Next, all important logic to get the right thumbnail and display it lives in a
mousemove listener for the controlbar:

    controlbar.addEventListener('mousemove',function(e) {
      // first we convert from mouse to time position ..
      var p = (e.pageX - controlbar.offsetLeft) * video.duration / 480;

      // ..then we find the matching cue..
      var c = video.textTracks[0].cues;
      for (var i=0; i p) {
              break;
          };
      }

      // ..next we unravel the JPG url and fragment query..
      var url =c[i].text.split('#')[0];
      var xywh = c[i].text.substr(c[i].text.indexOf("=")+1).split(',');

      // ..and last we style the thumbnail overlay
      thumbnail.style.backgroundImage = 'url('+c[i].text.split('#')[0]+')';
      thumbnail.style.backgroundPosition = '-'+xywh[0]+'px -'+xywh[1]+'px';
      thumbnail.style.left = e.pageX - xywh[2]/2+'px';
      thumbnail.style.top = controlbar.offsetTop - xywh[3]+8+'px';
      thumbnail.style.width = xywh[2]+'px';
      thumbnail.style.height = xywh[3]+'px';
    });

All done! Again, the [actual live Preview Thumbnails example][9] contains some
additional code. It includes the same logic for toggling playback and seeking,
as well as logic to show/hide the thumbnail when mousing in/out of the
controlbar.

## Timeline Search

Our last example offers yet another way to unlock your content, this time though
in-video search:

<iframe src="https://s3.amazonaws.com/demo.jwplayer.com/text-tracks/search.html" frameborder="0" height="400" width="495"></iframe>

This example re-uses an existing [captions VTT file][10], which is loaded into a
*captions* track. Below the video and controlbar, we print a basic search form:

<form>
    <input type="search">
    <button type="submit">Search</button>
</form>

Like with the thumbnails example, all key logic resides in a single function.
This time, it’s the event handler for submitting the form:

    form.addEventListener('submit',function(e) {
      // First we’ll prevent page reload and grab the cues/query..
      e.preventDefault();
      var c = video.textTracks[0].cues;
      var q = document.querySelector("input").value.toLowerCase();

      // ..then we find all matching cues..
      var a = [];
      for(var j=0; j -1) {
          a.push(c[j]);
        }
      }

      // ..and last we highlight matching cues on the controlbar.
      for (var i=0; i<a.length; i++) {
        var s = document.createElement("span");
        s.style.left = (a[i].startTime/video.duration*480-2)+"px";
        bar.appendChild(s);
      }
    });

Three time’s a charm! Like with the other ones, the [actual live Timeline Search
example][11] contains additional code for toggling playback and seeking, as well
as a snippet to update the controlbar help text.

## Wrapping Up

Above examples should provide you with enough knowledge to build your own
interactive videos. For some more inspiration, see our experiments around
[clickable hot spots][12], [interactive transcripts][13], or [timeline
interaction][14].

Overall, the [HTML5 `<track>` element][15] provides an easy to use, cross-
platform way to add interactivity to your videos. And while it definitely takes
time to author VTT files and build similar experiences, you will see higher
accessibility of and engagement with your videos. Good luck!

[1]: https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/HTML5
[2]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Video_Text_Tracks_Format
[3]: https://hacks.mozilla.org/2014/07/adding-captions-and-subtitles-to-html5-video/
[4]: http://demo.jwplayer.com/text-tracks/assets/chapters.vtt
[5]: http://demo.jwplayer.com/text-tracks/chapters.html
[6]: http://demo.jwplayer.com/text-tracks/assets/thumbs.vtt
[7]: http://demo.jwplayer.com/text-tracks/assets/thumbs.jpg
[8]: http://www.w3.org/TR/media-frags/
[9]: http://demo.jwplayer.com/text-tracks/thumbs.html
[10]: http://demo.jwplayer.com/text-tracks/assets/captions.vtt
[11]: http://demo.jwplayer.com/text-tracks/search.html
[12]: http://www.jwplayer.com/labs/experiments/hot-spots/
[13]: http://www.jwplayer.com/labs/experiments/interactive-transcripts/
[14]: http://www.jwplayer.com/labs/experiments/timeline-interaction/
[15]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/track