# jira-labelbot

Label Bot makes maintaining labels on Jira issues easy by automatically replacing, 
merging and removing labels based on a set of easily configurable rules.

Label Bot performs the following actions on labels found in your Jira instance:

1. Completely ignores labels matching a pattern.
2. Drops all labels from uppercase to lowercase.
3. Replaces one label with another.
4. Merges two or more labels into a single label.
5. Removes unwanted labels.

##	Requirements

Label Bot is a Perl script. It expects to find your perl executable under /usr/bin/perl.
The following Perl modules are required:

	Getopt::Long
	REST::Client
	JSON
	Set::Scalar


## Command Line Usage

Usage: labelbot [options]

Options:

	--rules=file     Specify a rules file (default=./rules.json).
	--cred=file      Specify a credentials file (default=./labelbot.cred).
	--mkcred=creds   Converts credentials to base64 format for credentials file.
	                 Example: --mkcred=username:password
	                 (Note: This is NOT encrypted. Protect your cred file.
	--test           Review issues and output a summary without modifying labels.
	--help           Output this usage.


## Setup Instructions

### 1. Familiarize yourself with the Command Line Usage (above)

In particular, always include --test until you know what
you are doing.

### 2. Create a Jira user (optional)

You may use an existing Jira user or create a new user
for Label Bot to use for managing labels. It is recommended
that you create a new user for this purpose to make it easier
to follow changes made by Label Bot. The user should have
permissions to view and edit all issues in your Jira instance.

### 3. Create a credentials file

The Jira API requires the username and password of a Jira user
be formatted in a certain way. This is done by taking the
username and password, sparated by a colon, and converted
into base64.

Label Bot will perform this formatting for you with the 
--mkcred option. Run:

```
./labelbot --mkcred=username:password > labelbot.cred
```

By default, labelbot looks for these credentials in the
./labelbot.cred file, but can be pointed elsewhere with the
--cred option.

Note: The contents of this file are saved in base64, but this
is *NOT* encrypted. The contents can easily be converted back
to obtain the cleartext username and password of your Jira
user. Set appropriate permissions on your credentials file.

### 4. Create your rules file

A default rules file has been included, called 
./rules.json. The rules file is in JSON (JavaScript
Object Notation) format, and can easily be edited by hand.
the sample into place like this:

The rules file has several directives:

* jiraurl
* ignore
* forcelowercase
* issuefilters
* replace
* merge
* ignore

The jiraurl directive is the full base URL to your Jira
instance. Example:

```
  "jiraurl": "https://jira.uwo.ca",
```

The ignore directive allows you to have Label Bot ignore any
labels that match certain patterns (using Perl regular
expressions). Example:

```
  "ignore": [
    "collector-.*"
  ],
```

This will ignore any labels starting with "collector-".

The forcelowercase directive tells Label Bot whether or not
you want to force all labels to be lowercase. "1" (default)
means yes and "0" means no.

```
	"forcelowercase": "1",
```

The issuefilters directive allows you to filter the jira
query to only a subset of the issues in your jira instance.
Maybe you just want to operate on issues in one project:

```
	"issuefilters": {
			"project": "RAD"
	},
```

You can remove the issuefilters directive completely if
you want to consider all issues in your jira instance.

The replace directive allows you to replace one label with another.
Example:

```
  "replace": {
    "mail": "email",
    "cert": "certificate"
  },
```

The merge directive allows you to merge two or more labels into 
a single label. All of the labels being merged must exist on
an issue for a merge to be performed. Example:

```
  "merge": [
    {
      "from": [ "password", "reset" ],
      "to": "password-reset"
    },
    {
      "from": [ "email", "quota" ],
      "to": "email-quota"
    }
  ],
```

The remove directive allows you to remove specific labels without
replacing them with anything. Example:

```
  "remove": [
    "badlabel",
    "verybadlabel"
  ]
```

### 5. Running Label Bot

Once you have labelbot.cred and rules.json in place, you're ready
to run Label Bot. Label Bot will output some details about what it's
doing with each issue. You'll likely want to redirect this output
to a log file. Example:

```
./labelbot > logs/labelbot.log
```

Or you can run the ./labelbot.run script which runs labelbot and 
directs this output to a new timestamped log file each time it's run.
This script also removes any log files older than 1 week. 

Modify ./labelbot.run as needed to suit your environment.

### 6. Running under crontab

The ./labelbot.run script is suitable for running Label Bot under
crontab. A sample crontab configuration file, ./labelbot.cron,
has been provided.

## Maintainers

* Andrew Culver (original author)
* burnettk

## Contributing

Send a pull request.
