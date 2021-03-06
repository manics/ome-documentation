Bio-Formats jobs
----------------

All jobs are listed under the :jenkinsview:`Bio-Formats` view tab of Jenkins.

.. list-table::
        :header-rows: 1

        -       * Job task
                * 5.x series

        -       * Builds the latest Bio-Formats artifacts using Ant
                * | :term:`BIOFORMATS-DEV-latest`
                    :term:`BIOFORMATS-DEV-latest-win`

        -       * Builds the latest Bio-Formats artifacts using Maven
                * :term:`BIOFORMATS-DEV-latest-maven`

        -       * Runs the daily Bio-Formats merge jobs
                * :term:`BIOFORMATS-DEV-merge-daily`

        -       * Merges the PRs
                * :term:`BIOFORMATS-DEV-merge-push`

        -       * Builds the merge Bio-Formats artifacts using Ant
                * | :term:`BIOFORMATS-DEV-merge-build`
                    :term:`BIOFORMATS-DEV-merge-build-win`

        -       * Builds the merge Bio-Formats artifacts using Maven
                * :term:`BIOFORMATS-DEV-merge-maven`

        -       * Runs the MATLAB tests
                * :term:`BIOFORMATS-DEV-merge-matlab`

        -       * Runs automated tests against the full repository on squig
                * | :term:`BIOFORMATS-DEV-merge-full-repository`
                    :term:`BIOFORMATS-DEV-merge-full-repository-win`

        -       * Runs automated tests against a subset of the data repository on squig
                * :term:`BIOFORMATS-DEV-merge-repository-subset`

        -       * Runs performance tests
                * :term:`BIOFORMATS-DEV-merge-performance`

        -       * Installs Bio-Formats using Homebrew
                * :term:`BIOFORMATS-DEV-merge-homebrew`

5.x.x series
^^^^^^^^^^^^

The branch for the 5.x series of Bio-Formats is develop.

.. glossary::

        :jenkinsjob:`BIOFORMATS-DEV-latest`

                This job builds the latest Bio-Formats artifacts using Ant

                #. |buildBF|
                #. Triggers downstream latest jobs

            See :jenkinsjob:`the build graph <BIOFORMATS-DEV-latest/lastSuccessfulBuild/BuildGraph>`

        :jenkinsjob:`BIOFORMATS-DEV-latest-win`

                This job builds the latest Bio-Formats artifacts using Ant
                on Windows

        :jenkinsjob:`BIOFORMATS-DEV-latest-maven`

            This job builds the latest Bio-Formats artifacts using Maven and
            uploads them to the `OME artifactory`_

        :jenkinsjob:`BIOFORMATS-DEV-merge-daily`

                This job runs the daily Bio-Formats jobs used for reviewing
                the PRs opened against the develop branch of Bio-Formats by
                running basic unit tests and checking for regressions across a
                representative subset of the data repository

                #. Triggers :term:`BIOFORMATS-DEV-merge-push`
                #. Triggers :term:`BIOFORMATS-DEV-merge-build` and :term:`BIOFORMATS-DEV-merge-maven`
                #. Triggers downstream merge projects

                See :jenkinsjob:`the build graph <BIOFORMATS-DEV-merge-daily/lastSuccessfulBuild/BuildGraph>`

        :jenkinsjob:`BIOFORMATS-DEV-merge-push`

                This job merges all the PRs opened against develop

                #. |merge|
                #. Pushes the branch to :bf_scc_branch:`develop/merge/daily`

        :jenkinsjob:`BIOFORMATS-DEV-merge-build`

                This job builds the merge Bio-Formats artifacts using Ant

                #. Checks out :bf_scc_branch:`develop/merge/daily`
                #. |buildBF|
                #. Triggers :term:`BIOFORMATS-DEV-merge-matlab`

        :jenkinsjob:`BIOFORMATS-DEV-merge-build-win`

                This job builds the merge Bio-Formats artifacts using Ant
                on Windows

        :jenkinsjob:`BIOFORMATS-DEV-merge-maven`

            This job builds the merge Bio-Formats artifacts using Maven

        :jenkinsjob:`BIOFORMATS-DEV-merge-matlab`

                This job runs the MATLAB tests of Bio-Formats

                #. Collects the MATLAB artifacts and unit tests from
                   :term:`BIOFORMATS-DEV-merge-build`
                #. Runs the MATLAB unit tests under
                   :file:`components/bio-formats/test/matlab` and collect the results

        :jenkinsjob:`BIOFORMATS-DEV-merge-full-repository`

                This job runs the automated tests against the curated data
                repository on Linux

                #. Checks out :bf_scc_branch:`develop/merge/daily`
                #. Runs automated tests against :file:`/ome/data_repo/curated/`

        :jenkinsjob:`BIOFORMATS-DEV-merge-full-repository-win`

                This job runs the automated tests against the curated data
                repository on Windows

                #. Checks out :bf_scc_branch:`develop/merge/daily`
                #. Runs automated tests against
                   :file:`\\\\squig.openmicroscopy.org.uk\\ome-data-repo\\curated`

        :jenkinsjob:`BIOFORMATS-DEV-merge-repository-subset`

                This job runs the automated tests against a subset of the data
                repository

                #. |merge|
                #. Runs automated tests against a subset of format directories
                   under :file:`/ome/data_repo/curated/`. The list of
                   directories to test by setting a space-separated list of
                   formats for the ``DEFAULT_FORMAT_LIST`` variable.

        :jenkinsjob:`BIOFORMATS-DEV-merge-performance`

                This job runs performance tests against directories on squig

                #. Checks out the :bf_scc_branch:`develop/merge/daily`
                #. Runs file-handles and openbytes-performance tests against
                   files specified by :file:`performance_files.txt`

        :jenkinsjob:`BIOFORMATS-DEV-merge-homebrew`

                This job builds Bio-Formats using MacOS X Homebrew

