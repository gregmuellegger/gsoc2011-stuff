Hi,

this week was the first week with official coding for my GSoC project. I
merged Bruno Renié's template based widget rendering branch in my django fork.
This will be the base of my on going work. I also made sure that all tests are
passing, since there were some issues with changes in the whitespace and
ordering of HTML attributes in the new widget rendering.

I except that my upcoming changes will propably change also some attributes
ordering and lots of whitespace. I took this as a motivation to write a
assertHTMLEqual TestCase method that compares two strings that contain HTML
markup. It parses the HTML into a object structure which can be easily
compared without beeing too pickey about e.g. trailing whitespace, attribute
ordering, closing tags etc.. See the tests to get an impression on how it
looks in action:
https://github.com/gregmuellegger/django/blob/soc2011%2Fform-rendering/tests/regressiontests/test_utils/tests.py#L121

Watch my progress on github:
https://github.com/gregmuellegger/django/tree/soc2011%2Fform-rendering

--
Servus,
Gregor Müllegger
