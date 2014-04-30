Scrolling Oddity in Sencha Touch
================================

Demonstration of scrolling + external link issue

This repo contains an XCode Project for a simple Sencha Touch 2.3.1 app.  It is meant to demonstrate an issue that I am coming up against with InAppBrowser.

Steps to reproduce

1) Clone the repo
2) Open the xcodeproj file in XCode and run the app in the Simulator
3) Click on any of the images.  You should get a popup confirming that you are about to navigate to an external link and would you like to open your native browser.  This is the desired behaviour when you click on an image (they're all the same link by the way)
4) Now, flick the screen to scroll it and while it's still scrolling, tap the screen.  It may take a few tries, but you should eventually get the undesired behaviour which is that the web page opens up within the browser itself.

Note this really only manifests itself in the native app.  I'm wrapping it with Cordova 3.4.1 and using the InAppBrowser 0.4.1.  Further, the only reason I put that little confirmation popup in there was because if I didn't, sometimes the app opened the link in the browser.  

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

You can watch a [video of the issue here](http://youtu.be/hmCryCAm3U4).

Tough to tell where the issue lies, Sencha Touch? InAppBrowser?  Who knows....