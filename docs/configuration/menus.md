---
title: Using menus
---
Creating menus
==============

Bolt has built-in functionality to create menus in your frontend templates. You
can define one or more menus in the file `config/bolt/menu.yml`, which can then
be inserted in your templates using the `{{ menu() }}` tag.

To change one or more of the menus, edit the file `config/bolt/menu.yml`. You
can add more separate menus, if you wish, and each menu can have one level of
items below it. See the default `menu.yml` for an example of the supported
options:


```yaml
main:
  - label: Home
    title: This is the first menu item. Fo shizzle!
    path: homepage
    class: first
  - path: entry/1
    label: Second item
    submenu:
      - label: Sub 1
        path: entry/2
      - label: Sub 2
        class: menu-item-class
        path: entry/3
```

In this case `main` is the name of the menu. The options are:

| Option     | Description |
|------------|-------------|
| `label` | override the 'title' of the record with a defined label. If omitted, the 'title' of the record is used. |
| `title` | used as a 'title'-attribute in the rendered HTML. If omitted this can be substituted for the `subtitle`-field in a record. |
| `class` | used to define an HTML `class`-attribute  |
| `path` | The 'path' to a record in Bolt, or a group of records. For example `path: page/about` will make this item link to a record of type 'page'  with the slug 'about'. `path: page/1` will link to the 'page' with id '1'. `path: entries` will link to the `/entries` overview page. |
| `link` | define an external link to another site. For example `link: https://bolt.cm`. Do not use `link` together with `path`! |
| `submenu` | defines a submenu. In the submenu you can define other items, with the same options as before. |

To insert a menu in your templates, use

```twig
{{ menu() }}
```

If you have more than one menu, you should use its name to make sure you get
the intended one:

```twig
{{ menu('foo') }}
```

By default, the menu is rendered using the template
`/bolt/templates/helpers/_sub_menu.twig`. You can 'override' the default by copying
this file to the root of your own theme folder. Bolt will pick your own version, and then
it will not be overwritten in a future update. However, it is good practice to
explicitly state which template file should be used to render a menu. Like
this:

```twig
{{ menu('foo', 'partials/_sub_menu.twig') }}
```

or

```twig
{{ menu('foo', '/partials/_menu_foo.twig') }}
```

You can specify other parameters besides the menu name and the template. For
example, you can also set the the `ul` class or whether or not the output
should contain submenus. You can control these using the so-called named
arguments in Twig. For example:

```twig
{{ menu(
    identifier = 'foo',
    template = 'partials/_menu_foo.twig',
    params = {'withsubmenus': false, 'class': 'myclass'}
) }}
```

Which is equivalent to this shorthand version:

```twig
{{ menu('foo', 'partials/_menu_foo.twig', {'withsubmenus': false, 'class': 'myclass'}) }}
```

Doing this will render the menu `foo`, using the template `_menu_foo.twig`. The
filename can be anything, but it's good practice to prefix it with `_menu`, so
it's always easily recognizable later, or to other people working with your
HTML.

<p class="note"><strong>Note:</strong> You can define more than one menu in
your <code>menu.yml</code> file, but you should define <em>only one</em> menu
in each template file. So, if you have multiple menus that should be rendered
with different HTML, you should have as many
<code>_menu_<em>menuname</em>.twig</code> files in your theme.</p>


A detailed example
------------------

In this section we'll show you a somewhat more elaborate example of how you can
create a menu, with submenus. First, start by adding a small menu to your
`config/bolt/menu.yml`-file:

```yaml
test:
  - label: Bolt
    link: https://bolt.cm
  - label: Example org
    link: http://example.org
  - label: Silex
    link: http://silex.sensiolabs.org
```

As you can probably guess, this menu does nothing but provide links to three
external websites. To get started, edit the template where you want this menu.
Usually, menus are used in 'headers', 'footers' or 'aside' includes, but you
can use them anywhere. For now, just insert this code, somewhere:

```twig
{{ menu('test', 'partials/_menu_test.twig') }}
```

This inserts the menu, using the template `partials/_menu_test.twig` template. The file
probably isn't present yet, so create it in your own `theme/`-folder.

```twig
<ul>
{% for item in menu %}
    <li>
        <a href="{{ item.link }}">{{item.label}}</a>
    </li>
{% endfor %}
</ul>
```

Refresh a page that uses the template that you've added the `{{ menu() }}`-tag
to in your browser, and you should see a very simple menu, with the following
HTML-markup:

```twig
<ul>
    <li>
        <a href="https://bolt.cm">Bolt</a>
    </li>
    <li>
        <a href="http://example.org">Example org</a>
    </li>
    <li>
        <a href="http://silex.sensiolabs.org">Silex</a>
    </li>
</ul>
```

As you can see, the `{% for %}`-loop iterated over all of the items in the
`menu`-array, and wrote out the HTML that you specified. Let's change our menu,
so it has a submenu, listing some content on our site. In this example, we'll
assume that you have a `pages` ContentType, and that records `1`, `2` and `3`
exist. If they don't, just replace them with some contenttype/id pairs that you
do have. Edit the `config/bolt/menu.yml`-file:

```yaml
test:
  - label: Bolt
    link: https://bolt.cm
  - label: All pages
    path: pages/
    submenu:
      - path: page/1
      - path: page/2
      - label: last page
        path: page/3
        class: my_class
  - label: Silex
    link: http://silex.sensiolabs.org
```

Now, the menu template needs to be extended, so that the submenu is output as
well. We'll do this by adding another `{% for %}`-loop. We'll wrap this loop in
an `{% if %}`-tag to prevent Bolt from outputting empty lists in the HTML. For
example:

