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
from datadog_checks.checks import AgentCheck
import random

class HelloCheck(AgentCheck):
    def check(self, instance):
        self.gauge('my_metric', random.randint(0, 1000))
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

## Visualizing Data
To create a dashboard with a timeseries widget, import that Datadog API and define the timeseries widget. In this example we set the metric used in our custom agent check to be visualized as a timeseries. We also used the anomoly function in a timeseries to show the number of MongoDB update operations per second. You can choose any widget type that is best for visualizing the metrics you need to visualize.

```Python
from datadog import initialize, api

options = {
    'api_key': '6995230b413764f1df36dbb30fee22d4',
    'app_key': '9b798128f85ae046ad16eafe3435a51f22656105'
}

initialize(**options)

title = 'My Metric'
widgets = [{
    'definition': {
        'type': 'timeseries',
        'requests': [
            {
                "q": "avg:my_metric{*}"
            }
        ],
        'title': 'My Metric'
    }
},
    {
    'definition': {
        'type': 'timeseries',
        'requests': [
            {'q': "anomalies(avg:mongodb.metrics.ttl.passesps{*}, 'basic', 2)"}
        ],
        'title': 'MongoDB # of Update Ops per sec'
    }
}]
layout_type = 'ordered'
description = 'A dashboard with metric info.'

api.Dashboard.create(title=title,
                     widgets=widgets,
                     layout_type=layout_type,
                     description=description)
```
## Dashboard UI
![Set timeboard time frame to the past 5 minutes, take a snapshot of the graph and use @ notation](https://user-images.githubusercontent.com/33497827/66783605-3bfe1580-eea6-11e9-8865-e0434e5f6e15.png)
Datadog's dashboard UI populates metrics in real-time. To access your dashboard, log into [Datadog](https://app.datadoghq.com/) and select Dashboards from the left hand navigation. Here you can view your dashboards, take snapshots and send timeboards, or export the entire dashboard as a JSON file. To send a snapshot, hover over the top right corner of the timeboard until the export icon appears. Choose send snapshot and use the @ symbol to send the file to yourself or others.

## Did you know?
Anomaly algorithms identify when a metric is behaving differently than before. These algorithms take into account trends, seasonal day-of-week, and time-of-day patterns and are well-suited for metrics with strong trends and recurring patterns that are hard to monitor.

# Blog Post
## Monitor your Auth0 logs with Datadog
![Datadog Logs](https://imgix.datadoghq.com/img/blog/announcing-logs/announcing-logs-hero-hg-b.png?fit=crop&w=1200&h=630)
Auth0 is a universal authentication and authorization platform for web, mobile, and native applications. Auth0 permits any application or API to connect to their service to eliminate unauthorized logins and improper access. Using the Auth0 Dashboard or the Management API logs endpoint, Auth0 can pull log data on actions performed by administrators using the Dashboard, operations performed via the Management API, and authentications made by your users.

---

Datadog log management enables you to collect, process, archive, explore, and monitor all of your logs. Ingesting logs from your hosts, containers, and cloud providers to generate live metrics lets you get a glimpse of what’s happening across all of your environments. With integrations and extensions, Datadog allows you to integrate all of your logs from different environments into one place. New custom integrations contributed by the Datadog community help expand your metric set beyond the scope of our own integrations, giving you the tools to monitor whatever services you need.

We’re excited to announce the Auth0 logs extension contributed by BetaProjectWave. With this new extension you can now integrate the Auth0 platform with Datadog to view your Auth0 logs in real-time. This easy to install and quick to configure extension lets you see your Auth0 logs in [Datadog log management](https://docs.datadoghq.com/logs/) so you can easily filter and visualize log data without having to log into the Auth0 platform.

To get started, log into your Auth0 account and visit the [Extensions Gallery](https://auth0.auth0.com/login?state=g6Fo2SBmQnJUcGZzeTZlSEF1RzRuLW1xbjloLVpqanZqSzZ0d6N0aWTZIFNDTW5BNFd5V0k0TExoRnFuRjBGdGF0THhVZHBpLUo2o2NpZNkgekVZZnBvRnpVTUV6aWxoa0hpbGNXb05rckZmSjNoQUk&client=zEYfpoFzUMEzilhkHilcWoNkrFfJ3hAI&protocol=oauth2&response_type=code&redirect_uri=https%3A%2F%2Fmanage.auth0.com%2Fcallback&scope=openid%20profile%20name%20email%20nickname%20created_at#/extensions). Click on the Datadog extension (linked above) or build locally using this [Github repo](https://github.com/BetaProjectWave/auth0-logs-to-datadog#filters).

Building this extension locally requires [NodeJS 6+](https://nodejs.org/en/download/). To build this extension locally, click on the clone/download repo button in Github to copy the URL and use Git or download the ZIP file. To download the repo using Git, open a shell of your choice and use the following command:

```Shell
git clone https://github.com/BetaProjectWave/auth0-logs-to-datadog.git
```

Once the files are available on your local host, run the command `yarn install —ignore engines` to install all of the appropriate dependencies. Once all dependencies are installed, bundle the app by running the `yarn build` command.

As a best practice, we recommend additionally configuring this extension to filter logs by a specific type such as LOG_TYPES or LOG_LEVEL. For example, to filter logs tied to a specific event such as when a user successfully logs in, set the `LOG_TYPES` filter to `s` in your Datadog logs dashboard. This will help filter out unnecessary logs and help you focus only on the logs that you require.

There is a list of filters and information [available here](https://github.com/BetaProjectWave/auth0-logs-to-datadog#filters) for the Auth0 logs extension via Github. Datadog also offers configuration of [Auth0 as a SAML IdP](https://docs.datadoghq.com/account_management/saml/auth0/#configuration) if you need further options for Auth0 configuration.

We want to give a special thanks to creators of this extension, [BetaProjectWave](https://github.com/BetaProjectWave/auth0-logs-to-datadog). The thought, time, and work they and others contribute to this extension is what empowers us to support custom integrations. To learn more about Datadog extensions and integrations and how to get started on your own custom integration, [read more here](https://docs.datadoghq.com/developers/integrations/new_check_howto/).
