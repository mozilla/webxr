# proposal for permissions request


### Philosophy
The user-agent will always be closer to the user than a spec. The user agent has greater access to current conditions than we (the spec writers) ever will. As a philosophy we should give the user agent *as much freedom as possible to “do the right thing”* under whatever circumstances are at the time a web application is started.

### scope

Currently we are having difficulties finding a solution that all parties can agree to. This is about the permissions for various properties in WebXR.  This proposal addresses the following concerns

* what permissions are available to be asked for
* how granular are these permissions
* are the permissions granted at page load / application start
* or are the permissions granted at the time of use.

### stating concerns

Concerns about granting at page load time are
* users may be requested to approve multiple permissions one after another, creating click fatigue.
* if all apps request permissions at start, the user has no information to judge whether or not they should grant this, because they haven’t entered the experience yet.

Concerns about granting requests once already inside of immersive mode are:
* access requests when inside immersive mode should be done with some sort of a secure method, such as a dedicated hardware button, so that applications cannot spoof such permissions.
* some devices or scenarios may not be able to provide such a secure method, thus doing the request before entering immersive mode is preferable.

Concerns about both systems include:
* the user agents know more about the user, the current scenario, and can make better decisions *at the time of use* than a spec can. Neither method give the user agent freedom to adapt.
* there may be new attacks devised or new flaws discovered after we have shipped, but user agents will not be able to address these because they are constrained by the spec.

There are most likely more concerns than just what I have listed above.

### Solution
To address these I propose the following: *do both*.  Let the developers have a single code path where they /request permissions at application start/ *and* when /the permission is actually needed/. The first request is considered the upper bounds of possible permissions. The user agent can decide whether to actually show user request dialogs at app time or when the permission is actually used, or some other scheme that the user agent deems to be better (such as warding off new attacks in the future).

### Details
From the developer’s perspective it would work like this.

The developer specifies the total permissions they are likely to need during the entire run of the application when requesting the XR Session, using an array with pre-defined permissions constants.  Something like this:

``` javascript
const perms = [
   MICROPHONE,
	 IMMERSIVE_MODE,
   HEADSET_POSE,
   //CAMERA_DATA,  developer decides not to use camera
]
navigator.xr.requestSession(perms).then(session => {
	//start up the rest of the app
})
```

At this point the user agent can either prompt the user for these permissions, or merely use them as an upper bound of permissions that could potentially be requested later.


Later, when the application needs to actually use a permission, say the microphone, the developer will request this access through a promise based API. Ex:

```javascript
navigator.mediaDevices.getUserMedia(opts).then(stream => {
	console.log("i have the microphone stream")
})
```

At this point the user agent can either prompt the user for the microphone permission, or immediately grant it if the user already approved it previously.  

Note that if the application did not include MICROPHONE as  the initial list of permissions to `requestSession` then the user agent *must* reject any later requests to `getUserMedia()`.  

*The initial list of permissions is the maximum set of permissions the application may request throughout it’s lifetime.*  This enables, say, a vr chat client to let the user enter a move and talk only mode and later request camera access because the user wants to share something, without having to bug the user initially; if the user agent supports immersive permission requests.  On user agents which do not support immersive permission requests, for whatever reason (form factor, user prefs, domain name, phase of moon, etc.), then the request for camera can be done at application start.


### Advantages
This system has two advantages over other approaches.
* the developer has a single code path that will always be followed.  They will not need to write conditional code to work around different user agent authentication systems.
* this gives the user agent maximum flexibility to adapt behavior to the current situation. 

User agents could implement many different behaviors which would be perfectly valid under this spec.

* request all permissions of the user at start, as a series of prompts
* collapse all permissions into a single prompt
* auto-approve certain permissions based on domain (ex: anything from mycompany.com is always approved)
* auto-approve certain permissions when others are requested. (ex: if dev asks for camera, we might as well give them microphone too)
* handle some permissions using a trusted immersive method such as an un-spoofable hardware button, or a sigil; but then use pre-immersive granting for certain permissions that are deemed extra dangerous.
* ignore permissions for a certain amount of time when set to kiosk mode


### Open questions:
* if an application wishes to request permissions that were previously out of the initial list of permissions (say a user creates an account and now wishes to increase the initial permissions).  We could advise the app to reload the page, forcing the user out of immersive mode.
* how do permissions transfer when jumping from one immersive page to another? Should they continue to be granted if on the same domain?  Auto-shut off? Always leave immersive mode when requesting particular permissions that are extra dangerous?
