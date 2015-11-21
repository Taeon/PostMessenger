# PostMessenger
Simple-to-use but elegant wrapper around JavaScript's postMessage() function, allowing communication between parent/iframe and parent/popup. Demo [here](https://taeon.github.io/PostMessenger/).

PostMessenger encourages secure practices by specifying the domain for both parties, and tying listeners to specific elements rather than listening to all incoming postMessage() events indiscriminately.

Fully compatible with IE10+, Chrome, Safari, iOS, Firefox (see caniuse: http://caniuse.com/#search=postMessage). Works in IE8/9, with limitations (see below).

No external dependencies, but supports jQuery-like syntax for element selection, or will accept a jQuery object.

Minified version weighs in at about 1.5KB, &lt;lKB GZipped.

Basic usage in an iframe
------------------------

Once you've included the Javascript, just instantiate like this:

    // In the outer (parent) page
    var PM = new PostMessenger( 'http://mydomain.com', '#my-iframe' );
    PM.AddListener( function( message, message_event ){ alert( 'Parent got a message' + message ); } );

    // In the iframe
    var PM = new PostMessenger( 'http://mydomain.com', parent );
    PM.AddListener( function( message, message_event ){ alert( 'Iframe got a message' + message ); } );

Now you can send like so (it works the same in the parent or the child):

    PM.Send( 'Hello there!' );

Note that you can send JSON data, as well as strings.

Namespaces and multiple listeners
---------------------------------

You can add as many listeners as you like, which when combined with namespaced messages, means you can direct different messages to different listeners:

    // In the parent
    var PM = new PostMessenger( 'http://mydomain.com', '#my-iframe' );
    PM.AddListener( function( message, message_event ){ alert( 'Parent got an important message' + message ); }, 'Important' );
    PM.AddListener( function( message, message_event ){ alert( 'Parent got a trvial message' + message ); }, 'Trivial' );

    // In the iframe
    var PM = new PostMessenger( 'http://mydomain.com', parent );
    PM.Send( "This is an important message!", "Important" );
    PM.Send( "This is a trivial message", "Trivial" );

Alternatively, you can set a namespace for the PostMessenger instance when you create it, and it will only send and receive using that namespace:

    // In the parent
    var PM = new PostMessenger( 'http://mydomain.com', '#my-iframe', 'Important' ); // Only listen for/send Important messages
    PM.AddListener( function( message, message_event ){ alert( 'Parent got an important message' + message ); } );

    // In the iframe
    var PM = new PostMessenger( 'http://mydomain.com', parent );
    PM.Send( "This is an important message!", "Important" ); // Will work
    PM.Send( "This is a trivial message", "Trivial" ); // Ignored

Domains, sources and security
-----------------------------

In truth, it's not required to set a domain when using postMessage(), but PostMessenger will default to the current domain if you don't provide one. PostMessenger checks the domain for incoming messages and disregards any that don't match, and also uses it when sending so that for example if the page in the iframe has changed to a different domain (or an iframe is being included inside a page hosted on different domain), it won't send.

If you don't care about the domain, use '*':

    var PM = new PostMessenger( '*', '#my-iframe' );

...but this is to be avoided if possible.

You don't HAVE to provide an element when listening (although obviously you do when sending). If you do provide an element to listen on, PostMessenger will compare this with the 'source' parameter of the postMessage() call to ensure that the message is coming from the expected element. 

This will listen to all messages from any domain and any element:

    var PM = new PostMessenger( '*', null );
    PM.AddListener( function( message, message_event ){ alert( 'Parent got a message' + message ); } );

...although clearly you wouldn't be able to send because you haven't specified an element to send to. Again, this is not necessarily an advisable approach.

Usage in a popup
----------------

See limitations, below. The main difference is how you refer to the parent:

    // In the outer (parent) page
    var my_popup = window.open( 'http://mydomain.com', '', 'width=500,height=500' );
    var PM = new PostMessenger( 'http://mydomain.com', my_popup );
    PM.AddListener( function( message, message_event ){ alert( 'Parent got a message' + message ); } );

    // In the popup
    var PM = new PostMessenger( 'http://mydomain.com', window.opener );
    PM.AddListener( function( message, message_event ){ alert( 'Popup got a message' + message ); } );

Known issues/limitations:
-------------------------

**All browsers:**
 - Cross-domain messaging (i.e sending/receiving messages between pages hosted on two different domains) is only possible when using iframes. It will not work when using popup windows.
 - JSON data will be dereferenced, meaning that changes to an object sent via postMessage after sending will not be reflected on the receiving page. Even if the script didn't use JSON.stringify/parse to serialize the data (for cross-browser compatibility) this would still be the case, as browsers will clone the data for security reasons.

**IE8/9**
 - Due to limitations of postMessage() in these browsers, messaging between a page and a popup is not supported at all.