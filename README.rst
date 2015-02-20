===================================
JMeter tests form ESD Review Tool
===================================


To run this tests a helping buildout is provided to get the calls correctly done. Just run the buildout and the helper scripts will be created inside directory bin::

    $ virtualenv .
    $ ./bin/buildout -vv


The generated scripts are the following::

    $ ./bin/logintest
    $ ./bin/editortest
    $ ./bin/reviewertest
    $ ./bin/eionetusertest

To run the tests manually, you need to pass the required parameters to each of the scripts::

    $ ./bin/logintest -n -JSERVERNAME=esddemo.eea.europa.eu -JUSERNAME=EIONET-USER-NAME -JPASSWORD=PASSWORD -JNUMOFTHREADS=3 -JLOOPCOUNT=5 -JRAMPUPPERIOD=1

This way, jmeter will create a results file called 'report_logintest.jtl'

The parameters that you need to set are the following:

- JSERVERNAME: full servername to be tested. ex; esddemo.eea.europa.eu
- JUSERNAME: valid EIONET username, with the appropriate permissions so that the test is successful
- JPASSWORD: valid EIONET username's password
- JNUMOFTHREADS: number of threads to be used in the test
- JLOOPCOUNT: number of repetitions of the test for each thread
- JRAMPUP-PERIOD: waiting time between tests


Running this tests with Jenkins
========================================================

Tests to be run with JMeter based on requirements of
https://taskman.eionet.europa.eu/issues/22685

- Create your tests using JMeter, but create them parametrized, this way you can create just one test case, and run in different scenarios: multiple users, different servers (dev/staging/production), and without adding user credentials (if needed) on your SCM.

  To do so, go to the **User Defined Variables** table in the Test Plan configuration in JMeter, and create all the needed variables.

  The tests created in this buildout use 6 variables. These are the values and its values:
    - username: ${__P(USERNAME)}
    - password: ${__P(PASSWORD)}
    - numofthreads: ${__P(NUMOFTHREADS)}
    - rampup-period: ${__P(RAMPUPPERIOD)}
    - loopcount: ${__P(LOOPCOUNT)}
    - servername: ${__P(SERVERNAME)}

  This allows to pass the correct parameters to JMeter when the test is run. You will need to use these variables (${username}, ${password}, etc), each time you need to get those values in JMeter. For instance, in the "HTTP Request Defaults" configuration you need to set ${servername} instead of the real servername.

- Install `Jenkins Performance plugin`_.

- Create a new Job using the `Freestyle project` template.

- Point to the git repository containing this buildout, this way Jenkins will download the latest version of the tests from the repository

- Check the "This build is parameterized" option and add the following parameters:
    - NUMOFTHREADS: type "string" default value "1"
    - LOOPCOUNT: type "string" default value "1"
    - RAMPUPPERIOD: type "string" default value "1"
    - SERVERNAME: type "string" default value "1"
    - USERNAME: type "string" default value "USERNAME"
    - PASSWORD: type "string" default value "PASSWORD"

  This parameters will be asked to the user before the execution of each build.

- Click on "Add build step" and select "Execute shell". Fill the command with the following::

    python bootstrap.py
    ./bin/buildout -vv
    ./bin/logintest -JSERVERNAME=${SERVERNAME} -JUSERNAME=${USERNAME} -JPASSWORD=${PASSWORD} -JNUMOFTHREADS=${NUMOFTHREADS} -JLOOPCOUNT=${LOOPCOUNT} -JRAMPUPPERIOD=${RAMPUPPERIOD}


- Replace ./bin/logintest, with each of the tests created in this buildout: logintest, editortest, reviewertest, eionetusertest and keep the rest of parameters. They will be filled with the user input on each build.

- Click on **Add post-build action"** and select **Publish Performance test result report**.
    - Under **Add a new report** select JMeter and enter the name of the report file as stated in the buildout file, for instance, **report_logintest.jtl**
    - Select **Error Threshold** mode and enter the threshold to mark the builds Unstable and Failed, you can enter 999 in both fields. You will be able to tune this setting later.
    - Enter the **relative thresholds for build comparison**, add **-0.01** and **0.01** as **Unstable % Range** and **-0.02** and **0.02** as **Failed % Range**. You will be able to tune this setting later.
    - Select **Average Reponse Time** in **Compare based on** dropdown.
    - Under **Performance display** check **Performance Per Test Case Mode** and **Show Troughput Chart** checkboxes

- Click on **Build with Parameters** to run the test, fill in the values of the parameters and wait for the results.

- When the test is completed, you have to go to the latest build, and click on the graph click on the graph icon on the left menu and then on **Response tieme trends for build** to se the response time graphs for each of the pages

.. _`Jenkins Performance plugin`: https://wiki.jenkins-ci.org/display/JENKINS/Performance+Plugin
