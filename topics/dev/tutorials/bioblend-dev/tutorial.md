Contributing to BioBlend as a developer

# 1  Introduction 

BioBlend [3] is a Python library for interacting with Galaxy [1].
Galaxy is a server for accessible, reproducible and transparent computational research. It includes a web interface where users can design and perform tasks in a visual and interactive manner. The server also exposes this functionality through its REST-based Application Programming Interface (API).
Computer programs can communicate with the Galaxy server through this API and perform similar tasks as can be achieved manually via the web interface. The program sends a network request to a URL of an API endpoint. The server then computes the result for this request and sends back a response to the program.
BioBlend provides classes and methods that handle the specific details of this communication for Python programs.
Similar libraries exists for interacting with Galaxy via other programming languages:
-  blend4j: https://github.com/galaxyproject/blend4j
-  blend4php: https://github.com/galaxyproject/blend4php
-  clj-blend: https://github.com/chapmanb/clj-blend


# 2  Development on GitHub 

Galaxy and BioBlend are developed as open source projects on GitHub and contributions are welcome!

## 2.1  GitHub repositories
 - Galaxy: https://github.com/galaxyproject/galaxy
 - BioBlend: https://github.com/galaxyproject/bioblend

## 2.2  Contributing on GitHub 

To contribute to BioBlend, a GitHub account is required.
Changes are proposed via a pull request (PR). This allows the project maintainers to review the changes and suggest improvements.
The general steps are as follows:
 -  Fork the BioBlend repository
 -  Make changes in a new branch
 -  Open a pull request for this branch in the upstream BioBlend repository
Note: It is generally a good idea to enable “Allow edits and access to secrets by maintainers”. Enabling this option gives maintainers more freedom to help with the PR.

# 3  Downloading Galaxy and BioBlend 

Now we are ready to set up our development environment! Since BioBlend communicates with the Galaxy API, we must also set up a Galaxy server in order to test BioBlend functionality.
We use the ‘git' versioning tool to download the repositories (https://git-scm.com).
For this we require a public SSH key associated with our GitHub account. It makes sense to set this up now, since pushing changes to GitHub without a public key will promt for credentials every time. See the GitHub Docs for information on setting up an SSH key.
To download Galaxy and set it up as a local repository, run:
git clone git@github.com:galaxyproject/galaxy.git
Likewise, to download BioBlend, run:
git clone git@github.com:galaxyproject/bioblend.git

# 4  Structure of the Galaxy API 

The source code for the API endpoints is contained in various files under ‘lib/galaxy/webapps/galaxy/api/’. Each of these files contains a controller which exposes functionality for a specific entity. For example, the dataset-related functionality is contained in the ‘DatasetsController’ class in the ‘datasets.py' file.
Additionally, the ‘builapp.py' file contains a complete listing of all the endpoints.
Note: Different versions of Galaxy differ in the functionality that is implemented. The development version is named 'dev' and includes the most recent changes. Release versions are named according to their release date. These Galaxy releases can be accessed via git branches. At the time of writing BioBlend supports Galaxy releases 17.09 and later, although some functionality is not supported by all of these versions.

## 4.1  Core concepts of the Galaxy interface 

