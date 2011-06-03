Hi,

this is the second RFC regarding template form rendering API/syntax. The first
one defined how a complete form or a subset of fields will be rendered. I'm
now proposing more tags to render more details like only the label of a form
field.

Currently we haven't discussed how we will substitute what we currently do
with using::

{{ form.non_field_errors }}
{{ form.errors }}
{{ form.field.errors }}
{{ form.field.help_text }}
{{ form.field.label_tag }}


We will implement these with template tags that are aware of the currently
used layout:

Substituting {{ \*.errors }}::

{% formerrors <form instance|field instance|any ErrorList/ErrorDict> %}

Some examples:
Display *all* errors of a form::
{% formerrors my_form %} (equals current {{ my_form.errors }})

Non-field errors::
{% formerrors my_form.non_field_errors %} equals {{ my_form.non_field_errors }}

Single-field errors::
{% formerrors my_form.firstname %} equals {{ my_form.firstname.errors }}

Basically it will render the error list/dict that is available in the *errors*
attribute of the passed in object. As an alternative (e.g. the
non_field_errors) you can pass in directly a error list that gets rendered::

{% formerrors my_custom_error_list %}
{% formerrors my_form.field.errors %}


Substituting ``{{ form.field.label_tag }}``

This one might be pretty obvious::
{% formlabel my_form.field %}


Substituting {{ form.field.help_text }}

Also pretty obvious::
{% formhelp my_form.field %}

But the naming is IMO not ideal. {% helptext %} as alternative is reasonable,
but Carl and I agreed on that we wanted the tag to start with form* but
{% formhelptext %} is to long. So we settled on {% formhelp %}
Better suggestions are welcome.


That's it basically. :-) I don't see any controversials in here, but as usual
with a RFC: comments are requested and will be integrated.
