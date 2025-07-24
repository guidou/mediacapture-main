# Echo Cancellation Mode

## Authors:

- Olga Sharanova (Google)
- Guido Urdaneta (Google)

## Participate
- https://github.com/w3c/mediacapture-main


## Introduction

The [getUserMedia](https://w3c.github.io/mediacapture-main/) API allows users to
share their microphone and camera with remote users, typically in
videoconferencing scenarios that involve [WebRTC](https://w3c.github.io/webrtc-pc/).

In many scenarios, it is desirable for user agents to remove echo (that is,
audio received from remote users and typically played out on loudspeakers)
from the microphone signal to ensure clear communication.

getUserMedia allows an application to enable or disable [echo cancellation](https://w3c.github.io/mediacapture-main/#def-constraint-echoCancellation)
via [constraints](https://w3c.github.io/mediacapture-main/#constrainable-interface).

This proposal adds the ability for Web applications to better control how much
of the user system playout is removed from the microphone signal.


## User-Facing Problem

In most cases when a web application requires echo cancellation to be applied to
a microphone signal, it wants as much of a user system playout as possible to be
removed from the microphone; at minimum the sound from incoming audio
[MediaStreamTracks](https://www.w3.org/TR/mediacapture-streams/#dom-mediastreamtrack)
sourced from WebRTC [RTCPeerConnections](https://www.w3.org/TR/webrtc/#dom-rtcpeerconnection)
needs to be removed, to facilitate 2-way RTC communication.

On the other hand, there are cases where it is not needed and it is desirable
to turn it off so that no audio artifacts are introduced.
The presence/absence of echo cancellation can be controlled by setting
the [echoCancellation](https://www.w3.org/TR/mediacapture-streams/#dfn-echocancellation)
constraintable property to `true` or `false`.

However:
* Some applications/use cases may be more privacy sensitive than others, and the
  only acceptable option for them would be removing all user system playoud from
  the microphone, to ensure no privacy-sensitive playout (screen readers,
  system notifications, etc.) is leaking via microphone.
* In other cases applications may want to remove echo from
  [RTCPeerConnections](https://www.w3.org/TR/webrtc/#dom-rtcpeerconnection),
  to enable 2-way RTC communication, while still capturing the rest of the local
  playout. One example is a remote music class, where a student sings along with
  some accompaniment produced by a local application. In this case, the Web
  application requires audio coming from the remote participant (i.e., the
  teacher) to be cancelled in order to avoid echo, but also requires that the
  accompaniment not be cancelled since the music teacher on the remote side
  needs to hear it together with the singing.

So, we are extending the set of values for
[echoCancellation](https://www.w3.org/TR/mediacapture-streams/#dfn-echocancellation)
with [EchoCancellationModeEnum](https://www.w3.org/TR/mediacapture-streams/#dom-echocancellationmodeenum),
allowing the web application to specify the desired echo cancellation behavior.

## Proposed approach
The proposed approach is to extend the type of the 
[echoCancellation](https://www.w3.org/TR/mediacapture-streams/#dfn-echocancellation)
constraintable property from `boolean` to `(boolean or EchoCancellationModeEnum)`.
In practice, this means adding two additional possible values to the 
`echoCancellation` property so that applications can support the use cases
mentioned above. The new set of values supported by the echoCancellation
property is:
* `false`: echo cancellation is disabled.
* `"all"`: The user agent must attempt to cancel all audio played out by the
   system, including audio output from `RTCPeerConnections`, screen readers and
   system notifications. This value aims to provide a higher level of privacy.
* `"remote-only"`: The user agent must attempt to cancel only played out audio
  from incoming `RTCPeerConnections`.
* `true`: The user agent decides what to cancel. It must attempt to cancel at
   least played out audio from incoming `RTCPeerConnections`, but it may also
   cancel other played out audio.

### Examples

* Checking if the user user agent can remove all user system playout  from the
  microphone

```js
const devices = await navigator.mediaDevices.enumerateDevices();
const mic = devices.filter(device => device.kind === 'audioinput')[0];
if (mic.getCapabilities().echoCancellation.indexOf("all") >= 0) {
   // Supported.
}
```

* Requesting all user system playout to be cancelled from the microphone
```js
const constraints = {
    audio: {
      echoCancellation: { exact: "all" }
    },
    video: false
  };


try { 
const stream = await navigator.mediaDevices.getUserMedia(constraints);
} catch (error) {
    if (error.name === 'OverconstrainedError') {
      // Not supported    
    }
}
```
* Requesting cancellation of only RTCPeerConnection playout
```js
const constraints = {
    audio: {
      echoCancellation: { exact: "remote-only" }
    },
    video: false
  };


try { 
const stream = await navigator.mediaDevices.getUserMedia(constraints);
} catch (error) {
    if (error.name === 'OverconstrainedError') {
      // Not supported    
    }
}
```

* Existing behavior: echo cancellation mode is chosen by the user agent
```js
const stream = await navigator.mediaDevices.getUserMedia({audio:{echoCancellation:true}});
```

* Existing behavior: disable echo cancellation
```js
const stream = await navigator.mediaDevices.getUserMedia({audio:{echoCancellation:false}});
```

### API
```webidl
enum EchoCancellationModeEnum {
  "all",
  "remote-only"
};

dictionary MediaTrackCapabilities {
  sequence<(boolean or DOMString)> echoCancellation;
}

dictionary MediaTrackSettings {
  (boolean or DOMString) echoCancellation;
}

dictionary ConstrainBooleanOrDOMStringParameters {
  (boolean or DOMString) exact;
  (boolean or DOMString) ideal;
};

typedef (boolean or DOMString or ConstrainBooleanOrDOMStringParameters) ConstrainBooleanOrDOMString;

dictionaty MediaTrackConstraintSet {
  ConstrainBooleanOrDOMString echoCancellation;
}
```

## Alternatives considered

### Additional echoCancellationMode constrainable property

This alternative leaves the echoCancellation property unmodified with type
`boolean`, and defines a new property of type `EchoCancellationMode` with two
possible values `"all"` and `"remote-only"`.

This solution would have worked similarly to the existing solution, but it
would introduce complexity for developers and implementers due to the
possibility of having contradicting or seemingly contradicting values for
the different properties.

For example, `getUserMedia{audio: {echoCancellation: {exact:false}, echoCancellationMode: {exact: "all"}})`
would fail with `OverconstrainedError`. 
However,  `getUserMedia{audio: {echoCancellation: false, echoCancellationMode: "all"}`, 
would succeed, as both values are ideal and therefore the call cannot fail.
The fitness distance for enabling or disabling is the same, so it is up
to the user agent to decide if it should do echo cancellation or not.
This problem is not unsurmountable, but it is more complex than the proposed
solution.

### Replace the type of `echoCancellation`

This alternative replaces the type of the echoCancellation property from
`boolean` to an `EchoCancellationMode` enum with four possible values:
`"all"`, `"remote-only"`, `"enabled"` and `"disabled"`.

This is a bit simpler than the proposed solution in that there are four possible
values, all of the same type. Therefore, it can solve the problem in exactly
the same way. The problem is that it is incompatible with existing applications
that assume `echoCancellation` is `boolean`, so it is not viable.

Other combinations of alternatives 1 and 2 are possible, but they all have
downsides. 

The proposed solution has the following advantages:
* It is compatible with existing applications
* All existing implementations are compliant, provided they do not report
 `"all"` and `"remote-only"` as capabilities.

The main downside of the proposed solution is that the property accepts values
of different types, but that is a very minor issue in practice as this is
directly supported by JavaScript.


## Accessibility, Privacy, and Security Considerations

The feature provides privacy guarantees when it matters for the user scenarios
by removing local playout from the microphone, while allowing flexibility
for scenarios where removing all system playout is undesirable.

From an accessibility perspective, it allows web applications to better
accommodate screen reader users:
* Cancelling all system playout means screen reader audio is removed from the
  microphone. 
* If it is not the default user agent behavior, the web application can request
  it specifically on behalf of a screen reader user, and if it is not available
  the web application can warn the user about that. 

## Stakeholder Feedback / Opposition
This proposal has had positive feedback from all major browser implementers and
has been integrated into the [Media Capture and Streams](https://www.w3.org/TR/mediacapture-streams) specification.
