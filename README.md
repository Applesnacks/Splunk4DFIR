# Splunk4DFIR

## Description

Quickly spin up a splunk instance with Docker to browse through logs and tools output during your investigations. 

This is a simple quality of life improvement project built upon the amazing work below:

- https://github.com/whikernel/evtx2splunk/tree/main
- https://github.com/splunk/docker-splunk
- https://github.com/SigmaHQ

## Motivation

SOC to DFIR is getting a natural career path and considering the current market shares of Splunk and Crowdstrike, familiarity with SPL(-like) query languages is getting widespread within the DFIR community.

In DFIR it is common to juggle between different VMs and operating systems to be able to use all your favourite tools. Using Docker is convenient for portability.

When on the field you may not have access  nor the time to transfer case artifact to a private cloud hosted lab and using a public cloud may be out of the question for coutless regulatory reasons, this project enables you to spin up in no time on whatever workstation you have in your hands an environment to demonstrate your SPL-fu and save the day.

  
## HOW-TO

Drop your files under the appropriate folder in `artifacts/` then build and run the container.

```
sudo docker build -t splunk4dfir .
sudo docker run --name splunk4dfir -e SPLUNK_START_ARGS=--accept-license -e SPLUNK_PASSWORD=changeme -p 8000:8000 -p 8089:8089 -v ./artifacts:/mnt/artifacts splunk4dfir:latest start
```

goto: http://127.0.0.1:8000/en-US/app/Splunk4DFIR/search

## Supported inputs

- `artifacts/json/`: drop there arbitrary json files
- `artifacts/csv/`: drop there arbitrary csv files
- `artifacts/cloudtrail/`: drop there exported cloudtrail logs
- `artifacts/evtx/`: drop there windows logs evtx files
- `artifacts/zeek/`: drop there your json zeek files
- `artifacts/suricata/`: drop there your eve.json suricata file

## Ingest evtx as json

Once splunk is up and running you can trigger the evtx logs ingestion with: 

```
sudo docker exec -it splunk4dfir sudo /opt/splunk/etc/apps/Splunk4DFIR/bin/ingest_evtx.sh
```

## Sigma Rules support

you can import sigma rules as savedsearches using the command below

```
sudo docker build -t sigma-cli sigma/
sudo docker run -it --name sigma-cli --rm -v ./Splunk4DFIR/default:/mnt/output -v ./sigma/rules/:/mnt/rules -v ./sigma/pipelines:/mnt/pipelines sigma-cli:latest pipenv run sigma convert -t splunk -p /mnt/pipelines/evtx2splunk.yml /mnt/rules/sigma/rules/windows/ -s  -o /mnt/output/savedsearches.conf
```

## Pcap to Zeek

Drop you pcap file under `artifacts/pcap/`, then build and run the zeek container to generate zeek json output files. 

```
sudo docker build -t zeek zeek/
sudo docker run -it -v ./artifacts:/mnt/artifacts --name zeek --rm zeek /opt/zeek/bin/zeek -r /mnt/artifacts/pcap/packetcapture.pcapng LogAscii::use_json=T Log::default_logdir=/mnt/artifacts/zeek/
```

## Pcap to suricata alerts

Drop you pcap file under `artifacts/pcap/`, then build and run the suricata container to generate the eve.json file. 

```
sudo docker build -t suricata suricata/
sudo docker run -it -v ./artifacts:/mnt/artifacts --name suricata --rm suricata suricata -S /var/lib/suricata/rules/suricata.rules -r /mnt/artifacts/pcap/packetcapture.pcapng -l /mnt/artifacts/suricata
```




