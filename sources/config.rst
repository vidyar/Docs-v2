:title: Shippable Configuration
:description: Complete guide to how Shippable is setup
:keywords: shippable, signup, login, registration
	
.. _setup:

A bit deeper
============

Build Minions and Build configuration are two things that you should care the most about when using Shippable. The sections below talk about these in greater detail.


**Minions**
-----------

Minions are Docker based containers that run your builds and tests. You can think of them as Build VMs. Each minion runs one build at a time, so the more minions you have, the more concurrency you will get.  

Minions are automatically provisioned whenever a build is triggered and it will get deleted after the build finishes execution. We will automatically add additional minions if you are upgrading your subscription plan.

Each minion starts from a base image and can be customized by specifying ``before_install`` scripts in the YML file. A minion can be configured to run any package, library, or service that your application requires. There are some preinstalled tools and services that you can use to customize your minions even further. 

Operating Systems
.................

All our Linux minions start from a vanilla base image from the Docker registry. We support all images as a starting point for your minion. Minions can be further customized by using the ``before_install`` and ``install`` tags in ``shippable.yml`` that is in the root of your code repository.

(Coming soon) Our Windows minions are based on AWS AMI for Windows 2012.



Common Tools
............

A set of common tools are available on all minions. The following is a list of available tools -

- Latest release of Git repository
- apt installer
- Networking tools  
  
  - curl
  - wget
  - OpenSSL

- At least 1 version of (check out the language documentation for specific versions)
  
  - Ruby
  - Node
  - Python 
  - OpenJDK
  - PHP
  - Go
  - Clojure

- Services
  
  - MySQL
  - Postgre
  - SQLLite
  - MongoDB
  - Redis
  - ElasticSearch
  - Selenium Server
  - Neo4j
  - Cassandra
  - CouchDB
  - RethinkDB

- Headless browser testing tools

  - xfvb
  - PhantomJS

- Libraries

  - ImageMagik
  - OpenSSL

- JDK Versions

  - Oracle JDK 7u6 (oraclejdk7)
  - OpenJDK 7 (alias: openjdk7)
  - OpenJDK 6 (openjdk6)
  - Oracle JDK 8 EA (oraclejdk8)

- Build Tools

  - Maven 3
  - Gradle 1.9
  - Make
  - SBT 0.12.1

- Preinstalled PIP Packages

  - mock
  - nosetests
  - py.test

- Gems

  - Bundler
  - Rake
 
- Addons
  
  - Firefox
  - Custom hostname
  - PostgreSQL

----------

**Configuration**
------------------

This section is generic to all build environments and all languages. If you are looking for language specific tags, please refer to our language guides for more information.

``shippable.yml``
.................

We believe that developers need complete control of their build configuration. We also realize that most developers don't want to log into a UI to make changes every single time. 

While mulling over the best way to give developers control without asking them to go through our UI, we came across ``.travis.yml``, an open source initiative that created the basic framework for this very problem. Following the same paradigm, we ask you to have ``shippable.yml`` in the root of the repository you want to build. The structure of shippable.yml closely mimics travis since we see no reason to reinvent the wheel. We do have additional tags for added functionality, and these will become more numerous as we evolve Shippable. 

Since shippable.yml is a superset of ``.travis.yml`` , we support ``.travis.yml`` natively as well. So if you have a travis.yml in the root of your repo, we will read the config and set up your CI.

At a minimum, Shippable needs to have your language and build version specified in the yml. We will then default to the most common commands.

Build Flow
..........

When we receive a build trigger through a webhook or manual run, we execute the following steps - 

1. Clone/Pull the project from Github. This depends on whether the minion is in pristine state or not
2. ``cd`` into the workspace
3. Checkout the commit that is getting built
4. Run the ``before_install`` section. This is typically used to prep your minion and update any packages
5. Run ``install`` section to install any project specific libraries or packages
6. Run ``before_script`` section to create any folders and unzip files that might be needed for testing. Some users also restore DBs etc. here
7. Run the ``script`` command which runs build and all your tests
8. Run ``after_script`` command
9. Run either ``after_success`` or ``after_failure`` commands


Build status will be determined based on the outcome of the above steps. They need to return an exit code of ``0`` to be marked as success. Everything else is treated as a failure.


----------

**Other useful configs**
------------------------

