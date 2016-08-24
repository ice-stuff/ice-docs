# Writing experiments

An experiment is a Python file that exposes, in its main namespace, two types
of functions: _tasks_ and _runners_. iCE provides with some annotations to let
user define the type of a function.

The experiment files call the [Fabric](http://www.fabfile.org/) API. The Fabric
API can enable experiment functions to:

* Transfer files between instance and host
* Execute local or remote shell commands
* _For runners_ Run tasks

iCE from Fabric's perspective is just a tool to facilitate the execution of
Fabric tasks for cloud experiments. Hence, the reader is encouraged to read
through the excellent [Fabric docs](http://docs.fabfile.org) for more
information on how to use the Fabric APIs.

## Task

A task is a function that runs for each host (instance) of the experiment.  It
receives the list of hostnames as the first argument and can have additional,
arbitrary, arguments.

Following example shows how to print the hostname of the instance the task will
run into:

```python
import fabric.api as fab
import ice

@ice.Task
def print_hostname(hosts, prefix=''):
    print '%s%s' % (prefix, fab.env.host_string)
```

It can receive an extra argument, `prefix`, which by default will be empty.
iCE provides the ability to set values to extra arguments or keyword arguments
through its experiment running API (see bellow).

The following example demonstrates some more useful Fabric operations. The aim
is to change the listening port of an Apache2 server, and restart the server:

```python
import StringIO
import fabric.api as fab
import ice

@ice.Task
def change_apache_port(hosts, port='8080'):
    # Download Apache configuration file
    old_file = StringIO.StringIO()
    fab.get(remote_path='/etc/apache2/ports.conf', local_path=old_file)

    # Process
    new_contents = old_file.getvalue().replace(
        'Listen 80', 'Listen %d' % int(port)
    )
    new_file = StringIO.StringIO(new_contents)

    # Replace file
    fab.put(local_path=new_file, remote_path='/etc/apache2/ports.conf')

    # Restart Apache
    fab.run('/etc/init.d/apache2 restart')
```

### Parallel tasks

The user can enable parallel execution of a task, between different instances.
To do so use the `@ice.ParallelTask` annotation instead.

## Runner

A runner is an iCE concept that can act as an orchestrator between different
iCE/Fabric tasks. They receive the list of hostnames (same as tasks). However
they don't run per-instance (as tasks do).

The following example uses the `execute` method of fabric to run `stop_server`
in a randomly selected subset of the instances:

```python
import random
import fabric.api as fab
import ice

@ice.Task
def stop_server(hosts):
    # ....
    pass

@ice.Runner
def run(hosts):
    failing_hosts = []
    for host in hosts:
        if bool(random.getrandbits(1)):
            failing_hosts.append(host)
    fab.execute(stop_server, hosts=failing_hosts)
```

iCE can pass additional arguments to runners, similarly to the way its done for
tasks. Finally if a user runs an iCE experiment without specifying the
task/runner name to execute, it will look for a `run` runner.

### Parallel runners

The user can enable parallel execution of every Fabric operation made within a
parallel runner. To do so use the `@ice.ParallelRunner` annotation instead of
the `Runner`.
