How to write in reStructuredText
=====================================

Math equations
--------------

This is a text with the equation :math:`3 · 3^2 + \Omega_2` in the middle.
Just start with "(:)math(:)" clausule and write the equation in LateX format between "`".

This another text with an equation in the next paragraph.

.. math::
    \Omega = 3 + 4

You can define a label for the equation so you can cite later:

.. math:: e^{i\pi} + 1 = 0
   :label: euler


The equation get a number and you can it cite as :eq:`euler`, and continue writting. Also you get an atomatic link to the equation.

read the docs for for information:

http://www.sphinx-doc.org/es/stable/ext/math.html


Cites
-----

It is also possible to write simple cites as this one [1]_ or this other [2]_:
This makes a link to the reference in the botom. 

Just write the citation list wherever you want

.. [1] Book or article reference, URL or whatever.
.. [2] Other Book or article reference.