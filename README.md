# Splunk4DFIR

## Description

`TODO`

## HOW-TO

```
sudo docker build -t splunk4dfir .
sudo docker run --name splunk4dfir -e SPLUNK_START_ARGS=--accept-license -e SPLUNK_PASSWORD=changeme -p 8000:8000 -p 8089:8089 -v ./artifacts:/mnt/artifacts splunk4dfir:latest start
```

goto: http://127.0.0.1:8000/en-US/app/launcher/home