This section provides succinct descriptions of the core concepts of the Galaxy interface, from the perspective of a BioBlend/Galaxy developer. This list is by no means complete, but it should help those not already familiar with Galaxy.
Note: BioBlend provides seperate clients for interacting which each of these concepts. For example, the BioBlend client for jobs is the ‘JobsClient’ class.
Job. Jobs are associated with the execution of various tasks on the Galaxy server. For example, creating a dataset amounts to a computation on the server and therefore has an associated job. The computation of each step in a workflow has an associated job as well. 
Jobs run as background processes on the Galaxy server. This is relevant because reading results before the appropriate jobs have finished can lead to missing data in the output. BioBlend methods must handle this by waiting for relevant jobs to finish before returning.
Workflow. A workflow represent a computation pipeline. It is created and contained in a history. A workflow is comprised of steps, which can be either inputs or tools. Its structure can be imagined as a directed graph of nodes that process inputs and pass on their outputs to the next nodes. The initial input to a workflow is commonly one or multiple datasets or dataset collections.
Invocation. An invocation represents the execution of a workflow on specified inputs and with certain specified parameters. Every invocation corresponds to only one workflow, and every step of the workflow has an associated invocation step.
Tool. A tool represent an actual algorithm that implements the functionality in a workflow step. It keeps track of its required dependencies. It also contains additional metadata such as the BibTex references to the associated research paper(s).
History. A history represents a container that keeps track of actions performed on its contents. Workflows and datasets can be created in a history. For example, when a workflow is invoked multiple times on different datasets, the history keeps track of the outputs corresponding to each of these invocations.
Dataset. A dataset represents digital data that can serve as input for a Job. It can be associated with (contained in) a history or a library. A dataset can be shared between users. Each time a dataset is copied between histories or libraries, a new History Dataset Association (HDA) or Library Dataset Association (LDA) is created. This naming scheme is used internally by the Galaxy relational database, but the terms might be encountered during BioBlend development as well.
Dataset Collection. A dataset collection represents a collection of related datasets. Like a dataset, it can be associated with a history or a library. Each time it is copied or shared, a new History Dataset Collection Association (HDCA) or a Library Dataset Collection Association (LDCA) is created.
Library. A library represent a container for datasets and dataset collections. Datasets that are contained in a library can be shared between multiple Galaxy users.


# 5  Structure of the BioBlend library 

BioBlend methods for the Galaxy API are stored under ‘bioblend/galaxy/’. The functionality for each Galaxy entity is in a seperate folder. For example, the methods for interacting with workflows are stored under ‘bioblend/galaxy/workflows’ in the ‘WorkflowsClient’ class.
The tests for these methods are stored under ‘tests/‘. For example, the tests related to workflows are in the file ‘tests/TestGalaxyWorkflows.py’.
The classic approach for accessing the Galaxy API is using the various clients and their methods. Each of these methods corresponds to one of the API endpoints. They send a request to the Galaxy server and return a Python dictionary representing the parsed JSON response. Alternatively, there is also BioBlend.objects [2], which provides an object-oriented API. Interaction with the API occurs via wrapper objects. These wrappers represent entity instances and provide methods to interact and manipulate them. For example, a wrapper could represent a single workflow. See section 9.1 for additional information.
Note: BioBlend also provides methods for interacting with the Galaxy ToolShed and Cloudman. However, this tutorial focuses on the part of BioBlend which provides access to the Galaxy API.


# 6  Hands On: Communicating with the Galaxy API 

We will make some manual requests to get a better feel for the Galaxy API.
We need a valid API key in order to communicate with the Galaxy server through the API.
A quick way to get an API key is to create a new user on our local Galaxy server. Navigate to the Galaxy base directory and execute the ‘run.sh' script. This starts the Galaxy server. Next, open ‘http://localhost:8080/' in a web browser. This should open the web interface of our Galaxy server. Create a new user by clicking ‘Login or Register' in the top menu. We can generate an API key for our user by navigating to ‘User > Preferences > Manage API Key' and clicking the ‘Create a new Key' button.
In the following examples we make use of a simple networking tool named ‘curl' (https://curl.se). Each command is listed together with the corresponding BioBlend and BioBlend.objects code.
We assume basic familiarity with HTTP request methods. For an overview, see the documentation at ‘https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods'.
Now, open a terminal window and let's make a few manual requests!


## 6.1  Making a GET request 

This is a simplest type of request. We request data from a specific URL, but we do not provide any further information. In this example we request a listing of all our invocations.
curl -H 'x-api-key: <API_KEY>' -X GET 'localhost:8080/api/invocations'
The ‘-H' flag is used to specify HTTP headers. We authenticate our requests by specifying our API key in the ‘x-api-key' header.
The ‘-X' flag specifies the HTTP method of our request. In this case we are making a GET request. Since GET is the default method, we could technically omit it here.
[{
    'history_id': '2f94e8ae9edff68a',
    'id': 'df7a1f0c02a5b08e',
    'model_class': 'WorkflowInvocation',
    'state': 'new',
    'update_time': '2015-10-31T22:00:22',
    'uuid': 'c8aa2b1c-801a-11e5-a9e5-8ca98228593c',
    'workflow_id': '03501d7626bd192f',
}]Example JSON response
from bioblend.galaxy import GalaxyInstance
gi = GalaxyInstance('http://localhost:8080', key=<API_KEY>)
invs = gi.invocations.get_invocations()
print(invs)BioBlend
from bioblend.galaxy.objects import GalaxyInstance
obj_gi = GalaxyInstance('http://localhost:8080', key=<API_KEY>)
previews = obj_gi.invocations.get_previews()
for preview in previews:
    print(preview.id)BioBlend.objects
