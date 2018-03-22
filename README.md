# snowtify
A utility to connect a CD pipeline to the ServiceNow API (often referred to here as `snow` for short).

## Usage
The utility is configured via environment variables - see the `config.js` tests
[here](https://github.com/UKHomeOffice/snowtify/blob/master/test/config.js), and the
[Input options](#input-options) section below for more details.

### As a Drone Plugin
This utility has been designed to be used (with minimal configuration) as a `drone.io` plugin. The
following configuration gives a basic example template of a deployment pipeline using the plugin:
```yaml
matrix:                 # setup some common env vars
  SNOW_USER:            # username for basic auth
    - my-CD-robot
  SNOW_INT_ID_FILE:     # file containing ServiceNow ID for this change
    - './internal-id'

pipline:
  setup:
    ...
  
  open-snow-change:
    image: quay.io/repository/ukhomeofficedigital/snowtify
    secrets: [ snow_password ]
    description: "Look out! We're hitting the big red button..."
  
  deploy:
    ...
  
  complete-snow-change:
    image: quay.io/repository/ukhomeofficedigital/snowtify
    secrets: [ snow_password ]
    comments: "Yay, it worked!"
    deployment_outcome: success
  
  cancel-snow-change:
    image: quay.io/repository/ukhomeofficedigital/snowtify
    secrets: [ snow_password ]
    comments: "Oooops, something went wrong!"
    deployment_outcome: fail
```
In the example above, the plugin is configured 3 times: first to send a notification of a new change
(the `open-snow-change` job), then to report either a successfully completed or failed deployment (the
`complete-snow-change` and `cancel-snow-change` jobs respectively). The options `description`,
`comments`, and `deployment_outcome` are set as part of the plugin's configuration - these options
correspond to the following environment variables:
[PLUGIN_DESCRIPTION](#snow_desc--plugin_description),
[PLUGIN_COMMENTS](#snow_comments--plugin_comments),
[PLUGIN_DEPLOYMENT_OUTCOME](#snow_status--plugin_deployment_outcome). All of the variable names
listed in the [Input options](#input_options) section that start with `PLUGIN_` can be configured
directly against the drone job (without the `PLUGIN_` prefix).

The `.drone.yml` file for this project
([here](https://github.com/UKHomeOffice/snowtify/blob/master/.drone.yml))
can also be used as a real example of how to notify ServiceNow as part of a deployment. The
"end-to-end" test jobs give an example of sending a change creation and completion notification.

### As a Docker container
The utility can also be run as a standard docker container, as shown in the following example:
```bash
docker run quay.io/repository/ukhomeofficedigital/snowtify \
    -e SNOW_USER=username \
    -e SNOW_PASS=password \
    -e SNOW_EXTERNAL_ID=my-project-id \
    -e SNOW_TITLE="Another deployment" \
    -e SNOW_DESC="It's time to release that thing we changed..."
```

## Input options
The following options exist to control - the kind of notification, and the details - that get sent
to ServiceNow (snow):

#### REPO_NAME | DRONE_REPO_NAME
When run as a drone plugin, drone sets this automatically. Otherwise this can be set to indicate the
project repository name, which is used to build a default external ID and change title if either are
not explicitly provided. See
 - [SNOW_EXTERNAL_ID | PLUGIN_EXTERNAL_ID](#snow_external_id--plugin_external_id), and
 - [SNOW_TITLE | PLUGIN_TITLE](#snow_title--plugin_title).

#### BUILD_NUMBER | DRONE_BUILD_NUMBER
When run as a drone plugin, drone sets this automatically. Otherwise this can be set to indicate the
build number for the current CD pipeline, which is used to build a default external ID and change
title if either are not explicitly provided. See
 - [SNOW_EXTERNAL_ID | PLUGIN_EXTERNAL_ID](#snow_external_id--plugin_external_id), and
 - [SNOW_TITLE | PLUGIN_TITLE](#snow_title--plugin_title).

#### SNOW_PROTOCOL | PLUGIN_PROTOCOL
The protocol part of the URL for the ServiceNow API endpoint - the default is "https".

#### SNOW_PROD_HOST | PLUGIN_PROD_HOST
The host part of the URL for the production version of the ServiceNow API endpoint - the default is
"lssiprod.service-now.com". Whether this host, or the Test host (described below) is used to construct
the URL used for the ServiceNow API endpoint can be configured by the `DEPLOY_TO` settings. See
 - [SNOW_TEST_HOST | PLUGIN_TEST_HOST](#snow_test_host--plugin_test_host), and
 - [SNOW_DEPLOY_TO | PLUGIN_DEPLOY_TO | DEPLOY_TO](#snow_deploy_to--plugin_deploy_to--deploy_to).

#### SNOW_TEST_HOST | PLUGIN_TEST_HOST
The host part of the URL for the sandbox version of the ServiceNow API endpoint - the default is
"lssitest.service-now.com". Whether this host, or the Production host (described above) is used to
construct the URL used for the ServiceNow API endpoint can be configured by the `DEPLOY_TO` settings.
See
 - [SNOW_PROD_HOST | PLUGIN_PROD_HOST](#snow_prod_host--plugin_prod_host), and
 - [SNOW_DEPLOY_TO | PLUGIN_DEPLOY_TO | DEPLOY_TO](#snow_deploy_to--plugin_deploy_to--deploy_to).

#### SNOW_PATH | PLUGIN_PATH
The path part of the URL for the ServiceNow API endpoint - the default is
"api/now/table/x_fho_siam_integra_transactions".

#### SNOW_ENDPOINT | PLUGIN_ENDPOINT
This defines the full exact endpoint for the ServiceNow API. If set it will override the other URL
options, and the `DEPLOY_TO` settings. See
 - [SNOW_PROTOCOL | PLUGIN_PROTOCOL](#snow_protocol--plugin_protocol),
 - [SNOW_PROD_HOST | PLUGIN_PROD_HOST](#snow_prod_host--plugin_prod_host),
 - [SNOW_TEST_HOST | PLUGIN_TEST_HOST](#snow_test_host--plugin_test_host),
 - [SNOW_PATH | PLUGIN_PATH](#snow_path--plugin_path), and
 - [SNOW_DEPLOY_TO | PLUGIN_DEPLOY_TO | DEPLOY_TO](#snow_deploy_to--plugin_deploy_to--deploy_to).

#### SNOW_DEPLOY_TO | PLUGIN_DEPLOY_TO | DEPLOY_TO
If set to "prod" (case insensitive), specifies the notification is to be sent to the production
URL - the default is to send notifications to the test URL. See 
 - [SNOW_PROD_HOST | PLUGIN_PROD_HOST](#snow_prod_host--plugin_prod_host),
 - [SNOW_TEST_HOST | PLUGIN_TEST_HOST](#snow_test_host--plugin_test_host).

#### SERVICE_NOW_USERNAME | SNOW_USER | PLUGIN_USERNAME
The username used for (basic) authentication to the ServiceNow API.

#### SERVICE_NOW_PASSWORD | SNOW_PASS | PLUGIN_PASSWORD
The password used for (basic) authentication to the ServiceNow API.

#### SNOW_EXTERNAL_ID | PLUGIN_EXTERNAL_ID
A project/build ID for the current CD pipeline. If not provided a default is constructed using the
template `${SERVICE_NOW_USERNAME}-${REPO_NAME}-${BUILD_NUMBER}`. See
 - [SERVICE_NOW_USERNAME | SNOW_USER | PLUGIN_USERNAME](#service_now_username--snow_user--plugin_username),
 - [REPO_NAME | DRONE_REPO_NAME](#repo_name--drone_repo_name), and
 - [BUILD_NUMBER | DRONE_BUILD_NUMBER](#build_number--drone_build_number).

#### SNOW_INT_ID_FILE | PLUGIN_INTERNAL_ID_FILE
Path to a file for storing the internal ID (when opening a new change), or retrieving the ID of the
target change (when updating its status). If this is not provided when opening a new change, be sure
to take note of the ID when it is printed, after the new change is successfully created. See
[SNOW_INTERNAL_ID | PLUGIN_INTERNAL_ID](#snow_internal_id--plugin_internal_id).

#### SNOW_INTERNAL_ID | PLUGIN_INTERNAL_ID
Used to identify a previously opened change when updating the status and closing. Setting this will
override the equivalent file-based option. See
[SNOW_INT_ID_FILE | PLUGIN_INTERNAL_ID_FILE](#snow_int_file--plugin_internal_id_file)

#### SNOW_NOTIFICATION_TYPE | PLUGIN_NOTIFICATION_TYPE
Indicates whether opening a new change (if set to "deployment"), or closing an existing one (if set
to "update" or "status update"). The value is not case sensitive. If omitted, the type will be
assumed to be a "deployment", unless the "status update" comments have been set. See
 - [SNOW_COMMENTS | PLUGIN_COMMENTS](#snow_comments--plugin_comments), and
 - [SNOW_COMMENTS_FILE | PLUGIN_COMMENTS_FILE](#snow_comments_file--plugin_comments_file).

#### SNOW_TITLE | PLUGIN_TITLE
The title for the new change being opened. If not provided a default is constructed using the
template: `Deployment #${BUILD_NUMBER} of ${REPO_NAME}`. See
 - [BUILD_NUMBER | DRONE_BUILD_NUMBER](#build_number--drone_build_number), and
 - [REPO_NAME | DRONE_REPO_NAME](#repo_name--drone_repo_name).

#### SNOW_END_TIME | PLUGIN_END_TIME
The expected completion time of the change. This value is expected to be a date-time string with the
format: `YYYY-MM-DD HH:mm:ss`. If omitted, a default of the current time plus 30 minutes will be used.

#### SNOW_DESC_FILE | PLUGIN_DESCRIPTION_FILE
Path to a file containing the full description of the new change - e.g. the relevant commit history,
user stories, etc. See
[SNOW_DESC | PLUGIN_DESCRIPTION](#snow_desc--plugin_description).

#### SNOW_DESC | PLUGIN_DESCRIPTION
The full description of the new change - e.g. the relevant commit history, user stories, etc.
Setting this will override the equivalent file-based option. See
[SNOW_DESC_FILE | PLUGIN_DESCRIPTION_FILE](#snow_desc_file--plugin_description_file).

#### SNOW_TESTING_FILE | PLUGIN_TESTING_FILE
Path to a file containing any unit/integration/end-to-end testing result logs to support the new
change. This is optional, as such there is no default value. See
[SNOW_TESTING | PLUGIN_TESTING](#snow_testing--plugin_testing).

#### SNOW_TESTING | PLUGIN_TESTING
Any unit/integration/end-to-end testing result logs to support the new change. This is optional, as
such there is no default value. Setting this will override the equivalent file-based option. See
[SNOW_TESTING_FILE | PLUGIN_TESTING_FILE](#snow_testing_file--plugin_testing_file).

#### SNOW_COMMENTS_FILE | PLUGIN_COMMENTS_FILE
Path to a file containing more information about the outcome of the change that is being closed.
Comments must be provided for a status update, either using this option or the non-file-based
option. See
[SNOW_COMMENTS | PLUGIN_COMMENTS](#snow_comments--plugin_comments).

#### SNOW_COMMENTS | PLUGIN_COMMENTS
More information about the outcome of the change that is being closed. Comments must be provided for
a status update, either using this option or the file-based option. Setting this will override the
equivalent file-based option. See
[SNOW_COMMENTS_FILE | PLUGIN_COMMENTS_FILE](#snow_comments_file--plugin_comments_file).

#### SNOW_STATUS | PLUGIN_DEPLOYMENT_OUTCOME
This is used to indicate the outcome of a change. Set it to "success" (case insensitive) if the
deployment succeeded, otherwise the change will be marked as a failure. If the utility is running as
a drone plugin (as in the [example](#as-a-drone-plugin) above), this option can be omitted and the
value will instead be picked up from the drone environment variable `DRONE_BUILD_STATUS`.

## Useful links
 - [Quay.io](https://quay.io/repository/ukhomeofficedigital/snowtify)

## License
This project is licensed under the GPL-3.0 License - see the `LICENSE` file for details.
