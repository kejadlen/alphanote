--- tags:
  - talks ---

# Twitter Tech Talk

(Some Twitter talk I attended many years ago.)

- ~55 engineers (want to get 150?)
- 19 teams
  - ad products
  - core infra
  - machine learning
  - mobile
  - sre/dre

## twitter streaming aggregator

- L7 reverse proxy for streaming
- allow streaming to scale horizontally
- c++11
- full reload w/out dropping connections

- no ssl, gzip internally (moar CPU for computing)

- single process, multithread
- async reactor model via libevent
- minimize buffering (scan for CRLF, don't parse until then, parse in place)

### testing

- { host: , hoses: , actions: } - run ssl, gzip, etc. matrix

- ~270k down, ~1.1M up, ~1.4M backend connections w/~60GB of RAM
- no GC

### hot restart

- previous process kept running - data kept in shared memory (stats, config,
  logging)
  - stop listening to connections
- all processes interact w/shared memory
- new process handles admin tasks
  - in control of old parents
- most bugs not bad enough that you want to actively kill old connections

- working on replacing current L7 proxy written in Java - not scaling as well
  due to GC
- GC hard on high-throughput network apps, instability and perf anomalies

- moving from hardware LB to L3/L4 software load balancing

### Q&A

- diversity?
- most diverse Twitter office (but still only single digits)

## MySQL

- own fork

- ~20M QPS (across all)
- >5k servers
- >50 master/slave replication clusters

- monitoring
  - monitoring: nagios, viz
  - config: puppet
  - automated tasks: ansible
  - HA: load balancers
  - tracking clusters: metadb

- metadb - replaces a wiki page w/a list of servers
  - database of databases
  - more scalable than a wiki page

- fork
  - lots of stuff I don't really care about, but...
  - audit logging
  - write user@host to binlogs
  - info about RBR events
  - user stats
  - table stats
  - more InnoDB status variables
  - bored of writing these down, so stopping now

- the future
  - HA - MHA, Galera
  - floating IPs via route injection (replace LBs and DNS failovers)

## loglens

- index, search, visualize services logs

- challenges
  - write heavy (100s of MB/s at start w/few services)
  - text indices are expensive relative to the generation cost
  - infrequent queries
  - value of data falls steeply w/time
  - horizontal scalability not good answer due to resource utilization costs
    - want fixed cost budget
  - hardware for indexing/query vs archival

- service tiers
  - near real time
    - indexed and available <2min (typically 40s)
    - retention: 6hr
  - on demand restore
    - indexes archived on HDFS for X period (default 7d)
    - wait time proportaional to time for copying data
  - on demand indexed
    - data archived w/out indexes
    - user specifies index, time range, index is build on demand
    - O(100MBps)
    - log storage is negligible compared to rest of data being stored

- stack
  - log handlers
  - scribe -> kafka
  - data loaders
  - archival/restore
  - elastic search / HDFS
  - kibana

- log handler
  - ease of onboarding
  - scribe directly to category (payload format requirements)
  - requires metadata - timestamp, service, machine, error level
    - going to add transaction ids, shard...

- scribe -> kafka
  - scribe: buffers a little, streams data to an endpoint (?)
    - intermediary for... fewer connections to kafka?
  - can drop data (which is ok)

- data loader (near real time)
  - kafka -> ES
  - stateless, runs on compute tier
  - manages index namespace
    - each service gets logical index
    - logical index has multiple physical indexes
    - physical indexes corresponds to time window (6hr by default)
  - want to drop things cheaply
  - split into multiple indexes

- data loader (on demand indexing)
  - kafka -> HDFS
  - kafka -> ES

- archival/restore service
  - keep indices around since they're expensive to create
  - removed after retention period (default 7d)
  - (logical_index_name, start_time, end_time) -> index!

- external monitor
  - important metrics
    - end to end latency
    - pipeline data loss
    - logical index size
    - logical index mssage rates
    - portal/es cluster health

- ES
  - improvements needed
    - replication/deployment
      - problems with replication when nodes go down (i.e., deployments)
      - no rolling restarts due to above issue (10min outage preferable)
      - if you take a node down, it tries to figure out other replicas,
        transactions still coming in during rolling upgrades, but when machine
        comes back up, it's got to catch up. for it to catch up, the lucene
        segments on both machines need to look the same. if they don't, it
        needs to copy. full data copy to re-build replica.  that's worse than
        taking service down for a few min.
    - plugin quality/support (upgrades)
  - easy to get started
  - do not want to index twice
  - most constrained, expensive resource? indexing
  - no reason to move off of ES

- kibana
  - using antiquated version
  - added index selection, expand (logical index, time range) -> set of
    physical indexes
  - figure out if they're going to stay with kibana
    - probably not good investment to upgrade or roll their own

- mostly interesting due to budget constraints (want fixed cost)

- Q: zipkin? (sp?)
- future plans: analyze exception stack?
- per data center
