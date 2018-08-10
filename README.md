# Mattermost Reference Galera Cluster Setup

This is a reference implementation for testing Mattermost against a Galera Cluster. While not for production use, this provides an example of how a Galera cluster is set up behind an HAProxy load balancer. This will be modified and updated to reflect improvements and optimizations to get the best performance.

## How to use

### 0. Install dependencies

To run this locally you'll have to install [Virtualbox](https://www.virtualbox.org/wiki/Downloads) and [Vagrant](https://www.vagrantup.com/docs/installation/).

### 1. Configure the Vagrant File

Set the following configuration values at the top of the `Vagrantfile`:

| Config value | Description | Default |
| --- | --- | --- |
| PUBLIC_IP | The IP address on your network for the | `192.168.1.104` |
| BRIDGE    | The host network interface that connects to the Internet| `eno1` |
| MATTERMOST_PASSWORD | The password for the Mattermost database user | `really_secure_password` |

### 2. Run Vagrant

Run this from the command line:

```
$ vagrant up
```

### 3. Update your Mattermost database configuration

Use the following value for the `DataSource` setting in your `config.json` file to let Mattermost connect to your Galera cluster:

```
"DriverName": "mysql",
"DataSource": "mysql://mmuser:really_secure_password@192.168.1.104:5432/mattermost?sslmode=disable\u0026connect_timeout=10",
```

If you changed the `PUBLIC_IP` or `MATTERMOST_PASSWORD` configuration values in step 1 make sure to replace them in those values.

**Note:** You do not need to set up data source replicas to use Galera with Mattermost.

### Finally...

Restart Mattermost

```
$ sudo service mattermost restart
```

*Tip:* Make sure to free up system resources when not using the Galera cluster by running `vagrant halt`

## Feedback appreciated!

If you see improvements or optimizations that can be made to this configuration please create an issue and let us know!