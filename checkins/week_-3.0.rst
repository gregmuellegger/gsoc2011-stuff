Hi,

let me introduce myself a bit before I start talking about my upcoming work
during the summer. My name is Gregor Müllegger and I was accepted as a Google
Summer of Code Student for Django, doing the project called "Revised form
rendering". Next Monday will start my fourth year at the university, I'm
studying Computer Science at the University of Augsburg which is 60km West of
Munich, Germany. I'm also doing freelancing work with Django to finance my
studies.

My project will basically try to make the current form rendering possibilities
more flexible and to remove all the hardcoded HTML from the django source
related to forms. You can have a look at the django-dev mailing list
discussion about my proposal online:
http://groups.google.com/group/django-developers/browse_thread/thread/8eb1f07bfd949ab7

My mentor is Carl Meyer, I already spoke to him and I'm very sure that we will
make a great team in bringing this project to a successful ending in August.
We will use the next weeks to speak intensively about the design issues that
still needs to be resolved and other things that needs to be done before the
GSoC time starts in May.

Today we discussed how we want to approach the design phase. We still need to
finalize for example the exact syntax of the proposed form template tag. We
came up with the idea of a sample project that we will put online in a
repository on github. This repo will first contain some scenarios in the
possible usage of the form tag, exploring it's behaviour in edge cases etc.
Others could fork the repo and contribute different syntax ideas, making it
easy to compare all variations. In the end we will have a resource were we can
point at in questions about why we have chosen this API.

In an IRC chat between Carl, Jannis and me on #django-dev, we choose to
develop the project directly against django's trunk. This means that we
decided against the alternative of developing the proposed API
as an external app. We still want to find an easy way for interested testers
to check out the current state in an already existing django project.


Here are some resources you might want to track during the summer if you are
interested in this GSoC project:

* My django fork on github will have a soc2011/form-rendering branch that will
contain my updates that might get merged into django's trunk at the end of
summer [1]. My changes compared to django's trunk will be visible there as
well [2].

* Keep track of the django-form-rendering-api repository if you want to know
how things go in the design process. [3] There's nothing in there yet, but
this will change during the next week.

* Join us in #django-dev on Freenode! Carl and I will use this as our IM
solution so that you can jump into the discussion if you are interested.

[1] https://github.com/gregmuellegger/django/tree/soc2011%2Fform-rendering
[2] https://github.com/gregmuellegger/django/compare/django:master...gregmuellegger:soc2011/form-rendering
[3] https://github.com/gregmuellegger/django-form-rendering-api

That's it so far this week. Thanks for your attention :-)


--
Servus,
Gregor Müllegger
