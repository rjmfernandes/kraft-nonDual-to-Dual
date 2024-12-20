# Kraft Controllers: From dedicated to dual 

- [Kraft Controllers: From dedicated to dual](#kraft-controllers-from-dedicated-to-dual)
  - [Disclaimer](#disclaimer)
  - [Start](#start)
  - [Change Controller to Dual](#change-controller-to-dual)
  - [Replicate Topics to Dual Controller](#replicate-topics-to-dual-controller)
  - [Now with just Dual Controller](#now-with-just-dual-controller)
  - [Cleanup](#cleanup)

We will try for experimental reasons to migrate a cluster based on kraft dedicated controllers to a single dual node one.

## Disclaimer

The code and/or instructions here available are **NOT** intended for production usage. 
It's only meant to serve as an example or reference and does not replace the need to follow actual and official documentation of referenced products.

## Start

```shell
cd run
cp ../start/compose.yml .
docker compose up -d
```

Let's create a topic:

```shell
kafka-topics --bootstrap-server localhost:9091 --topic test --create --partitions 3 --replication-factor 1
```

And we can list all topics:

```shell
kafka-topics --bootstrap-server localhost:9091 --list
```

## Change Controller to Dual

Now we can change our controller to dual:

```shell
rm compose.yml
cp ../controller-to-dual/compose.yml .
docker compose up -d
```

We run again the list of topics:

```shell
kafka-topics --bootstrap-server localhost:9091 --list
```

## Replicate Topics to Dual Controller

Now if we run:

```shell
kafka-topics --bootstrap-server localhost:9091 --describe --topic test
```

We get something like:

```
Topic: test	TopicId: Wrdu19lVS1KF4_eOYIMFlQ	PartitionCount: 3	ReplicationFactor: 1	  Configs: min.insync.replicas=1
	Topic: test	Partition: 0	Leader: 2	Replicas: 2	Isr: 2	Offline: Elr: N/A	ownElr: N/A
	Topic: test	Partition: 1	Leader: 2	Replicas: 2	Isr: 2	Offline: Elr: N/A	ownElr: N/A
	Topic: test	Partition: 2	Leader: 2	Replicas: 2	Isr: 2	Offline: Elr: N/A	ownElr: N/A
```

We need to make sure everything gets replicated to our dual controller.

So we execute:

```shell
kafka-reassign-partitions --bootstrap-server localhost:9091 --reassignment-json-file ../reassignment.json --execute
```

We can check again:

```shell
kafka-topics --bootstrap-server localhost:9091 --describe --topic test
```

## Now with just Dual Controller

Now we do:

```shell
docker compose down kafka-1
rm compose.yml
cp ../only-dual/compose.yml .
docker compose restart
```

And we check:

```shell
kafka-topics --bootstrap-server localhost:9092 --describe --topic test
```

So we update:

```shell
kafka-reassign-partitions --bootstrap-server localhost:9092 --reassignment-json-file ../reassignmentBack.json --execute
```

And check:

```shell
kafka-topics --bootstrap-server localhost:9092 --describe --topic test
```

## Cleanup

```shell
docker compose down -v
rm -fr *
cd ..
```