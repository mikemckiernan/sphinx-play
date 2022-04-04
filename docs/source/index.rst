Proposed automation for documentation
=====================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

Goals
-----

The goal is to add some automation for documentation to GitHub projects.

- When I open a PR, perform the following steps:

  - Build the HTML documentation as part of the Python build and test workflow.

  - Create a review directory for the PR in GitHub Pages.

  - Add the HTML to the review directory.

  - Update the PR with a comment that includes a review URL.

- When I push updates to a PR, rebuild the documentation and update the review directory in GitHub Pages.

- When I close a PR, remove the review directory from GitHub Pages.

- When I merge a PR, rebuild the documentation, and update the ``main`` directory for GitHub Pages.

Run the workflow for pull requests that are based from the ``main`` branch only.


Prerequisites
-------------

* Enable GitHub Pages, typically on the ``gh-pages`` branch.

* Add a ``.nojekyll`` file to the root directory on the GitHub Pages branch.


Anticipated implementation
--------------------------

1. Update the workflow that builds the Python project and add a few steps:

   - Build the documentation.

   - Store the result of the documentation build  and pull request information
     with ``actions/upload-artifact``.

   .. code-block:: yaml

      - name: Make HTML
        run: |
          pushd docs
          make html
          popd
      - name: Upload HTML
        uses: actions/upload-artifact@v2
        with:
          name: html-build-artifact
          path: docs/build/html
          if-no-files-found: error
          retention-days: 1
      - name: Store PR information
        run: |
          mkdir ./pr
          echo ${{ github.event.number }}              > ./pr/pr.txt
          echo ${{ github.event.pull_request.merged }} > ./pr/merged.txt
          echo ${{ github.event.action }}              > ./pr/action.txt
      - name: Upload PR information
        uses: actions/upload-artifact@v2
        with:
          name: pr
          path: pr/

  A workflow that triggers on a pull request does not have access to the
  data about the pull request, so the pull request information is stored
  as an artifact to pass the data to the second workflow.

  Storing the pull request information as an artifact has a kludgy feel, but
  seems to be a standard practice. The following two URLs show nearly identical
  content.

  - https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#using-data-from-the-triggering-workflow

  - https://securitylab.github.com/research/github-actions-preventing-pwn-requests

2. Add a workflow that is triggered by the build workflow.
   This second workflow downloads the artifacts and updates GitHub Pages.

   It's kind of dull reading, so peek in ``.github/workflows/docs-preview-pr.yaml``.

   .. note::

      As of 08MAR, it seems that the workflow files must be pushed to the ``main``
      branch of the repo.
      Two attempts were made to add the workflow files on a pull request.
      On both attempts, the workflow files did not run--that makes some sense
      from the perspective of not running untrusted code.


Security considerations
-----------------------

Because the team uses public repositories, it is important to have the documentation workflow operate for forks.

Working with forks is a complicating factor for the two goals of adding a comment to the pull request and
adding HTML to the branch that is used for GitHub Pages.

The following page shows that a forked repository has ``read`` access from the ``GITHUB_TOKEN`` on all
scopes and specifically for the ``issues`` and ``pages`` scopes:

  https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token

The proposed method for providing automation for pull requests and limiting risk is shown in the
following blog:

  https://securitylab.github.com/research/github-actions-preventing-pwn-requests/


Questions and next steps
------------------------

Use of secrets.GITHUB_TOKEN
~~~~~~~~~~~~~~~~~~~~~~~~~~~

I felt that it was worthwhile to use the ``gh`` CLI whenever possible just to reduce complexity.

I referred to the following documentation from GitHub:

- https://docs.github.com/en/actions/using-workflows/using-github-cli-in-workflows

- https://github.blog/2021-03-11-scripting-with-github-cli/

The GitHub documentation routinely shows the following pattern:

.. code-block:: yaml

   - run: gh ...some-command...
     env:
       GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

I need to know if using ``${{ secrets.GITHUB_TOKEN }}`` is a deal breaker.


Preserve the review HTML for a little longer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

My inclination is to revise the workflow so that it does not delete the
``review/pr-PR_NO`` directory immediately when a pull request is closed.

I think there is value to leaving the review HTML for a week after a
pull request is closed.
It provides a quick way to stare-and-compare between what a pull request
submitted and the HTML for the ``main`` branch.

My proposal is to add a second workflow on a daily schedule that performs
the following actions:

- Check out the branch that is used for GitHub Pages.

- Find the directories that are older than a week.

- Use the pull request number from the directory name to
  query the GitHub API to determine if the related pull request
  is closed.

- If the pull request is closed, delete the review directory.
