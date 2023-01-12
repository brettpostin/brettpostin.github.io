---
layout: post
title:  "Chrome 50 and table-cell percentage heights"
date:   2016-04-16 21:01:17 +0000
categories: Development
tags: chrome
---

With the release of **Chrome 50** there appears to be a fundamental breaking change in the rendering behaviour of content nested within table cells.

See this [jsFiddle](https://jsfiddle.net/zc21pg9q/1/) for an example.

Previously, Chrome (along with Safari, IE11 and Opera) would pass percentage heights implicitly to the child cell. So if height 100% was set on the table, the table cell would also be height 100%, making the div fill the full height of the table.

This was not the case with Firefox and versions of IE from 10 down. In fact, this behaviour has been [reported as a bug](https://bugzilla.mozilla.org/show_bug.cgi?id=10212) with Mozilla since 1999, which seems to be acknowledged as such.

Chrome 50 appears to have joined this club, with complete disregard to the impact this change would have out in the wild. Based on [my Chromium bug report](https://bugs.chromium.org/p/chromium/issues/detail?id=603835&start=100&num=100), it seems we are not the only ones affected by this breaking change.

The justification for the change in [this bug report](https://bugs.chromium.org/p/chromium/issues/detail?id=583670&q=label%3ACr-Blink-Layout-Table&colspec=ID%20Pri%20M%20Stars%20ReleaseBlock%20Cr%20Status%20Owner%20Summary%20OS%20Modified) states:

> The new behavior is more spec compliant and, as mentioned above, there’s an easy workaround: add height:100% to the table cell.

Whilst in isolation the workaround is simple, dealing with a legacy web application consisting of thousands of pages now broken due to this change is a more serious problem.

Within this legacy application we apply a shim to “fix” this behaviour globally for Firefox and IE10. However there is a noticeable degrade in UI experience with doing this. Chrome has now been added to the list of browsers needing this shim.

The implementation may not be perfect, but hopefully it may help others who are dealing with the fallout of this high impact change.

```javascript
(function ($, sr) {
 
    var debugEnabled = false;
 
    // debouncing function from John Hann
    // http://unscriptable.com/index.php/2009/03/20/debouncing-javascript-methods/
    var debounce = function (func, threshold, execAsap) {
        var timeout;
 
        return function debounced() {
            var obj = this, args = arguments;
            function delayed() {
                if (!execAsap)
                    func.apply(obj, args);
                timeout = null;
            };
 
            if (timeout)
                clearTimeout(timeout);
            else if (execAsap)
                func.apply(obj, args);
 
            timeout = setTimeout(delayed, threshold || 100);
        };
    }
 
    // smartresize init
    jQuery.fn[sr] = function (fn) { return fn ? this.bind('resize', debounce(fn)) : this.trigger(sr); };
 
    function isFullHeight(element) {
        return (element.css('height') === '100%' ||
                element.attr('height') === '100%' ||
                element.attr('isFullHeight') === 'true' ||
                element[0].style.height === '100%' ||
                (element[0].tagName === 'TD' && element[0].style.height == '' && !element.attr('height')));
    }
 
    function propagateCellHeights() {
 
        // **
        // ** Global fix solution that involves no markup changes
        // **
 
        var tables = $('table');
        tables.each(function (i, table) {
 
            table = $(table);
 
            if ((table.css('height') === '100%' || table.attr('height') === '100%') && table.is(':visible')) {
 
                debug('Table (' + table.attr('id') + ') qualified.');
 
                var cells = table.find('> tbody > tr > td');
                var fullHeightCells = [];
 
                cells.each(function (j, cell) {
                    cell = $(cell);
                    if (isFullHeight(cell)) {
                        debug('Cell (' + cell.attr('id') + ') qualified.');
                        fullHeightCells.push(cell);
                    }
                });
 
                $(fullHeightCells).each(function(j, cell) {
                    cell.children().each(function(k, child) {
                        child = $(child);
                        if (isFullHeight(child)) {
                            child.attr('isFullHeight', 'true');
                            child.height(0);
                        }
                    });
                });
 
                $(fullHeightCells).each(function (j, cell) {
                    var cellHeight = Math.floor(cell.height());
                    cell.children().each(function (k, child) {
                        child = $(child);
                        if (isFullHeight(child)) {
                            child.outerHeight(cellHeight);
                        }
                    });
                });
            }
        });
    }
 
    function debug(message) {
        if (debugEnabled) {
            console.log('[ie-tablecell-fix] : ' + message);
        }
    }
 
    $(window).load(function () {
        propagateCellHeights();
    });
 
    $(window).smartresize(function () {
        propagateCellHeights();
    });
 
})(window.jQuery, 'smartresize');
```

**Note: This shim has been used in production for nearly two years without major issues, other than the stated degradation in UI experience.**