Shippable uses Docker containers to provide you with isolation and a dedicated build environment. Our command sessions are not sticky throughout the build, but they are sticky within the same section of the build. For e.g. ``cd`` is sticky within the ``before_script`` tag of ``shippable.yml``

script
......
You can run any script file as part of your configuration, as long as it has a valid shebang command and the right ``chmod`` permissions. 

.. code-block:: python
        
        # script file 
        script: ./minions/do_something.sh 



command collections
...................
``shippable.yml`` supports collections under each tag. This is nothing more than YML functionality and we will run it one command at a time.

.. code-block:: python
        
  # collection scripts 
  script: 
   - ./minions/do_something.sh 
   - ./minions/do_something_else.sh 

In the example above, our minions will run ``./minions/do_something.sh`` and then run ``./minions/do_something-else.sh``. The only requirement is that all of these operations return a ``0`` exit code. Else the build will fail.


git submodules
..............
Shippable supports git submodules. This is a cool functionality of breaking your projects down into manageable chunks. We automatically initialize the ``.gitmodules`` file in the root of the repo. 

.. note::

  If you are using private repos, add the deploy keys so that our minion ssh keys are allowed to pull from the repo. This can be done via shippable.com

If its your own public repos then do this

.. code-block:: python
        
  # for public modules use
  git://github.com/someuser/somelibrary.git

  # for private modules use
  git@github.com:someuser/somelibrary.git

If you would like to turn submodules off completely -

.. code-block:: python
        
  # for public modules use
  git:
   submodules: false


  
common environment variables
.............................

You will have the following environment variables available to you for every build. You can use these in your scripts if required -

- BRANCH : Name of branch being built

- BUILD_NUMBER : Build number for current build

- BUILD_URL : Direct URL link to the Build Output

- COMMIT : Commit id that is being built and tested

- DEBIAN_FRONTEND : noninteractive

- JOB_ID : id of job in Shippable

- JRUBY_OPTS : --server -Dcext.enabled=false -Xcompile.invokedynamic=false

- LANG : en_US.UTF-8

- LC_ALL : en_US.UTF-8

- MERB_ENV : test

- PULL_REQUEST : Pull request id if the job is a pull request. If not, this will be set to 'None'

- RACK_ENV : test

- RAILS_ENV : test

- USER : shippable

- SHIPPABLE_ARTIFACTS_URL : URL to download artifacts

- ARTIFACTS_UPLOAD_SUCCESSFUL : Value of this variable will be true if archive is successful else this will be set as false.

- SHIPPABLE_API_TOKEN : Api access token for current build
 

user specified environment variables
.....................................

You can set your own environment variables in the yml. Every statement of this command will trigger a separate build with that specific version of the environment variables. 

.. code-block:: python
        
  # environment variable
  env:
   - FOO=foo BAR=bar
   - FOO=bar BAR=foo


.. note::

  Env variables can create an exponential number of builds when combined with ``jdk`` & ``rvm , node_js etc.`` i.e. it is multiplicative

In this setting **4 individual builds** are triggered in a build group

.. code-block:: python
        
  # npm builds
  node_js:
    - 0.10.24
    - 0.8.14
  env:
    - FOO=foo BAR=bar
    - FOO=bar BAR=foo

.. _secure_env_variables:

Secure environment variables
.............................

Shippable allows you to encrypt the environment variable definitions and keep your configurations private using **secure** tag. Go to the org account or individual dasboard page from where you have enabled your project and click on **ENCRYPT ENV VARIABLE** button on the top right corner of the page. Enter the env variable and its value in the text box as shown below. 

.. code-block:: python

    name=abc

Click on the encrypt button and copy the encrypted output string and add it to your yml file as shown below:


.. code-block:: python
   
   env:
     secure: <encrypted output>


To encrypt multiple environment variables and use them as part of a single build, enter the environment variable definitions in the text box as shown below 

.. code-block:: python

  name1="abc" name2="xyz"    

This will give you a single encrypted output that you can embed in your yml file.


You can also combine encrypted output and clear text environments using **global** tag. 

.. code-block:: python
 
   env:
     global:
       - FOO="bar"
       - secure: <encrypted output>


To encrypt multiple environment variables separately, configure your yml file as shown below: 

.. code-block:: python
  
  env:
    global:
      #encrypted output of first env variable
      - secure: <encrypted output> 
      #encrypted output of second env variable
      - secure: <encrypted output>
    matrix:
      #encrypted output of third env variable
      - secure: <encrypted output>


