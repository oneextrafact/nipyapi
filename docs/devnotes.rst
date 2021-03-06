.. highlight:: shell

=================
Development Notes
=================

A collection point for information about the development process for future collaborators


Decision Points
---------------

* Using Swagger 2.0 instead of OpenAPI3.0 as it (currently as of Aug2017) has wider adoption and completed codegen tools
* We use Google style Docstrings to better enable Sphinx to produce nicely readable documentation


Docker Test Environment
-----------------------

There is an Apache NiFi image available on Dockerhub::

    docker pull apache/nifi:latest

There are a couple of configuration files for launching various Docker environment configurations in ./test_env_config for convenience.

Testing on OSX
--------------

There is a known issue with testing newer versions of Python on OSX.
You may receive an error reporting [SSL: CERTIFICATE_VERIFY_FAILED] when trying to install packages from Pypi

You can fix this by running the following commands::

    export PIP_REQUIRE_VIRTUALENV=false
    /Applications/Python\ 3.6/Install\ Certificates.command

Generate Swagger Client
-----------------------

The NiFi and NiFi Registry REST API clients are generated using swagger-codegen, which is available via a variety of methods:

- the package manager for your OS
- github: https://github.com/swagger-api/swagger-codegen
- maven: http://central.maven.org/maven2/io/swagger/swagger-codegen-cli/2.3.1/swagger-codegen-cli-2.3.1.jar
- pre-built Docker images on DockerHub (https://hub.docker.com/r/swaggerapi/swagger-codegen-cli/)

In the examples below, we'll use Homebrew for macOS::

    brew install swagger-codegen

NiFi Swagger Client
~~~~~~~~~~~~~~~~~~~

1. build relevant version of NiFi from source
2. use swagger-codegen to generate the Python client::

    mkdir -p ~/tmp && \
    echo '{ "packageName": "nifi" }' > ~/tmp/swagger-nifi-python-config.json && \
    rm -rf ~/tmp/nifi-python-client && \
    swagger-codegen generate \
        --lang python \
        --config swagger-nifi-python-config.json \
        --api-package apis \
        --model-package models \
        --template-dir /path/to/nipyapi/swagger_templates \
        --input-spec /path/to/nifi/nifi-nar-bundles/nifi-framework-bundle/nifi-framework/nifi-web/nifi-web-api/target/swagger-ui/swagger.json \
        --output ~/tmp/nifi-python-client

3. replace the embedded clients::

    rm -rf /path/to/nipyapi/nipyapi/nifi && cp -rf ~/tmp/nifi-python-client/nifi /path/to/nipyapi/nipyapi/nifi

4. review the changes and submit a PR!

NiFi Registry Swagger Client
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Fetch the definition from a running Registry instance at URI: /nifi-registry-api/swagger/swagger.json
2. use swagger-codegen to generate the Python client::


    mkdir -p ~/tmp && \
    echo '{ "packageName": "registry" }' > ~/tmp/swagger-registry-python-config.json && \
    rm -rf ~/tmp/nifi-registry-python-client && \
    swagger-codegen generate \
        --lang python \
        --config swagger-registry-python-config.json \
        --api-package apis \
        --model-package models \
        --template-dir /path/to/nipyapi/swagger_templates \
        --input-spec /path/to/nifi-registry/nifi-registry-web-api/target/swagger-ui/swagger.json \
        --output ~/tmp/nifi-registry-python-client

3. replace the embedded clients::

    rm -r /path/to/nipyapi/nipyapi/registry && cp -rf /tmp/nifi-registry-python-client/swagger_client /path/to/nipyapi/nipyapi/registry

4. review the changes and submit a PR!



Release Process
---------------

This assumes you have virtualenvwrapper, git, and appropriate python versions installed, as well as the necessary test environment:

- update History.rst
- check setup.py
- check requirements.txt and requirements_dev.txt
- Commit all changes
- in bash::

    cd ProjectDir
    source ./my_virtualenv/bin/activate
    bumpversion patch|minor|major
    python setup.py develop
    tox
    python setup.py test
    python setup.py build_sphinx
    # check docs in build/sphinx/html/index.html
    python setup.py sdist bdist_wheel
    mktmpenv  # or pyenv virtualenvwrapper mktmpenv if using pyenv
    pip install path/to/nipyapi-0.3.1-py2.py3-none-any.whl  # for example
    # Run appropriate tests, such as usage tests etc.
    deactivate
    # You may have to reactivate your original virtualenv
    twine upload dist/*
    # You may get a file exists error, check you're not trying to reupload an existing version
    git push --tags

- check build in TravisCI
- check docs on ReadTheDocs
- check release published on Github and PyPi
