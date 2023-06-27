# Moonriver

![](https://user-images.githubusercontent.com/83699173/201930807-6ea0c7cc-6fd0-43df-a833-bdd99a8f79f7.jpg)

### Guide-in-Moonriver-network-validator

In this guide you will find a detailed description of how to create a validator on the Moonriver network In this guide you will find a detailed description of how to create a validator on the Moonbeam network

### Official Links

[Website](https://moonbeam.network/) [Twitter](https://twitter.com/MoonbeamNetwork) [Discord](https://discord.gg/moonbeam)

### Explorer

[Explorer](https://telemetry.polkadot.io/#list/0x401a1f9dca3da46f5c4091016c8a2f26dcea05865116b286f60f668207d1474b)

### Install Node Guide

**Setting up variables**

First you need to download the binary file of the current version

```bash
wget https://github.com/PureStake/moonbeam/releases/download/v0.27.2/moonbeam
```

**To verify that you have downloaded the correct version, you can run moonbeam in your terminal, you should receive the following output:**

```bash
sha256sum
```

**you should receive the following output:**

```bash
b497f3e9f6f9c1e8591fbcbde5c18e24a5a5b4cdb98bc641ac7e643890a523fe
```

**Setup the Service**

**The following commands will set up everything regarding running the service.**

**1. Create a service account to run the service:**

```bash
adduser moonriver_service --system --no-create-home
```

**2. Create a directory to store the binary and data**

```bash
mkdir /var/lib/moonriver-data
```

**3. Move the binary built in the last section to the created folder**

```bash
mv ./moonbeam /var/lib/moonriver-data
```

**4. Make sure you set the ownership and permissions accordingly for the local directory that stores the chain data:**

```bash
sudo chown -R moonriver_service /var/lib/moonriver-data
```

**Create the Configuration File**

The next step is to create the systemd configuration file. If you are setting up a collator node, make sure to follow the code snippets for Collator. Note that you have to:

**- Replace YOUR-NODE-NAME in two different places**

**- Replace <50% RAM in MB> for 50% of the actual RAM your server has. For example, for 32 GB RAM, the value must be set to 16000. The minimum value is 2000, but it is below the recommended specs**

**- Double-check that the binary is in the proper path as described below (ExecStart)**

**- Double-check the base path if you've used a different directory**

**Create file**

```bash
nano /etc/systemd/system/moonriver.service
```

**Insert the configuration below into the created file and make the necessary corrections**

```bash
[Unit]
Description="Moonriver systemd service"
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=on-failure
RestartSec=10
User=moonriver_service
SyslogIdentifier=moonriver
SyslogFacility=local7
KillSignal=SIGHUP
ExecStart=/var/lib/moonriver-data/moonbeam \
     --validator \
     --port 30333 \
     --rpc-port 9933 \
     --ws-port 9944 \
     --execution wasm \
     --wasm-execution compiled \
     --trie-cache-size 0 \
     --db-cache <50% RAM in MB> \
     --base-path /var/lib/moonriver-data \
     --chain moonriver \
     --name "YOUR-NODE-NAME" \
     -- \
     --port 30334 \
     --rpc-port 9934 \
     --ws-port 9945 \
     --execution wasm \
     --name="YOUR-NODE-NAME (Embedded Relay)"

[Install]
WantedBy=multi-user.target
```

**Then go to the directory where the binary is located and grant permissions**

```bash
cd /var/lib/moonriver-data/moonbeam
```

```bash
chmod +x moonbeam
chown moonriver_service moonbeam
```

**Go back to original directory**

```bash
cd 
```

**Run the Service**

Almost there! Register and start the service by running:

```bash
systemctl enable moonriver.service
systemctl start moonriver.service
```

**And lastly, verify the service is running:**

```bash
systemctl status moonriver.service
```

**You can also check the logs by executing:**

```bash
journalctl -f -u moonriver.service
```

**Update the Client**

If you want to update your client, you can keep your existing chain data in tact, and only update the binary by following these steps:

**1. Stop the systemd service:**

```bash
sudo systemctl stop moonriver.service
```

**2. Remove the old binary file:**

```bash
rm  /var/lib/moonriver-data/moonbeam
```

**3. Get the latest version of Moonbeam from the** [**Moonbeam GitHub Release page**](https://github.com/PureStake/moonbeam/releases/)

**4. If you're using the release binary, update the version and run:**

```bash
wget https://github.com/PureStake/moonbeam/releases/download/<NEW VERSION TAG HERE>/moonbeam
```

**5. Move the binary to the data directory:**

```bash
mv ./moonbeam /var/lib/moonriver-data
```

**6. Then go to the directory where the binary is located and grant permissions**

```bash
cd /var/lib/moonriver-data/moonbeam
```

```bash
chmod +x moonbeam
chown moonriver_service moonbeam
```

**7. Go back to original directory**

```bash
cd 
```

**8. Register and start the service by running:**

```bash
systemctl enable moonriver.service 
systemctl start moonriver.service 
```

**9. You can also check the logs by executing:**

```bash
journalctl -f -u moonriver.service 
```

```bash
sudo systemctl stop moonriver.service
sudo rm /etc/systemd/system/moonriver.service
sudo systemctl disable moonriver.service
rm -rf /var/lib/moonriver-data
```
