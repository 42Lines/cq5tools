A collection of scripts to allow one to administrate parts of
Adobe/Day Software's CQ5/WCM without having to use the GUI and thus
hopefully avoid RSI.  Especially useful if one is creating many
instances on many machines.

## Scripts

* change-cq5-passwords - change the admin password in the five (CQ5.4) or
the two (CQ5.5) places one needs to change it for it to be changed everywhere
* cq5-replication - display replication agent logs, test replication agent configuration, and create new replication agents all from the command line
* cq5-users - create an initial set of users

## change-cq5-passwords

Defaults are localhost, port 4502, existing password of `admin`, and version
5.5 (as opposed to 5.4)

    change-cq5-passwords --newpw s3cr3t

## cq5-replication

Assumes that one wants to create replication agents that replicate to an
instance that is on a port number one greater than the source instance.
For example, here's the default replication configuration:

    cq5-replication --command=report --pw=s3cr3t

    Name                 Blocked Paused Enabled Title                transportUri                             Description         
    flush                                       Dispatcher Flush     http://localhost:8000/dispatcher/invalidate.cache                     
    publish              false   false  true    Default Agent        http://localhost:4503/bin/receive?sling:authRequestLogin=1                     
    publish_reverse      false   false  true    Reverse Replication Agent http://localhost:4503/bin/receive?sling:authRequestLogin=1                     
    static                              false   Static Agent         static:///                   
Now let's add three replication agents; one in our cage and three in the cloud:

    cq5-replication --command=create --pw=s3cr3t --agents cage1 --fqdn cage.example.com
    Copying publish author agent to cage1
    cq5-replication --command=create --pw=s3cr3t --agents cloud1,cloud2 --fqdn cloud.example.com
    Copying publish author agent to cloud1
    Copying publish author agent to cloud2

    cq5-replication --command=report --pw=s3cr3t

    Name                 Blocked Paused Enabled Title                transportUri                             Description         
    cage1                false   false  true    cage1                http://cage1.cage.example.com:4503/bin/receive?sling:authRequestLogin=1 Replication agent for cage1
    cloud1               false   false  true    cloud1               http://cloud1.cloud.example.com:4503/bin/receive?sling:authRequestLogin=1 Replication agent for cloud1
    cloud2               false   false  true    cloud2               http://cloud2.cloud.example.com:4503/bin/receive?sling:authRequestLogin=1 Replication agent for cloud2
    flush                                       Dispatcher Flush     http://localhost:8000/dispatcher/invalidate.cache                     
    publish              false   false  true    Default Agent        http://localhost:4503/bin/receive?sling:authRequestLogin=1                     
    publish_reverse      false   false  true    Reverse Replication Agent http://localhost:4503/bin/receive?sling:authRequestLogin=1                     
    static                              false   Static Agent         static:///  

*Note* that the default `publish` agent uses a `transportUri` of `localhost:4503`.  If your source instance is running on another port, say `5502`, you'll need to change the default `publish` agent `transportUri` to `localhost:5503`.  You can do this easily with:

    cq5-replication --command=pubfix --port=5502 --pw=s3cr3t

Can also run tests for replication agents and display their logs:

    cq5-replication --command=test --agents=publish --pw=s3cr3t
    cq5-replication --command=log --agents=publish --pw=s3cr3t

## cq5-users

Setting up anything more than one or two users for an instance is time consuming in the GUI.  `cq5-users` allows one to create users quickly and add them to groups.

    cq5-users --command=create --pw=s3cr3t --installation FOO --trace

    Creating user jpage
    Adding group c/contributor to user jpage
    Adding group c/content-authors to user jpage
    Adding group w/workflow-users to user jpage
    Adding group w/workflow-editors to user jpage
    Creating user jbonham
    Adding group c/contributor to user jbonham
    Adding group c/content-authors to user jbonham
    Adding group w/workflow-users to user jbonham
    Adding group w/workflow-editors to user jbonham
    Adding group a/administrators to user jbonham

Note that the simple format in the script allows users to be a part of
more than one installation.  For example, John Bonham has the field
`FOOBAR` which will include him when `--installation` is FOO or BAR.
