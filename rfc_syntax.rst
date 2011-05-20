Request for comment on form tag syntax
======================================

Hi,
like you might know I've prepared as pre-work to my GSoC project a repository
[1] with examples for two different approaches to my upcoming work on the form
rendering. The first approach is called "single_tag" [2] the second one
"modifier_tags" [3]. I agreed with my mentor Carl that we will follow more the
style of the "modifier_tags" syntax.

I tried here to summerize a bit how the new tags will look like. I call now
comments and will really appreciate any feedback on them. Beeing it about
their naming or if you have ideas for other tags. Be picky, this is something
lots of people will use -- and though we want it to be as easy as possible to
get started with.

::

    {% renderform <form instance> %}

The renderform tag obviously renders the form. It doesn't do anything else,
but it will take into account of what option other form modifier tags have
set.

One such modifier tag is the formlayout tag::

    {% formlayout <layout name> %}

Every {% renderform %} that is following in the templates will use the
specified layout for rendering. We will provide layouts called "table", "ul"
and "p". Users can specify new layouts just by putting some templates in the
right template path.

The {% form %} tag is limiting the scope of these modifier tags. If a modifier
tag is wrapped in such a form tag, then it will lose its influence on form
rendering outside of the form block-tag. ::

    {% form %}
        {% formlayout "table" %}
        {% renderform my_form %} {# will use table layout #}
    {% endform %}

    {% renderform my_form %} {# will use default layout #}

The description of the form-tag implies that the modifier tags are able to
set/modify the global state in the current template. That is something that is
clearly wanted. This way you can set a "formlayout" in the head of your base
template and any other extending template will have this default for the form
rendering.

Here is are some more tags:

**renderfield**: same as renderform but will only render on given field::

    {% renderfield <bound formfield> %}

**widget**: You can change the widget used to render a set of fields::

    {% widget <widget class or instance> for <field type> [with <varname>=<varvalue> ...] %}

In this example (and the following) <field type> means that you can specify
one of three things as argument:

1. A bound field, means that the widget will be rendered only for this field.
2. A field class/type that will match for all fields that are using this
   formfield class. Example::

    {% widget PasswordInput for CharField %}

   will render a <input type="password" /> for all charfields.

3. A field name (string). This will apply to all fields that have that
   particular field name. It's only usefull if the option should apply to more
   than one form.

Some examples::

    {% widget Textarea for my_form.comment %} (case 1.)
    {% widget DatePicker for DateField %} (case 2.)
    {% widget PasswordInput for "password" %} (case 3.)

The "with varname=varvalue" bit in the widget tag is meant as possibility to
pass extra arguments into the template that will be used to render the widget.
This will use the same syntax as django's ``include`` tag [4]. We assume at
this point that until this GSoC project is finished we will likely have
template based widget rendering like currently developed by Bruno [5].

**excludefield**: Prevent rendering fields in following renderform tags::

    {% excludefield <field type> %}

**hiddenfield**: Mark fields that they will be rendered as hidden fields::

    {% hiddenfield <field type> %}

Thanks if read that far. Now please start commenting :-)

Gregor

| [1] https://github.com/gregmuellegger/django-form-rendering-api
| [2] https://github.com/gregmuellegger/django-form-rendering-api/blob/master/formapi/templates/api_samples/single_tag.html
| [3] https://github.com/gregmuellegger/django-form-rendering-api/blob/master/formapi/templates/api_samples/modifier_tags.html
| [4] http://docs.djangoproject.com/en/dev/ref/templates/builtins/#std:templatetag-include
| [5] http://code.djangoproject.com/ticket/15667
