---
title: BaseAudioContext.createChannelMerger()
slug: Web/API/BaseAudioContext/createChannelMerger
tags:
  - API
  - Audio
  - AudioContext
  - BaseAudioContext
  - Method
  - Reference
  - Web Audio API
  - createChannelMerger
browser-compat: api.BaseAudioContext.createChannelMerger
---
<p>{{ APIRef("Web Audio API") }}</p>

<p>The <code>createChannelMerger()</code> method of the {{domxref("BaseAudioContext")}} interface creates a {{domxref("ChannelMergerNode")}},
  which combines channels from multiple audio streams into a single audio stream.</p>

<div class="notecard note">
  <p><strong>Note:</strong> The {{domxref("ChannelMergerNode.ChannelMergerNode", "ChannelMergerNode()")}} constructor is the
    recommended way to create a {{domxref("ChannelMergerNode")}}; see
    <a href="/en-US/docs/Web/API/AudioNode#creating_an_audionode">Creating an AudioNode</a>.</p>
</div>

<h2 id="Syntax">Syntax</h2>

<pre class="brush: js">createChannelMerger(<em>numberOfInputs</em>)</pre>

<h3 id="Parameters">Parameters</h3>

<dl>
  <dt>numberOfInputs</dt>
  <dd>The number of channels in the input audio streams, which the output stream will
    contain; the default is 6 if this parameter is not specified.</dd>
</dl>

<h3 id="Returns">Returns</h3>

<p>A {{domxref("ChannelMergerNode")}}.</p>

<h2 id="Example">Example</h2>

<p>The following example shows how you could separate a stereo track (say, a piece of
  music), and process the left and right channel differently. To use them, you need to use
  the second and third parameters of the {{domxref("AudioNode/connect", "AudioNode.connect(AudioNode)")}}
  method, which allow you to specify both the index of the channel to connect from and the
  index of the channel to connect to.</p>

<pre class="brush: js">var ac = new AudioContext();
ac.decodeAudioData(someStereoBuffer, function(data) {
 var source = ac.createBufferSource();
 source.buffer = data;
 var splitter = ac.createChannelSplitter(2);
 source.connect(splitter);
 var merger = ac.createChannelMerger(2);

 // Reduce the volume of the left channel only
 var gainNode = ac.createGain();
 gainNode.gain.setValueAtTime(0.5, ac.currentTime);
 splitter.connect(gainNode, 0);

 // Connect the splitter back to the second input of the merger: we
 // effectively swap the channels, here, reversing the stereo image.
 gainNode.connect(merger, 0, 1);
 splitter.connect(merger, 1, 0);

 var dest = ac.createMediaStreamDestination();

 // Because we have used a ChannelMergerNode, we now have a stereo
 // MediaStream we can use to pipe the Web Audio graph to WebRTC,
 // MediaRecorder, etc.
 merger.connect(dest);
});</pre>

<h2 id="Specifications">Specifications</h2>

{{Specifications}}

<h2 id="Browser_compatibility">Browser compatibility</h2>

<p>{{Compat}}</p>

<h2 id="See_also">See also</h2>

<ul>
  <li><a href="/en-US/docs/Web/API/Web_Audio_API/Using_Web_Audio_API">Using the Web Audio API</a></li>
</ul>