---
title: Twig filters
---

Twig filters
============

### excerpt

Excerpt creates a short, text-only, excerpt of a record or a string. It's
useful to get short blurbs of text for overview pages, listings, et cetera. If
you pass it a string, it will simply strip out HTML and, reduce it to a given
length:

```twig
{% set text = "Bonum patria: miserum exilium. Ut optime, secundum" %}
{{ text|excerpt(10) }}

=> Bonum pat…
```

If you get an excerpt of a Record, Bolt will attempt to get an excerpt that's
representative of the Record. If it has a recognisable title, it will start
with that, and it will use the other text-fields to complete it. In fact, it's
the same function that's used in the Bolt backend, on the dashboard. See also [extras][extras].

```twig
{% setcontent page = "pages/1" %}
{{ page|excerpt(200) }}

=> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Videsne quam sit magna
dissensio? Cum ageremus, inquit, vitae beatum et eundem supremum diem, scribebamus haec.
Duo Reges: constructio int…
```

It is also possible to highlight a keyword in an excerpt, which can be used in
search results.

```twig
{% set keyword = 'ageremus' %}{# this is the keyword you want to highlight #}
{% set include_title = false %}{# this will include the title in the results #}
{% setcontent page = "pages/1" %}
{{ page|excerpt(200, include_title, keyword|default('')) ) }}

=> …consectetur adipiscing elit. Videsne quam sit magna dissensio? Cum <mark>ageremus</mark>,
inquit, vitae beatum et eundem supremum diem, scribebamus haec. Duo Reges: constructio int…
```

### localedatetime

Outputs a localized, readable version of a timestamp, based on the `locale`
setting in the `config.yml`-file. See the [Locales][locales-page] page for
more information on locales. If the locale you've set in `config.yml` does not
work, you should verify that the locale is properly installed on your system.

In Bolt dates are stored with each record for the date the record was created,
when it was last edited, and optionally when it was published. These dates are
stored in a way that makes it easier for the database to work with them when it
comes to sorting or selecting a specific period. They look like:
`2020-02-18 09:41:10`, which isn't suitable to output on the website itself.
The localedatetime filter transforms the ugly timestamp to a readable,
localized text. Examples:

```twig
'{{ record.datepublish }}' is the same as
'{{ record.datepublish|localedatetime("%A %B %e") }}'
```

Outputs:

 - '2020-12-05 06:51:16' is the same as 'mánudagur desember 5', if your locale is set to
    `is_IS`,  or
 - '2020-12-05 06:51:16' is the same as 'Monday December 5', if it's set to `en_GB`. Note
    that it correctly uses capitals according to the chosen language's conventions.

Some other examples:

```twig
<ul>
    <li> Created: {{ record.datecreated|localedatetime("%c") }}</li>
    <li> Published: {{ record.datepublish|localedatetime("The %A in week %V of %Y") }}</li>
    <li> Last changed: {{ record.datechanged|localedatetime("%B %e, %Y %r ") }}</li>
</ul>
```

Outputs:

 - Created: Fri 9 Nov 10:55:19 2020
 - Published: The Sunday in week 07 of 2020
 - Last changed: February 17, 2020 01:09:30 pm

The `localedatetime`-filter uses the PHP `strftime()` function internally. For all
possible options, see the official [strftime()][strftime] page on php.net.

### localedate

