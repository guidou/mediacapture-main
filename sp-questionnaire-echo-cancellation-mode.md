# Security and Privacy questionnaire

### 2.1. What information does this feature expose, and for what purposes?

This feature adds the ability to specify what audio should be cancelled from
a microphone signal obtained with [geUserMedia](https://w3c.github.io/mediacapture-main/).
The following information is exposed, guarded by microphone permission and/or
utilization:
* The echo cancellation mode used by the a microphone track.
* The echo cancellation modes available for a given microphone or track.

This API makes it possible to know the pixel ratio of a captured
surface. This information will be gated by the display-capture permission,
which grants access to all the pixels in the captured surface, requires explicit
user authorization via a dialog, is not persistent, and expires once the capture
session ends.

### 2.2. Do features in your specification expose the minimum amount of information necessary to implement the intended functionality?
Yes.

### 2.3. Do the features in your specification expose personal information, personally-identifiable information (PII), or information derived from either?
No.

### 2.4. How do the features in your specification deal with sensitive information?
Although related to microphone capture, which is sensitive, this feature is an
addition to existing APIs that does not deal with sensitive information.

### 2.5. Does data exposed by your specification carry related but distinct information that may not be obvious to users?
No.

### 2.6. Do the features in your specification introduce state that persists across browsing sessions?
No.

### 2.7. Do the features in your specification expose information about the underlying platform to origins?
It exposes if the platform can support some echo cancellation modes, which might
not be available on all platforms. This is guarded by the microphone permission, though.

### 2.8. Does this specification allow an origin to send data to the underlying platform?
No.

### 2.9. Do features in this specification enable access to device sensors?
No.

### 2.10. Do features in this specification enable new script execution/loading mechanisms?
No.

### 2.11. Do features in this specification allow an origin to access other devices?
No.

### 2.12. Do features in this specification allow an origin some measure of control over a user agent’s native UI?
No.

### 2.13. What temporary identifiers do the features in this specification create or expose to the web?
None.

### 2.14. How does this specification distinguish between behavior in first-party and third-party contexts?
No distinction.

### 2.15. How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?
No distinction.

### 2.16. Does this specification have both "Security Considerations" and "Privacy Considerations" sections?
This is a minor addition to an existing specification. The existing specification has a "Privacy and security considerations" section.

### 2.17. Do features in your specification enable origins to downgrade default security protections?
No

### 2.18. What happens when a document that uses your feature is kept alive in BFCache (instead of getting destroyed) after navigation, and potentially gets reused on future navigations back to the document?
Nothing remarkable. Active microphone captures would end, but information about
available echo cancellation modes would continue to be available.

### 2.19. What happens when a document that uses your feature gets disconnected?
Microphone captures end, but nothing specific happens in relation to echo
cancellation modes.


### 2.20. Does your spec define when and how new kinds of errors should be raised?
This feature does not produce new kinds of errors other than existing overconstrained
errors in getUserMedia when an unsupported echo cancellation mode is requested.

### 2.21. Does your feature allow sites to learn about the user’s use of assistive technology?
No. In fact, it helps conceal the use of assistive technology by making it
easier for an application to cancel audio coming from screen readers, for example.

### 2.22. What should this questionnaire have asked?
The questions seem appropriate.
