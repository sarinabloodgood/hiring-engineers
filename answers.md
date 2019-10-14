# Collecting Metrics

## Adding Tags to the Agent Config File
![Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.
](https://user-images.githubusercontent.com/33497827/66730147-b898e180-ee1d-11e9-802d-11740702bc86.png) 

To get started with Datadog on MacOS, download the [Datadog Agent](https://docs.datadoghq.com/getting_started/agent/?tab=datadogussite) and launch the agent GUI from your terminal.

```Shell
datadog-agent launch-gui
```

Go to Integrations in the left navigation and follow the instructions for your integration.

To add tags to the agent config file, open the datadog.yaml file located in /etc/datadog-agent/ and create a list of tags. We recommended configuring tags in `<KEY>:<VALUE>` format. Container and cloud environments heavily rotate through hosts, so these tags will allow for aggregation of the metrics you're receiving.

```yaml
#########################
## Basic Configuration ##
#########################

## @param api_key - string - required
## The Datadog API key to associate your Agent's data with your organization.
## Create a new API key here: https://app.datadoghq.com/account/settings
#
api_key: "6995230b413764f1df36dbb30fee22d4"

tags:
    - role:database
    - size:small
    - us:east
```

## Integrating MongoDB with Datadog
![Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.](https://user-images.githubusercontent.com/33497827/66730236-60161400-ee1e-11e9-8272-fa10f7cb4efd.png)

First, install MongoDB locally on your machine. Follow the [MongoDB Community Installation guide](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/) to properly set up MongoDB. Once completed, follow the instructions below to complete the integration.

To integrate MongoDB 3.0 or higher with Datadog, authenticate as an admin user and use the createUser command.
```Shell
# Authenticate as the admin user.
use admin
db.auth("datadog", "datadog)

# On MongoDB 3.x or higher, use the createUser command.
db.createUser({
  "user":"datadog",
  "pwd": "datadog",
  "roles" : [
    {role: 'read', db: 'admin' },
    {role: 'clusterMonitor', db: 'admin'},
    {role: 'read', db: 'local' }
  ]
})
```

Edit the mongo.d/conf.yaml file in the conf.d folder at the root of your Agent’s configuration directory.
```Shell
  init_config:
  instances:
      ## @param server - string - required
      ## Specify the MongoDB URI, with database to use for reporting (defaults to "admin")
      ## E.g. mongodb://datadog:LnCbkX4uhpuLHSUrcayEoAZA@localhost:27016/admin
      #
    - server: "mongodb://datadog:datadog@localhost:27017/admin"

      ## @param replica_check - boolean - required - default: true
      ## Whether or not to read from available replicas.
      ## Disable this if any replicas are inaccessible to the agent.
      #
      replica_check: true

      ## @param additional_metrics - list of strings - optional
      ## By default, the check collects a sample of metrics from MongoDB.
      ## This  parameter instructs the check to collect additional metrics on specific topics.
      ## Available options are:
      ##   * `metrics.commands` - Use of database commands
      ##   * `tcmalloc` -  TCMalloc memory allocator
      ##   * `top` - Usage statistics for each collection
      ##   * `collection` - Metrics of the specified collections
      #
      additional_metrics:
        - metrics.commands
        - tcmalloc
        - top
        - collection
```

## Creating a Custom Agent Check
![Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.
](https://user-images.githubusercontent.com/33497827/66730278-8d62c200-ee1e-11e9-83b5-24d102510c70.png)

To create a custom agent check, create a new Python file in etc/checks.d. In this script, import the random module and the check base class. Create a function that inherits the check itself and submits a metric with a random value between 0 and 1000.

```Python
import random

# the following try/except block will make the custom check compatible with any Agent version
try:
    # first, try to import the base class from old versions of the Agent...
    from checks import AgentCheck
except ImportError:
    # ...if the above failed, the check is running in Agent version 6 or later
    from datadog_checks.checks import AgentCheck

# content of the special variable __version__ will be shown in the Agent status page
__version__ = "1.0.0"


class HelloCheck(AgentCheck):
    def check(self, instance):
        self.gauge('my_metric', random.randint(1,1000), tags=['TAG_KEY:TAG_VALUE'])
```

## Change a Check Collection Interval

To change a check collection interval, create a corresponding YAML file that is titled the same as your Python file. For example, if you python file is named `my_metric.py`, your YAML file must be named `my_metric.yaml`.

In this example, the check's collection interval is set to once every 45 seconds. However, you can set the interval as required.

```YAML
init_config:

instances:
  - min_collection_interval: 45
```

### Did You Know?
The collection interval can be configured without modifying the Python check file. To do this, set the collection interval in the YAML file as noted aboved instead of configuring the interval in the Python file.