include & exclude branches
..........................

You can build specific branches or exclude them if needed. 

.. code-block:: python

  # exclude
  branches:
    except:
      - test1
      - experiment2

  # include
  branches:
    only:
      - stage
      - prod


build matrix
............

This is another powerful feature that Shippable has to offer. You can trigger multiple different test passes for a single code push. You might want to test against different versions of ruby, or different aspect ratios for your Selenium tests or best yet, just different jdk versions. You can do it all with Shippable's matrix build mechanism.

.. code-block:: python

  rvm:
    - 1.8.7 # (current default)
    - 1.9.2
    - 1.9.3
    - rbx
    - jruby
    - ruby-head
    - ree
  gemfile:
    - gemfiles/Gemfile.rails-2.3.x
    - gemfiles/Gemfile.rails-3.0.x
    - gemfiles/Gemfile.rails-3.1.x
    - gemfiles/Gemfile.rails-edge
  env:
    - ISOLATED=true
    - ISOLATED=false

The above example will fire 36 different builds for each push. Whoa! Need more minions?
 

**exclude**

It is also possible to exclude a specific version using exclude tag. Configure your yml file as shown below to exclude a specific version.

.. code-block:: python

   matrix:
     exclude:
       - rvm: 1.9.2
        


**include**

You can also configure your yml file to include entries into the matrix with include tag.

.. code-block:: python

   matrix:
     include:
       - rvm: 2.0.0
         gemfile: gemfiles/Gemfile.rails-3.0.x
         env: ISOLATED=false


**allow-failures**

Allowed failures are items in your build matrix that are allowed to fail without causing the entire build to be shown as failed. You can define allowed failures in the build matrix as follows:

.. code-block:: python

  matrix:
    allow_failures:
      - rvm: 1.9.3



----------

**Services**
-----------------
Shippable offers a host of pre-installed services to make it easy to run your builds. In addition to these you can install other services also by using the ``install`` tag of ``shippable.yml``. 

All the services are turned off by default and can be turned on by using the ``services:`` tag.

MongoDB
.......

.. code-block:: bash
  
  # Mongo binds to 127.0.0.1 by default
  services:
   - mongodb

Sample PHP code using `mongodb <https://github.com/Shippable/sample_php_mongo>`_ .


MySQL
.....

.. code-block:: bash
  
  # MySQL binds to 127.0.0.1 by default and is started on boot. Default username is shippable with no password
  # Create a DB as part of before script to use it

  before_script:
      - mysql -e 'create database myapp_test;'
                                 
Sample javascript code using `mysql <https://github.com/Shippable/sample_node_mysql>`_.


SQLite3
.......

SQLite is a software library that implements a self-contained, serverless, zero-configuration, transactional SQL database engine. So you can use SQLite, if you do not want to test your code behaviour with other databases.

Sample python code using `SQLite <https://github.com/Shippable/sample_python_sqllite>`_.


Elastic Search
..............

.. code-block:: bash

  # elastic search is on default port 9200
  services:
      - elasticsearch

Sample python code using `Elastic Search <https://github.com/Shippable/sample_python_elasticsearch>`_.

Memcached
..........

.. code-block:: bash

  # memcached runs on default port 11211
  services:
      - memcached

Sample python code using `Memcached <https://github.com/Shippable/sample_python_memcache>`_ .


Redis
.....

.. code-block:: bash

  # redis runs on default port 6379
  services:
      - redis


Sample python code using `Redis <https://github.com/Shippable/sample_python_redis>`_.


Neo4j
.....

.. code-block:: bash
 
 #neo4j runs on default port 7474
 services:
  - neo4j

Sample javascript code using `Neo4j <https://github.com/Shippable/sample_node_neo4j>`_ .

Cassandra
..........

.. code-block:: bash
 
 # cassandra binds to the default localhost 127.0.0.1 and is not started on boot. 
 services:
   - cassandra

Sample ruby code using `Cassandra <https://github.com/Shippable/sample_ruby_cassandra>`_ .

CouchDB
.........

.. code-block:: bash

 # couchdb binds to the default localhost 127.0.0.1 and runs on default port 5984. It is not started on boot.
 services:
   - couchdb

Sample ruby code using `CouchDB <https://github.com/Shippable/sample-ruby-couchdb/blob/master/shippable.yml>`_ .

