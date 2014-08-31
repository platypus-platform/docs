Platypus Platform
=================

A collection of agents that work against common data stores to get software
deployed into clusters of computers.

_This is a work-in-progress prototype._

Getting Started
---------------

Make sure you have a working go build environment, with `$GOPATH` set etc.
Refer to go documentation.

Then, build everything:

    for x in pp-preparer pp-iptables pp-setlive; do
      go get github.com/platypus-platform/$x
    done

Run consul to serve as an intent store:

    consul agent -server -bootstrap-expect 1 -data-dir /tmp/consul

Set up some test data in the intent store:

    cd $GOPATH/src/github.com/platypus-platform/pp-store
    ruby20 test_data.rb

Run through a deployment. You will need a test artifact to deploy, of which one
is not currently provided (TODO: fix that). These can be run in any order,
they will exit gracefully if a precondition isn't met yet (for instance, you
can't set live an artifact until it has been prepared).

    > pp-preparer -repo=file:///tmp/local-repo/
    INFO  [2014-08-31 02:42:58.076] Polling intent store
    INFO  [2014-08-31 02:42:58.077] Checking spec for slug
    INFO  [2014-08-31 02:42:58.078] Does not exist: /tmp/slug/installs/slug_e928e5ad8814441e7c503d7f6c9e55d72584c006
    WARN  [2014-08-31 02:42:58.078] TODO: Fetching artifact
    INFO  [2014-08-31 02:42:58.078] Extracting /tmp/local-repo/slug/slug_e928e5ad8814441e7c503d7f6c9e55d72584c006.tar.gz to /tmp/preparer458731992
    INFO  [2014-08-31 02:42:58.158] Moving /tmp/preparer458731992 to /tmp/slug/installs/slug_e928e5ad8814441e7c503d7f6c9e55d72584c006
    INFO  [2014-08-31 02:42:58.159] Does not exist: /tmp/slug/installs/slug_56d459e7c581b913ad5afa627064d62aea2cfac3
    WARN  [2014-08-31 02:42:58.159] TODO: Fetching artifact
    INFO  [2014-08-31 02:42:58.159] Extracting /tmp/local-repo/slug/slug_56d459e7c581b913ad5afa627064d62aea2cfac3.tar.gz to /tmp/preparer546603863
    INFO  [2014-08-31 02:42:58.233] Moving /tmp/preparer546603863 to /tmp/slug/installs/slug_56d459e7c581b913ad5afa627064d62aea2cfac3
    > sudo pp-iptables -cmd=/data/bin/port_authority -config-dir=/data/etc/port_authority.d
    INFO  [2014-08-31 02:37:22.551] Polling intent store
    INFO  [2014-08-31 02:37:22.553] Checking spec for slug
    INFO  [2014-08-31 02:37:22.554] refreshing portauthority
    > sudo pp-setlive -config=/etc/servicebuilder.d -runit=/var/service -runit-stage=/var/service-stage/
    INFO  [2014-08-31 02:38:04.9] Polling intent store
    INFO  [2014-08-31 02:38:04.902] Checking spec for slug
    INFO  [2014-08-31 02:38:04.903] slug: Stopping
    INFO  [2014-08-31 02:38:05.326] slug: Configuring service builder
    INFO  [2014-08-31 02:38:05.362] slug: Symlinking
    INFO  [2014-08-31 02:38:05.362] slug: Starting

Development
-----------

The default configuration for each agent should enable you to run them locally,
even if you don't have all the right dependencies.  See individual READMEs for
specific protips.

Principles
----------

* Users change the system by expressing _intent_ in the intent store. They do
  not direct how that change comes about.
* Agents inspect the intent store and converge the system towards it.
  Therefore, the system is eventually consistent. It is a _distributed_ system,
  after all. It could not be any other way.
* Agents must be able to run outside a datacenter, so they can be developed
  easily.
* Agents must not know about each other. For instance, `pp-setlive` doesn't
  care how artifacts are put in place, it just cares whether they are there.
  Agents observe their slice of the world and converge it. They are _not_ told
  what the world looks like by an external process.
* Since agents always try to converge, restarting an agent is always safe. They
  are idempotent.
* An agent that cannot converge should keep trying. It will be inferred by
  external monitors (not done yet).
* Agents should do one thing well. Like unix.
