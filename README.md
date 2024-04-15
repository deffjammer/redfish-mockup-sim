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

# This is run in the Redfish-Interop-Validator directory created from redfish-validator-profiles steps in redfish-validator-profiles/README.md
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


## csm-redfish-emulator
After setup
csm-redfish-emulator$ bash ./setup.sh -a root:initial0:Administrator -m EX425

cd ../Emulator
export AUTH_CONFIG=root:initial0:Administrator
export MOCKUPFOLDER=EX425

./venv/bin/python emulator.py -d


### Mockup Docker w/ IP

Pick a docker file.
Used this one for CMMs. Change the MOCKUPFOLDER and hostname
```bash
+++ b/mockups/EX425/Dockerfile
@@ -28,7 +28,8 @@
 # ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
  # POSSIBILITY OF SUCH DAMAGE.

   -FROM arti.dev.cray.com/baseos-docker-master-local/alpine:3.13 AS base
   +FROM alpine:3.13
   +#FROM arti.dev.cray.com/baseos-docker-master-local/alpine:3.13 AS base

     # Insert our emulator extentions
      COPY src /app
      @@ -54,10 +55,10 @@ RUN set -ex \
           && pip3 install wheel \
                && pip3 install -r /app/requirements.txt

                 -EXPOSE 5000
                 +EXPOSE 443
                  ENV MOCKUPFOLDER="EX425"
                  -ENV AUTH_CONFIG=""
                  -ENV PORT=5000
                  +ENV AUTH_CONFIG="root:initial0:Administrator"
                  +ENV PORT=443
```


```bash
docker build -t ex425-rf:latest -f mockups/EX425/Dockerfile .
# Or for CMM changes
docker build -t cmm-rf:latest -f mockups/EX425/Dockerfile .
docker run --net=mynetwork --ip=192.168.10.2 cmm-rf:latest
docker run --net=mynetwork --ip=192.168.10.4 ex425-rf:latest

$ curl -k -u root:initial0 -w %{http_code} -sXGET "https://192.168.10.2:443/redfish/v1/Chassis/Enclosure" | jq .Model,.Manufacturer
"CMM"
"HPE"

$ curl -k -u root:initial0 -w %{http_code} -sXGET "https://192.168.10.4:443/redfish/v1/Systems/Node0" | jq .Model,.Manufacturer
"HPE CRAY EX425 (MILAN)"
"HPE"
```