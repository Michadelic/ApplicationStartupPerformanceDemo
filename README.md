# TechEdLatency
Latency can be a drag: Sample Code

Important: Set your browser language to en_US to avoid additional requests to i18n.properties files. No 404 response codes should appear in network trace.

Explain the code: 
- UI5 bootstrap, loading UI5 from local server (e.g. HCP instance where Web IDE is running)
- several libraries are loaded, also typical setup
- preload: sync (since this is the default and many still use it)

Run this app and show in network trace. 
- Show latency in network trace, should be around or above 170ms, sometimes significantly higher

<!DOCTYPE HTML>
<html>

	<head>
		<meta http-equiv="X-UA-Compatible" content="IE=edge" />
		<meta charset="UTF-8">

		<title>LatencyDemo</title>

		<script id="sap-ui-bootstrap"
			src="../../resources/sap-ui-core.js"
			data-sap-ui-libs="sap.m, sap.ui.table, sap.ui.commons, sap.ui.ux3"
			data-sap-ui-theme="sap_bluecrystal"
			data-sap-ui-compatVersion="edge"
			data-sap-ui-preload="sync"
			data-sap-ui-resourceroots='{"LatencyDemo": ""}'>
		</script>

		<script>

		jQuery.sap.require('LatencyDemo.util.File1');
		jQuery.sap.require('sap.ui.core.mvc.XMLView');

		new sap.ui.xmlview({viewName:"LatencyDemo.view.View1"}).placeAt("content");
				
		</script>
	</head>

	<body class="sapUiBody" id="content">
	</body>

</html>

Next step: Switch to async preloading

Observe in network trace that preload file are loaded in parallel, but total calls go up from 30 to almost 100. 

What happened? Both script tags are processed sequentially. While preload was still set to "sync" only sync XHr requests were sent to module loading was completed by the time the second script tag was reached. 
Now, script simply requests all preloads, but does not block until the reponses are loaded. Thus, any require call will trigger a modules loading cascade. 

<html>

	<head>
		<meta http-equiv="X-UA-Compatible" content="IE=edge" />
		<meta charset="UTF-8">

		<title>LatencyDemo</title>

		<script id="sap-ui-bootstrap"
			src="../../resources/sap-ui-core.js"
			data-sap-ui-libs="sap.m, sap.ui.table, sap.ui.commons, sap.ui.ux3"
			data-sap-ui-theme="sap_bluecrystal"
			data-sap-ui-compatVersion="edge"
			data-sap-ui-preload="async"
			data-sap-ui-resourceroots='{"LatencyDemo": ""}'>
		</script>

		<script>

		jQuery.sap.require('LatencyDemo.util.File1');
		jQuery.sap.require('sap.ui.core.mvc.XMLView');

		new sap.ui.xmlview({viewName:"LatencyDemo.view.View1"}).placeAt("content");
				
		</script>
	</head>

	<body class="sapUiBody" id="content">
	</body>

</html>

What is missing? The sync point offered by SAPUI5 is the init event. Add the statement as below. 

<html>

	<head>
		<meta http-equiv="X-UA-Compatible" content="IE=edge" />
		<meta charset="UTF-8">

		<title>LatencyDemo</title>

		<script id="sap-ui-bootstrap"
			src="../../resources/sap-ui-core.js"
			data-sap-ui-libs="sap.m, sap.ui.table, sap.ui.commons, sap.ui.ux3"
			data-sap-ui-theme="sap_bluecrystal"
			data-sap-ui-compatVersion="edge"
			data-sap-ui-preload="async"
			data-sap-ui-resourceroots='{"LatencyDemo": ""}'>
		</script>

		<script>

		sap.ui.getCore().attachInit(function(){
    		jQuery.sap.require('LatencyDemo.util.File1');
    		jQuery.sap.require('sap.ui.core.mvc.XMLView');
    
    		new sap.ui.xmlview({viewName:"LatencyDemo.view.View1"}).placeAt("content");		  
		});
				
		</script>
	</head>

	<body class="sapUiBody" id="content">
	</body>

</html>

Now already better, less requests. 

Next, switch to AKAMAI deployment. Now, all requests to SAPUI5 resources will go to nearest AKAMAI server.

- Show latency in network trace, should be around or below 30ms

