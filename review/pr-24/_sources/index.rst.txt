.. sphinx-play documentation master file, created by
   sphinx-quickstart on Tue Mar  1 16:25:36 2022.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to sphinx-play's documentation!
=======================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

Any heading at all
------------------

Add any content.


Add a para.

Hello from branch docs-hello.

Let's see if the workflow identifies the review branch to delete.
It didn't.
Trying again.

The HTML review directory is deleted.
Now, check if the HTML review directory is identified in the commit message.

I cannot explain why, but the `synchronize` state for a `pull_request` is reported
in the GitHub UI, but it is not reported in the events that can be downloaded with `curl`:

.. code-block:: shell

   curl -H "Accept: application/vnd.github.v3+json" https://api.github.com/repos/mikemckiernan/sphinx-play/events

Regardless, I'll make believe that workflows for the `pull_request` type are run after each push.

Pushes to a branch that has an open PR seem to run the actions again.
In the case of this workflow, the HTML is build again and deployed.

Let's see if it can be deployed more than once.
Let's also see if the comment is added to the PR when the PR is opened.
The comment is supposed to contain a URL to the proposed documentation change.

Regarding the comment with the URL to the proposed changes, I had a boo-boo
in my logic:

.. code-block:: yaml

   if: github.event.pull_request.action == 'opened'

Silly me.  The `action` field is not a child of the `pull_request`.
It is a child of `github.event`.


Add a separate workflow
-----------------------

I added a second workflow so that I can understand the pull_request event better.


PRs that do not trigger a workflow
----------------------------------

Either GitHub is broken, or I've done something incredibly stupid.
I can't tell.


More headings
-------------

* How do we add a comment to the PR with the URL to the preview?


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