```twig
<ul>
{% for item in menu %}
    <li class="{{ item.class }}">
        <a href="{{ item.link }}">{{item.label}}</a>
        {% if item.submenu is defined %}
            <ul>
            {% for item in item.submenu %}
                <li class="{{ item.class }}">
                    <a href="{{ item.link }}">{{item.label}}</a>
                </li>
            {% endfor %}
            </ul>
        {% endif %}
    </li>
{% endfor %}
</ul>
```

The output in HTML might look like this now:

```twig
<ul>
    <li class="">
        <a href="https://bolt.cm">Bolt</a>
    </li>
    <li class="">
        <a href="/pages">All pages</a>
            <ul>
                <li class="">
                    <a href="/page/sic-consequentibus-vestris">Sic consequentibus vestris</a>
                </li>
                <li class="">
                    <a href="/page/sublatis-prima-tolluntur">Sublatis prima tolluntur</a>
                </li>
                <li class="my_class">
                    <a href="/page/tria-genera-bonorum">last page</a>
                </li>
            </ul>
    </li>
    <li class="">
        <a href="http://silex.sensiolabs.org">Silex</a>
    </li>
</ul>
```

Dynamic menus
-------------

You can use other `menu.yml` parameters to make a more dynamic menu. In this
example we will use taxonomies combined with the menu to create taxonomy-based
submenus. Let's say you want to have a few static pages to be listed as
submenus under "Pages" in your menu.

Start with creating a new taxonomy in `taxonomy.yml` to control what pages are
to be listed under "Pages":

```yaml
menu:
    name: Menu
    singular_name: Menu item
    behaves_like: categories
    multiple: false
    options: [ about, pages, more ]
```

Then, in your `menu.yml` change your "Pages" to the following.

```yaml
- label: Pages
      path: pages
      list:
          contenttype: pages
          where:
              menu: pages
              limit: 5
```

Now all that's left is to modify your submenu template (`_sub_menu.twig`) so that it adds the pages with the "pages" taxonomy.

```twig
{% macro display_menu_item(item, loop, extraclass, withsubmenus) %}
    {% from _self import display_menu_item %}
    {% apply spaceless %}
    <li class="index-{{ loop.index -}}
        {{ item.path|default('') == 'homepage' ? ' menu-text' -}}
        {{ loop.first ? ' first' -}}
        {{ loop.last ? ' last' -}}
        {{ (item.submenu|default(false) and withsubmenus) ? ' is-dropdown-submenu-parent' -}}
        {{ item|current ? ' active' }}">

        <a href="{{ item.link }}" title='{{ item.title|default('')|escape }}' class='{{ item.class|default('') }}'>
            {{- item.label|default('-') -}}
        </a>

        {% set list = [] %}

        {% if item.submenu is defined and withsubmenus %}
            <ul class="menu submenu vertical" data-submenu>
                {% for submenu in item.submenu %}
                    {{ display_menu_item(submenu, loop) }}
                {% endfor %}
                {% if item.list|default(false) %}
                    {% setcontent listedcontent = item.list.contenttype where item.list.where %}
                    {% for listitem in listedcontent %}
                        {% set list = list|merge([{title: listitem.title, link: listitem.link, label: listitem.title}]) %}
                    {% endfor %}
                    <ul class="menu submenu vertical" data-submenu>
                        {% for submenu in list %}
                            {{ display_menu_item(submenu, loop) }}
                        {% endfor %}
                    </ul>
                {% endif %}
            </ul>
        {% elseif item.list|default(false) %}
            {% setcontent listedcontent = item.list.contenttype where item.list.where %}
            {% for listitem in listedcontent %}
                {% set list = list|merge([{title: listitem.title, link: listitem.link, label: listitem.title}]) %}
            {% endfor %}
            {% if list is not empty %}
            <ul class="menu submenu vertical" data-submenu>
                {% for submenu in list %}
                    {{ display_menu_item(submenu, loop) }}
                {% endfor %}
            </ul>
            {% endif %}
        {% endif %}

    </li>
    {% endapply %}
{% endmacro %}
```

Further customizations
----------------------

That's basically all there's to it. Since the menus use standard Twig tags, we
can enhance the lists with extra features, to automatically give special
classes to the first or last item, or highlight the 'current' page.

Some of the more commonly used 'tricks' are:

  - `index-{{ loop.index }}` - Add the current index of the loop, like
    `index-1`, `index-2`, etc.
  - `{% if loop.first %}first{% endif %}` - Output `first`, but only for the
    first item in the loop.
  - `{% if loop.last %}last{% endif %}` - Output `last`, but only for the last
    item in the loop.
  - `{% if item|current %}active{% endif %}` - Output `current`, but only if
    we're on the page that the item links to.
  - `{% if item.title is defined %}title='{{ item.title|escape }}'{% endif %}`
    - Add a `title` attribute, but only if it's defined in our `.yml`-file, or
    if the ContentType has a `subtitle` field.
  - `{% if item.class is defined %}class='{{item.class}}'{% endif %}` - Add a
    `class` attribute, but only it the item has a `class` defined in the
    `.yml`-file.

See the default `/bolt/templates/helpers/_sub_menu.twig` file for an in-depth
example of all of the things you can do with menus. Remember that you should
always copy this file to your own theme folder, or create your own from
scratch. If you modify the default file, it will most likely get overwritten
when you update Bolt to a newer version.

Normally you will only need the basic properties of each of the menu items, but
sometimes you might need to do more with the items. For this reason, each
`item` has access to the entire record. You can use `{{ item.record }}` like
you would use any other record. For instance, `{{ item.record.taxonomy }}`, or
`{{ dump(item.record) }}`.
