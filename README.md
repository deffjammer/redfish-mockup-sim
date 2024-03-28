# redfish-mockup-sim

Combine all the Redfish Mockup bits into a usable repo. 

## Native

```bash
 python3 Redfish-Mockup-Server/redfishMockupServer.py  -D mockup-data/Cray/windom
 curl -sXGET "http://127.0.0.1:8000/redfish/v1/"
```

## Docker Using local &  mockup data

### Use Redfish-Mockup-Server instructsion for building docker

 In the Redfish-Mockup-Server directory (submodule)

```bash
 docker pull dmtf/redfish-mockup-server:latest
 docker build -t dmtf/redfish-mockup-server:latest .
```

### Run the Docker with correct options 

```bash
docker run -p 443:8000 --rm -v ~/HPECode/redfish-mockup-sim/mockup-data/Cray/windom/:/mockup dmtf/redfish-mockup-server:latest -D /mockup -H admin
curl -XGET "http://0.0.0.0:443/redfish/v1/"
```