<html>

	<head>
		<meta http-equiv="X-UA-Compatible" content="IE=edge" />
		<meta charset="UTF-8">

		<title>LatencyDemo</title>

		<script id="sap-ui-bootstrap"
			src="https://sapui5.hana.ondemand.com/resources/sap-ui-core.js"
			data-sap-ui-libs="sap.m, sap.ui.table, sap.ui.commons, sap.ui.ux3"
			data-sap-ui-theme="sap_bluecrystal"
			data-sap-ui-compatVersion="edge"
			data-sap-ui-preload="async"
			data-sap-ui-resourceroots='{"LatencyDemo": ""}'>
		</script>

		<script>

		sap.ui.getCore().attachInit(function(){
    		jQuery.sap.require('LatencyDemo.util.File1');
    		jQuery.sap.require('sap.ui.core.mvc.XMLView');
    
    		new sap.ui.xmlview({viewName:"LatencyDemo.view.View1"}).placeAt("content");		  
		});
				
		</script>
	</head>

	<body class="sapUiBody" id="content">
	</body>

</html>

Switch the project type to "SAPUI5 Client Build"

Deploy to HCP and notice in network that no component preload is used. Reason: No component used for application startup. 

<head>
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta charset="UTF-8">

    <title>LatencyDemo</title>

    <script id="sap-ui-bootstrap"
        src="https://sapui5.hana.ondemand.com/resources/sap-ui-core.js"
        data-sap-ui-libs="sap.m, sap.ui.table, sap.ui.commons, sap.ui.ux3"
        data-sap-ui-theme="sap_bluecrystal"
        data-sap-ui-compatVersion="edge"
        data-sap-ui-preload="async"
        data-sap-ui-resourceroots='{"LatencyDemo": ""}'>
    </script>

    <script>

    sap.ui.getCore().attachInit(function(){
    	
    	jQuery.sap.require('LatencyDemo.util.File1');
    	jQuery.sap.require('sap.ui.core.mvc.XMLView');   	

	new sap.ui.core.ComponentContainer({
            name : "LatencyDemo"
        }).placeAt("content");

    });

    </script>
</head>

<body class="sapUiBody" id="content">
</body>

Again, deploy and show in network trace. Notice how still serveral application resources are loaded which are however part of the component preload. The reason for this is that the require calls are done in the init event, but are in fact related to the "second" sync point in terms of preloading - the component init. All required which are related to application code must be in the init or later. 

Next, move the require calls to the component.js

```javascript
sap.ui.define([
	"sap/ui/core/UIComponent",
	"sap/ui/Device",
	"LatencyDemo/model/models"
], function(UIComponent, Device, models, File1, XMLView) {
	"use strict";

	return UIComponent.extend("LatencyDemo.Component", {

		metadata: {
			manifest: "json"
		},

		/**
		 * The component is initialized by UI5 automatically during the startup of the app and calls the init method once.
		 * @public
		 * @override
		 */
		init: function() {
			// call the base component's init function
			UIComponent.prototype.init.apply(this, arguments);
			
			jQuery.sap.require('LatencyDemo.util.File1');
    			jQuery.sap.require('sap.ui.core.mvc.XMLView');  

			// set the device model
			this.setModel(models.createDeviceModel(), "device");
		}
	});

});
```

And remove the calls from the index.html
<head>
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta charset="UTF-8">

    <title>LatencyDemo</title>

    <script id="sap-ui-bootstrap"
        src="https://sapui5.hana.ondemand.com/resources/sap-ui-core.js"
        data-sap-ui-libs="sap.m, sap.ui.table, sap.ui.commons, sap.ui.ux3"
        data-sap-ui-theme="sap_bluecrystal"
        data-sap-ui-compatVersion="edge"
        data-sap-ui-preload="async"
        data-sap-ui-resourceroots='{"LatencyDemo": ""}'>
    </script>

    <script>

    sap.ui.getCore().attachInit(function(){
    	
		new sap.ui.core.ComponentContainer({
            name : "LatencyDemo"
        }).placeAt("content");

    });

    </script>
</head>

<body class="sapUiBody" id="content">
</body>

Now, deploy again and check the results in the network trace.

Final version: 
- Preload files for SAPUI5 are loaded from AKAMAI with low latency and in parallel
- Preload file for application is loaded, no other application modules are requested any more by the browser

