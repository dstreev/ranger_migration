# Ranger Migration Utilities
Assistance when upgrading to CDP

## Requirements
Python 2.7.5+
Install 'requests' library for Python
`pip install requests`

## Ranger Policy Migration
Upgrading from Ambari-DC to CDP-DC does not leverage the same repositories between these two management platforms.  Whether the Ranger service is migrated, configured and integrated with the other services is outside the scope of this tool.

After the migration from Ambari to Cloudera Manager, you will have multiple service repositories in each service.  The new repository will have a name like 'cm_*' and will have the basic defaults installed for new deployments, in addition to being properly integrated with the other services under Cloudera Manager management.

Using the tool, we'll migrate your old policies to the new service repository created by the Ranger Service. Policies that don't exist in the new service will be created with the original policy name, appended with an `epoch` timestamp value.  Policy names are 'NOT' the key for these entries.  It is the policy 'resources'.

This tool uses public Ranger REST API's to migrate policies and is intended for the initial conversion.  Repeated executions may yield unexpected results.

The tool will NOT remove policies when using mode (upsert) `-m u`.  Policies that are in the "to" `-t` service that are NOT in the "from" `-f` service will NOT be removed or adjusted. 

### Usage
```
Usage: ranger_policy_migration.py [options]

Options:
  -h, --help            show this help message and exit
  -l LOG, --log=LOG     Activity Log

  Connection:
    URL, User and password information needed to connect to Ranger

    -u HOST_URL, --ranger-host-url=HOST_URL
                        Ranger Host Base URL(required)
    -n USER, --username=USER
                        Username(required)
    -p PASSWORD, --password=PASSWORD
                        Password(required)

  Repo Details:
    Repositories details for migration

    -f REPO_FROM, --from-repo=REPO_FROM
                        Ranger Repository "from" Name(required)
    -t REPO_TO, --to-repo=REPO_TO
                        Ranger Repository "to" Name(required)
    -m METHOD, --migration-method=METHOD
                        Migration Method: m(move) or u(upsert). u is default

  Migration Control:
    Optional, define include/exclude policy "id's" to handle

    -e EXCLUDE_POLICIES, --exclude-policies=EXCLUDE_POLICIES
                        Comma separated list of policy id's to exclude
    -i INCLUDE_POLICIES, --include-policies=INCLUDE_POLICIES
                        Comma separated list of policy id's to include
```

### Examples

#### Upsert policies from repo 'hdp_hdfs' to repo 'cm_hdfs' (Recommended)

The `-t` and `-f` repositories must be of the same type.

```
./ranger_policy_migration.py -u http://my_ranger.server.org:6080 -n admin -p admin -f hdp_hdfs -t cm_hdfs
```
This will match and update policies in the 'to' repo with ones in the 'from' repo.  Matching is done based on the *Resource Path*.  When a matching policy isn't found, a new one will be created. 

#### Move Policies between Repositories with the `-m` option

The `-t` and `-f` repositories must be of the same type.

```
./ranger_policy_migration.py -u http://my_ranger.server.org:6080 -n admin -p admin -f hdp_hdfs -t cm_hdfs -m m
```

#### Restrict / Include Policies between Repositories with the `-i` and `-e` option

The `-t` and `-f` repositories must be of the same type.

The `-i` option is the Policy Id in the *from* repository.  Only matching policies will be considered.

```
./ranger_policy_migration.py -u http://my_ranger.server.org:6080 -n admin -p admin -f hdp_hdfs -t cm_hdfs -i 207
```

The `-e` option is the Policy Id in the *from* repository.  Only non-matching policies will be considered.

```
./ranger_policy_migration.py -u http://my_ranger.server.org:6080 -n admin -p admin -f hdp_hdfs -t cm_hdfs -e 210
```

These two options can be used together.

### Run Process

Gather the matching *from* and *to* repositories for each service type and run this tool for each set.  These can include, but not limited to: HDFS, Hive, YARN, HBase, Kafka, KNOX, Solr, and Atlas.  This process can also migration KMS policies in Ranger.  Key admins in Ranger use a difference login.  Be sure to make this adjustment in credentials when dealing with KMS policies. 