Scrolling Oddity in Sencha Touch
================================

Demonstration of scrolling + external link issue

This repo contains an XCode Project for a simple Sencha Touch 2.3.1 app.  It is meant to demonstrate an issue that I am coming up against with InAppBrowser.

###Update / Resolution###
See the Sencha Forum Threads [Cordova InAppBrowser Strangeness](http://www.sencha.com/forum/showthread.php?273944-Cordova-InAppBrowser-Strangeness) and [Links in HTML Page in Native App](http://www.sencha.com/forum/showthread.php?284954-Links-in-HTML-Page-in-Native-App).  

I finally figured out a way around this.  Insead of listening for an event on the Viewport component, I'm now listening for an event on the actual body element

	// Need to make sure we're only responding to a tap, not a drag.  
	Ext.getBody().dom.addEventListener( 'mousedown', function(e){
		var el = Ext.fly( e.target );
		if ( el.is( 'a[target="_blank"]' ) || el.parent( 'a[target="_blank"]' ) ){
			this.startPoint = { x : e.screenX, y: e.screenY }
		}
	});
	Ext.getBody().dom.addEventListener( 'click', function(e){
		var el = Ext.fly( e.target );
		if ( this.startPoint ){
			e.preventDefault();
			if ( ( Math.abs( this.startPoint.x - e.screenX ) < 8 )
			&&   ( Math.abs( this.startPoint.y - e.screenY ) < 8 ) ){
			  	window.open( el.dom.href || el.parent().dom.href, "_system" );
			}
			delete this.startPoint;
			return false;
		}
	});


###Steps to reproduce###

1. Clone the repo
2. Open the xcodeproj file in XCode and run the app in the Simulator
3. Click on any of the images.  You should get a popup confirming that you are about to navigate to an external link and would you like to open your native browser.  This is the desired behaviour when you click on an image (they're all the same link by the way)
4. Now, flick the screen to scroll it and while it's still scrolling, tap the screen.  It may take a few tries, but you should eventually get the undesired behaviour which is that the web page opens up within the browser itself.

*Note:* This really only manifests itself in the native app.  I'm wrapping it with Cordova 3.4.1 and using the InAppBrowser 0.4.1.  The issue occurs both in the simulator and on the real device.

Further, the only reason I put that little confirmation popup in there in the first place was because if I didn't, sometimes the app opened the link in the app itself.  It's like InAppBrowser is being bypassed.

Here's the relevant Sencha Touch code, at line 39344 of app.js

	Ext.Viewport.onBefore(
		'tap',
		function(e){
			e.preventDefault();
			the_app.app.confirm(
				{
					id: 'update', 
					title: WP.__("Leaving App"),
					html: WP.__("You have requested to view a web page outside of this app.  Continue to your default browser?"),
					hideOnMaskTap: false,
					handler: function(){
						// sometimes it's a lower down element that is the target (i.e. an <img> tag).  If that's the case, then we need to
						// go up the DOM until we find the A.  
						window.open( e.target.href || Ext.fly( e.target ).parent( 'a', true ).href, "_system");
					}
				}
			);
			return false;
		},
		this,
		{
			delegate: 'a[target="_blank"]',
			element: 'element'
		}
	);

You can watch a [screencapture of the issue here](http://youtu.be/hmCryCAm3U4).

Tough to tell where the issue lies, Sencha Touch? InAppBrowser?  Who knows....