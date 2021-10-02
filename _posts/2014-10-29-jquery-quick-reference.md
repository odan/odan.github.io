---
title: jQuery quick-reference
layout: post
comments: true
published: false
description: 
keywords: 
---

```js
// jquery plugin adapter (jpa)
$.fn.nameofclass = function(options) {
    this.each(function() {
        $(this).data('nameofclass', new myClass(this, options));
    });
};

// Remove all CSS classes
// Calling removeClass with no parameters will remove all of the item's classes.
$("#item").removeClass();

// -----------------------------------------------------------------------------

// Setting a Dropdown value by value is quite easy:
// select option element by value
$('#dropdown_id').val(2);

// If you need to set a dropdown list item by text you need a little bit more code:
// select option element by text
var strTextValue = 'Test';
$('#dropdown_id' + ' option').each(function() {
    if($(this).text() === strTextValue) {
        $(this).prop('selected', true);
    };
});

// find nearest parent
$(selector).closest(selector2);

// set checkbox checked
$('.myCheckbox').prop('checked', true);
$('.myCheckbox').prop('checked', false);

```