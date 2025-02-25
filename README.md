# streamlit-webrtc
**Handling and transmitting real-time video/audio streams over the network with Streamlit** [![Open in Streamlit](https://static.streamlit.io/badges/streamlit_badge_black_white.svg)](https://share.streamlit.io/whitphx/streamlit-webrtc-example/main/app.py)

[![Tests](https://github.com/whitphx/streamlit-webrtc/workflows/Tests/badge.svg?branch=main)](https://github.com/whitphx/streamlit-webrtc/actions?query=workflow%3ATests+branch%3Amain)
[![Frontend Tests](https://github.com/whitphx/streamlit-webrtc/workflows/Frontend%20tests/badge.svg?branch=main)](https://github.com/whitphx/streamlit-webrtc/actions?query=workflow%3A%22Frontend+tests%22+branch%3Amain)

[![PyPI](https://img.shields.io/pypi/v/streamlit-webrtc)](https://pypi.org/project/streamlit-webrtc/)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/streamlit-webrtc)](https://pypi.org/project/streamlit-webrtc/)
[![PyPI - License](https://img.shields.io/pypi/l/streamlit-webrtc)](https://pypi.org/project/streamlit-webrtc/)
[![PyPI - Downloads](https://img.shields.io/pypi/dm/streamlit-webrtc)](https://pypi.org/project/streamlit-webrtc/)

[![GitHub Sponsors](https://img.shields.io/github/sponsors/whitphx?label=Sponsor%20me%20on%20GitHub%20Sponsors&style=social)](https://github.com/sponsors/whitphx)

<a href="https://www.buymeacoffee.com/whitphx" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" width="180" height="50" ></a>

<table>
<tr>
<td width="48%">
  <a href="https://share.streamlit.io/whitphx/streamlit-webrtc-example/main/app.py">
    <img src="https://aws1.discourse-cdn.com/business7/uploads/streamlit/original/2X/a/af111a7393c77cb69d7712ac8e71ca862feaeb24.gif" />
  </a>
</id>
<td width="48%">
  <a href="https://share.streamlit.io/whitphx/style-transfer-web-app/main/app.py">
    <img src="https://aws1.discourse-cdn.com/business7/uploads/streamlit/original/2X/b/b3cb8aa60eb746366e06726a9137720583c02c3a.gif" />
  </a>
</id>
</tr>
</table>

## Examples
### [⚡️Showcase including following examples and more](https://github.com/whitphx/streamlit-webrtc-example): [🎈Online demo](https://share.streamlit.io/whitphx/streamlit-webrtc-example/main/app.py)

* Object detection
* OpenCV filter
* Uni-directional video streaming
* Audio processing

You can try out this sample app using the following commands on your local env.
```
$ pip install streamlit-webrtc opencv-python-headless matplotlib pydub
$ streamlit run https://raw.githubusercontent.com/whitphx/streamlit-webrtc-example/main/app.py
```

### [⚡️Real-time Speech-to-Text](https://github.com/whitphx/streamlit-stt-app): [🎈Online demo](https://share.streamlit.io/whitphx/streamlit-stt-app/main/app_deepspeech.py)

It converts your voice into text in real time.
This app is self-contained; it does not depend on any external API.

### [⚡️Real-time video style transfer](https://github.com/whitphx/style-transfer-web-app): [🎈Online demo](https://share.streamlit.io/whitphx/style-transfer-web-app/main/app.py)
It applies a wide variety of style transfer filters to real-time video streams.

### [⚡️Video chat](https://github.com/whitphx/streamlit-video-chat-example)
(Online demo not available)

You can create video chat apps with ~100 lines of Python code.

### [⚡️Tokyo 2020 Pictogram](https://github.com/whitphx/Tokyo2020-Pictogram-using-MediaPipe): [🎈Online demo](https://share.streamlit.io/whitphx/tokyo2020-pictogram-using-mediapipe/streamlit-app)
[MediaPipe](https://google.github.io/mediapipe/) is used for pose estimation.

## Install
```shell
$ pip install -U streamlit-webrtc
```

## Quick tutorial

See also ["Developing Web-Based Real-Time Video/Audio Processing Apps Quickly with Streamlit"](https://towardsdatascience.com/developing-web-based-real-time-video-audio-processing-apps-quickly-with-streamlit-7c7bcd0bc5a8).

---

Create `app.py` with the content below.
```py
from streamlit_webrtc import webrtc_streamer

webrtc_streamer(key="sample")
```
Unlike other Streamlit components, `webrtc_streamer()` requires the `key` argument as a unique identifier. Set an arbitrary string to it.

Then run it with Streamlit and open http://localhost:8501/.
```shell
$ streamlit run app.py
```

You see the app view, so click the "START" button.

Then, video and audio streaming starts. If asked for permissions to access the camera and microphone, allow it.
![Basic example of streamlit-webrtc](./docs/images/streamlit_webrtc_basic.gif)

Next, edit `app.py` as below and run it again.
```py
from streamlit_webrtc import webrtc_streamer
import av


class VideoProcessor:
    def recv(self, frame):
        img = frame.to_ndarray(format="bgr24")

        flipped = img[::-1,:,:]

        return av.VideoFrame.from_ndarray(flipped, format="bgr24")


webrtc_streamer(key="example", video_processor_factory=VideoProcessor)
```

Now the video is vertically flipped.
![Vertically flipping example](./docs/images/streamlit_webrtc_flipped.gif)

As an example above, you can edit the video frames by defining a class with a callback method `recv(self, frame)` and passing it to the `video_processor_factory` argument.
The callback receives and returns a frame. The frame is an instance of [`av.VideoFrame`](https://pyav.org/docs/develop/api/video.html#av.video.frame.VideoFrame) (or [`av.AudioFrame`](https://pyav.org/docs/develop/api/audio.html#av.audio.frame.AudioFrame) when dealing with audio) of [`PyAV` library](https://pyav.org/).

You can inject any kinds of image (or audio) processing inside the callback.
See examples above for more applications.

Note that there are some limitations in this callback. See the section below.

## Limitations
The callback methods (`VideoProcessor.recv()` and similar ones) are executed in threads different from the main thread, so there are some limitations:
* Streamlit methods (`st.*` such as `st.write()`) do not work inside the callbacks.
* Variables outside the callbacks cannot be referred to from inside, and vice versa.
  * It's impossible even with the `global` keyword, which also does not work in the callbacks properly.
* You have to care about thread-safety when accessing the same objects both from outside and inside the callbacks.

### A technique to pass values between inside and outside the callbacks
As stated above, you cannot directly pass variables from/to outside and inside the callback and have to consider about thread-safety.

Usual cases are
* to change some parameters used in the callback to new values passed from the main scope.
* to refer to the results of some processing inside the callback from the main scope.

The solution is to use the properties of the processor object which is accessible via the context object returned from `webrtc_streamer()` as below.
```python
class VideoProcessor:
    def __init__(self):
        self.some_value = 0.5

    def recv(self, frame):
        img = frame.to_ndarray(format="bgr24")

        ...
        self.do_something(img, self.some_value)  # `some_value` is used here
        ...

        return av.VideoFrame.from_ndarray(img, format="bgr24")


ctx = webrtc_streamer(key="example", video_processor_factory=VideoProcessor)

if ctx.video_processor:
    ctx.video_processor.some_value = st.slider(...)  # `some_value` is set here
```

If the passed value is a complex object, you may also have to consider about using something like [`threading.Lock`](https://docs.python.org/3/library/threading.html#threading.Lock) or [`queue.Queue`](https://docs.python.org/3/library/queue.html#queue.Queue) for thread-safety.

[The sample app, `app.py`](https://github.com/whitphx/streamlit-webrtc/blob/main/app.py) has many cases where this technique is used and can be a hint for this topic.

## Serving from remote host
When deploying apps to remote servers, there are some things you need to be aware of.

### HTTPS
`streamlit-webrtc` uses [`getUserMedia()`](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia) API to access local media devices, and this method does not work in an insecure context.

[This document](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia#privacy_and_security) says
> A secure context is, in short, a page loaded using HTTPS or the file:/// URL scheme, or a page loaded from localhost.

So, when hosting your app on a remote server, it must be served via HTTPS if your app is using webcam or microphone.
If not, you will encounter an error when starting using the device. For example, it's something like below on Chrome.
> Error: navigator.mediaDevices is undefined. It seems the current document is not loaded securely.

[Streamlit Cloud](https://streamlit.io/cloud) is a recommended way for HTTPS serving. You can easily deploy Streamlit apps with it, and most importantly for this topic, it serves the apps via HTTPS automatically by defualt.

### Configure the STUN server
To deploy the app to the cloud, we have to configure the *STUN* server via the `rtc_configuration` argument on `webrtc_streamer()` like below.

```python
webrtc_streamer(
    # ...
    rtc_configuration={  # Add this config
        "iceServers": [{"urls": ["stun:stun.l.google.com:19302"]}]
    }
    # ...
)
```

This configuration is necessary to establish the media streaming connection when the server is on a remote host.

`streamlit_webrtc` uses WebRTC for its video and audio streaming. It has to access a "STUN server" in the global network for the remote peers (precisely, peers over the NATs) to establish WebRTC connections.
As we don't see the details about STUN servers here, please google it if interested with keywords such as STUN, TURN, or NAT traversal, or read these articles ([1](https://towardsdatascience.com/developing-web-based-real-time-video-audio-processing-apps-quickly-with-streamlit-7c7bcd0bc5a8#1cec), [2](https://dev.to/whitphx/python-webrtc-basics-with-aiortc-48id), [3](https://www.3cx.com/pbx/what-is-a-stun-server/)).

The example above is configured to use `stun.l.google.com:19302`, which is a free STUN server provided by Google.

You can also use any other STUN servers.
For example, [one user reported](https://github.com/whitphx/streamlit-webrtc/issues/283#issuecomment-889753789) that the Google's STUN server had a huge delay when using from China network, and the problem was solved by changing the STUN server.

For those who know about the browser WebRTC API: The value of the rtc_configuration argument will be passed to the [`RTCPeerConnection`](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/RTCPeerConnection) constructor on the frontend.

### Configure the TURN server if necessary
Even if the STUN server is properly configured, media streaming may not work in some network environments.
For example, in some office or public networks, there are firewalls which drop the WebRTC packets.

In such environments, setting up a [TURN server](https://webrtc.org/getting-started/turn-server) is a solution. See https://github.com/whitphx/streamlit-webrtc/issues/335#issuecomment-897326755.

## Logging
For logging, this library uses the standard `logging` module and follows the practice described in [the official logging tutorial](https://docs.python.org/3/howto/logging.html#advanced-logging-tutorial). Then the logger names are the same as the module names - `streamlit_webrtc` or `streamlit_webrtc.*`.

So you can get the logger instance with `logging.getLogger("streamlit_webrtc")` through which you can control the logs from this library.

For example, if you want to set the log level on this library's logger as WARNING, you can use the following code.
```python
st_webrtc_logger = logging.getLogger("streamlit_webrtc")
st_webrtc_logger.setLevel(logging.WARNING)
```

In practice, `aiortc`, a third-party package this library is internally using, also emits many INFO level logs and you may want to control its logs too.
You can do it in the same way as below.
```python
aioice_logger = logging.getLogger("aioice")
aioice_logger.setLevel(logging.WARNING)
```

## API
Currently there is no documentation about the interface. See the example [app.py](./app.py) for the usage.
The API is not finalized yet and can be changed without backward compatiblity in the future releases until v1.0.

### For users since versions `<0.20`
`VideoTransformerBase` and its `transform` method have been marked as deprecated in v0.20.0. Please use `VideoProcessorBase#recv()` instead.
Note that the signature of the `recv` method is different from the `transform` in that the `recv` has to return an instance of `av.VideoFrame` or `av.AudioFrame`.

Also, `webrtc_streamer()`'s `video_transformer_factory` and `async_transform` arguments are deprecated, so use `video_processor_factory` and `async_processing` respectively.

See the samples in [app.py](./app.py) for their usage.

## Resources
* [Developing web-based real-time video/audio processing apps quickly with Streamlit](https://www.whitphx.info/posts/20211231-streamlit-webrtc-video-app-tutorial/)
  * A tutorial for real-time video app development using `streamlit-webrtc`.
  * Crosspost on dev.to: https://dev.to/whitphx/developing-web-based-real-time-videoaudio-processing-apps-quickly-with-streamlit-4k89
* [New Component: streamlit-webrtc, a new way to deal with real-time media streams (Streamlit Community)](https://discuss.streamlit.io/t/new-component-streamlit-webrtc-a-new-way-to-deal-with-real-time-media-streams/8669)
  * This is a forum topic where `streamlit-webrtc` has been introduced and discussed about.