RethinkDB
...........

.. code-block:: bash

 # rethinkdb binds to the default localhost 127.0.0.1 and is not started on boot.
 services:
   - rethinkdb

Sample javascript code using `RethinkDB <https://github.com/Shippable/sample-node-rethinkdb>`_.
 
RabbitMQ
.........

.. code-block:: bash

  # rabbitmq binds to 127.0.0.1 and is not started on boot. Default vhost "/", username "guest" and password "guest" can be used.
  services:
    - rabbitmq

Sample python code using `RabbitMQ <https://github.com/Shippable/sample_python_rabbitmq>`_ .


Selenium
.........

Selenium is not started on boot. You will have to enable it using **services** tag and start xvfb (X Virtual Framebuffer) on display port 99.0, so that all your test suites will run on the server without a display. Configure your yml file as shown below to start selenium on firefox.

.. code-block:: bash
   
     addons:
        firefox: "23.0"
     services:
       - selenium
     before_script:
       - "export DISPLAY=:99.0"
       - "/etc/init.d/xvfb start"
     after_script:
       - "/etc/init.d/xvfb stop"

     
Sample javascript code using `Selenium <https://github.com/Shippable/sample_node_selenium>`_ .


--------

**Addons**
----------

firefox
..........

We support different firefox versions like "18.0", "19.0", "20.0", "21.0", "22.0", "23.0", "24.0", "25.0", "26.0", "27.0", "28.0", "29.0". To select a specific firefox version, add the following to your shippable.yml file.

.. code-block:: python

	addons:
  	   firefox: "21.0"

custom host name
..................

You can also set up custom hostnames using the **hosts** addons. To set up the hostnames in /etc/hosts file, add the following to your shippable.yml file.
   
.. code-block:: python

        addons:
           hosts: 
    	    - google.com
            - asdf.com

PostgreSQL
...........

.. code-block:: bash

  # Postgre binds to 127.0.0.1 by default and is started on boot. Default username is "postgres" with no password
  # Create a DB as part of before script to use it

  before_script:
    - psql -c 'create database myapp_test;' -U postgres

Sample java code using `PostgreSQL <https://github.com/Shippable/sample_java_postgres>`_.

We support PostgreSQL 9.1, 9.2 and 9.3 versions and by default, version 9.2 is installed on our minions. Configure your yml file using **PostgreSQL** addons to select different versions. Add the following to your yml file to select the version 9.3.


.. code-block:: python

          addons:
           postgresql : "9.3"
  
PostGIS 2.1 packages are pre-installed in our minions along with the PostgreSQL versions 9.1, 9.2 and 9.3.


----------

**Test and Code Coverage visualization**
----------------------------------------
Test results
............
To set up test result visualization for a repository.

* Output test results to shippable/testresults folder. 
* Make sure test results are in junit format.

For example, here is the .yml file for a Python repo -

.. code-block:: bash

  before_script: mkdir -p shippable/testresults
  script:
    - nosetests python/sample.py --with-xunit --xunit-file=shippable/testresults/nosetests.xml

Examples for other languages can be found in our :ref:`Code Samples <samplesref>` .

Code coverage
.............
To set up code coverage result visualization for a repository.

* Output code coverage output to shippable/codecoverage folder. 
* Make sure code coverage output is in cobertura xml format.

For example, here is the .yml file for a Python repo -

.. code-block:: bash

  before_script: mkdir -p shippable/codecoverage
  script:
    - coverage run --branch python/sample.py
    - coverage xml -o shippable/codecoverage/coverage.xml python/sample.py

Examples for other languages can be found in our :ref:`Code Samples <samplesref>`.


-------------

**Notifications**
-----------------
Shippable can notify you about the status of your build. If you want to get notified about the build status (success, failure or unstable), you need to follow the rules below to configure your yml file. Shippable will send the consolidated build reports in individual emails for matrix build projects. By default Shippable will send the email notifications to the last committer.


Email notification
..................


You can configure the email notification by specifying the recipients id in ``shippable.yml`` file.

.. code-block:: bash

  notifications:
      email:
          - exampleone@org.com
          - exampletwo@org.com


You can also specify when you want to get notified using change|always|never. Change means you want to be notified only when the build status changes on the given branch. Always and never mean you want to be notified always or never respectively.


.. code-block:: bash
 
  notifications:
       email:
           recipients:
               - exampleone@org.com
               - exampletwo@org.com
           on_success: change
           on_failure: always


