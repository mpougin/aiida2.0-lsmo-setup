# LSMO AiiDA project setup

Welcome to the wild world of setting up the aiida-lsmo plugin! Hold onto your keyboards, because we're about to embark on the epic journey of 'How many LSMO-group members does it take to install a plugin?' Spoiler: Just one... that's you! 🚀

Note: These instructions have been tested for Ubuntu 22.04.3 LTS

## Install Python and services

```
sudo apt install git python3-dev python3-pip postgresql postgresql-server-dev-all postgresql-client rabbitmq-server
```

Check the Python version with 

```
python3 --version
```

Start the services (you might have to do this again after reboot):

```
sudo service postgresql start
sudo service rabbitmq-server start
```

## Setting up the Python environment

First we're going to install `aiida-project` with `pipx`, which allows you to globally install Python package in separated environments.
If you haven't installed `pipx`, here are the installation instructions:

https://pypa.github.io/pipx/installation/

Once `pipx` is installed, you can install `aiida-project` via:

```
pipx install aiida-project
```

Then initialise `aiida-project` via:

```
aiida-project init
```

And finally create the Python environment via:

```
aiida-project create aiida
```

and activate the project environment

```
cda aiida
```

This will also immediately change your directory to the `aiida` project one.
Update `pip` to the latest version 

```
pip install --upgrade pip
```

Install the AiiDA v2.X support branch for the `aiida-lsmo` package:

```
pip install git+https://github.com/mpougin/aiida-lsmo.git@fix/update-aiida2
```

```
git clone -b fix/update-aiida2 --single-branch https://github.com/mpougin/aiida-lsmo.git
```

## Set up an AiiDA profile

### Creating a database

```
sudo -u postgres createuser lsmo 2>/dev/null
sudo -u postgres psql -U postgres -c "ALTER USER lsmo WITH PASSWORD 'database';" 2>/dev/null
```

```
sudo -u postgres createdb -O lsmo aiida 2>/dev/null
```

### Set up the profile

Now it's time to set up the profile!
You can do so in several ways, but most convenient is probable to create a YAML file called `lsmo.yaml` and put in the following content, replacing the `<INSERT YOUR HOME DIRECTORY HERE!!!!>` with your home directory:

```
profile: lsmo
email: guiseppi.verdi@epfl.ch
first_name: Guiseppi
last_name: Verdi
institution: Milan-Conservatory
db_name: aiida
db_username: lsmo
db_password: database
repository: <INSERT YOUR HOME DIRECTORY HERE!!!!>/project/aiida/repository/lsmo
```

> IMPORTANT: This will only work if you actually replace `<INSERT YOUR HOME DIRECTORY HERE!!!!>` with your home directory.

You can then set up the profile using:

```
verdi setup -n --config lsmo.yaml
```

### RabbitMQ incompatibility

When you now run `verdi status`, it's very likely you'll still get a warning related to `rabbitmq`:

```
$ verdi status
 ✔ version:     AiiDA v2.4.0
 ✔ config:      /home/lsmo/project/aiida/.aiida
 ✔ profile:     lsmo
 ✔ storage:     Storage for 'lsmo' [open] @ postgresql://lsmo:***@localhost:5432/aiida / DiskObjectStoreRepository: 3979d1b4ae684b1490044b11aa0eafae | /home/lsmo/project/aiida/repository/lsmo/container
Warning: RabbitMQ v3.10.7 is not supported and will cause unexpected problems!
Warning: It can cause long-running workflows to crash and jobs to be submitted multiple times.
Warning: See https://github.com/aiidateam/aiida-core/wiki/RabbitMQ-version-to-use for details.
 ⏺ rabbitmq:    Incompatible RabbitMQ version detected! Connected to RabbitMQ v3.10.7 as amqp://guest:guest@127.0.0.1:5672?heartbeat=600
 ⏺ daemon:      The daemon is not running.
```

For more context, you can check the [following wiki post on the `aiida-core` repository](https://github.com/aiidateam/aiida-core/wiki/RabbitMQ-version-to-use), but the gist of it is that we have to reconfigure `rabbitmq` a bit for longer workflows.
Edit the `/etc/rabbitmq/rabbitmq.conf` file with sudo, e.g. with `vim`:

```
sudo vim /etc/rabbitmq/rabbitmq.conf 
```

and add in the following content:

```
# 1000 hours in milliseconds (increase if you expect your workflows to run longer)
consumer_timeout = 3600000000
```

Save the file, and let AiiDA know you've configured `rabbitmq` properly:

```
verdi config set warnings.rabbitmq_version false
```

You should not be all set!
Check it with `verdi status`:

```
$ verdi status
 ✔ version:     AiiDA v2.4.0
 ✔ config:      /home/lsmo/project/aiida/.aiida
 ✔ profile:     lsmo
 ✔ storage:     Storage for 'lsmo' [open] @ postgresql://lsmo:***@localhost:5432/aiida / DiskObjectStoreRepository: 3979d1b4ae684b1490044b11aa0eafae | /home/lsmo/project/aiida/repository/lsmo/container
 ✔ rabbitmq:    Connected to RabbitMQ v3.10.7 as amqp://guest:guest@127.0.0.1:5672?heartbeat=600
 ⏺ daemon:      The daemon is not running.
```
## Set up Computer and Codes

General instructions for setting up a (remote) computer resource, setting up a code on this computer and submitting calculations through AiiDA can be found on [the `Aiida-HowTo` manual](https://aiida.readthedocs.io/projects/aiida-core/en/latest/howto/run_codes.html).
I summarise the important steps here to use the common AiiDA plugins on the servers used by the LSMO-group.

### Set up SSH connections

AiiDA communicates with remote computers via the SSH protocol. To set up an SSH connection for AiiDA you first need to generate an SSH key. You can find more information on [the `AiiDA-HowTo` manual](https://aiida.readthedocs.io/projects/aiida-core/en/latest/howto/ssh.html).

To connect to the CSCS-clusters (eiger and daint) you need to use the `ProxyJump` or `ProxyCommand` feature of SSH using the `ela.cscs` proxy server.

### Computer setup

The configuration of computers happens in two steps: setting up the public metadata associated with the Computer in AiiDA provenance graphs, and configuring private connection details. 
I collected the `setup.yaml` files for the computers used by the lsmo-team in the `/computer` folder.

> IMPORTANT: When you copy the `.yaml` files make sure to change your username!!!

To create a new computer you can use the information provided in the configurations files:

```
verdi computer setup --config computer-setup.yml
```

### Computer connection configuration

The second step configures private connection details. Here, you can use and `configure.yaml` files in the `/computer` folder:

```
verdi computer configure core.ssh computer --config computer-configure.yaml
```

After the setup and configuration have been completed, let’s check that everything is working properly:

```
verdi computer test COMPUTERNAME
```

After running the test, if everything's working fine, it's time for a victory dance in the office! 💃🕺 (We will take a picture for your graduation.) Nearly there!!!!

### Create a code

Last step, before you can finally run a calculation, you need to define a "code". This will tell AiiDA what code the calculation should execute and how it should be executed. Again you don't have to worry, I provided the necessary information (modules, paths,...) for the commonly used codes for you, check the `/codes` folder. You can then set up the code via the configuration files:

```
verdi code create core.code.installed --config code.yaml
```


**If you've made it this far without throwing your computer out the window, congratulations! You're officially awesome. 🚀🎉 Remember to hydrate, stretch, and give yourself a well-deserved high-five!**