The above response would mean that our user only once invoked a workflow. The workflow's encoded ID is listed, as well as the encoded ID of the history containing this workflow. The state ‘new’ indicates that the request to invoke this workflow is still pending and that the invocation has not been scheduled yet.


## 6.2  Making a GET request with query parameters 

In this example we include two query parameters in our request, ‘limit’ and ‘include_terminal’. With ‘limit=5’ we limit the result to at most 5 invocations. With ‘include_terminal=False’ we indicate that invocations in a terminal state should be excluded from the results.
curl -H 'x-api-key: <API_KEY>' \
    -X GET 'localhost:8080/api/invocations?limit=5&include_terminal=False'
invs = gi.invocations.get_invocations(limit=5, include_terminal=False)
print(invs)BioBlend
invs = obj_gi.invocations.list(limit=5, include_terminal=False)
for invocation in invs:
    print(invocation.id)BioBlend.objects


## 6.3  Making a POST request with a payload 

curl -X POST \
    -H 'x-api-key: <API_KEY>' \
    -H 'Content-Type: application-json' \
    -d '{"name":"MyNewHistory"}' \
    'localhost:8080/api/histories'
This creates a new history with the name ‘MyNewHistory’.
history = gi.histories.create_history('MyNewHistory')
print(history)BioBlend
history = obj_gi.histories.create('MyNewHistory')
print(history.name)BioBlend.objects
Note: For additional information regarding the difference between query parameters and the payload, see section 9.6.


# 7  Hands On: Creating a BioBlend pull request 