.. _bf_breaking:

Breaking jobs
^^^^^^^^^^^^^

Breaking jobs are jobs used to review breaking changes, for instance model
changes. The branch for the breaking series of Bio-Formats is develop.

.. glossary::
        :jenkinsjob:`BIOFORMATS-DEV-breaking-trigger`

                This job runs the daily Bio-Formats jobs used for reviewing
                the PRs opened against the develop branch

                #. Triggers :term:`BIOFORMATS-DEV-breaking-push`
                #. Triggers :term:`BIOFORMATS-DEV-breaking-build` and
                   :term:`BIOFORMATS-DEV-breaking-maven`
                #. Triggers downstream jobs

                See :jenkinsjob:`the build graph <BIOFORMATS-DEV-breaking-trigger/lastSuccessfulBuild/BuildGraph>`

        :jenkinsjob:`BIOFORMATS-DEV-breaking-push`

                This job merges all the breaking PRs opened against develop

                #. |merge| labeled as breaking
                #. Pushes the branch to :bf_scc_branch:`develop/breaking/trigger`

        :jenkinsjob:`BIOFORMATS-DEV-breaking-build`

                This job builds the breaking Bio-Formats artifacts using Ant

                #. Checks out :bf_scc_branch:`develop/breaking/trigger`
                #. |buildBF|
                #. Triggers :term:`BIOFORMATS-DEV-breaking-matlab`

        :jenkinsjob:`BIOFORMATS-DEV-breaking-maven`

                This job build the breaking Bio-Formats artifacts using Maven

        :jenkinsjob:`BIOFORMATS-DEV-breaking-matlab`

                This job runs the MATLAB tests of Bio-Formats

                #. Collects the MATLAB artifacts and unit tests from
                   :term:`BIOFORMATS-DEV-breaking-build`
                #. Runs the MATLAB unit tests under
                   :file:`components/bio-formats/test/matlab` and collect the results


        :jenkinsjob:`BIOFORMATS-DEV-breaking-full-repository`

                This job runs the automated tests against the curated data
                repository on Linux

                #. Checks out :bf_scc_branch:`develop/breaking/trigger`
                #. Runs automated tests against :file:`/ome/data_repo/curated/`

        :jenkinsjob:`BIOFORMATS-DEV-breaking-repository-subset`

                This job runs the automated tests against a subset of the data
                repository on Linux

                #. |merge| labeled as breaking
                #. Runs automated tests against a subset of format directories under
                   :file:`/ome/data_repo/curated/`. The list of directories to
                   test by setting a space-separated list of formats for the
                   ``DEFAULT_FORMAT_LIST`` variable.

        :jenkinsjob:`BIOFORMATS-DEV-breaking-performance`

                This job runs performance tests against directories on squig

                #. Checks out the :bf_scc_branch:`develop/breaking/trigger`
                #. Runs file-handles and openbytes-performance tests against
                   files specified by :file:`performance_files.txt`

