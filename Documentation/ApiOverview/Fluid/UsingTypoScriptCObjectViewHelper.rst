.. include:: /Includes.rst.txt
.. index::
   TypoScript ViewHelper
   Fluid; f:cObject

==================
cObject ViewHelper
==================

The cObject ViewHelper combines Fluid with TypoScript.
The following line in the HTML template will be replaced with the referenced
TypoScript object.

.. code-block:: html
   :caption: EXT:my_extension/Resources/Private/Templates/SomeTemplate.html

   <f:cObject typoscriptObjectPath="lib.title"/>

Now we only have to define :typoscript:`lib.title` in the TypoScript
Setup:

.. code-block:: typoscript
   :caption: EXT:my_extension/Configuration/TypoScript/setup.typoscript

   lib.title = TEXT
   lib.title.value = Extbase and Fluid

»Extbase and Fluid« will be outputted in the template. Now we can output an
image (e.g. headlines with unusual fonts) by changing the TypoScript to:

.. code-block:: typoscript
   :caption: EXT:my_extension/Configuration/TypoScript/setup.typoscript

   lib.title = IMAGE
   lib.title {
      file = GIFBUILDER
      file {
         10 = TEXT
         10.value = Extbase and Fluid
      }
   }

.. sidebar:: TypoScript

   TypoScript is a flexible configuration language, which can control
   the rendering of a page in much detail. It consists of TypoScript objects
   (also known as :typoscript:`Content` object or :typoscript:`cObject`) and
   their configuration options.

   The simplest :typoscript:`Content` object is :typoscript:`TEXT`
   which outputs unmodified text. The TypoScript object :typoscript:`IMAGE`
   can be used to generate images, and database entries can be outputted
   with :typoscript:`CONTENT`.

So far, it's not a "real world" example because no data is
being passed from Fluid to the TypoScript. We'll demonstrate how to pass
a parameter to the TypoScript with the example of a user counter. The value
of our user counter should come from the Blog-Post. (Every Blog-Post should
count how many times it's been viewed in this example).

In the Fluid template we add:

.. code-block:: html
   :caption: EXT:my_extension/Resources/Private/Templates/SomeTemplate.html

   <f:cObject typoscriptObjectPath="lib.myCounter">{post.viewCount}</f:cObject>

Alternatively, we can use a self-closing tag. The data is being passed
with the help of the :html:`data` attribute.

.. code-block:: html
   :caption: EXT:my_extension/Resources/Private/Templates/SomeTemplate.html

   <f:cObject typoscriptObjectPath="lib.myCounter" data="{post.viewCount}" />

Also advisable for this example is the inline notation, because you can
easily read it from left to right:

.. code-block:: html
   :caption: EXT:my_extension/Resources/Private/Templates/SomeTemplate.html

   {post.viewCount -> f:cObject(typoscriptObjectPath: 'lib.myCounter')}

Now we still have to evaluate the passed value in our TypoScript
template. We can use the :typoscript:`stdWrap` attribute :typoscript:`current`
to achieve this. It works like a switch: If set to 1, the value, which we
passed to the TypoScript object in the Fluid template will be used. In our
example, it looks like this:

.. code-block:: typoscript
   :caption: EXT:my_extension/Configuration/TypoScript/setup.typoscript

   lib.myCounter = TEXT
   lib.myCounter {
      current = 1
      wrap = <strong>|</strong>
   }

This TypoScript snippet outputs the current number of visits written
in bold.

Now for example we can output the user counter as image instead of
text without modifying the Fluid template. We simply have to use the
following TypoScript:

.. code-block:: typoscript
   :caption: EXT:my_extension/Configuration/TypoScript/setup.typoscript

   lib.myCounter = IMAGE
   lib.myCounter {
      file = GIFBUILDER
      file {
         10 = TEXT
         10.text.current = 1
      }
   }

At the moment, we're only passing a single value to the TypoScript.
It's more versatile, though, to pass multiple values to the TypoScript object
because then you can select which value to use in the TypoScript, and the
values can be concatenated. You can also pass whole objects to the
ViewHelper in the template:

.. code-block:: html
   :caption: EXT:my_extension/Resources/Private/Templates/SomeTemplate.html

   {post -> f:cObject(typoscriptObjectPath: 'lib.myCounter')}

Now, how do you access individual properties of the object in the
TypoScript-Setup? You can use the property :typoscript:`field` of
:typoscript:`stdWrap`:

.. code-block:: typoscript
   :caption: EXT:my_extension/Configuration/TypoScript/setup.typoscript

   lib.myCounter = COA
   lib.myCounter {
      10 = TEXT
      10.field = title
      20 = TEXT
      20.field = viewCount
      wrap = (<strong>|</strong>)
   }

Now we always output the title of the blog, followed by the amount of
page visits in parenthesis in the example above.

You can also combine the :typoscript:`field` based approach with
:typoscript:`current`: If you set the property :html:`currentValueKey`
in the cObject ViewHelper, this value will be available in
the TypoScript template with :typoscript:`current`. That is especially useful
when you want to emphasize that the value is very
*important* for the TypoScript template. For example, the
*amount of visits* is significant in our view
counter:

.. code-block:: html
   :caption: EXT:my_extension/Resources/Private/Templates/SomeTemplate.html

   {post -> f:cObject(typoscriptObjectPath: 'lib.myCounter', currentValueKey: 'viewCount')}

In the TypoScript template you can now use both, :typoscript:`current`
and :typoscript:`field`, and have therefor the maximum flexibility with the
greatest readability. The following TypoScript snippet outputs the same
information as the previous example:

.. code-block:: typoscript
   :caption: EXT:my_extension/Configuration/TypoScript/setup.typoscript

   lib.myCounter = COA
   lib.myCounter {
      10 = TEXT
      10.field = title
      20 = TEXT
      20.current = 1
      wrap = (<strong>|</strong>)
   }

The cObject ViewHelper is a powerful option to use the
best advantages of both worlds by making it possible to embed TypoScript
expressions in Fluid templates.
