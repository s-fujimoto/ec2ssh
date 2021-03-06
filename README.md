# sshec2
[![Build Status](https://travis-ci.org/s-fujimoto/sshec2.svg?branch=master)](https://travis-ci.org/s-fujimoto/sshec2)
[![codecov](https://codecov.io/gh/s-fujimoto/sshec2/branch/master/graph/badge.svg)](https://codecov.io/gh/s-fujimoto/sshec2)
[![Documentation Status](https://readthedocs.org/projects/sshec2/badge/?version=master)](http://sshec2.readthedocs.io/ja/master/)

SSH login utility for Amazon EC2 instance. Test only OSX.

![](https://raw.githubusercontent.com/s-fujimoto/sshec2/master/docs/sshec2.gif)

## Documentation

Japanese document page.

##### [http://sshec2.readthedocs.io/](http://sshec2.readthedocs.io/)

## Features
- ssh login 
- display instance list (only running status instances)
    - Name tag and instance ID
- search and filter instance by name
- access via specify bastion
- access via vpn connection
- access via SOCKS proxy
- specify key name
- specify username
- transfer file via scp
    - from EC2 to local
    - from local to EC2

## Usage
### Installation
Install from PyPI repository.

```
$ pip install sshec2
```

### Uninstallation
Uninstall from pip packages

```
$ pip uninstall sshec2
```

### Support python version
- 2.7
- 3.3
- 3.4
- 3.5
- 3.6

### Option
Command option. Option setting is available for Environment.

```
$ sshec2 --help
usage: sshec2 [-h] [-k KEY_PATH] [-u USERNAME] [--bastion] [-b BASTION_NAME]
              [-e BASTION_KEY_PATH] [-s BASTION_USERNAME] [-p PROFILE]
              [-v VIF] [-r REGION] [--scp-to-ec2] [--scp-from-ec2]
              [--source-path SOURCE_PATH] [--target-path TARGET_PATH]
              [--debug]

Simple argparse CLI

optional arguments:
  -h, --help            show this help message and exit
  -k KEY_PATH, --key-path KEY_PATH
                        Specify private key path
  -u USERNAME, --username USERNAME
                        Specify login user name
  --bastion             Enabled selecting bastion mode
  -b BASTION_NAME, --bastion-name BASTION_NAME
                        Specify bastion instance name
  -e BASTION_KEY_PATH, --bastion-key-path BASTION_KEY_PATH
                        Specify bastion private key path
  -s BASTION_USERNAME, --bastion-username BASTION_USERNAME
                        Specify bastion user name
  -p PROFILE, --profile PROFILE
                        Specify profile name for AWS credentials
  -v VIF, --vpn-interface VIF
                        Specify interface name for vpn
  -r REGION, --region REGION
                        Specify region name
  --scp-to-ec2          SCP from local to EC2 mode
  --scp-from-ec2        SCP from EC2 to local mode
  --source-path SOURCE_PATH
                        Specify source path
  --target-path TARGET_PATH
                        Specify target path
  --debug               logging debug mode
```

```-k, --key-path```  
Specify private key file path. 
* Default ```$HOME/.ssh/<<INSTANCE_KEY_PAIR_NAME>>.pem```
* ENVIRONMENT_KEY : ```KEY_PATH```

```-u, --username```  
Specify login username.
* Default ```ec2-user```
* ENVIRONMENT_KEY : ```EC2SSH_USERNAME```

```-p, --profile```  
Specify aws credential profile name.
aws credential file is ```$HOME/.aws/credential```
* ENVIRONMENT_KEY : ```EC2SSH_AWS_PROFILE```

```--bastion```
Select bastion mode.

```-b, --bastion-name```  
If ssh access via bastion, specify bastion instance name.
* ENVIRONMENT_KEY : ```EC2SSH_BASTION_NAME```

```-e, --bastion-key-path```  
If ssh access via bastion, specify bastion private key file path.
* Default ```$HOME/.ssh/<<BASTION_INSTANCE_KEY_PAIR_NAME>>.pem```
* ENVIRONMENT_KEY : ```EC2SSH_BASTION_KEY_PATH```

```-s, --bastion-username```  
If ssh access via bastion, specify bastion login username.
* Default ```ec2-user```
* ENVIRONMENT_KEY : ```EC2SSH_BASTION_USERNAME```

```-v, --vpn-interface```  
If ssh access via vpn connection, specify vpn interface name.
* ENVIRONMENT_KEY : ```EC2SSH_VPN_INTERFACE```

```-r, --region```
Region name of logging in EC2 instances.
ex. ap-northeast-1, us-east-1 etc
* ENVIRONMENT_KEY : ```EC2SSH_AWS_REGION```

```--scp-to-ec2```
SCP from local to EC2 mode

```--scp-from-ec2```
SCP from EC2 to local mode

```--src-path```
**SCP Mode option** Specify source path

```--dst-path```
**SCP Mode option** Specify target path

```--debug```
logging debug mode

### Usage
Only exec command ```sshec2```

#### Use not default profile (AWS credential)
sshec2 using default profile is ```default```.  
sshec2 is able to not default profile. sshec2 searches profile name in ```~/.aws/credentials```.   

For example...  
- profile name : profile1

```
$ sshec2 -p profile1
```

#### via bastion
Nested SSH login. 
Local -(public access)-> Bastion -(private aceess)-> Target

For example...
- Bastion instance name : bastion
- Bastion instance key pair path : ~/.ssh/bastion.pem
- Bastion instance user name : root

```
$ sshec2 -b bastion -e ~/.ssh/bastion.pem -s root 
```

#### via vpn connection
Access via vpn connection. Add target instance having public ip address rule to local routing table. Exec ```sudo route add <target_ip>/32 -interface <vpn_interface_name>```  
Local -(VPN routing)-> Bastion

For example...  
- VPN interface name : utun2

```
$ ifconfig
<snip>
utun2: flags=8051<UP,POINTOPOINT,RUNNING,MULTICAST> mtu 1500
	inet 192.168.0.100 --> 192.168.0.100 netmask 0xffffffff

$ sshec2 -v utun2
<select target instance number>
Input sudo password if required sudo password.
Password: <input sudo password>
```

#### SCP Mode
Transfer directory or file by SCP command. Exec (From local to EC2).  
```scp -r <source_path> <target_ip>:<destination_path>```  
Support to transfer via bastion.

For example transfer to EC2 from local.

- Source File : README.md
- Destination Directory : /tmp/

```
$ sshec2 --scp-to --src README.md --dst /tmp/
```