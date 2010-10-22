# tablesorter

This is a fork of the jQuery [tablesorter](http://tablesorter.com/) plug-in by Christian Bach.

The goal of this fork is to allow parser detection by using an entire column instead of just the first cell. This is because for some uses, it isn't possible to detect a parser by only looking at one cell. For example, let's say we have a table with columns that can be letter grades (A,A-,B+,...,F). We also have another column that contains the names of programming languages. With the old way parsers were implemented, the only way you could auto-detect a column was by looking at the first cell. So what if the first cell contained "C"? Is this a letter grade column or a programming language name column? We don't know. A way to get a better idea of the column type would be to look at all of the cells in the column. Instead of passing the first cell as an argument to the parser, we pass the entire column being checked as a jQuery object.

Here is our attempt to create a parser using the old way:

<code>
    $.tablesorter.addParser({
        id: 'grades',
        is: function(first_cell_value) {
            if (!/^[ABCDF][+-]?$/i.test(first_cell_value)) {
              return true;
            }
            return false;
        },
        format: function(value) {
            grades = ["A", "A-", "B+", "B", "B-", "C+", "C", "C-", "D+", "D", "D-", "F"];
            return grades.reverse().indexOf(value);
        },
        // set type, either numeric or text
        type: 'numeric'
    });
</code>

This parser would detect a column containing the values "C", "Ruby", "Python", "C++" as being a grade column. That's no good.

Let's try the new way:
<code>
    $.tablesorter.addParser({
        id: 'grades',
        is: function(column, config) {
            for (i = 0; i < column.length; i++) {
                if (!/^[ABCDF][+-]?$/i.test(column[i].innerHTML)) {
                    return false;
                }
            }
            return true;
        },
        format: function(value) {
            grades = ["A", "A-", "B+", "B", "B-", "C+", "C", "C-", "D+", "D", "D-", "F"];
            return grades.reverse().indexOf(value);
        },
        type: 'numeric'
    });
</code>

This goes through the entire column making sure that each value is a proper letter grade. Much more accurate, but a little slower [O(n) instead of O(1)]. So let's say you still only needed to check the first cell in the column. You can still do that in constant time. Let's look at one of the default parsers for example:

<code>
    ts.addParser({
        id: "currency",
        is: function(text) {
            text = $.trim($.tablesorter.getElementText(config, column[0]));
            return /^[£$€?.]/.test(text);
        },
        format: function(s) {
            return $.tablesorter.formatFloat(s.replace(new RegExp(/[^0-9.]/g), ""));
        },
        type: "numeric"
    });
</code>

This is how it looked before:

<code>
    ts.addParser({
        id: "currency",
        is: function(s) {
            return /^[£$€?.]/.test(s);
        },
        format: function(s) {
            return $.tablesorter.formatFloat(s.replace(new RegExp(/[^0-9.]/g),""));
        },
        type: "numeric"
    });
</code>

It's a little bit more complicated, but it's worth it IMO. However, I may consider eventually making both ways possible by using different names for the function.