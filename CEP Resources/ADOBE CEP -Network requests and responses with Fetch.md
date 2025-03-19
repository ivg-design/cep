# Network requests and responses with Fetch

Many Creative Cloud app extensions require the ability to talk to API services on the web. With both Chromium Embedded Framework (CEF) and Node.js at its core, CEP gives you the flexibility to make network calls from within your extension in the way that makes sense for your workflow.

In this guide, we will cover how to call a third-party API service, update the panel UI according to the API response, and interact with the host app's creative asset based on the API response.

![](readme-assets/weather-panel-ps.png)

By the end of this guide, we will have a CEP extension for Photoshop and InDesign that:

1. Calls the Dark Sky API to get the current weather for a particular city.
1. Displays a string in the panel UI that tells the user the current weather.
1. Lets a user click a button to dynamically alter the open asset based on the weather.

<!-- doctoc command config: -->
<!-- $ doctoc ./readme.md --title "## Contents" --entryprefix 1. --gitlab --maxlevel 2 -->

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Contents

1. [Technology Used](#technology-used)
1. [Prerequisites](#prerequisites)
1. [Configuration](#configuration)
1. [Client-side: HTML Markup](#client-side-html-markup)
1. [Client-side: Service API interaction](#client-side-service-api-interaction)
1. [Client-side: Creative Cloud host app interaction](#client-side-creative-cloud-host-app-interaction)
1. [Host app: Automation with ExtendScript](#host-app-automation-with-extendscript)
1. [Other Resources](#other-resources)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## Technology Used

- Supported Host Applications: Photoshop, InDesign
- Libraries/Frameworks/APIs:
    - Adobe-specific: [CEP](https://www.adobe.io/apis/creativecloud/cep.html), ExtendScript for Photoshop and InDesign
    - Other: [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API), [Dark Sky API](https://darksky.net/dev)


## Prerequisites

This guide will assume that you have installed all software and completed all steps in the following guides:

- [Getting Started](../readme.md)
- [Debugging your Adobe panel](https://medium.com/adobe-io/debugging-your-adobe-panel-cf73f00f6961)


## Configuration

### Set up the sample extension

The following steps will help you get the sample extension for this guide up and running:

1. Install the `./com.cep.weather-simple/` directory in your `extensions` folder. ([See the Cookbook](https://github.com/Adobe-CEP/CEP-Resources/blob/master/CEP_8.x/Documentation/CEP%208.0%20HTML%20Extension%20Cookbook.md#extension-folders) if you are unsure where your `extensions` folder is.)
1. [Download CEP's `CSInterface.js` library](https://github.com/Adobe-CEP/CEP-Resources/blob/master/CEP_8.x/CSInterface.js) and move it to `./com.cep.weather-simple/client/js/lib/CSInterface.js`.
1. Get a free API Key from [Dark Sky](https://darksky.net/dev).
1. Make a file named `config.js` at this path: `./com.cep.weather-simple/client/js/config.js`.
1. In your newly created `config.js` file, add this code, substituting in your Dark Sky API Key:

    ```
    const darkSkyKey = "YOUR_API_KEY";
    ```

After following these steps, you'll be able to run the sample extension within the host apps indicated in the [Technology Used](#technology-used) section of this guide.


### Configure `manifest.xml`

As noted in the [Getting Started guide](../readme.md), the `manifest.xml` file is where, among other things, you indicate which Creative Cloud host apps and version numbers your extension supports.

For this guide, we'll make an extension that supports Photoshop and InDesign. So in the `manifest.xml`, make sure you list the supported host apps within the `<HostList>` element:

```xml
<!-- ...  -->
<ExecutionEnvironment>
  <HostList>
    <Host Name="PHSP" Version="19" /> <!-- Photoshop -->
    <Host Name="PHXS" Version="19" /> <!-- Photoshop -->
    <Host Name="IDSN" Version="13" /> <!-- InDesign -->
  </HostList>

  <!-- // ... -->
</ExecutionEnvironment>

<!-- // ... -->

```

Note that the versions indicted in the example code above only target a single version of each host app, for the sake of demo simplicity. Most extension developers will want to target a range of host app versions. To learn how to support multiple host app versions, [see the Cookbook](https://github.com/Adobe-CEP/CEP-Resources/blob/master/CEP_8.x/Documentation/CEP%208.0%20HTML%20Extension%20Cookbook.md#extension-manifest).



## Client-side: HTML Markup

The user interface for CEP extensions is written in HTML. For this sample, the HTML document is located at `./com.cep.weather-simple/client/index.html` and contains the following code (see comments **#1-3**):

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>Weather Simple</title>
	<link rel="stylesheet" href="styles/style.css">
</head>
<body>
  <!-- 1) An empty paragraph element -->
	<p id="weather-summary"></p>

  <!-- 2) A button -->
	<button id="apply-weather-button">Apply weather</button>

  <!-- 3) Your scripts, including CEP's CSInterface.js library -->
	<script type="text/javascript" src="js/lib/CSInterface.js"></script>
	<script type="text/javascript" src="js/config.js"></script>
	<script type="text/javascript" src="js/index.js"></script>
</body>
</html>
```

The document body in the code above contains two simple HTML elements: `button` and `p`, both uniquely identifiable by `id` attributes that we'll use in the steps below.

Note that, while this guide will not cover styling an extension with CSS, the CSS stylesheet for this sample panel can be found at `./com.cep.weather-simple/client/styles/style.css`.


## Client-side: Service API interaction

As we saw in the previous section's `index.html` code, the client-side JavaScript for this extension is located at `./com.cep.weather-simple/client/js/index.js`. We will look at this `index.js` file in this section.

Note that nothing in this section is specifically unique to CEP. This section is all about client-side JavaScript that will set the stage for interacting with CEP features in the next sections.


### Create references to the UI elements

In `index.js`, we'll first reference the elements in our `index.html`:

```javascript
const applyWeatherButton = document.querySelector("#apply-weather-button");
const weatherSummary = document.querySelector("#weather-summary");
```

We'll work with these UI elements in the next step.


### Map API responses to user-readable strings

As we will see in the next step, the Dark Sky API JSON response will include an `.icon` property that indicates the weather. The value of the `.icon` property will be a kebob-case slug, like `"clear-day"`.

Let's map all the possible slugs to user-readable strings in a constant:

```javascript
const weatherTypes = {
  "clear-day": "a clear day",
  "clear-night": "a clear night",
  "rain": "raining",
  "snow": "snowing",
  "sleet": "sleeting",
  "wind": "windy",
  "fog": "foggy",
  "cloudy": "cloudy",
  "partly-cloudy-day": "a partly cloudy day",
  "partly-cloudy-night": "a partly cloudy night"
}
```

This makes it convenient to translate an API response like `"clear-day"` to something that's more natural for the user to read, like `"It looks like it's a clear day."`.

We'll use this `weatherTypes` constant in the next step.


### Request and display the weather for a city

Our extension will automatically get and display the weather for New York City when it loads.

![](readme-assets/weather-panel-ps.png)

In `index.js`, let's make a `getWeather()` helper method that is immediately invoked. See comments **#1-8** in the code below:

```javascript
(function getWeather() {
  /* 1) Create an object to contain relevant city data */
  let cityObj = {name: "New York", lat: "40.40", lon: "-73.56"};

  /* 2) Set up the request URL for the Dark Sky API */
  let reqUrl = `https://api.darksky.net/forecast/${darkSkyKey}/${cityObj.lat},${cityObj.lon}`;

  /* 3) Make a request to the URL constructed above */
  fetch(reqUrl)
    .then(function(res) {
      /* 4) If the response is ok, return the Dark Sky JSON response */
      if (res.ok) return res.json();
    })
    .then(function(json) {
      /* 5) Get the current weather slug from the Dark Sky JSON response */
      const currentWeatherSlug = json.currently.icon;

      /* 6) Display the weather information in the panel's HTML DOM */
      const currentWeatherString = `It looks like it\'s ${weatherTypes[currentWeatherSlug]} in ${cityObj.name}.`;
      weatherSummary.textContent = currentWeatherString;

      /* 7) Add weather information to the button dataset for convenient access later */
      applyWeatherButton.dataset.currentWeatherSlug = currentWeatherSlug;
      applyWeatherButton.dataset.currentWeatherString = currentWeatherString;
    })
    .catch(function(err) {
      /* 8) Handle errors */
      weatherSummary.textContent = `We had trouble getting the weather for ${cityObj.name}. Please try again.`;
    });
})();
```

In step **#3**, we take advantage of [the `fetch()` API](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch), provided by Chromium Embedded Framework in CEP, to make a network request. The `fetch()` API returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise), which is why we follow up the request with `.then()/.catch()` syntax.

In step **#6**, we used our `weatherTypes` constant created in the previous step to get the user-readable string.

![](readme-assets/weather-panel-ps.png)

If the call is successful, you'll see a string in the panel UI that tells you the current weather in New York City. Otherwise, you'll see a string that says `"We had trouble getting the weather for New York City. Please try again."`.


### Alternatives to `fetch()`

Note that `fetch()` is not the only way that CEP gives you to make network requests.

Since Chromium Embedded Framework is essentially a browser, you can use an `XMLHttpRequest` (or a client-side library that wraps it, such as jQuery). You can also take advantage of Node.js within CEP, which gives you even more alternatives for making network requests.


## Client-side: Creative Cloud host app interaction

### Instantiate `CSInterface`

For any CEP panel, you'll need an instance of `CSInterface`, which, among other things, gives you a way to communicate with the host app's scripting engine:

```javascript
const csInterface = new CSInterface();
```

We'll make use of this `csInterface` constant later on.


### Add a click handler to the button

We'll add a click handler to `applyWeatherButton`:

```javascript
applyWeatherButton.addEventListener("click", applyWeatherToAsset);
```

We'll make the `applyWeatherToAsset()` helper method in the next step.


### Communicate with the host app

To communicate with the host app's scripting engine, we'll make use of the `csInterface.evalScript()` method. (If you need a refresher on the `.evalScript()` method, refer to the [Getting Started guide](../readme.md).)

In this sample app, our `.evalScript()` call will be wrapped in a helper method called `applyWeatherToAsset()`, which we attached to our button's click handler in the step above. See comments **#1-3** in the code below:

```javascript
function applyWeatherToAsset(e) {
  /* 1) Get the dataset from the click event */
  const dataset = e.target.dataset;

  /* 2) Pass an ExtendScript function call to the host app with .evalScript() */
  csInterface.evalScript(`applyWeatherToAsset("${dataset.currentWeatherSlug}", "${dataset.currentWeatherString}")`);
}
```

We could interpret the `.evalScript()` call in the last line of code above as meaning:

> Hey host app, call the `applyWeatherToAsset()` function from my ExtendScript file.

We’ll need to make sure that ExtendScript function exists in the next section.

## Host app: Automation with ExtendScript

Many CC host apps like Photoshop and InDesign (and many more!) can be automated with ExtendScript. In this sample extension, we're going to alter the active asset in Photoshop or InDesign based on the weather.

Note once again that this sample app is extremely simple for the purpose of focus. ExtendScript provides many deep features to automate work in CC host apps; you can explore more ExtendScript features in our Scripting Guides.

The ExtendScript file for this extension is located at `./com.cep.weather-simple/host/index.jsx`.


### Create an ExtendScript function

In the extension's `index.jsx` file, let's create our function declaration:

```javascript
function applyWeatherToAsset(currentWeatherSlug, currentWeatherString) {

  // Code will go here

}
```

The function takes the `currentWeatherSlug` and `currentWeatherString` string arguments that we provided in the client-side `.evalScript()` call.


### Create document element references

Through ExtendScript, we can, among other things, interact with the host app's DOM to interact with elements in an open document. The available DOM properties and methods vary for each host app, so we'll make use of the global `app.name` property to determine which app the extension is running in (see comments **#1-4**):

```javascript
function applyWeatherToAsset(currentWeatherSlug, currentWeatherString) {
  var layer;
  var frame;

  /* 1) If we're running in Photoshop */
  if (app.name === "Adobe Photoshop") {

    /* 2) Make a reference to the first art layer of the first open document */
    layer = app.documents[0].artLayers[0];
  }
  /* 3) Otherwise, if we're running in InDesign */
  else if (app.name === "Adobe InDesign") {

    /* 4) Create a text frame on the first page of the first open document */
    frame = app.documents.item(0).pages.item(0).textFrames.add({geometricBounds: [72, 72, 110, 300]});
  }

  // To be continued...
}
```

So depending on the app the extension is running in, we'll either reference an art layer (Photoshop) or create a text frame (InDesign).


### Alter the open asset in the host app

As a final step, we'll alter the open asset depending on the weather. We'll make a `switch` statement to control what happens based on the `currentWeatherSlug` (see comments **1-8**):

```javascript
function applyWeatherToAsset(currentWeatherSlug, currentWeatherString) {
  var layer;
  var frame;

  if (app.name === "Adobe Photoshop") {
    layer = app.documents[0].artLayers[0];
  }
  else if (app.name === "Adobe InDesign") {
    frame = app.documents.item(0).pages.item(0).textFrames.add({geometricBounds: [72, 72, 110, 300]});
  }

  /* 1) Switch based on the currentWeatherSlug we got from the Dark Sky API */
  switch (currentWeatherSlug) {

    /* 2) If it's a clear day */
    case "clear-day":
      /* 3) If we're running in Photoshop */
      if (app.name === "Adobe Photoshop") {
        /* 4) Adjust the color balance of the art layer */
        layer.adjustColorBalance([0,0,0], [-10,0,0], [-50,0,0]);
      }
      /* 5) If it's InDesign */
      else if (app.name === "Adobe InDesign") {
        /* 6) Use the currentWeatherString as the text frame contents */
        frame.contents = currentWeatherString;
      }
      break;

    /* 7) Repeat for clear night... */
    case "clear-night":
      if (app.name === "Adobe Photoshop") {
        layer.adjustColorBalance([0,0,33], [-39,-14,100], [-10,0,17]);
      }
      else if (app.name === "Adobe InDesign") {
        frame.contents = currentWeatherString;
      }
      break;

    /* 8) And so on for all possible currentWeatherSlug strings... */
    }
}
```

When you click the button, the output will look like this:

![](readme-assets/photoshop.png)
![](readme-assets/indesign.png)

In other words, if the panel is running in Photoshop, we'll adjust the color balance of an art layer; if the panel is running in InDesign, we'll add a text frame to the document that tells us the weather.


## Other Resources
- [CEP Cookbook](https://github.com/Adobe-CEP/CEP-Resources/blob/master/CEP_8.x/Documentation/CEP%208.0%20HTML%20Extension%20Cookbook.md)
