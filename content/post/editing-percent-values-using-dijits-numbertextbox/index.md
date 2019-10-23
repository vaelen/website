---
date: "2011-07-28 20:50:28"
title: "Editing Percent Values Using Dijit's NumberTextBox"
---
<a href="http://dojotoolkit.org/widgets" target="_blank">Dijit </a>is a web UI toolkit built on top of the <a href="http://dojotoolkit.org/" target="_blank">Dojo framework</a>. One of its widgets is called <a href="http://dojotoolkit.org/reference-guide/dijit/form/NumberTextBox.html" target="_blank">NumberTextBox</a>. This widget allows you to show and edit formatted numbers easily.

For example, I can create an instance of <a href="http://dojotoolkit.org/reference-guide/dijit/form/CurrencyTextBox.html" target="_blank">CurrencyTextBox</a>Â (a subclass of NumberTextBox)Â and call set("value", Â 2589632). Â This will display the value as follows (assuming that my locale is set to en_US):

<a href="payroll-display.jpg"><img class="alignnone size-full wp-image-258" title="payroll-display" src="payroll-display.jpg" alt="" width="252" height="36" /></a>

If I click in the box to edit the value, it changes back to just numbers and looks like this:

<a href="payroll-edit.jpg"><img class="alignnone size-full wp-image-260" title="payroll-edit" src="payroll-edit.jpg" alt="" width="257" height="42" /></a>

Exiting the field will reformat the displayed value to match my locale's number formatting guidelines. Â However, when I call the get("value") method on the widget, I will get back the number 2589632 instead of the formatted string. Â This makes it easily to implement locale sensitive number formatting for web applications.

The problem I've been having is that percents aren't handled quite right. Setting the "type" constraint to "percent" will properly display percent values so that setting the value of the widget to "0.02" will display the string "2.00%".

<a href="percent-display.jpg"><img class="alignnone size-full wp-image-261" title="percent-display" src=percent-display.jpg" alt="" width="366" height="36" /></a>

However, When the user clicks on the box to edit the value, the "2.00%" is replaced by it's real value: "0.02".

<a href="percent-edit.jpg"><img class="alignnone size-full wp-image-259" title="percent-edit" src="percent-edit.jpg" alt="" width="369" height="34" /></a>

This causes two problems:
<ol>
	<li>The user is expecting to enter a percentage, not a decimal value. Â Users are often confused when the value they were expecting to see suddenly changes. Â Also, a user might enter "2" in the field, expecting it to show "2.00%". Â Instead it will show "200.00%".</li>
	<li>Truncation can occur when the number being edited has too many decimal places. Setting the "places" constraint on the widget to "2" will display the value "0.0256" as "2.56%". However, when you edit the value it truncates back to 2 decimal places, showing you only "0.02". Those extra two decimal places are now lost forever.</li>
</ol>
There is an <a href="http://bugs.dojotoolkit.org/ticket/10582" target="_blank">open defect</a>Â for this issue, but I couldn't wait for them to fix it in the main dojo code. Â To get around these issues I started digging into the NumberTextBox code and found that I can replace the parse() method on the NumberTextBox object to get around the problem. This solution also appends a percent symbol to the end of the string if one was left off, making it easier for a user to simply enter "10" and have that converted to "10.00%" when they move to the next field.

Here is my solution:
<pre>function createPercentTextBox(name, title, extraParams) {
    if(!dojo.isObject(extraParams)) {
        extraParams = {};
    }

    var parseFunction = function(expression, options) {
        if(dojo.isString(expression)) {
            expression = dojo.trim(expression);
            var i = expression.lastIndexOf("%");
            if(i == -1) {
                expression = expression + "%";
            }
        }
        var value = dojo.number.parse(expression, { pattern: '0%' });
        return value;
    };

    var params = {
        id: name,
        name: name,
        label: title ? title + ":" : "",
        title: title ? title : "",
        constraints: {
            type: 'percent',
            places: 2
        },
        editOptions: {
            pattern: "##0.00%"
        },
        parse: parseFunction
    };

    dojo.mixin(params, extraParams);

    return new dijit.form.NumberTextBox(params);
}</pre>
Note: I'm using <a title="Dojo 1.6 API Documentation" href="http://dojotoolkit.org/api/1.6/" target="_blank">Dojo 1.6</a> for this example. Future versions of Dojo may resolve this issue more elegantly.