Note: This example is based on the pull request named “Improving Tools API coverage” in the BioBlend repository (https://github.com/galaxyproject/bioblend/pull/390).
We will extend the BioBlend ‘ToolsClient’ class by adding a new method named ‘uninstall_dependencies'. This method will send a request to the Galaxy server to uninstall any dependencies for the tool specified by the user.


## 7.1  Identifying the relevant Galaxy endpoint 

We should check the Galaxy controller to verify whether this functionality is already implemented in the Galaxy API. Let's check the ‘dev' version of Galaxy. Our method interacts with the tools API, so let's check the ‘api/tools.py' file. It turns out that a corresponding ‘uninstall_dependencies' endpoint method is already available on Galaxy ‘dev'. Lucky us! If this was not the case, then we would first have to implement the endpoint in Galaxy before a BioBlend method could interact with it.

## 7.2  Constructing the correct method signature 

Let's look at the signature of the endpoint method.
def uninstall_dependencies(self, trans: GalaxyWebTransaction, id, **kwds):File api/tools.py, line 291
The ‘GalaxyWebTransaction' is an object which represents our API request. It is internal to Galaxy. We do not specify it as a parameter in BioBlend.
We can see that it also expects an ‘id' parameter. The ‘kwds' argument indicates that there might be aditional keyword arguments as well. 
Let's check the method documentation.
"""
DELETE /api/tools/{tool_id}/dependencies
Attempts to uninstall requirements via the dependency resolver

parameters:

    index:
        index of dependency resolver to use when installing dependency.
        Defaults to using the highest ranking resolver

    resolver_type: 
        Use the dependency resolver of this resolver_type to install                     dependency
"""
The endpoint expects a DELETE request at URL ‘/api/tools/{tool_id}/dependencies'. The ‘id’ parameter in the method signature corresponds to the variable ‘{tool_id}’ present in the endpoint URL ‘/api/tools/{tool_id}/dependencies’.
There are two additional parameters: ‘index' and ‘resolver_type'. These parameters specify how the server should resolve the tool dependencies. For an average API user these options are probably too specific as they require knowledge of the resolvers used by the Galaxy server backend. We will skip these parameters for this pull request.
Lastly, we need to find out what the endpoint returns. Unfortunately the return type is not listed in the method documentation. It is a reality that various parts of the Galaxy API lack complete documentation, although this is being worked on.
Let's look at the return statement of the endpoint.
return tool.tool_requirements_status
Here the tool API controller calls the ‘tool_requirements_status' method of the ‘Tool’ class instance that represents the tool corresponding to the ‘id’ parameter. Let's check the definition of this method.
def tool_requirements_status(self):
    """
    Return a list of dictionaries for all tool dependencies with their 
    associated status
    """
    return self._view.get_requirements_status(
        {self.id: self.tool_requirements},
        self.installed_tool_dependencies
    )File tools/__init__.py, line 1833
The documentation of this method mentions that it returns a list of dictionaries for all tool dependencies with their associated status. Now we know the return type of the endpoint method!
Our Bioblend method will receive this list of dictionaries encoded as JSON in the server response. It will parse the JSON and return the list. Thus, the signature of our new ‘uninstall_dependencies' method will be as follows:
def install_dependencies(self, tool_id: str) -> List[dict]:
The ‘self’ parameters corresponds to the ‘ToolsClient’ instance. The parameter ‘tool_id’ corresponds to the encoded ID of a given tool. This ID will be substituted in place of ‘{tool_id}’ in the request URL.
Note: In Galaxy and BioBlend, encoded IDs are hexadecimal strings. See section 9.4 for additional information.

## 7.3  Adding the method body 

Our method is part of the ‘ToolsClient' class. This means that our method has access to all its helper methods.
Let's recall that the endpoint expects a DELETE request at URL ‘/api/tools/{tool_id}/dependencies'.
To translate this into BioBlend source code, we do the following:
-  We use the ‘self._make_url' helper method to construct a valid URL with the supplied 'tool_id' parameter. Finally, we need to append ‘/dependencies' for our endpoint.
url = self._make_url(tool_id) + '/dependencies'
-  BioBlend clients include helper methods for making request of various types. In this case we want to make a DELETE request, so we use the ‘self._delete' helper method with our constructed URL as argument.  It also expects a ‘payload' argument. Our payload will be empty, since we use ‘tool_id' to construct the URL. This helper method automatically parses the JSON reponse into a python object, so we can directly return its result.
return self._delete(payload={}, url=url)

## 7.4  Adding the docstring 

We should document the method and its parameter for the user.
"""
Uninstall dependencies for a given tool via a resolver.

:type tool_id: str
:param tool_id: id of the requested tool

:rtype: list of dicts
:return: Tool requirement statuses
"""
This is the docstring format used in BioBlend. At the top we provide a short general description of the method. Then we document each parameter with its type and a short description. The return value of the method is documented last.
Checking Galaxy version compatibility
In case certain versions of Galaxy do not support our method's functionality, we should append a note to the method's docstring.
.. note::
    This method is only supported by Galaxy 19.09 or later.
Previously we made sure that Galaxy ‘dev' supported our method. Let's now check if older versions of Galaxy also support it. At the time of writing, Galaxy 17.09 is the earliest version supported by BioBlend, so it makes sense to check it first. 
It turns out the Galaxy 17.09 also supports our method! This means that all later versions do as well, and thus we do not need to add a note.


## 7.5  Putting it all together 

def install_dependencies(self, tool_id: str) -> List[dict]:
    """
    Install dependencies for a given tool via a resolver.

    :type tool_id: str
    :param tool_id: id of the requested tool

    :rtype: list of dicts
    :return: Tool requirement statuses
    """
    url = self._make_url(tool_id) + '/install_dependencies'
    return self._delete(payload={}, url=url)

## 7.6  Writing a test for our new method 

It is best to keep BioBlend tests simple. We need to check that we get an expected response, but testing minute details is unnecessary, since those are the responsiblity of the Galaxy server itself.
Therefore, let's check that, when run on a given tool ID, the returned status indicates that the dependency is ‘null'. Let's reuse the tool ID ‘CONVERTER_fasta_to_bowtie_color_index' from the ‘test_tool_requirements' test method.
def test_tool_dependency_uninstall(self):
    statuses = self.gi.tools.uninstall_dependencies(
         'CONVERTER_fasta_to_bowtie_color_index'
    )
    self.assertEqual(statuses[0]['model_class'], 'NullDependency')
Note: It might sometimes be necessary to skip a test for Galaxy versions that do not support the tested functionality. See section 9.2 for additional information.


# 8  Running the BioBlend tests 

Finally, we should run our test to make sure that it passes. This would indicate that our method works correctly. After testing we will be ready to push our changes to GitHub and open a pull request!
This section outlines two approaches for running BioBlend tests: A simple approach that should suffice for basic testing, and a more involved approach that speeds up testing.
Note: On our local machine we only test against Galaxy 'dev', so it might happen that tests on GitHub still fail because they test against all supported Galaxy versions. When this happens we simply need to update our changes to fix those errors as well.


## 8.1  Approach 1: Using the ‘run_bioblend_tests.sh’ script 

This script is provided in the repository and can be used to run the BioBlend tests. It works well for small tests that require little to no debugging or experimentation.
Since BioBlend requires a Galaxy server, we must specify the path to the Galaxy server directory.
-  -g <galaxy_path>
Two useful optional parameters:
-  -e <python_version>
-  -t <path_to_test>
We can specify a subset of tests to run by supplying this argument. This is specified in 'pytest' format. See the documentation for more information.
Examples:
Let's assume that our galaxy directory is ‘../galaxy_dev’.
Then this command runs all BioBlend tests:
./run_bioblend_tests -g ../galaxy_dev -e py39
And this command runs only the test named ‘test_get_jobs‘:
./run_bioblend_tests \
-g ../galaxy_dev \
-e py39 \
-t tests/TestGalaxyJobs.py::TestGalaxyJobs::test_get_jobs
Downside of this approach:
The script starts and stops a new instance of the Galaxy server every time it is run. This makes it robust. However, it also means we have to wait for the server to start up every time. This makes it painful to debug more complicated tests.


## 8.2  Approach 2: Running the tests directly 

We can speed up our test environment by controlling the Galaxy server and the BioBlend tests directly. The initial setup will be more involved, but this way we can keep the server running in the background while we do our testing.

### 8.2.1  Caveat 

When the Galaxy server is not reset between test runs, tests that implicitly depend on a clean server state might start failing after the first run. 
For example, let's assume there is only a single test that creates a new history and checks that the total number of histories is equal to 1. This test will pass for a new Galaxy server because no histories exist yet. However, the next run there would be two histories and the test would fail.
Although this is generally not a big issue, it is something to watch out for. There are a few BioBlend tests that will fail in this manner. Most of these tests measure an array length property that increases with every run and therefore fail after the first run.
Note: It is preferable to write test that are robust in this regard!


### 8.2.2  Starting the Galaxy server 

Open a terminal in the Galaxy base directory and execute the following commands:
Note: We could of course collect the commands in a script, but for the sake of this tutorial they are presented one by one.
Step 1: Declare an API key of our choosing to use for testing. We will include this key in the Galaxy server configuration. The server will then accept incoming requests that use this key.
API_KEY=LetMeIn
Step 2: Create a temporary directory
TEMP_DIR=$(mktemp -d)
echo "Galaxy directory: $TEMP_DIR"
Step 3: Export environment variables required by the Galaxy server
export SKIP_GALAXY_CLIENT_BUILD=1
export GALAXY_LOG_FILE=$TEMP_DIR/main.log
export GALAXY_CONFIG_FILE=$TEMP_DIR/test_galaxy.ini
Step 4: Copy a sample tool configuration
cp config/tool_conf.xml.sample $TEMP_DIR/tool_conf.xml.sample
Step 5: Declare the desired logging level
LOG_LEVEL=DEBUG
Note: The available logging levels can be found in the documentation of the Python ‘logging' library.
Step 5: Create the Galaxy configuration file
cat << EOF > $GALAXY_CONFIG_FILE
[server:main]

use = egg:Paste#http
port = 8080

[app:main]

log_level = $LOG_LEVEL
paste.app_factory = galaxy.web.buildapp:app_factory
database_connection = sqlite:///$TEMP_DIR/universe.sqlite?isolation_level=IMMEDIATE
file_path = $TEMP_DIR/files
new_file_path = $TEMP_DIR/tmp
tool_config_file = $TEMP_DIR/tool_conf.xml.sample
conda_auto_init = True
job_working_directory = $TEMP_DIR/jobs_directory
allow_library_path_paste = True
admin_users = test@test.test
allow_user_deletion = True
allow_user_dataset_purge = True
enable_beta_workflow_modules = True
master_api_key = $API_KEY
enable_quotas = True
cleanup_job = onsuccess
EOF
Step 6: Start the Galaxy server
./run.sh
8.2.3  Running tests 

Now we have a Galaxy server running with our specified configuration. We can run tests against this server in a separate terminal.
Navigate to the BioBlend base directory and execute the following commands:
Step 1: Declare the same API key we used for the Galaxy server
API_KEY=LetMeIn
Step 2: Export environment variables required by BioBlend
export BIOBLEND_GALAXY_API_KEY=$API_KEY
export BIOBLEND_GALAXY_URL=http://localhost:8080/
Step 3: Active the virtual environment
source <path_to_galaxy_directory>/.venv/bin/activate
Step 4: Declare the desired test logging level
LOG_LEVEL=WARNING
Step 5: Run the desired BioBlend test(s)
# selection format: tests/<file>::<module>::<test>
pytest --override-ini log_cli_level=$LOG_LEVEL \
    tests/TestGalaxyWorkflows.py::TestGalaxyWorkflows::test_show_versions
Step 6: We can leave the virtual environment when we are done testing
deactivate
Note: The temporary directory that we created will contain useful information for debugging tests, such as the ‘main.log' file and the ‘universe.sqlite' database.


# 9  Tips 

This sections contains additional tips on concepts that might be encountered in the course of BioBlend development.

## 9.1  Difference between GalaxyInstance and objects.GalaxyInstance 

The ‘GalaxyInstance’ class represents an instance of a galaxy user session. It includes as member variables the various Galaxy API client modules.
In the BioBlend tests, ‘self.gi’ corresponds to the ‘GalaxyInstance'.
The exception to this rule is ‘TestGalaxyObjects.py’. This file contains tests for the various API wrapper classes of the BioBlend object-oriented API. These wrappers have their own version of the ‘GalaxyInstance’, namely the ‘objects.GalaxyInstance’ class, which includes additional functionality specific to the object-oriented API. In the ‘TestGalaxyObjects.py’ file, ‘self.gi’ points to the ‘objects.GalaxyInstance'. The normal ‘GalaxyInstance’ is accessed as ‘self.gi.gi’.


## 9.2  Use of the ‘@test_util.skip_unless_galaxy’ decorator 

On our local machine we mostly test our changes against a single version of Galaxy, for example 'dev'. However, the GitHub Actions Continuous Integration workflow runs the BioBlend tests against all supported versions of Galaxy.
At the time of writing BioBlend supports Galaxy 17.09 and later. However, certain functionality might not be available in all these versions. For example, ‘download_dataset_collection' is only supported by Galaxy 18.01 and later. We don't want to run tests specific to this functionality on earlier versions of Galaxy. They would simply fail. We can skip a test for specific versions of Galaxy by adding the ‘@test_util.skip_unless_galaxy’ decorator above the test method in question.
Example:
@test_util.skip_unless_galaxy('release_19.09')
def test_some_functionality(self):
This test will only be run against Galaxy 19.09 and later.

## 9.3  Controllers versus managers in Galaxy 

The various API methods that are exposed to the outside are contained in controller classes. These contain the endpoint methods corresponding to the URLs of the Galaxy API. These endpoint methods handle the incoming requests from BioBlend.
A focus for development of the Galaxy API is to make these controllers as “terse” as possible. This basically means that any logic not strictly required by the API endpoint method is moved to a corresponding manager class. This appreach seperates the API more cleanly from internal functionality.
At the time of writing this transition is ongoing, therefore source code of methods for one controller might look and function differently than those of another controller. This might be encountered when debugging a failing request in BioBlend.

## 9.4  IDs versus encoded IDs 

The Galaxy server assigns a number ID to every entity, which serves as its unique identifier. For example, each workflow, tool, history and dataset is assigned a unique ID.
The Galaxy server encodes interal IDs into hexademimal strings before sending them to external API clients. These are referred to as encoded IDs. BioBlend only deals with these encoded IDs.

## 9.5  Galaxy API parameter formats 

Implementing a BioBlend method often requires knowledge about the parameters which are accepted by the corresponding Galaxy API endpoint. At the time of writing there are a few different formats in which endpoints might accept their input parameters.

### 9.5.1  Direct parameters 

A parameter can be specified explicitly in the signature of the endpoint method.
Example:
In the history API 'delete' endpoint method, the 'id' parameter is specified explicitly.
def delete(self, trans, id, **kwd):

### 9.5.2  Keywords parameters 

Other parameters might get passed to the endpoint as an element of ‘kwd'. If the documentation of the endpoint is lacking, these keyword parameters can be painful to identify.

### 9.5.3  The 'q' and 'qv' format 

Filtering parameters might also be expected as pairs of ‘q' and ‘qv' values. These are then parsed into filters by the Galaxy server. 
The general format is ‘q={filter}-{operation}' and ‘qv={value}’. The accepted values for these parameters must be identified in order to construct valid ‘q' and ‘qv' pairs.
Example:
The Datasets API 'index' method uses this format. The ‘q' and ‘qv' parameters are passed to the endpoint as part of 'kwd'. They are not used in the endpoint itself, but rather implicitely passed into the ‘self.parse_filter_params' helper method as part of ‘kwd'.


## 9.6  Params vs payload 

In BioBlend, the ‘Client._get’ method is used for GET requests. It takes a ‘params’ argument. The ‘Client._post’ and ‘Client._put’ helper methods make POST and PUT request respectively. They expect a ‘payload’ argument. In this section we briefly explain the difference.
A HTTP request is comprised of (1) a request line, (2) headers with metadata, (3) an empty line and (4) an optional message body. The HTTP GET method uses query parameters to specify which resource should be retreived from the server. These parameters are included in the request line as part of the URL. The HTTP POST and PUT methods create or update resources. This data is specified in the payload, which is included in the message body of the request. 
We can see the difference using ‘curl’ and ‘ncat’. Open a terminal and run ‘ncat’ in listen mode on port 8080. It will print out any incoming requests.
ncat --listen --port 8080 --keep-open
Parameters
Open a second terminal and send a GET request with a query parameter.
curl -m 1 -X GET localhost:8080/?message=Hello
GET /?message=Hello HTTP/1.1
Host: localhost:8080
User-Agent: curl/7.76.1
Accept: */*

Output of ‘ncat’
The two blank lines indicate that the payload is empty.
Payload
Send a POST request with payload ‘My name is curl’.
curl -m 1 -X POST -d 'My name is curl' localhost:8080
POST / HTTP/1.1
Host: localhost:8080
User-Agent: curl/7.76.1
Accept: */*
Content-Length: 15
Content-Type: application/x-www-form-urlencoded

My name is curl Output of ‘ncat’


## 9.7  Determining the Galaxy version in BioBlend 

In certain situations it might be useful to determine the version of the Galaxy server from BioBlend. We can determine the version using the ‘self.gi.config.get_version’ method.
Example:
if self.gi.config.get_version()['version_major'] >= '21.01':
print('Newer version of Galaxy!')
else:
print('Older version of Galaxy!')
An example where this is useful, at the time of writing, is the ‘download_dataset_collection’ method. It downloads a dataset collection as an archive file. Early versions of Galaxy return a ‘tar.gz’ archive. Later versions return a ‘zip’ archive. The BioBlend method uses the Galaxy version to determine the archive type.

# 10  Conclusion 

We covered the basics of BioBlend development.
Compared to Galaxy, BioBlend is a smaller project with limited complexity. Changes are generally not very hard to implement, which makes it a good candidate for contributions from those not yet familiar with Galaxy in its entirety.
Good luck!
Bibliography 
[1]  Enis Afgan, Dannon Baker, Bérénice Batut, Marius Van den Beek, Dave Bouvier, Martin Čech, John Chilton, Dave Clements, Nate Coraor, Björn A. Grüning, Aysam Guerler, Jennifer Hillman-Jackson, Saskia Hiltemann, Vahid Jalili, Helena Rasche, Nicola Soranzo, Jeremy Goecks, James Taylor, Anton Nekrutenko, and Daniel Blankenberg. The Galaxy platform for accessible, reproducible and collaborative biomedical analyses: 2018 update. Nucleic Acids Research, 46(1):537–544, 07 2018.
[2]  Simone Leo, Luca Pireddu, Gianmauro Cuccuru, Luca Lianas, Nicola Soranzo, Enis Afgan, and Gianluigi Zanetti. BioBlend.objects: metacomputing with Galaxy. Bioinformatics, 30(19):2816–2817, 06 2014.
[3]  Clare Sloggett, Nuwan Goonasekera, and Enis Afgan. BioBlend: automating pipeline analyses within Galaxy and CloudMan. Bioinformatics, 29(13):1685–1686, 04 2013.
