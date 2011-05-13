Hi,

I used the last week to setup the 'test environment' for proof-of-concept'ing
the API we will be implementing. The first thing there is that I setup a (yet)
simplistic template for

1. the form tag, the monolithic approach how I have proposed it
2. multiple tags like proposed by Russell.

Both can be found in the form-rendering-api repo on github [1 & 2].
Feel free to fork the repo, adding more of these proposals or changing the API
of already proposed syntaxes.

The other thing I did was a quick proof-of-concept of the idea that came up
for a revised media handling. The problem with media handling is, that there
might arise the need in the middle of the template to inject some CSS
definitions in the head of the page. It turned out that this can be done with
the current template engine. See the github repo for implementation details
[3]. It also contains tests where you can see the tag in action.

Here a short teaser for the {% media %} tag::

    {% load media_tags %}
    <head>
        {% media "css" %}
    </head>
    <body>
        {% addmedia "css" %}
        <style type="text/css">
            p { color: #ff0000; }
        </style>
        {% endaddmedia %}
        <p>This text will be red.</p>
    </body>

... will render to ... ::

    <head>
        <style type="text/css">
            p { color: #ff0000; }
        </style>
    </head>
    <body>
        <p>This text will be red.</p>
    </body>

| [1] https://github.com/gregmuellegger/django-form-rendering-api/blob/master/formapi/templates/api_samples/single_tag.html
| [2] https://github.com/gregmuellegger/django-form-rendering-api/blob/master/formapi/templates/api_samples/modifier_tags.html
| [3] https://github.com/gregmuellegger/django-form-rendering-api/blob/master/formapi/media_framework/templatetags/media_tags.py

--
Servus,
Gregor Müllegger
