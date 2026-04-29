IPAM Performance Test
=====

Motivation
-----
We wanted to test the behavior of the IPAM controller under various scenarios
by mocking and monitoring the edges that the controller interacts with. This has the following goals:

- Save time on testing
- Simulate various behaviors cheaply
- Observe and model the ideal behavior of the IPAM controller code

Currently the test runs through the 4 different IPAM controller modes for cases where the Kubernetes API QPS is
equal to, or significantly less than, the number of nodes being added to observe and quantify behavior.

How to run
-------

```shell
# In kubernetes root path
cd test/integration/ipamperf
./test-performance.sh
```

The runner scripts support a few different options:

```shell
./test-performance.sh -h
usage: ./test-performance.sh [-h] [-d] [-r <pattern>] [-o <filename>]
usage: ./test-performance.sh <options>
 -h display this help message
 -d enable debug logs in tests
 -r <pattern> regex pattern to match for tests
 -o <filename> file to write JSON-formatted results to
 -p <id> enable CPU and memory profiles, output written to mem-<id>.out and cpu-<id>.out
 -c enable custom test configuration
 -a <name> allocator name, one of RangeAllocator, CloudAllocator, IPAMFromCluster, IPAMFromCloud
 -k <num> API server QPS for allocator
 -n <num> number of nodes to simulate
 -m <num> API server QPS for node creation
 -l <num> GCE cloud endpoint QPS
```

The tests follow the pattern TestPerformance/{AllocatorType}-KubeQPS{X}-Nodes{Y}, where AllocatorType
is one of

- RangeAllocator
- IPAMFromCluster
- CloudAllocator
- IPAMFromCloud

and X represents the QPS configured for the Kubernetes API client, and Y is the number of nodes to create.

The -d flag sets the -v level for glog to 6, enabling nearly all of the debug logs in the code.

To run the test for CloudAllocator with 10 nodes, one can run:

```shell
./test-performance.sh -r /CloudAllocator.*Nodes10$
```

At the end of the test, JSON-formatted results for all the tests run are printed. Passing the -o option
also saves this JSON to a named file.

### Profiling the code
It's possible to get the CPU and memory profiles of code during test execution by using the ```-p``` option.
The CPU and memory profiles are generated in the same directory with the file names set to ```cpu-<id>.out```
and ```mem-<id>.out```, where ```<id>``` is the argument value. A typical pattern is to use the number
of nodes being simulated as the ID, or 'all' when running the full suite.

### Custom Test Configuration
It's also possible to run a custom test configuration by passing the -c option. With this option, it is
possible to specify the number of nodes to simulate and the API server QPS values for creation,
IPAM allocation, and cloud endpoint, along with the allocator name to run. The default values for the
QPS parameters are 30 for IPAM allocation, 100 for node creation, and 30 for the cloud endpoint, and the
default allocator is the RangeAllocator.

Code Organization
-----
The tests are defined in [ipam_test.go](ipam_test.go), using the t.Run() helper to control parallelism
because the master should only be started once. [cloud.go](cloud.go) contains the mock of the cloud server endpoint
and can be configured to behave differently as needed by the various modes. The tracking of node behavior and
creation of the test results data is in [results.go](results.go).
