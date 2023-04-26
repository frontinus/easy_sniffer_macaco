# Easy Sniffer
### Network tool for A/D CTF

This tool allow you to automatically start tcpdump session on remote host and save capture files (.pcap) on local machine (inspired by [remote_sniffer.py](https://github.com/Shotokhan/kanku-sho/blob/master/src/remote_sniffer.py))

Thanks to **rsync** it is possibile to transfer quickly files from remote host to local machine

Dependencies:
- python3
- coloredlogs (from pip)
- tcpdump (on remote host)
- rsync (both on local machine and remote host)
- inotify-tools

*It's suggested to use Linux or WSL 2.0*

### Basic config
All configuration must be done in **_config.json_**
Example:
```json
{
    "connection_info":{
        "vulnbox_ip": "IP_YOUR_VIRTUAL_MACHINE",
        "vulnbox_port" : 22,
        "user" : "root",
        "ssh_key" : "YOUR_SSH_KEY"

    },
    "tcpdump_info": {
        "interface" : "any",
        "remote_pcap_folder" : "",
        "local_pcap_folder" : "",
        "max_packets" : 500,
        "max_size" : 1,
        "packet_name" : "remote_capture",
        "sleep_time" : 15,
        "script_name" : "rename_pcap"
    },
    "verbose": true
}
```
*Connection Info*: information regarding remote connection to the vulnerable virtual machine
- **vulnbox_ip**: ip address of the vulnerable virtual machine
- **vulnbox_port**: ssh port of the vulnerable virtual machine (default = 22)
- **user**: username of the virtual machine (default = root)
- **ssh_key**: ssh key used to access remote machine with ssh without using the password

*tcpdump info: information regarding packet capture generation and transport to local machine*
- **interface**: network interface on which tcpdump listen for traffic
- **remote_pcap_folder**: folder (on the virtual machine) where to save generated pcap
- **local_pcap_folder**: folder (on the locale machine) where to save generated pcap
- **max_packets**: maximum number of packets that tcpdump generates. After the N-th packet it will override starting from the first one.
- **max_size**: maximum size of each packet in MByte.
- **packet_name**: prefix name of the generated packets. They will be named like "packet_name000", "packet_name001" and so on.
- **sleep_time**: time (in seconds) to wait between one call to rsync and another to download packet captures to the local machine
- **script_name**: script executed after the generation of the packet capture from tcpdump. This is needed because tcpdump doesn't automatically add the .pcap extension but Caronte needs it to work properly. If you set *packet_name* to be "packet_name.pcap" then tcpdump will generate packets named: "packet_name.pcap000", which is not good for Caronte.

*Global*
- **verbose**: set logging level to DEBUG if true or INFO if falze

### SSH Key Generation
You must first generate a SSH key to be able to execute remote commands with ssh

To do, follow:
```bash
    ssh-keygen -f key_name
    ssh-copy-id -i key_name user@vulnbox_ip 
```


### Attention
**Make sure**:
- **rename_pcap** is present in same the same folder of **_remote_pcap_folder_** (the script automatically upload this file, but double check anyway)
- **tcpdump** is up to date on remote host
- to execute `aa-complain /bin/tcpdump` on remote host (apt-get install apparmor-utils). This is needed to make `-z postcommand` of tcpdump run properly (error such as permission denied could be issued and packets are not renamed in packetnameXXX.pcap, but remains packetname.pcapXXX - not good for Caronte)
- **rsync** is installed both on local and remote machine

For more information on point (3), look at:

https://ubuntuforums.org/showthread.php?t=1501339

https://answers.launchpad.net/ubuntu/+source/tcpdump/+question/168402

If you want something easier, you could create and run a script `auto_rename` using _inotify_ on the remote host which rename packets generated by tcpdump when the latter close the file descriptor.
Or eventually you could use -G instead of -C and use the "timestamp string template" of tcpdump, like `name_%(H)_%(M).pcap`



### Caronte integration
Thanks to [@eciavatta](https://github.com/eciavatta) it is pretty easy to integrate Caronte. Just use `./feedCaronte .` inside the **local pcap folder** and keep it running.
In order to use it, you need to install `inotify-tools` on the local machine

### TODO
- [ ] Check if tcpdump is already running on remote host
- [ ] Add better timing for rsync to be called
- [x] Add integration with Caronte and ~~Flower~~ (automatic pcap uploader)
- [ ] Add basic web interface for easy settings management and for basic pcap analyzing (similiar to Flower)
- [ ] Dockerize when finished





