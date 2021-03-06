# HPExplorer-HPComposer-HPfabric

Nuestro objetivo es instalar un ambiente de DEV con Hyperledger Fabric, además de Hyperledger Composer y finalmente configurar una herramienta de monitoreo y visualización de los compomentes que conforman la blockchain como es Hyperledger Explorer.

Asumimos que se cuenta con un ambiente de Hyperledger Composer para desarrollo instalado siguiendo con la documentación oficial: https://hyperledger.github.io/composer/latest/installing/development-tools

A continuación vamos a describir detalladamente el proceso que tuve que realizar para poder configurar el framework Hyperledger Explorer, válido aclarar que difiere de la documentación oficial en algunos aspectos como por ejemplo en el formato del fichero de configuración.

También decidí levantar todo el ambiente en contenedores por lo qué tanto la base de datos postgesql, así como el propio explorer corren en contenedores sobre docker.

Pasos:

1. Primeramente ahí que modificar el docker-compose.yml de la instalación del Hyperledger Fabric para añadir una variable a la construcción del servicio "peer0.org1.example.com", la opción a añadir es:

- CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051
- CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.example.com:7051

Quedando el servicio como se muestra a continuación:

```
  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    image: hyperledger/fabric-peer:1.2.1
    environment:
      - CORE_LOGGING_LEVEL=debug
      - CORE_CHAINCODE_LOGGING_LEVEL=DEBUG
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_PEER_ID=peer0.org1.example.com
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.example.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=composer_default
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/peer/msp
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb:5984
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    ports:
      - 7051:7051
      - 7053:7053
    volumes:
        - /var/run/:/host/var/run/
        - ./:/etc/hyperledger/configtx
        - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/peer/msp
        - ./crypto-config/peerOrganizations/org1.example.com/users:/etc/hyperledger/msp/users
    depends_on:
      - orderer.example.com
      - couchdb
```
2. Seguimos con la documentación Oficial del Hyperledger Explorer. Clonamos el repositorio de Github y nos movemos para la carpeta examples/net1.

A continuación copiamos todos los materiales criptograficos desde la instalación del Hyperledger Composer para la carpeta crypto como indica la documentación de la propia página. La única diferencia con la presente documentación es el contenido del fichero de configuración config.json, de configurarlo como explica la documentación oficial no vas a lograr que se conecte correctamente el Explorer a la infraestructura de Fabric que tienes en ejecución.

A continuación les dejo un ejemplo de archivo de configuración que me funcionó correctamente:
```
{
	"network-config": {
	  "org1": {
		"name": "Org1",
		"mspid": "Org1MSP",
		"tlsEnable": false,
		"peer1": {
		  "requests": "grpc://172.17.0.1:7051",
		  "events": "grpc://172.17.0.1:7053",
		  "server-hostname": "peer0.org1.example.com",
		  "tls_cacerts":
			"/tmp/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"
		},
		"admin": {
		  "key":
			"/tmp/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore",
		  "cert":
			"/tmp/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts"
		}
	  }
	},
	"channel": "composerchannel",
	"orderers": [
	  {
		"mspid": "OrdererMSP",
		"server-hostname": "orderer.example.com",
		"requests": "grpc://172.17.0.1:7050",
		"tls_cacerts":
		  "/tmp/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt"
	  }
	],
	"keyValueStore": "/tmp/fabric-client-kvs",
	"configtxgenToolPath": "/path to the fabric sample/fabric-samples/bin",
	"SYNC_START_DATE_FORMAT": "YYYY/MM/DD",
	"syncStartDate": "2018/01/01",
	"eventWaitTime": "30000",
	"license": "Apache-2.0",
	"version": "1.1"
  }
  
```
Aclarar que cuando se instala el Hyperledger Composer no constamos con la carpeta de los script de Fabric, por lo tanto seguí los paso como si fuera a instalar Hyperledger Fabric la Basic Network, pero solamente realicé los pasos hasta que me descargó los script.

3. Finalmente ejecutamos el script de despligue de los contenedores pasándole como argumentos la carpeta donde está el fichero de configuración y la red de docker donde están desplegados los contenedores de Fabric.
#./deploy_explorer.sh net1 composer_default

Espero que les resulte de ayuda!!!.
