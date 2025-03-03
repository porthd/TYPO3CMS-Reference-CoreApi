.. include:: /Includes.rst.txt
.. highlight:: typoscript
.. index:: LinkHandlers
.. _linkhandler:

===================
The LinkHandler API
===================

The LinkHandler API currently consists of 7 LinkHandler classes and the
:php:`TYPO3\CMS\Backend\LinkHandler\LinkHandlerInterface`. The
LinkHandlerInterface can be implemented to create custom LinkHandlers.

..  versionchanged:: 12.0
    Due to the integration of EXT:recordlist into EXT:backend the namespace of
    LinkHandlers has changed from
    :php:`TYPO3\CMS\Recordlist\LinkHandler`
    to
    :php:`TYPO3\CMS\Backend\LinkHandler`.
    For TYPO3 v12 the moved classes are available as an alias under the old
    namespace to allow extensions to be compatible with TYPO3 v11 and v12.

Most LinkHandlers cannot receive additional configuration, they are marked as
:php:`@internal` and contain neither hooks nor events. They are therefore
of interest to Core developers only.

Current LinkHandlers:

*  :ref:`pagelinkhandler`: for linking pages and content
*  :ref:`recordlinkhandler`: for linking any kind of record
*  UrlLinkHandler: for linking external urls
*  FileLinkHandler: for linking files in the :ref:`fal`
*  FolderLinkHandler: for linking to directories
*  MailLinkHandler: for linking email addresses
*  TelephoneLinkHandler: for linking phone numbers

.. note::

   In the system extension :file:`core` there are also classes ending on
   "LinkHandler". However those implement the interface :php:`LinkHandlingInterface`
   and are part of the LinkHandling API, not the LinkHandler API.

The following LinkHandlers are of interest:

.. toctree::
   :titlesonly:

   PageLinkHandler
   RecordLinkHandler
   CustomLinkHandlers

The links are now stored in the database with the syntax
`<a href="t3://record?identifier=anIdentifier&amp;uid=456">A link</a>`.

#. TypoScript is used to generate the actual link in the frontend.

   .. code-block:: typoscript

      config.recordLinks.anIdentifier {
          // Do not force link generation when the record is hidden
          forceLink = 0
          typolink {
              parameter = 123
              additionalParams.data = field:uid
              additionalParams.wrap = &tx_example_pi1[item]=|&tx_example_pi1[controller]=Item&tx_example_pi1[action]=show
          }
      }

   .. important::

      Do not change the identifier after links have been created  using the LinkHandler. The identifier will be
      stored as part of the link in the database.


.. index::
   pair: LinkHandler; Page TSconfig
   TCEMAIN; linkHandler
.. _linkhandler-pagetsconfig:

LinkHandler page TSconfig options
=================================

The minimal PageTSconfig Configuration is:

.. code-block:: typoscript
   :caption: EXT:some_extension/Configuration/page.tsconfig

   TCEMAIN.linkHandler.anIdentifier {
       handler = TYPO3\CMS\Backend\LinkHandler\RecordLinkHandler
       label = LLL:EXT:extension/Resources/Private/Language/locallang.xlf:link.customTab
       configuration {
           table = tx_example_domain_model_item
       }
   }

The following optional configuration is available:

:typoscript:`configuration.hidePageTree = 1`
   Hide the page tree in the link browser

:typoscript:`configuration.storagePid = 84`
   The link browser starts with the given page

:typoscript:`configuration.pageTreeMountPoints = 123,456`
   Only records on these pages and their children will be displayed

Furthermore the following options are available from the LinkBrowser Api:

:typoscript:`scanAfter = page` or :typoscript:`scanBefore = page`
   define the order in which handlers are queried when determining the responsible tab for an existing link

:typoscript:`displayBefore = page` or :typoscript:`displayAfter = page`
   define the order how the various tabs are displayed in the link browser.

Example: news records from one storage pid
------------------------------------------

The following configuration hides the page tree and shows news records only
from the defined storage page:

.. code-block:: typoscript
   :caption: EXT:some_extension/Configuration/page.tsconfig

   TCEMAIN.linkHandler.news {
       handler = TYPO3\CMS\Backend\LinkHandler\RecordLinkHandler
       label = News
       configuration {
           table = tx_news_domain_model_news
           storagePid = 123
           hidePageTree = 1
       }
       displayAfter = email
   }

It is possible to have another configuration using another storagePid which
also contains news records.

This configuration shows a reduced page tree starting at page with uid 42:

.. code-block:: typoscript
   :caption: EXT:some_extension/Configuration/page.tsconfig

   TCEMAIN.linkHandler.bookreports {
       handler = TYPO3\CMS\Backend\LinkHandler\RecordLinkHandler
       label = Book Reports
       configuration {
           table = tx_news_domain_model_news
           storagePid = 42
           pageTreeMountPoints = 42
           hidePageTree = 0
       }
   }

The page TSconfig of the LinkHandler is being used in sysext `backend`
in class :php:`\TYPO3\CMS\Backend\LinkHandler\RecordLinkHandler`
which does not contain Hooks.

Enable page id field
--------------------

It is possible to enable an additional field in the link browser to enter the uid of a page.
The uid will be used directly instead of selecting it from the page tree.

This only works for the :php:`PageLinkHandler`.
It will **not** work for custom added LinkHandler configurations.

.. figure:: Images/LinkBrowserTSConfigExamplepageIdSelector.png
   :alt: The link browser field for entering a page uid.

Enable the field with the following page TSConfig:

.. code-block:: typoscript
   :caption: EXT:some_extension/Configuration/page.tsconfig

   TCEMAIN.linkHandler.page.configuration.pageIdSelector.enabled = 1


.. index::
   pair: LinkHandler; TypoScript
   TypoScript; config.recordLinks
.. _linkhandler-typoscript:

LinkHandler TypoScript options
==============================

A configuration could look like this:

.. code-block:: typoscript
   :caption: EXT:some_extension/Configuration/TypoScript/setup.typoscript

   config.recordLinks.anIdentifier {
       forceLink = 0

       typolink {
           parameter = 123
           additionalParams.data = field:uid
           additionalParams.wrap = &tx_example_pi1[item]=|
       }
   }

The TypoScript Configuration of the LinkHandler is being used in sysext `frontend`
in class :php:`TYPO3\CMS\Frontend\Typolink\DatabaseRecordLinkBuilder`.

Example: news records displayed on fixed detail page
----------------------------------------------------

The following displays the link to the news on a detail page:

.. code-block:: typoscript
   :caption: EXT:some_extension/Configuration/TypoScript/setup.typoscript

   config.recordLinks.news {
      typolink {
         parameter = 123
         additionalParams.data = field:uid
         additionalParams.wrap = &tx_news_pi1[controller]=News&tx_news_pi1[action]=detail&tx_news_pi1[news]=|
      }
   }

Once more if the book reports that are also saved as `tx_news_domain_model_news` record should be displayed on their own
detail page you can do it like this:

.. code-block:: typoscript
   :caption: EXT:some_extension/Configuration/TypoScript/setup.typoscript

   config.recordLinks.bookreports  {
      typolink {
         parameter = 987
         additionalParams.data = field:uid
         additionalParams.wrap = &tx_news_pi1[controller]=News&tx_news_pi1[action]=detail&tx_news_pi1[news]=|
      }
   }
