#PDF Reader

## Description

This module extends the android project here - https://github.com/joniks/Android-MuPDF/

## Usage v1.9

```javascript
var enabled = true;
var count = 0;
var keyword = "the";

var win = Ti.UI.createWindow({
	backgroundColor : 'white',
	exitOnClose : true
});

var READER_MODULE = require("com.mykingdom.mupdf");

//Make sure the file exists
var file = Ti.Filesystem.getFile(Ti.Filesystem.externalStorageDirectory, "sample.pdf");

if (!file.exists()) {
	var source = Ti.Filesystem.getFile(Ti.Filesystem.resourcesDirectory, "sample.pdf");
	file.write(source.read());
}

console.log(">>EXISTS>>>" + file.exists());

var processDialog = Ti.UI.Android.createProgressIndicator({
	message : 'Searching...',
	location : Ti.UI.Android.PROGRESS_INDICATOR_DIALOG,
	type : Ti.UI.Android.PROGRESS_INDICATOR_INDETERMINANT,
	cancelable : false
});

var pdfReader = READER_MODULE.createView({
	file : file
});

//pdfReader.loadPDFFromFile(file);

//READER_MODULE.DIRECTION_VERTICAL || READER_MODULE.DIRECTION_HORIZONTAL (default)
pdfReader.setScrollingDirection(READER_MODULE.DIRECTION_VERTICAL);

win.add(pdfReader);

/*
 *
 * Available methods:
 * ------------------
 * pdfReader.getCurrentPage() - returns current page
 * pdfReader.setCurrentPage(pageNum) - set current page
 * pdfReader.getPageCount() - returns total number of pages
 *
 */

pdfReader.addEventListener("change", function(evt) {
	/*
	 *
	 * properties of evt
	 * currentPage - being viewed
	 * count - number of pages in pdf
	 *
	 */
	console.log("Viewing " + evt.currentPage + " / " + evt.count);
});

pdfReader.addEventListener("click", function(evt) {
	console.log("you just clicked on pdf reader");
});

win.addEventListener("open", function(e) {
	var activity = win.getActivity();
	activity.onCreateOptionsMenu = function(e) {
		var searchItem = e.menu.add({
			title : "Search",
			showAsAction : Ti.Android.SHOW_AS_ACTION_ALWAYS
		});
		searchItem.addEventListener("click", function(e) {
			var toast = Ti.UI.createNotification({
				message : "Search for the total occurences of keyword '" + keyword + "' in the entire pdf. Note : Touch events will be disabled during search",
				duration : Ti.UI.NOTIFICATION_DURATION_LONG
			});
			toast.show();
			count = 0;
			processDialog.show();
			pdfReader.onSearch(searchResult);
			//start search from page no. 1.
			//third parameter is optional, defaults to false. Disable the rendering of the search. If true the page will be rendered with results highlighted
			pdfReader.search(keyword, 1, false);
		});
		var previousItem = e.menu.add({
			title : "Previous",
			showAsAction : Ti.Android.SHOW_AS_ACTION_IF_ROOM
		});
		previousItem.addEventListener("click", function(e) {
			pdfReader.moveToPrevious();
		});
		var nextItem = e.menu.add({
			title : "Next",
			showAsAction : Ti.Android.SHOW_AS_ACTION_IF_ROOM
		});
		nextItem.addEventListener("click", function(e) {
			pdfReader.moveToNext();
		});
		var searchPreviousItem = e.menu.add({
			title : "Search Previous",
			showAsAction : Ti.Android.SHOW_AS_ACTION_IF_ROOM
		});
		searchPreviousItem.addEventListener("click", function(e) {
			pdfReader.onSearch(logSearch);
			pdfReader.search(keyword, pdfReader.getCurrentPage() - 1, true);
		});
		var searchNextItem = e.menu.add({
			title : "Search Next",
			showAsAction : Ti.Android.SHOW_AS_ACTION_IF_ROOM
		});
		searchNextItem.addEventListener("click", function(e) {
			pdfReader.onSearch(logSearch);
			pdfReader.search(keyword, pdfReader.getCurrentPage() + 1, true);
		});
		var toggleHightLight = e.menu.add({
			title : "Toggle highlight",
			showAsAction : Ti.Android.SHOW_AS_ACTION_IF_ROOM
		});
		toggleHightLight.addEventListener("click", function(e) {
			pdfReader.setHighlightColor( enabled ? "#500000FF" : "transparent");
			pdfReader.onSearch(logSearch);
			//search and render results for the current page
			pdfReader.search(keyword, pdfReader.getCurrentPage(), true);
			enabled = !enabled;
		});
	};
	activity.invalidateOptionsMenu();
});

function logSearch(evt) {
	console.log(evt);
}

function searchResult(result) {
	console.log("Searh Result: ", result);
	count += result.count;
	if (result.currentPage < pdfReader.getPageCount()) {
		// search for next page until end of the pdf
		pdfReader.search(keyword, result.currentPage + 1);
	} else {
		processDialog.hide();
		if (count == 0) {
			alert("No matches found");
		} else {
			alert("Total occurence : " + count);
		}
	}
}

Ti.Gesture.addEventListener("orientationchange", function() {
	pdfReader.setCurrentPage(pdfReader.getCurrentPage());
});

win.open();
```