If you do not want to get notified, you can configure email notifications to false.

.. code-block:: bash

  notifications:
     email: false


----------

**Pull Request**
------------------


Shippable will integrate with github to show your pull request status on CI. Whenever a pull request is opened for your repo, we will run the build for the respective pull request and notify you about the status. You can decide whether to merge the request or not, based on the status shown. If you accept the pull request, Shippable will run one more build for the merged repo and will send email notifications for the merged repo.
To rerun a pull request build, go to your project's page -> Pull Request tab and then click on the **Build this Pull Request** button.
 
--------

**Permissions**
------------------

We will automatically add your collaborators when you login to shippable and it will be updated in the user-interface. Go to the org dashboard and click on the **Permissions** button on the right side of the page to view your collaborators. 


There are two types of roles that users can have for an organisation-

**Owner :** 
Owner is the highest role. This role permits users to create, run and delete a project. 


**Collaborator :** 
Collaborator can run or manage projects that are already setup. They have full visibility into the project and can trigger the build.



--------

**Build Termination**
-----------------------

We will teminate the build forcefully in the following scenarios:

* Script or test suite hangs for a long time 
* There hasn't been any log output in 20 minutes  
* It is still executing after 20 minutes 

and the status of the build will be updated as **timeout** .
 


 
--------

**Skipping a build**
-----------------------

Any changes to your source code will trigger a build automatically on Shippable. So if you do not want to run build for a particular commit, then add **[ci skip]** or **[skip ci]** to your commit message. 

Our webhook processor will look for the string  **[ci skip]** or **[skip ci]** in the commit message and if it exists, then that particular webhook build will not be executed and the status of that build will be marked as **skipped** on UI .

--------

**Using Shippable with Gitlab or other types of source control**
----------------------------------------------------------------

At the moment, Shippable supports repositories hosted either on GitHub or Bitbucket.
However, your development setup may involve using a different provider or even hosting the repository server on your own.
In both cases, the easiest way to make your code available to Shippable is to set up a mirror of your repository with either of the supported services. 

As `GitLab <https://about.gitlab.com/>`_ is a very popular choice among organizations managing their own repositories,
the instructions below outline how to set up a mirror of a repository hosted on GitLab Community Edition 7.2.1.
Other self-hosted solutions can be integrated in a very similar manner, the only differences being the locations of the files.
If you experience any problems setting up a mirror using a different technology, please do not hesitate to reach out to us.
Please also note that this method can be applied to mirror repositories using different VCS than Git or Mercurial, if only an extension to push the changes to Git is available.
Hence, it allows using VCS of your choice (such as Perforce or SVN) with Shippable.

First step of setting up a mirror is to create a target repository on either GitHub or Bitbucket.
Please note that in the case of Bitbucket, you can create unlimited number of private repositories for free, granted that no more than 5 users will have access to them.
It makes it especially appealing to host mirrors then, as you only need to associate two users (the account you use with Shippable and an extra account for your repository server) with all the mirrors.

Please write down the Git url of your newly created mirror. For the sake of convenience, we will use the following urls throughout this guide:

* GitHub: ``git@github.com:Shippable/shippable-mirror-test.git``
* Bitbucket: ``git@bitbucket.org:Shippable/shippable-mirror-test.git``

Next, you need to grant write access for your repository server to the mirror.

Granting access to a GitHub mirror
..................................

In case of GitHub, it can be done by `adding a deployment key <https://developer.github.com/guides/managing-deploy-keys/#deploy-keys>`_ to the repository. 
GitHub requires the deployment keys to be unique, so we need to create a dedicated SSH key for every repository.

Switch to the ``git`` user on your GitLab server and create the key, using a filename that clearly associates the key with the repository.
Leave the passphrase empty:

.. code-block:: bash

  # su - git
  $ bash
  $ pwd
  /var/opt/gitlab
  $ ssh-keygen -f .ssh/shippable-mirror_key

