.. sphinx-play documentation master file, created by
   sphinx-quickstart on Tue Mar  1 16:25:36 2022.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to sphinx-play's documentation!
=======================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

GitHub workflows for documentation
----------------------------------

The goal is to add some automation to GitHub projects for documentation.

- [ ] When I open a PR, build the documentation, add it to GitHub Pages, and update the PR with a comment that includes a review URL.

- [ ] When I push updates to a PR, rebuild the documentation.

- [ ] When I close a PR, remove the review HTML from GitHub Pages.

- [ ] When I merge a PR, rebuild the documentation, and update the ``main`` directory for GitHub Pages.

The workflow only runs on pull requests that are based from the ``main`` branch.


Security considerations
-----------------------

The initial attempt worked for pushes and PRs by someone with access to push to the repo.
For public repositories, it is important to have the documentation workflow operate for forks.

The complicating factor is that it is very helpful to add a comment to the PR with the URL
of the documentation preview.
However, adding a comment to a PR is a privileged operation that requires ``write`` permission to the ``issue`` type.
The suggested method for limiting the risk is covered in the following blog post:

https://securitylab.github.com/research/github-actions-preventing-pwn-requests/


Some things I learned
---------------------

I cannot explain why, but the `synchronize` state for a `pull_request` is reported
in the GitHub UI, but it is not reported in the events that can be downloaded with `curl`:

.. code-block:: shell

   curl -H "Accept: application/vnd.github.v3+json" https://api.github.com/repos/mikemckiernan/sphinx-play/events

Pushes to a branch that has an open PR seem to run the actions again.
In the case of this workflow, the HTML is build again and deployed.

Regarding the comment with the URL to the proposed changes, I had a boo-boo
in my logic:

.. code-block:: yaml

   if: github.event.pull_request.action == 'opened'

Silly me.  The `action` field is not a child of the `pull_request`.
It is a child of `github.event`.


One mystery
-----------

I have an update on the mystery.
I'm fairly sure it was triggered by a race due to having the
``store-html`` job and ``remove-html`` job run simultaneously.
Both jobs checked out the GitHub Pages branch.
On a merge, the ``store-html`` job added HTML to the ``main`` directory and pushed.
At approximately the same time, the ``remove-html`` job removed the review directory and pushed.

I believe the fix is to consolodate the two jobs into a single job that handles all the permutations.

PR is merged
  Replace the ``main`` directory with the HTML artifact and remove the review HTML directory.

PR is closed but not merged
  Remove the review HTML directory.

PR is updated
  Replace the HTML review directory with the HTML artifact.

...Original demonstration of ignorance follows...

I'm unsure why the process in the ``store-html`` job somehow
seems to have missing commits.

.. code-block:: text

   Changes to be committed:
     (use "git restore --staged <file>..." to unstage)
           modified:   main/_sources/index.rst.txt
           modified:   main/index.html
           modified:   main/searchindex.js

  [gh-pages 3daa0dd] Adding HTML directory.
   3 files changed, 25 insertions(+), 1 deletion(-)
   rewrite main/searchindex.js (64%)
  To https://github.com/mikemckiernan/sphinx-play
   ! [rejected]        gh-pages -> gh-pages (fetch first)
  error: failed to push some refs to 'https://github.com/mikemckiernan/sphinx-play'
  hint: Updates were rejected because the remote contains work that you do
  hint: not have locally. This is usually caused by another repository pushing
  hint: to the same ref. You may want to first integrate the remote changes
  hint: (e.g., 'git pull ...') before pushing again.
  hint: See the 'Note about fast-forwards' in 'git push --help' for details.
  Error: Process completed with exit code 1.

There's something that I need to learn about Git.
I have a misconception that a checkout would get the latest work.


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