## Usage 1.9.3

```javascript
var READER_MODULE = require("com.mykingdom.mupdf");

//Make sure the file exists
var file = Ti.Filesystem.getFile(Ti.Filesystem.externalStorageDirectory, "sample.pdf");

if (!file.exists()) {
	var source = Ti.Filesystem.getFile(Ti.Filesystem.resourcesDirectory, "sample.pdf");
	file.write(source.read());
}

var win = Ti.UI.createWindow({
	backgroundColor : 'white',
	exitOnClose : true
});

var pdfViewer = READER_MODULE.createView({
    file : file,
    width : Ti.UI.FILL,
    height : Ti.UI.FILL
});

function displayPDF(e) {

  win.add(pdfViewer);

  if (OS_ANDROID) {
    Alloy.Globals.navWindows.push();
    win.open();
  }
}

pdfViewer.addEventListener('successEvent', displayPDF);

pdfViewer.setScrollingDirection(READER_MODULE.DIRECTION_VERTICAL);

win.addEventListener("open", function(e){
    var activity = $.pdfViewerWindow.getActivity();
    activity.onCreateOptionsMenu = function(e) {

      if( !pdfViewer.getNeedsPassword() ){
        var printMenu = e.menu.add({
            title : L("print", "Print")
        });
        printMenu.addEventListener("click", function(e) {
            pdfViewer.print();
        });
      }

      var closeMenu = e.menu.add({
          title : L("close", "Close")
      });
      closeMenu.addEventListener("click", function(e) {
          win.close();
      });
    };
    activity.invalidateOptionsMenu();
});

Ti.Gesture.addEventListener("orientationchange", function() {
    pdfViewer.setCurrentPage(pdfViewer.getCurrentPage());
});
```

## Building

To build the project use "ant"

## Changelog

* 1.9.3
	* Updated Titanium SDK to 5.1.2.GA
	* Added support for password protected files
	* Added support for printing files

* 1.9.0
	* Updated MuPDF library to the latest version (1.8) which addresses several crash issues.
	* Updated Titanim SDK to 5.1.1.GA

* 1.8
	* Verical Scrolling
	* Updated Titanim SDK to 3.1.3.GA

* 1.7
	* Search callback was never called when no results found on a page - fixed

* v1.6
	* crash issues fixed

* v1.5
	* replaced createPDFReader with createView
	* updated mupdf library to 1.7
	* enchancements on search method

* v1.4
	* added methods setHighlightColor
	* added events click and change
