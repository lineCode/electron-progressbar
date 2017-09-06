![npm](https://img.shields.io/npm/dt/electron-progressbar.svg?style=flat-square)
![Hits](https://hitt.herokuapp.com/AndersonMamede/electron-progressbar.svg)

electron-progressbar
================
> electron-progressbar is an easy way of showing progress bar window in Electron applications

electron-progressbar provides an easy-to-use and highly customizable API to show and control progress bars on Electron application.

You can customize windows (electron's BrowserWindow), progress bars' visual aspects (CSS), texts and also all visible information.

[![NPM](https://nodei.co/npm/electron-progressbar.png?downloads=true&stars=true)](https://www.npmjs.com/package/electron-progressbar)

Installation
------------

Install with `npm`:

``` bash
$ npm install electron-progressbar --save
```


Example of an **indeterminate** progress bar - used when your application can't calculate how long the process will last
-------

``` js
const {app} = require("electron");
const ProgressBar = require("electron-progressbar");

app.on("ready", function(){
  // electron-progressbar must be created only after electron application
  // is "ready" because electron's BrowserWindow module doesn't work before this;
  // alternatively, you can pass the "app" object (electron) to ProgressBar's
  // constructor as a second parameter, e.g. ProgressBar({options...}, app)
  var progressBar = new ProgressBar({
    indeterminate: true,
    text: "Preparing data...",
    detail: "Wait...",
    browserWindow: {
      icon: "icon.ico"
    }
  });
  
  progressBar
    .on("completed", function(value){
      console.info(`completed... ${value}`);
    })
    .on("aborted", function(value){
      console.info(`aborted... ${value}`);
    });
  
  // there is no value for indeterminate progress bar so it
  // should be just set as complete after all work is done
  setTimeout(function(){
    progressBar.complete();
  }, 3000);
});
```


Example of a **determinate** progress bar - used when your application can accurately calculate how long the process will last
-------

``` js
const {app} = require("electron");
const ProgressBar = require("electron-progressbar");

// instead of waiting for electron's "ready" event, we pass electron's app
// reference ("app") as a second parameter
var progressBar = new ProgressBar({
  maxValue: 120, // used to determine when process is completed; default: 100
  text: "Preparing data...",
  detail: "Wait...",
  browserWindow: {
    icon: "icon.ico"
  }
}, app);

progressBar
  .on("progress", function(value){
    progressBar.detail = `${value} / ${progressBar.getOptions().maxValue}`;
  })
  .on("completed", function(value){
    clearInterval(interval);
  })
  .on("aborted", function(value){
    console.info(`aborted... ${value}`);
  });

// update progress bar status;
// here we are just simulating a work being done
var interval = setInterval(function(){
  progressBar.value += 1;
}, 50);
```


## API

#### `constructor(options [,electronApp])`

Create a new progress bar. Because electron's BrowserWindow is used to display the progress bar and it only works after electron's "ready" event, you have wait for the "ready" event before creating a progress bar; optionally, you can just pass electron's app as a second parameter (`electronApp`).

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| [options] | <code>object</code> |  |  |
| [options.abortOnError] | <code>boolean</code> | <code>false</code> | Whether progress bar should abort and close if an error occurs internally. |
| [options.indeterminate] | <code>boolean</code> | <code>false</code> | Whether progress bar should be **indeterminate**. If false, progress bar will be **determinate** |
| [options.initialValue] | <code>number</code> | <code>0</code> | Progress bar's initial value. |
| [options.maxValue] | <code>number</code> | <code>100</code> | Progress bar's max value. When progress bar's value reaches this number, it will be set as completed and event `complete` will be fired. Used only for determinate progress bar. |
| [options.closeOnComplete] | <code>boolean</code> | <code>true</code> | Whether progress bar's window should be automatically closed after completed. If false, the progress bar must be manually closed by calling its `close` method. |
| [options.title] | <code>string</code> | <code>Wait...</code> | Text shown on the title bar. |
| [options.text] | <code>string</code> | <code>Wait...</code> | Text shown inside the window and above the progress bar. |
| [options.detail] | <code>string</code> | <code>null</code> | Text shown between the `text` and the progress bar. Used to display the current status, i.e., what part of the whole process is being done. Setting this property later should be more useful because your application can determine and display, in real time, what is currently happening. |
| [options.style] | <code>object</code> | <code></code> | Visual styles. All elements properties are purely CSS, just the way it is used in an CSS file. |
| [options.style.text] | <code>object</code> | <code></code> | An object containing any CSS properties for styling the text element. |
| [options.style.detail] | <code>object</code> | <code></code> | An object containing any CSS properties for styling the detail element. |
| [options.style.bar] | <code>object</code> | <code>{"width":"100%", "background-color":"#DEDEDE"}</code> | An object containing any CSS properties for styling the bar in the progress bar. |
| [options.style.value] | <code>object</code> | <code>{"background-color":"#22328C"}</code> | An object containing any CSS properties for styling the value in the progress bar. |
| [options.browserWindow] | <code>object</code> | <code></code> | [`Electron's BrowserWindow options`](https://github.com/electron/electron/blob/master/docs/api/browser-window.md#new-browserwindowoptions). Although only a few properties are used per default, you can specify any of [`Electron's BrowserWindow options`](https://github.com/electron/electron/blob/master/docs/api/browser-window.md#new-browserwindowoptions). |
| [options.browserWindow.parent] | <code>instance of BrowserWindow</code> | <code>null</code> | A BrowserWindow instance. If informed, the progress bar's window will block its parent window so user can't interact with it until the progress bar is completed or aborted. |
| [options.browserWindow.modal] | <code>boolean</code> | <code>true</code> | Whether this is a modal window. This actually only works when the window is a child window, i.e., when its `parent` is informed. |
| [options.browserWindow.resizable] | <code>boolean</code> | <code>false</code> | Whether window is resizable. |
| [options.browserWindow.closable] | <code>boolean</code> | <code>false</code> | Whether window is closable. |
| [options.browserWindow.minimizable] | <code>boolean</code> | <code>false</code> | Whether window is minimizable. |
| [options.browserWindow.maximizable] | <code>boolean</code> | <code>false</code> | Whether window is maximizable. |
| [options.browserWindow.width] | <code>number</code> | <code>450</code> | Progress bar window's width in pixels. |
| [options.browserWindow.height] | <code>number</code> | <code>175</code> | Progress bar window's height in pixels. |

#### `getOptions()` ⇒ <code>object</code>

Return a copy of all current options.

#### `on(eventName, listener)` ⇒ <code>reference to this</code>

Adds the listener function to the end of the listeners array for the event named `eventName`. No checks are made to see if the `listener` has already been added. Multiple calls passing the same combination of `eventName` and `listener` will result in the `listener` being added, and called, multiple times.

Returns a reference to `this` so that calls can be chained.

Available events:

| Event name | Receives parameter | Description |
| --- | --- | --- |
| progress | value | Available only for **determinate** progress bar. Fired every time the progress bar's value is changed. The listener receives, as first parameter, the current progress bar's value. |
| completed | value | Fired when progress bar is completed, i.e., its value reaches `maxValue` or method `close` is called. |
| aborted | value | Fired if progress bar is closed when it's not completed yet, i.e., when user closes progress bar's window or method `close` is called before progress bar is completed. |

#### `complete()`

Set progress bar as complete. This means the whole process is finished.

If progress bar is **indeterminate**, a manual call to this method is required because it's the only way to complete the process and trigger the `complete` event, otherwise the progress bar would be displayed forever.

#### `close()`

Close progress bar window. If progress bar is not completed yet, it'll be aborted and event `aborted` will be fired.

#### `isInProgress()` ⇒ <code>boolean</code>

Return true if progress bar is currently in progress, i.e., it hasn't been completed nor aborted yet, otherwise false.

#### `isCompleted()` ⇒ <code>boolean</code>

Return true if progress bar is completed, otherwise false.


## License ##

MIT. See [LICENSE.md](http://github.com/AndersonMamede/electron-progressbar/blob/master/LICENSE) for details.