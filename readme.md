# gtt 

[![npm](https://img.shields.io/npm/dt/gitlab-time-tracker.svg?style=flat-square)](https://www.npmjs.com/package/gitlab-time-tracker) [![npm](https://img.shields.io/npm/v/gitlab-time-tracker.svg?style=flat-square)](https://www.npmjs.com/package/gitlab-time-tracker) [![Travis](https://img.shields.io/travis/kriskbx/gitlab-time-tracker.svg?style=flat-square)](https://travis-ci.org/kriskbx/gitlab-time-tracker) [![npm](https://img.shields.io/npm/l/gitlab-time-tracker.svg?style=flat-square)](https://www.npmjs.com/package/gitlab-time-tracker)

> A command line interface for GitLabs time tracking feature.

gtt monitors the time you spent on an issue or merge request locally and syncs it to GitLab. 
It also allows you to create reports in various formats from time tracking data
stored on GitLab.

## contents

* [requirements](#requirements)
* [installation](#installation)
* [updating](#updating)
* [docker](#docker)
* [commands](#commands)
    * [I) configuration](#i-configuration)
    * [II) time tracking](#ii-time-tracking)
    * [III) reports](#iii-reports)
* [configuration](#configuration)
    * [config file](#config-file)
    * [time format](#time-format)
* [how to use gtt as a library](#how-to-use-gtt-as-a-library)
* [faqs](#faqs)
* [contributing](#contributing)
* [buy me a beer 🍺](#buy-me-a-beer)
* [license](#license)

## requirements

* [node.js](https://nodejs.org/en/download) version >= 6
* [npm](https://github.com/npm/npm) or [yarn](https://yarnpkg.com/en/docs/install)

## installation

Install the gtt command line interface using yarn:

```shell
yarn global add gitlab-time-tracker --prefix /usr/local
```

... or using npm (recommended on **Windows** operating systems):

```shell
npm install -g gitlab-time-tracker
```

Run the config command to create a config file and open it in your default editor.
If nothing happens, open the file manually: `~/.gtt/config.yml` - on Windows: `C:\Users\YourUserName\.gtt\config.yml`

```shell
gtt config
```

Add your GitLab API url and your [GitLab API personal token](https://gitlab.com/profile/personal_access_tokens) 
to the config file and save it afterwards:

```yaml
url: https://gitlab.com/api/v4/
token: 01234567891011
```

## updating

**Updating from version <= 1.5? Please [click here](https://github.com/kriskbx/gitlab-time-tracker/blob/master/upgrade.md)!**

Update gtt via yarn:

```shell
yarn global upgrade gitlab-time-tracker
```

... or npm:

```shell
npm install -g gitlab-time-tracker
```

## docker

You don't need to have node and gtt installed on your system in order to use gtt,
you can use the official [Docker image](https://hub.docker.com/r/kriskbx/gitlab-time-tracker) as well:

```shell
docker run \
       --rm -it \
       -v ~:/root \
       kriskbx/gitlab-time-tracker \
       --help
```

`--rm` removes the container after running, `-it` makes it interactive, `-v ~:/root` mounts your home directory to the
home directory inside the container. If you want to store the config in another place, mount another directory: 
 
 
 ```shell
 docker run \
        --rm -it \
        -v /path/to/gtt-config:/root \
        kriskbx/gitlab-time-tracker \
        --help
 ```

... or use a Docker volume:

```shell
docker volume create gtt-config

docker run \
      --rm -it \
      -v gtt-config:/root \
      kriskbx/gitlab-time-tracker \
      --help
```
 
I highly recommend creating an alias and adding it to your `bashrc`:
 
```shell
echo "alias gtt='docker run --rm -it -v ~:/root kriskbx/gitlab-time-tracker'" >>~/.bash_rc
```

Now you can simply write `gtt` instead of the bulky Docker command before. Try it out: `gtt --help`

**Note:** If you want to save reports via the `--file` parameter, make sure to save them in `/root` or another 
mounted directory that you have access to on your host machine. Take a look at the [Docker documentation](https://docs.docker.com/engine/tutorials/dockervolumes/) about how Docker handles data and volumes.

## commands

### I) configuration

*[Click here](#configuration) for a complete list of configuration options.*

**Edit/create the global configuration file, stored in your home directory:**

```shell
gtt config
```

**Edit/create a local configuration file `.gtt.yml` in the current working directory:**

```shell
gtt config --local
```

If a local configuration file is present it will extend of the global one and overwrites global settings.
If you don't want to extend the global configuration file, set the `extend` option in your local config to `false`.
So you can use gtt easily on a per project basis. Make sure to add .gtt.yml to your gitignore file if using a local configuration.

If nothing happens, open the file manually: `~/.gtt/config.yml` - on Windows: `C:\Users\YourUserName\.gtt\config.yml`

### II) time tracking

Time tracking enables you to monitor the time you spent on an issue or merge request locally.
When you're done, you can sync these time records to GitLab: gtt adds the time spent to the given 
issue or merge request and automagically keeps everything in sync with your local data.
 
Did you forgot to stop time monitoring locally and accidentally synced it to GitLab? 
No worries, just edit the local record and run sync again.

**Start local time monitoring for the given project and issue id:**

```shell
gtt start "kriskbx/example-project" 15
gtt start 15
```

If you configured a project in your config you can omit the project.

**Start local time monitoring for a merge request:**

```shell
gtt start --type=merge_request "kriskbx/example-project" 15
gtt start --type=merge_request 15
```

If you configured a project in your config you can omit the project.

**Show the current time monitoring status:**

```shell
gtt status
```

**Stop local time monitoring and save as a new time record:**

```shell
gtt stop
```

**Cancel local time monitoring and discard the time record:**

```shell
gtt cancel
```

**Show a list of all local time records (including their ids and meta data):**

```shell
gtt log
```

**Edit a local time record by the given id:**

```shell
gtt edit 2XZkV5LNM
gtt edit
```

You can omit the id to edit the last added time record.

**Delete a local time record by the given id:**

```shell
gtt delete 2XZkV5LNM
gtt delete
```

You can omit the id to delete the last added time record.

**Sync local time records with time tracking data on Gitlab:**

```shell
gtt sync
gtt sync --proxy="http://localhost:8888"
```

You can pass an url to the proxy option if you want to use a proxy server.

### III) reports

Get a report for the time tracking data stored on GitLab. If you want to include your local data make sure to sync it
before running the report command. The report command has a lot of options to filter data and output, make sure to 
read through the docs before using it.

#### Get a report

```shell
gtt report ["namespace/project"] [issue_id]
gtt report "kriskbx/example-project"
gtt report "kriskbx/example-project" 145
gtt report "kriskbx/example-project" 145 209 45 54
gtt report
gtt report 145
gtt report 123 345 123
```

If you configured a project in your config file you can omit it. By passing a or multiple ids you can limit the
report to the given issues or merge requests. *Note: if you're passing a or multiple ids, gtt will fetch issues
with these ids by default. If you want to fetch merge requests you can change the query type using the
`--query` option.*

#### Query groups and subgroups

```shell
gtt report ["namespace"] --type=group
gtt report example-group --type=group
gtt report example-group --type=group --subgroups
```

Query all projects from the given group and combine the data into a single report. The option `--subgroups`
includes subgroups.

#### Query multiple projects or groups and combine the data in one report

```shell
gtt report ["namespace/project"] ["namespace/project"] ...
gtt report "kriskbx/example-project" "kriskbx/example-project-2"
gtt report ["namespace"] ["namespace"] ... --type=group
gtt report example-group example-group-2 --type=group
```

*Hint: use the `project_id` or `project_namespace` columns in your report.*

#### Choose an output for your report

```shell
gtt report --output=table
gtt report --output=markdown
gtt report --output=csv
gtt report --output=pdf --file=filename.pdf
```

Defaults to `table`. `csv` and `markdown` can be printed to stdout, `pdf` needs the file parameter.

#### Print the output to a file

```shell
gtt report --file=filename.txt
```

#### Only get time records from the given time frame

```shell
gtt report --from="2017-03-01" --to="2017-04-01"
```

*Note: `--from` defaults to 1970-01-01, `--to` defaults to now. Make sure to use an [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) compatible time/date format.*

#### Include closed issues/merge requests

```shell
gtt report --closed
```

#### Limit to the given user

```shell
gtt report --user=username
```

#### Limit to the given milestone

```shell
gtt report --milestone=milestone_name
```

#### Limit issues and merge requests to the given labels 

```shell
gtt report --include_by_labels=critical --include_by_labels=important
```

#### Exclude issues and merge requests that have the given labels

```shell
gtt report --exclude_by_labels=wont-fix --exclude_by_labels=ignore
```

#### Limit the query to the given data type

```shell
gtt report --query=issues
gtt report --query=merge_requests
```

#### Include issues and merge requests in the report that have no time records

```shell
gtt report --show_without_times
```

#### Limit the parts in the final report

```shell
gtt report --report=stats --report=issues
```

*Note: These parts are available: `stats`, `issues`, `merge_requests`, `records`*

#### Hide headlines in the report

```shell
gtt report --no_headlines
```

#### Hide warnings in the report

```shell
gtt report --no_warnings
```

#### Set date format for the report

```shell
gtt report --date_format="DD.MM.YYYY HH:mm:ss"
```

*Note: [Click here](http://momentjs.com/docs/#/displaying/format/) for a further documentation on the date format.*

#### Set time format for the report

```shell
gtt report --time_format="[%sign][%days>d ][%hours>h ][%minutes>m ][%seconds>s]"
```

*Note: [Click here](#time-format) for a further documentation on the time format.*

#### Set columns included in the time record table

```shell
gtt report --record_columns=user --record_columns=date --record_columns=time
```

*Note: Available columns to choose from: `user`, `date`, `type`, `iid`, `project_id`, `project_namespace`, `time`*

#### Set columns included in the issue table

```shell
gtt report --issue_columns=iid --issue_columns=title --issue_columns=time_username
```

*Note: Available columns to choose from: `id`, `iid`, `title`, `project_id`, 
`project_namespace`, `description`, `labels`, `milestone`, `assignee`, `author`,
`closed`, `updated_at`, `created_at`, `state`, `spent`, `total_spent`, `total_estimate`*

*You can also include columns that show the total time spent by a specific user 
by following this convention: `time_username`*

#### Set columns included in the merge request table

```shell
gtt report --merge_request_columns=iid --merge_request_columns=title --merge_request_columns=time_username
```

*Note: Available columns to choose from: `id`, `iid`, `title`, `project_id`,
`project_namespace`, `description`, `labels`, `milestone`, `assignee`, `author`,
`updated_at`, `created_at`, `state`, `spent`, `total_spent`, `total_estimate`*

*You can also include columns that show the total time spent by a specific user 
by following this convention: `time_username`*

#### Add columns for each project member to issue and merge request table, including their total time spent

```shell
gtt report --user_columns
```

#### Only include the given labels in the report

```shell
gtt report --include_labels=pending --include_labels=approved
```

#### Exclude the given labels from the report

```shell
gtt report --exclude_labels=bug --exclude_labels=feature
```

#### Use a proxy server

```shell
gtt report --proxy="http://localhost:8080"
```

#### Output verbose debug information

```shell
gtt report --verbose
```

## configuration

The configuration uses the [yaml file format](http://www.yaml.org/spec/1.2/spec.html). 
[Click here](#i-configuration) for more information how to edit and create a config file.

### Config File

Here's a sample configuration file including all available options:

```yaml
# Url to the gitlab api
# Make sure there's a trailing slash
# [required]
url: http://gitlab.com/api/v4/

# GitLab personal api token
# [required]
token: abcdefghijklmnopqrst

# Use a proxy server
# defaults to false
proxy: http://localhost:8080

# Project
# defaults to false
project: namespace/projectname

# Include closed issues and merge requests
# defaults to false
closed: true

# Limit to the given milestone
# defaults to false
milestone: milestone_name

# Exclude issues and merge requests that have the given labels
# defaults to an empty array
excludeByLabels:
- wont-fix
- ignore

# Limit issues and merge requests to the given labels
# defaults to an empty array
includeByLabels:
- critical
- important

# Limit the query to the given data type
# available: issues, merge_requests
# defaults to [issues, merge_requests]
query:
- issues

# Limit the parts in the final report
# available: stats, issues, merge_requests, records
# defaults to [stats, issues, merge_requests, records]
report:
- stats
- records

# Hide headlines in the report
# defaults to false
noHeadlines: true

# Hide warnings in the report
# defaults to false
noWarnings: true

# Include issues and merge requests in the report that have no time records
# defaults to false
showWithoutTimes: true

# Hours per day
# defaults to 8
hoursPerDay: 8

# Include the given columns in the issue table
# See --issue_columns option for more information
# defaults to iid, title, spent, total_estimate
issueColumns:
- iid
- title
- estimation

# Include the given columns in the merge request table
# See --merge_request_columns option for more information
# defaults to iid, title, spent, total_estimate
mergeRequestColumns:
- iid
- title
- estimation

# Include the given columns in the time record table
# See --record_columns option for more information
# defaults to user, date, type, iid, time
recordColumns:
- user
- iid
- time

# Add columns for each project member to issue and
# merge request table, including their total time spent
# defaults to true
userColumns: true

# Date format
# Click here for format options: http://momentjs.com/docs/#/displaying/format/
# defaults to DD.MM.YYYY HH:mm:ss
dateFormat: DD.MM.YYYY HH:mm:ss

# Time format
# See time format configuration below
# defaults to "[%sign][%days>d ][%hours>h ][%minutes>m ][%seconds>s]"
timeFormat: "[%sign][%days>d ][%hours>h ][%minutes>m ][%seconds>s]"

# Output type
# Available: csv, table, markdown, pdf
# defaults to table
output: markdown

# Exclude the given labels from the report
# defaults to an empty array
excludeLabels:
- bug
- feature

# Only include the given labels in the report
# defaults to an empty array
includeLabels:
- pending
- approved

# Change number of concurrent connections/http queries
# Note: Handle with care, we don't want to spam GitLabs API too much
# defaults to 10
_parallel: 20

# Change rows per page (max. 100)
# defaults to 100
_perPage: 100

# Only works if using a local configuration file!
# Extend the global configuration if set to true, pass a string to extend 
# the configuration file stored at the given path
# defaults to true
extend: true
```

### Time format

##### `[%sign]`, `[%days]`, `[%hours]`, `[%minutes]`, `[%seconds]`

Prints out the raw data.
 
**Example config:** 
 
```yaml
timeFormat: "[%sign][%days]d [%hours]h [%minutes]m [%seconds]s"
```

**Example outputs:**

```shell
0d 0h 30m 15s
-1d 10h 15m 0s
```
 
##### `[%days>string]`, `[%hours>string]`, `[%minutes>string]`, `[%seconds>string]`

This is a conditional: it prints out the raw data and appends the given string if the
data is greater than zero.

**Example config:** 
 
```yaml
timeFormat: "[%sign][%days>d ][%hours>h ][%minutes>m ][%seconds>s]"
```

**Example outputs:**

```shell
30m 15s
-1d 10h 15m
```

##### `[%Days]`, `[%Hours]`, `[%Minutes]`, `[%Seconds]`, `[%Days>string]`, `[%Hours>string]`, `[%Minutes>string]`, `[%Seconds>string]`

First letter uppercase adds leading zeros.

**Example config:** 
 
```yaml
timeFormat: "[%sign][%Days]:[%Hours]:[%Minutes]:[%Seconds]"
```

**Example outputs:**

```shell
00:00:30:15
-01:10:15:00
```

##### `[%days_overall]`, `[%hours_overall]`, `[%minutes_overall]`, `[%seconds_overall]`

Instead of printing out the second-, minute-, hour-, day-part of the duration this
prints the complete time in either seconds, minutes, hours or days.

**Example config:** 
 
```yaml
timeFormat: "[%sign][%minutes_overall]"
```

**Example outputs:**

```shell
30.25
1095
```

##### `[%days_overall_comma]`, `[%hours_overall_comma]`, `[%minutes_overall_comma]`, `[%seconds_overall_comma]`

Use a comma for float values instead of a point. (Useful for some european countries)

**Example config:** 
 
```yaml
timeFormat: "[%sign][%minutes_overall]"
```

**Example outputs:**

```shell
30,25
1095
```

## how to use gtt as a library

Add as a dependency using yarn:

```shell
yarn add gitlab-time-tracker
```

... or using npm:

```shell
npm install --save gitlab-time-tracker
```

Use it in your project:

```js
// require modules
const Config = require('gitlab-time-tracker/src/include/config');
const Report = require('gitlab-time-tracker/src/models/report');

// create a default config
let config = new Config();

// set required parameters
config.set('token', 'abcdefghijklmnopqrst');
config.set('project', 'namespace/project');

// create report
let report = new Report(config);

// query and process data
try {
    await report.getProject()
    await report.getIssues()
    await report.getMergeRequests()
    await report.processIssues()
    await report.processMergeRequests()
} catch (error) {
    console.log(error)
}
      
// access data on report
report.issues.forEach(issue => {
    // time records on issue
    console.log(issue.times);
    // time spent of single time record
    console.log(issue.times[0].time);
});
report.mergeRequests.forEach(mergeRequest => {
    // time records on merge requests
    console.log(mergeRequests.times);
    // user of single time record
    console.log(issue.times[0].user);
});
```

## faqs

#### What is the difference between 'total spent' and 'spent'?

`total spent` is the total amount of time spent in all issues after filtering.
It can include times outside the queried time frame. `spent` on the other hand
is the total amount of time spent in the given time frame.

#### Why 'total spent' and 'spent' are showing different amounts.

gtt can only track time records from notes/comments. If you start your 
issue or merge request with `/spent [time]` in its description, gtt won't
take it into consideration (for now).

## contributing

I would love to integrate unit testing in this project, but unfortunately my knowledge of 
testing in the JavaScript/Node.js world is very limited. (I'm actually a PHP dev) 
So this would be a very helpful thing to contribute but of course all contributions are very welcome.

* Please work in your own branch, e.g. `integrate-awesome-feature`, `fix-awful-bug`, `improve-this-crappy-docs`
* Create a pull request to the `dev` branch

## buy me a beer

gtt is an open source project, developed and maintained completely in my free time.

If you enjoy using gtt you can support the project by [contributing](#contributing) to the code base, 
sharing it to your colleagues and co-workers or monetarily by [donating via PayPal](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=DN9YVDKFGC6V6). 
Every type of support is helpful and thank you very much if you consider supporting the project 
or already have done so. 💜

## license

GPL v2