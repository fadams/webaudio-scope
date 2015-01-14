# webaudio-scope
Simple self-contained demo of HTML5 Web Audio APIs used with the WebRTC getUserMedia API
used to capture live audio input from the microphone.

It doesn't do anything terribly exciting, it just loops the input stream to the
output and renders a simple oscilloscope visualisation.

The main aim is just to play with the API and include references to the APIs used
so I can find them quickly again :-)

**WebRTC:**
https://developer.mozilla.org/en-US/docs/NavigatorUserMedia.getUserMedia

    navigator.getUserMedia(constraints,
	    				   successCallback,
	    				   errorCallback);
https://developer.mozilla.org/en-US/docs/Web/API/MediaStream_API#LocalMediaStream

**WebAudio:**
https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Basic_concepts_behind_Web_Audio_API

https://developer.mozilla.org/en-US/docs/Web/API/AudioContext
https://developer.mozilla.org/en-US/docs/Web/API/AudioNode
https://developer.mozilla.org/en-US/docs/Web/API/MediaStreamAudioSourceNode
https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode

**"Interesting" Browser Issues**
A couple of browser quirks caused a bit of pain as I was experimenting with getUserMedia and WebAudio.

**Firefox (Using FF 34):**
1. By default Firefox getUserMedia seemed to be horribly distorted, but worked
fine on Chrome and Opera. I *eventually* tracked it down to Firefox doing some
post-processing of the input by default.


> about:config

search for "getusermedia"

> media.getusermedia.agc_enabled media.getusermedia.aec_enabled

(Automatic Gain Control & Acoustic Echo Cancellation) should both be set to false.

It would be nice if these were controllable via getUserMedia constraints, but for
Firefox at least a global about:config setting is currently required.
2. MediaStreamAudioSourceNode get prematurely garbage-collected due to a bug in Firefox
 https://bugzilla.mozilla.org/show_bug.cgi?id=934512 which causes the microphone
to spontaneously switch off after five or so seconds. This normally occurs because
the MediaStreamAudioSourceNode tends to get created in the getUserMedia success
callback in a line like this:

> var input = context.createMediaStreamSource(stream);

The issue (for Firefox) is the local reference in the scope of the gUM success
callback. If, however, we use a more global reference to the input variable it
won't get prematurely garbage-collected.

**Chrome (39.0.2171.99 (64-bit)) & Opera (Using opera 26.0):**
1. Chrome and Opera have their own premature garbage-collection issue. In their
case it relates to ScriptProcessorNodes (not actually used in webaudio-scope).

An AudioNode *should* live as long as there are any references to it *including*
a connection reference which occurs if another AudioNode is connected to it. As
per the specification:

http://www.w3.org/TR/webaudio/#lifetime-AudioNode (clause 3.)

but with Chrome and Opera the ScriptProcessorNode gets garbage collected when the
normal object reference goes out of scope. This is a bug covered in these references:

https://bugs.webkit.org/show_bug.cgi?id=112521
https://code.google.com/p/chromium/issues/detail?id=82795
http://sriku.org/blog/2013/01/30/taming-the-scriptprocessornode