Next, take the contents of the ``.ssh/shippable-mirror_key.pub`` file and add it as a deploy key in the GitHub settings panel for your mirror repository.
To ensure that the right key gets picked when ``git`` establishes the connection with the mirror, we will add a special host entry in the SSH config.
Open ``/var/opt/gitlab/.ssh/config`` file (create it if it doesn't exist) with your favorite editor and add the following section:

.. code-block:: bash

  Host shippable-mirror
  IdentityFile /var/opt/gitlab/.ssh/shippable-mirror_key
  HostName github.com  
  User git

Now, connecting to ``shippable-mirror`` host (replace this alias with your repository name) will result in establishing a connection with ``github.com`` as user ``git``,
but using our dedicated deployment key. Test it by issuing the following command (still as ``git`` user on your GitLab server):

.. code-block:: bash

  $ ssh shippable-mirror
  Warning: Permanently added the RSA host key for IP address '192.30.252.129' to the list of known hosts.
  PTY allocation request failed on channel 0
  Hi Shippable/shippable-mirror-test! You've successfully authenticated, but GitHub does not provide shell access.
  Connection to github.com closed.

If you see a message like this above, you have successfully set the deployment key up.

Granting access to a Bitbucket mirror
.....................................

In case of Bitbucket, you need to create an SSH key and associate it with a user account.
As it is not advisable to deploy to a remote server the key that grants access to your private account,
we recommended creating a separate user on Bitbucket.org for authenticating your GitLab server.

You can create one key for ``git`` user on your GitLab server and use it for all the services, but for security reasons you may create
a separate key for Bitbucket.  Switch to the ``git`` user on your GitLab server and create the key, leaving the passphrase empty:

.. code-block:: bash

  # su - git
  $ bash
  $ pwd
  /var/opt/gitlab
  $ ssh-keygen -f .ssh/bitbucket_key

Next, take the contents of the ``.ssh/bitbucket_key.pub`` file and 
`add the key for it in the account management panel <https://confluence.atlassian.com/display/BITBUCKET/Add+an+SSH+key+to+an+account>`_.
To ensure that the right key gets picked when ``git`` establishes the connection with the mirror, we will add a special host entry in the SSH config.
Open ``/var/opt/gitlab/.ssh/config`` file (create it if it doesn't exist) with your favorite editor and add the following section:

.. code-block:: bash

  Host bitbucket
  IdentityFile /var/opt/gitlab/.ssh/bitbucket_key
  HostName bitbucket.org
  User git

Now, connecting to ``bitbucket`` host will result in establishing a connection with ``bitbucket.org`` as user ``git``,
but using our dedicated deployment key. Test it by issuing the following command (still as ``git`` user on your GitLab server):

.. code-block:: bash

  $ ssh bitbucket
  Warning: Permanently added the RSA host key for IP address '131.103.20.168' to the list of known hosts.
  PTY allocation request failed on channel 0
  logged in as Shippable.

  You can use git or hg to connect to Bitbucket. Shell access is disabled.
  Connection to bitbucket.org closed.

If you see a message like this above, you have successfully set the deployment key up.

Setting a git hook
..................

The next step is to add mirror as a remote to the repository on your GitLab server.
Still as ``git`` user, go to the directory that contains the repository and issue the following command:

.. code-block:: bash

  $ cd /var/opt/gitlab/git-data/repositories/<GitLab namespace for the repo>/<repo name>.git
  # for GitHub
  $ git remote add mirror --mirror=push shippable-mirror:Shippable/shippable-mirror-test.git
  # for Bitbucket 
  $ git remote add mirror --mirror=push bitbucket:Shippable/shippable-mirror-test.git        

Next, add the ``post-receive`` hook in the same directory (you can read more about ``git`` hooks `here <http://git-scm.com/docs/githooks.html>`_).

.. code-block:: bash

  $ echo "exec git push --quiet mirror &" >> hooks/post-receive
  $ chmod 755 hooks/post-receive

You are now all set! After you push new changes to the GitLab, they whole repository will be automatically mirrored to either GitHub or Bitbucket.
If any errors occur, they should be visible in the output of your local ``git push`` command.

---------

**Docker hub**
---------------

Shippable allows you to push the containers to docker registry after a successfull build. To avail this option, you will have to enable the Docker hub from shippable account first. Follow the steps below to enable and push the container to docker registry.

1. Go to individual dashboard's page.
2. Click on the **Docker Hub** button and then enter the docker hub credentials.
3. Configure your yml file as shown below to push the container.


.. code-block:: bash

    commit_container: username/sample_project

-------

**Build Badge**
-------------------

You can find the build badges on the project's page. Click on the **Badge** button and copy the markdown to your README file to display the status of most recent build on your Github or Bitbucket repo page.
