GSoC 2011 Proposal - Revised form rendering
============================================

Hi my name is Gregor Müllegger. I'm a Computer Science student in Germany at
the University of Augsburg currently in the fourth year of my studies. I first
came to django shortly before 0.96 was released and a lots of awesomeness was
introduced with the magic removal branch.

I'm also doing some django freelancing work since 2008 to finance my studies
and attended DjangoCon EU in 2010. This year I would like to apply to
Google Summer of Code helping to improve Django with something that bugs me
since quite a while: It's builtin ability to render a form straight away into
HTML.

Motiviation
-----------

Why would I like to change the current behaviour? There are some reasons for
this:

1. It's hard to change the default rendering.

   It is very easy to use e.g. django's builtin rendering of a ``ul`` based
   output like ``{{ myform.as_ul }}``. But what if you want to use your own
   *layout* like ``{{ myform.as_dl }}``? You can create a new method on your
   existing forms, but that involves writing python code (out of the designers
   hand) and might not be possible for thirdparty forms.

2. Maybe the ``as_ul`` rendering is fine for me. But maybe I want to skip that
   unnecessary field that the thirdparty form is providing. The solution would
   be to write down every single field in your template, reimplementing the
   ``ul`` layout::

        <li>{{ form.field.label_tag }}: {{ form.field }}
            {{ form.field.help_text }} {{ form.field.errors }}</li>
        {# skipping field2 here #}
        <li>{{ form.field3.label_tag }}: {{ form.field3 }}
            {{ form.field3.help_text }} {{ form.field3.errors }}</li>
        ...

   We all love DRY, so this is not acceptable.

3. The designers I worked with are often interested on adding custom css class
   or an attribute to a form field. Most of the time this is really a pain to
   do if you don't have control over the python form code. Imagine a reusable
   app that ships with urls, views, forms. To add a single class you would
   need to: (a) overwrite the predefined url because you want (b) to specify
   an additional parameter for the view which is (c) your custom subclass of
   the thirdparty form::

       class ChangedAttributeForm(ThirdPartyForm):
           field = forms.CharField(max_length=50,
               widget=TextInput(attrs={'class': 'fancy-text-input'}))

   btw: This also violates the DRY principle since you have to redefine the
   used field type, it's attributes (like ``max_length=50``) and the widget.

   I want to tell the designer how he can do this without my help in the
   template.

Goals I want to accomplish
--------------------------

After showing some of the problems that I see, are here the higher goals I
want to achieve during the summer:

1. All of the rendering formats should be extracted from django's python source
   into a well defined structure of customizable templates.
2. Make it possible to reorder the use of form fields in the template without
   needing to write down the complete form.
3. Support for *chrome* that can be added dynamically in the template to a set
   of fields, changing some aspects of the form output.
4. Ensuring that the DRY principle is applyable to your form templates (in
   case of reordering/skipping some fields, like shown above)
5. Ensuring total backwards compatibility for ``{{ form.as_p }}`` etc. but
   deprecating it.
6. Converting the admin to use all of the new goodness. First, to make django
   eating its own dogfood. Second, to prove that the stuff I have developed is
   really a step forward and to show up big problems that occur in reallife
   scenarios, not taken into account in the blueprint.
7. Documenting all the newly introduced templatetags, how to extend the
   rendering behaviour, how to create your own form layout ...

Let's get a bit more detailed. How will this look like?

**1. No HTML in python source**

I will push the formating of ``as_table``, ``as_ul`` and ``as_p`` into a
specified template structure. A formating (e.g. ``ul``) will be called a
layout and will live in the template directory ``forms/layouts/<layout>/...``.
This directory will contain a single file::

    forms/layouts/<layout name>/row.html

Creating a new layout is as simple as putting a new directory into your
template path and adding one file. Why putting the file into a directory? The
reason is because we can make the layout even more customizable. You are able
in the ``row.html`` to ``{% include %}`` a ``errors.html`` and a
``helptext.html`` that can be overriden by the designer without completely
rewriting the whole layout.

Ok, but how will this look in the template where I use the form? For this we
need a new template tag::

    {% form myform using layout "p" %}

    {% form myform using layout "my_custom_layout" %}

    ...

**2. Reordering/skipping fields**

Thats pretty straigh forward with the newly introduced template tag::

    {% form myform using fields "firstname" "lastname" "country" %}

You can list any fields you want, skipping some of them, reordering them etc.
Here is a more advanced example with using a layout and injecting fields of a
second form into another one::

    {% form myform using layout "p" and fields "firstname" "lastname" %}
    {% form fancyform.favourite_color using layout "p" %}
    {% form myform using layout "p" and fields "country" %}

What have we added to the syntax?

a. We can use a single field instead of a form as first argument
b. We can use multiple *rendering modifiers* after the keyword ``using``.

*Rendering modifiers* are the bits after ``using``. They have a name (like
``layout`` or ``fields``) and take as many arguments as they want. You can
seperate multiple modifiers with ``and``.

The goal will be to make these modifiers defineable. Which means that you
can create your own modifiers to support some of your extravagant form
rendering needs. To support this we will need to have a rendering modifier
*registry* or something similiar, where you can ideally load new ones with the
existing ``{% load %}`` tag. The same will apply for custom *chrome*,
described below.

**3. Chrome**

The principle of a *chrome* is a shameless plug of Russell's earlier
suggestions on a revised form rendering [1]. Let me quickly summarize what a
chrome might be able to do for you.

A chrome might be useful to change the rendering of a widget. For example you
have a simple ``<input type="text" />`` input for datetime field. But you want
to enhance it with a calendar popup (like in django's admin). You would simply
apply the ``calendar`` chrome to your field::

    {% form myform.birthday using calendar %}

So what's the difference here between a chrome and a rendering modifier? The
difference is only visible in the python level since chromes will be way
easier to implement.

They opperate only on fields and widgets not on a complete form. They won't
need to hassle with argument parsing since they have a common syntax that
will be::

    {% form <form instance> using <chrome name> [<list of arguments>] [for <filtering fields>] %}

So a chrome is called with the ``<list of arguments>`` that are already
evaluated, previously beeing template variables or static strings. The chrome
is also called once for every field, which simplifies implementation again.
The template tag syntax also provides ways of limiting the use of the chrome
to a set of fields (but the chrome implementor doesn't need to care about
them). It's easier to explain this with some examples::

    Will add the attribute required to _all_ fields of the form (no field filters specified):
    {% form myform using attribute "required" %}

    Calendar chrome for two specified fields:
    {% form myform using calendar for myform.birthday myform.member_since %}

    Adding the class "error" to all fields that have errors:
    {% form myform using class "error" for errors %}

    Calendar for all forms.DateTimeField fields in all of the used forms:
    {% formblock using calendar for type "DateTimeField" %}
        {% form myform %}
        {% form fancyform %}
        {% form another_form_with_lots_of_datetime_fields %}
    {% endformblock %}

    Autocomplete email addresses from your addressbook for all fields that are using the EmailInput:
    {% form myform using autocomplete "/addressbook/emails/" for widget "EmailInput" %}

A sample chrome implementation would look like::

    def attribute(bound_field, name, value=None):
        if value is None:
            value = ''
        bound_field.field.widget.attrs[name] = value

Look again at the example from above::

    {% form myform using attribute "placeholder" "Type in your name ..." for myform.firstname myform.lastname %}

This template tag will call the ``attribute`` chrome on both the fields
``firstname`` and ``lastname`` like::

    attribute(myform['firstname'], 'placeholder', 'Type in your name ...')
    attribute(myform['lastname'], 'placeholder', 'Type in your name ...')

There will also be the possibility of template only chromes which means that
you don't need any python code for some simple modifications.

Template based chromes will live in ``forms/chromes/<chrome name>.html`` and
will get the same arguments as the proposed signature of the python function:
``bound_field`` and ``args``. This makes things possible like creating
Javascript triggers after the widget::

    in the form template:

        {% form myform.client using autocomplete "/customers/" %}

    in forms/chromes/autocomplete.html:

        {{ bound_field }}
        {# ^--- will render the used widget as usual #}
        <script>
            ... javascript triggers ...
            autocomplete({ url: {{ args.0 }} });
        </script>

**4. Keeping your form templates DRY**

The example in **2.** is already much better than the current situation but it
still violates the DRY principle somehow. We repeat ourselfs by listing the
used layout three times. We can do better by grouping the modifiers with a
*formblock*::

    {% formblock using layout "p" %}
        {% form myform using fields "firstname" "lastname" %}
        {% form fancyform.favourite_color %}
        {% form myform using fields "country" %}
    {% endformblock %}

This will remember the modifiers and chromes that are used in the
``formblock`` and will apply them to all ``{% form %}`` tags that are used
inside.

**5. Backwards compatibility**

Backwards compatibility is a serious thing but straight forward in the case of
this proposal. We can fall back to use the internals of the ``{% form %}`` tag
while rendering the form via ``{{ myform }}`` or ``{{ myform.as_ul }}``. A bit
trickier is the use of ``{{ myform.field.label_tag }}`` and
``{{ myform.field.errors }}``. The proposal above doesn't include these cases.

But this is also possible to solve. Goal 1. suggests to refactor all HTML out
of the python source. This must include lables and errors as well. For this
case we would create some new templates::

    forms/layouts/default/label.html
    forms/layouts/default/errors.html

They get the bound field that is used passed in and can render there output like
the ``label_tag`` method and the ``errors`` attribute. In the template we would
use::

    {% form myform.birthday display errors %}
    instead of {{ myform.birthday.errors %}

    {% form myform.birthday display label %}
    instead of {{ myform.birthday.label_tag %}

Storing these templates in the layouts directory has also some nice side
effects. We can for example use some alternative styling of the labels and
errors based on the current layout::

    {% formblock using layout "plain" %}
        {% form myform.birthday display label field %}
           ^--- uses the "plain" layout to render the label and the field definition
        {% form myform.birthday display errors using layout "ul" %}
           ^--- the specified "ul" layout overwrites the "p" layout from the
                formblock and displays a list of errors instead of errors
                seperated by <br> that might be used in the "p" layout.
        {% form myform.birthday display helptext %}
    {% endfor %}

**6. Admin integration**

The admin integration will be a real fun part for me. I already worked many
many hours with customizing the admin. Many clients want another widget to be
used here, adding some style changes to a field there etc.

All the stuffs is already possible ofcourse through custom widgets but we
still need to overwrite the forms in the admin code. A cool thing would be to
make the use of chromes easier here.

We will add some hooks to the ``ModelAdmin`` class that allow you to add
chromes to fields, maybe in a way like ``formfield_overrides`` works but
propably in a more flexible manner. This is already easy to use for most
designers since reading the ``admin.py`` file is usually very straight forward
and only requires some basic knowledges about assignments, lists and dicts.
However it would be even cooler to do this in the template.

First we will create a custom form layout for the admin. It's easy to
overwrite as described above so that you can change the look of the field
rows, errors, helptext pretty easily. Additionally there will be a block in
the ``change_form.html`` template that can be overridden by the user (the
example is showing the template ``admin/<appname>/change_form.html``)::

    {% extends "admin/change_form.html" %}

    {% block form %}
        {% form adminform using autocomplete "/customers/" for adminform.client %}
    {% endblock %}

Besides the integration, there will be some need to convert the existing
gimmicks like calendar popup, "add new" icon next to ForeingKey dropdowns,
etc. into chrome implementations.

**7. Documentation**

Lots of work but nothing to specify here...

Media aka. JS/CSS
-----------------

One of the other mainpoints in the discussions I reviewed for this proposal was
the use of JS and CSS files that must be maintained somehow to display them
how we already do through the media framework (e.g. ``{{ form.media }}``).

The problem with this is that with the new template tag we can change some
of the widgets in the template and introducing new dependencies. Thats why I
would like to have an alternative for the ``using`` argument in the
``{% form %}`` tag.

If ``using`` is replaced with ``configure``, the ``{% form %}`` tag will _not_
output the HTML in the current place. However it will record and remember the
usage of widgets and fields to determine which media files are required. An
example template would look like::

    {% block extrahead %}
        {% form myform configure widget "CalendarInput" for myform.birthday %}
                       ^--- The new widget for the field birthday will be
                            recorded, but the form will not be rendered.
        {{ myform.media.css }}
           ^--- Outputting all necessary css files.
    {% endblock %}

    {% block content %}
        {% form myform %}
           ^--- The form will be rendered as usual but with the
                "CalendarInput" widget that was specified in the other tag.
    {% endblock %}

    {% block footer %}
        {{ myform.media.js }}
           ^--- Outputting all necessary js files at the end of the document.
    {% endblock %}

I will also check the possibility and difficulty of a new ``{% formmedia %}``
tag that hooks into the template parsing system, reading until the end of the
template and analyzing the use of the ``{% form %}`` tag. This way it could
determine all changes that will be applied to the form before it gets
rendered, including all the necessary CSS dependencies that needs to be
imported in the header of the page.

It is not clarified yet, if the ``{% formmedia %}`` is possible at all with
the current template parsing implementation. There might be some risks that
need to be sorted out before starting with the implementation:

* By parsing from the ``{% formmedia %}`` tag until the end of the template
  might result in that all content after this tag is represented as a child
  node of it. What side effects are implied? Does it produce backwards
  incompatibilities with thirdparty template tags?

* What happens if the ``{% form %}`` tag is changing the widget of the form
  based on a context variable?

Estimates
---------

That's it so far with the proposal. In the following I will go a bit into the
timeline that I have in mind for the implementation.

1st week: Starting to layout the documentation. The form tag syntax based on
discussions from the mailing list should already be finalized.

2nd week: Examing what unittests are available for the current form rendering
and making sure they are stable for testing backwards compatibility during the
project.

3rd week: I will attend DjangoCon EU, hopefully giving a talk about the
revised form rendering and collecting more feedback in an open space.

4th week: Converting the current layouts into template based renderers, ensuring
backwards compatibility.

Goal: no HTML should be left now in the python source.

5th week: Mainly working on documentation and integrating the very last changes
based on discussions from DjangoCon EU.

Goal: All public APIs should be stable.

6th week: Starting to write tests and implementing the ``{% form %}`` tag to
be able to emulate all the rendering that is currently possible.

7th week: Implementing the necessary rendering modifiers like "fields",
"layout" etc. and the API for chrome.

8th week: Implementing the ``{% formmedia %}`` tag.

Goal: Project should be feature complete.

9th - 11th week:

* Validating backwards compatibility for the ``{% formmedia %}`` parsing
  implementation with thirdparty modules (see Media section).
* Converting the admin to use the new form rendering.
* Integrating lessons learned from the admin.
* Bugfixes and regression tests for problems that showed up in the work with
  the admin.

Goal: Code should be ready to be used in sample projects.

12th week: Finalizing, bugfixes and tweaking the documentation.

Unfortunatelly university is running in germany during the time that I will
work on the project. However based on my experience with last years of
university I think that I can provide about 30 hours per week for work on the
project, discussing questions and suggestions on the mailing list and so on.


So thanks a lot for reading so far, I really appreciate it and would love to
hear your feedback. Please keep in mind that the syntax is product of my own
taste and I haven't shown it to someone else yet. I'm confident that it will
be usable for most template authors and is also very flexible on the python
side for customizations. But there might still be a lot of room for critiques
and situations that I have not considered yet.

Gregor

(A formated version of this proposal is available at [2])


Addendum
--------

Most of my inspiration came from the *Revised form rendering* thread [1]
kicked off by Russell last year after DjangoCon EU. Thanks also goes to Jannis
and Bruno for giving me some advice and making clear what the current state of
template based widgets is and much more.

My first thought was to include template based widgets also in this proposal.
But Bruno has already made a great effort with developing
**django-floppyforms** that might also get merged into django. He is already
working on a patch [3].


* [1] http://groups.google.com/group/django-developers/browse_thread/thread/cbb3aee22a0f8918
* [2] https://gist.github.com/898375
* [3] https://github.com/brutasse/django/compare/15667-template-widgets
