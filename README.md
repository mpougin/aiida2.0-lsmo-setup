# LSMO AiiDA project setup

(These instructions have been tested for Ubuntu 22.04.3 LTS)

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
