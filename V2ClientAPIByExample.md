# v2 iCE client API by example

This document is a work in progress description (by example) of the 
[v2](https://github.com/glestaris/iCE/tree/v2) iCE programmatic API. It has 
nothing to do with the v1 programmatic API which is currently in use. Also, 
given its WIP status, it is subject of frequent changes.

## Registry

Here is how to create a registry client:

```python
import ice.registry.client as registry_client

registry_cfg = registry_client.CfgRegistryClient(
    host='localhost', 
    port=5000
)
logger = logging.getLogger('ice.registry')
registry = registry_client.RegistryClient(registry_cfg, logger)
```

### Session

#### Create session

```python
from ice import entities

session = entities.Session()
registry.submit_session(session)
```

#### Destroy session

```python
registry.destroy_session(session.id)
```

### Instances

#### Get instances

```python
instances = registry.get_instances_list(session.id)
```

#### Delete instances

```python
for inst in instances:
    registry.get_instance(inst.id)
```

### User data

Generates the user data that will make a VM join the registry:

```python
user_data = registry.get_user_data(session.id)
```

## EC2

Here is how to creare an EC2 client:

```python
from ice import ec2_client

cloud_cfg = ec2_client.CfgEC2CloudAuth(
    ec2_url='...',
    aws_access_key='...',
    aws_secret_key='...'
)
logger = logging.getLogger('ice.ec2')
cloud = ec2_client.EC2Client(cloud_config, logger)
```

### Create EC2 VMs

```python
vm_spec = ec2_client.CfgEC2VMSpec(
    'ami-...',
    'key-...',
    user_data=user_data # see above
)
reservation = cloud.create(10, vm_spec)
```

### Stop EC2 VMs

```python
instance_ids = [inst.id for inst in reservation.instances]
cloud.destroy(instance_ids)
```

## Loading and running experiments

### Loading from a file

```python
from ice import experiment_loader

logger = logging.getLogger('ice.experiment_loader')
loader = experiment_loader.ExperimentLoader(logger)
exp_module = loader.load('experiments/test.py')
```

### Creating an object

User may use the `ice.experiment_loader` to load an experiment to acquire a
`exp_module` instance. Alternatvely the user can import a module themselves:

```python
from ice import experiment

logger = logging.getLogger('ice.experiment')
exp = experiment.Experiment(exp_module, logger)
```

### Running 

In the following example `arg1` and `arg2` are additional keyword arguments
passed to the runner/task to run:

```python
hosts = [inst.get_host_string() for inst in instances] # see instances above
return_value = exp.run(
    hosts,                  # List of host name strings
    '/path/to/ssh-key',     # SSH key file pah
    'run',                  # task/runner name
    arg1=12, arg2='test'    # additional arguments
)
```