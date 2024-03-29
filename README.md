# redfish-mockup-sim

Combine all the Redfish Mockup bits into a usable repo.

## Redfish Mockup Creator

```bash
 python3 ./redfishMockupCreate.py -u admin -p admin -r x3000c0s33b4n0:443 -H -d xd2000 --trace  -D hpe.redfish.mockup -S --Auth Basic
```

## Redfish Mockup Server

### Native

```bash
 python3 Redfish-Mockup-Server/redfishMockupServer.py  -D mockup-data/Cray/windom
 curl -sXGET "http://127.0.0.1:8000/redfish/v1/"
```

### Docker Using local &  mockup data

#### Use Redfish-Mockup-Server instructsion for building docker

 In the Redfish-Mockup-Server directory (submodule)

```bash
 docker pull dmtf/redfish-mockup-server:latest
 docker build -t dmtf/redfish-mockup-server:latest .
```

#### Run the Docker with correct options

```bash
docker run -p 443:8000 --rm -v ~/HPECode/redfish-mockup-sim/mockup-data/Cray/windom/:/mockup dmtf/redfish-mockup-server:latest -D /mockup -H admin
curl -XGET "http://0.0.0.0:443/redfish/v1/"
```

## Redfish Service Validator

### Cray Branch

See redfish-validator-profiles/README.md

Running against MockupServer

```bash
 cat config/CSM.ini 
   [Tool]
   Version = 2
   Copyright = Redfish DMTF (c) 2021
   verbose = 0

   [Host]
   username = root
   description = "Redfish Endpoint"
   forceauth = False
   authtype = Basic

   [Validator]
   payload = Tree /redfish/v1/
   logdir = ./logs
   oemcheck = False
   debugging = False

 python3 RedfishInteropValidator.py -c config/CSM.ini -i http://0.0.0.0:443 ../redfish-validator-profiles/profiles/CSMRedfishProfile.v1_2_0.json -p initial0

```

Results are in logs/

### DMTF
Install the tool locally.

```bash
 pip install redfish_service_validator
```

Run against mockup server/data

```bash
 rf_service_validator -u root -p initial0 -r http://0.0.0.0:443/
```

## Redfish Service Emulator

How does this different from Mockup Server
