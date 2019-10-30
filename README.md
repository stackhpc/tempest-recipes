# How to use

A number of different test configurations have been defined, to test different
aspects of the system.

### Bare metal fixed network

This configuration tests bare metal, accessed via a fixed network rather than a
floating IP.

Create a new verifier and ensure that it is configured correctly. In
production, use the config/production directory. For bare metal, we need to use
a fork of the tempest repo with some changes to support rate limiting
deployments.

```
(rally) $ rally verify create-verifier --name tempest-bare-metal-fixed-ip --type tempest --source https://github.com/VerneGlobal/tempest --version bare-metal
```

In candidate:
```
(rally) $ rally verify configure-verifier --reconfigure --extend config/alt1/bare-metal-fixed-network.conf
```

In production:
```
(rally) $ rally verify configure-verifier --reconfigure --extend config/production/bare-metal-fixed-network.conf
```

Alternatively, use an existing verifier.

```
(rally) $ rally verify use-verifier --id tempest-bare-metal-fixed-ip
```

For these tests we use a pre-create hook script that waits for sufficient bare
metal compute resources to become available before creating a server. This
script should be copied to /tmp/rally-node-count.sh:

```
cp tools/rally-node-count.sh /tmp/rally-node-count.sh
```

You will need to export some environment variables for this script:
```
export RALLY_NODE_COUNT_VENV=/path/to/virtualenv
export RALLY_NODE_COUNT_OPENRC=/path/to/openrc.sh
export RALLY_NODE_COUNT_RESOURCE_CLASS=Compute_1
```

We are using a list of tests from Refstack, with all non-compute tests skipped.
We use a concurrency of 1 to ensure that there is no contention for the bare
metal nodes.  Run all tests:

```
(rally) $ rally verify start --load-list test-lists/next-test-list.txt --skip-list blacklists/bare-metal-fixed-network --concurrency 1
```

Generate a report.

```
(rally) $ rally verify report --type html --to ~/rally-reports/$(date -d "today" +"%Y%m%d%H%M").html
```