Alias for [localedatetime](#localedatetime)

### date

```twig
{{ content.datecreated|date("M d, ’y")}}
```

See the various options for 'date' on the [PHP website][date].

<p class="note"><strong>Note:</strong> This filter does <em>not</em> display a
localized version of the date. Use the <code>{{ localedatetime }}</code>-filter if
you want to display dates in other languages than English.</p>


### current

Checks if a given record corresponds to the page being shown in the browser.
Useful for adding 'active' states to menus and such.

```twig
{% if page|current %}class="current"{% endif %}
```

or:

```twig
{% if page|current %}
    Yes, {{ page.title }} is the current page.
{% else %}
    No, you're viewing another page than {{ page.title}}
{% endif %}
```


### round, ceil and floor

The `round` modifier can be used to round numbers (or
strings containing a numerical-like values) to the nearest integer, which
basically means "whole number".

```twig
{% set pi = 3.141592 %}

Rounded, Pi is {{ pi|round }} {# "3" #}

The constant Pi is somewhere between {{ pi|round(1, 'floor') }} and {{ pi|round(1, 'ceil') }}
{# "3 and 4" #}
```

If you need fancier number formatting than this, you can use the built-in Twig
`number_format`-filter. See the [docs here][number].


### slug

The `slug` filter can be used to transform any string into a slug-like value.
This can be very useful when you're hand-crafting links to categories, pages or
other structures where you need a URL-safe representation of a string.

In this example, we build links to all category listing pages:

```twig
<ul>
{% for category in app.config.get('taxonomy/categories/options') %}
    <li><a href="/category/{{ category|slug }}">{{ category }}</a></li>
{% endfor %}
<ul>
```

### ucwords

Converts the first character of every word into upper case.

### shy

The "soft hyphenate" filter can be used for strings without spaces, that would
otherwise break the layout of your page. By adding these soft hyphens, the
browser knows it can wrap to the next line. For example:

```
|                         |
| Before: {{ file }}      |
| MyVeryLongFilenameWithoutSpacesOrDashesOrWhatever.jpg |
|                         |
| After: {{ file|shy }}   |
| MyVeryLongFilenameWith- |
| outSpacesOrDashesOrWha- |
| tever.jpg               |
|                         |
```

### safestring

Use this modifier to return a "safe" version of the string. For example:

```twig
{% set text = "Bonum patria: miserum exilium. Ut optime, secundum" %}
{{ text|safestring(true) }}

=> bonum-patria-miserum-exilium-ut-optime-secundum
```

Characters in the string are converted to lowercase, accented ones are converted to their lowercase ASCII equivalent.
Punctuation signs, and trailing space if present, are replaced by hyphens.

You can specify two parameters: strict mode and extrachars.

  - Strict mode (boolean, default to false): spaces are converted to hyphens.
  - Extra chars (string, default to empty): A string containing extra non-alphabetical characters to keep in result.

```twig
{# Default settings #}
{% set text = "Bonum patria: miserum exilium. Ut optime, secundum" %}
{{ text|safestring() }}

=> bonum patria-miserum exilium-ut optime-secundum

{# Strict mode #}
{% set text = "Bonum patria: miserum exilium. Ut optime, secundum" %}
{{ text|safestring(true) }}

=> bonum-patria-miserum-exilium-ut-optime-secundum

{# Keep dots #}
{% set text = "my beautiful image.jpg" %}
{{ text|safestring(true, ".") }}

=> my-beautiful-image.jpg
```

### plaintext

Use this modifier to return a "plaintext" version of the string. For example:

```twig
{% set text = "Bonum patria: señor & <em>éxilium</em>!" %}
{{ text|plaintext }}

=> Bonum patria: señor & éxilium!
```

This returns a string with letters, numbers and common extra characters, but
without HTML tags. This makes the output very suited for - for example - use in
`<title>` tags.

The main difference between this filter and `|striptags` is that the output of
this filter is marked as HTML in Twig, meaning that characters like `&` and `'`
will not be escaped. If you wish these to be escaped, use `|striptags` instead.

### showimage

Use this filter to insert an image in the HTML. You can optionally provide the
width, height and cropping parameters, like you can do with the `thumbnail`
filter.

```twig
{{ record.photo|showimage(800, 600) }}
or
{{ showimage("2020-03/foo.jpg", 800, 600) }}
```

### thumbnail

Use this modifier to create a link to an automatically generated thumbnail of a
size of your choosing. For example:

```twig
<img src="{{ content.image|thumbnail(320, 240) }}">
```

If `content.image` is an image in your `files/` folder, like `foo.jpg`, this
modifier will output a link like `/thumbs/320x240/foo.jpg`. This is useful for
creating absolute links to a thumbnail, regardless of whether Bolt is installed
in the root of your domain, a subdomain or a folder.

You can specify three parameters: the width, height, and the mode of cropping.
The mode of cropping is important if you're requesting a thumbnail that has
different proportions than the original image. Valid options for cropping are:

  - `c` (crop, default) - Makes sure you always get an image that is the
    specified width and height. The image is not transformed, so it will be
    cropped to fit the boundaries is necessary.
  - `f` ('fit') - The image will not be cropped but resized to fit within the
    given maximum width and height. This means that you will always get an image
    with the exact same width and height that you specified. The resulting image
    might be deformed, and will _not_ have the same aspect ratio as the
    original.
  - `b` (borders) - Will add a border to the image, in order to make it fit
    within the given boundaries.
  - `r` (resize) - Will resize the image to fit the boundaries, without
    cropping. This means your thumbnail might be smaller than the width/height
    given, but the the image will always maintain the aspect ratio of the
    original image.

Use the cropping parameter like this:

```twig
<img src="{{ content.image|thumbnail(100, 100, "r") }}">
```

If you omit the width and height altogether, the thumbnail will use the
'default' size and cropping mode. Remember to add quotes around the cropping
mode.

```twig
<img src="{{ content.image|thumbnail }}">
```

You can set the size in your `config.yml`, like this:

```yaml
thumbnails: [ 160, 120, c ]
```

To use a defined [thumbnail alias](../configuration/thumbnails#thumbnail-aliases),
you just need to pass in your alias name like so:

```twig
<img src="{{ content.image|thumbnail('cover') }}">
```


### image

Use this modifier to create a link to an image of your choosing. For example:

```twig
<img src="{{ content.photo|image }}">
```

If `content.photo` is an image in your `files/` folder, like `2020-11/foo.jpg`,
this modifier will output a link like `/files/2020-11/foo.jpg`. This is useful
for creating absolute links to an image, regardless of whether Bolt is installed
in the root of your domain, a subdomain or a folder.

You can specify three parameters: the width, height, and the mode of cropping.
By doing so, the image will be resized, and it behave exactly like the
[thumbnail filter](#thumbnail).

```twig
<img src="{{ content.photo|image(100, 100, "r") }}">
```

To scale an image proportionally to a given width or height,
set the other dimension to `null`, and set cropping mode to resize.

```twig
<img src="{{ content.image|image(400, null, "r") }}">
```

See also [extras][extras].

### raw

If the content contains HTML-fields, they will be rendered with escaped
characters by default. If you want to use the HTML as-is, add the raw modifier:

```twig
{{ page.tite|raw }}
```

If we didn't add the `raw` modifier, all '<' and '>' characters in the body
would be output as '&amp;lt;' and '&amp;gt;' respectively. If 'body' is an HTML
field in our ContentType, we want it to be output as normal HTML, so we have to
add the `raw` modifier.

### order

In most cases the results of `{% setcontent %}` or `{{ record|related() }}` are
in the desired order. In some cases you might want to reorder them, by using the
`order`-filter. The filter takes one or two parameters: the names of the fields
you wish to order the results on:

```twig
{% set relatedrecords = record|related() %}
<p class="meta">Related content:
    <ul>
    {% for related in relatedrecords|order('datepublish') %}
        <li><a href="{{ related|link }}">{{ related.title }}</a></li>
    {%  endfor %}
    </ul>
</p>
```

or:

```twig
{# get the 10 latest entries by date, but sort them on the title field #}
{% setcontent entries = "entries/latest/10" %}
<ul>
{% for entry in entries|order('title', 'subtitle') %}
    <li><a href="{{ entry|link }}">{{ entry.title }}</a></li>
{%  endfor %}
</ul>
```

**Note:** Ordering with the `order`-filter is case sensitive. This means that
'banana' will come _before_ 'Apple'. If you're sorting on a title or name field
and this case sensitivity is undesirable, you can use `|order('slug')` instead.
The slug is always lowercase, so this will normalize the ordering.

### shuffle

Randomly shuffles the passed array.


### previous(*byColumn = 'id'*, *sameContentType=true*)

Returns the previous record from the database query based on the passed
parameters. By default, `|previous` finds the left adjacent element for the
same contenttype using the record's database id.

### next(*byColumn = 'id', *sameContentType=true*)

Returns the next record from the database query based on the passed parameters.
Uses the same logic as the [previous filter](#previous)

### edit_link

Returns the edit link for the record in the Bolt backend.

### taxonomies

Returns an array of all taxonomies linked to the record.

### label

Returns the label of the field, as defined in the field's `contenttypes.yaml`
definition.

### type

Returns the field type of the field, as defined in the field's
`contenttypes.yaml` definition.

### selected

Returns all selected records from the content select field. Note, this filter
should only be used on select fields that select from a list of Content, as
opposed to a list of items.

### markdown

Transforms the given markdown content into HTML content, i.e. parses markdown
into HTML.

### popup

See [popup function](#popup-magnific-popup)

### media

Returns the media array associated with the field. Note, this should only be
used with image and file fields.

### json_records

Encodes the given array of records into json.

### normalize_records

Returns an array of normalized records.

### preg_replace

Makes PHPs `preg_replace()` function available as twig filter. Example usage:

```twig
{{ content.text|preg_replace('/[^a-z]+/', '_') }}
```

### related(*name=null*, *contenttype=null*, *bidirectional=true*, *publishedonly=true*)

Returns an array of records that are related to the given record.

```twig
{% set relatedrecords = record|related() %}
<p class="meta">Related content:
    <ul>
    {% for related in relatedrecords %}
        <li><a href="{{ related|link }}">{{ related|title }}</a></li>
    {%  endfor %}
    </ul>
</p>
```

### related_all(*bidirectional=true*, *limit=true*, *pubishedonly=true*)

Returns an array of all records that are a relation with any other record.

See [related filter](#related-name-null-contenttype-null-bidirectional-true-publishedonly-true).

### related_first(*name=null*, *contenttype=null*, *bidirectional=true*, *publishedonly=true*)

Returns the first record related to the given record.

### translated(locale)

Returns the field translated into the given locale.

[widgets-page]: ../templating/widgets
[debugging-page]: ../debugging
[select-page]: ../fields/select
[locales-page]: ../other/locales
[linkintpl]: ../templating/linking-in-templates
[linkintpl-asset]: ../templating/linking-in-templates#using-asset-to-link-to-assets-or-files
[linkintpl-pathurl]: ../templating/linking-in-templates#using-path-and-url-to-link-to-named-routes
[linkintpl-current]: ../templating/linking-in-templates#linking-to-the-current-page
[twig]: http://twig.sensiolabs.org/doc/templates.html
[inc]: http://twig.sensiolabs.org/doc/functions/include.html
[inheritance]: http://twig.sensiolabs.org/doc/templates.html#template-inheritance
[number]: http://twig.sensiolabs.org/doc/filters/number_format.html
[popup]: http://dimsemenov.com/plugins/magnific-popup/
[strftime]: http://php.net/manual/en/function.strftime.php
[date]: http://php.net/manual/en/function.date.php
[for]: http://twig.sensiolabs.org/doc/tags/for.html
[switch]: http://php.net/manual/en/control-structures.switch.php
[extras]: ./twig-components/extras
