# BrewBlox iSpindel service

This a BrewBlox service to support [iSpindel](https://github.com/universam1/iSpindel/)
an electronic hydrometer.


This project is under active development.


## How does it work ?

The iSpindel is configured to send metrics using the generic HTTP POST protocol.
 
When the iSpindel wake up (like every 2 minutes) it submits a POST request containing the metrics to the iSpindel BrewBlox service.

The service then publish metrics to the event-bus, the BrewBlox history service is in charge to persist the metrics into the InfluxDB database.

## Configuration
 
### iSpindel

- Switch iSpindel on
- Press the reset button 3-4 times which sets up an access point
- Connect to the Wifi network "iSpindel"
- Open a browser on [http://192.168.4.1](http://192.168.4.1)
- From the "Configuration" menu, configure the Wifi access, then
  - Service Type: `HTTP`
    - Token:
    - Server Address: `<BREWBLOX_IP>`
    - Server Port: `9000`
    - Server URL: `/ispindel/ispindel`


### Deploy on the BrewBlox stack

Add the service to your docker compose file:
```yaml
  ispindel:
    image: brewblox-ispindel:latest
    labels:
      - "traefik.port=5000"
      - "traefik.frontend.rule=PathPrefix: /ispindel"
```


## Subscribe to the history service

To persist the metrics into InfluxDB the history service must subscribe to the iSpindel events:

```bash
curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/octet-stream' -d '{ \ 
   "relay": "data", \ 
   "exchange": "brewblox", \ 
   "routing": "iSpindel000" \ 
 }' 'http://localhost:9000/history/subscribe'
```

## Development

### Setup dev environment

```bash
sudo pip3 install pipenv
pipenv lock
pipenv sync -d
```

### Run tests

```bash
pipenv run pytest
```

### Build a docker image

```bash
rm -rf ./dist/ ./docker/dist
python3 setup.py sdist
mv dist docker/dist
docker build --tag brewblox-ispindel:latest --file docker/amd/Dockerfile  docker/
```

### Simulate iSpindel request

```bash
curl -XPOST http://localhost:9000/ispindel/ispindel
-d'{"name":"iSpindel000","ID":4974097,"angle":83.49442,"temperature":21.4375,"temp_units":"C","battery":4.035453,"gravity":30.29128,"interval":60,"RSSI":-76}'
```

### View influxdb data

```sql
docker exec -it brewblox-ui_influx_1 influx
> USE brewblox
> SELECT * FROM "iSpindel000"
name: iSpindel000
time                angle    battery  gravity   rssi temperature
----                -----    -------  -------   ---- -----------
1546121491626257000 83.49442 4.035453 30.29128  -76  21.4375
1546121530861939000 84.41665 4.035453 30.75696  -75  19.125

```

## Limitations

- There is no security on the ispindel endpoint
