# HPExplorer-HPComposer-HPfabric

Nuestro objetivo es instalar un ambiente de DEV con Hyperledger Fabric, además de Hyperledger Composer y finalmente configurar una herramienta de monitoreo y visualización de los compomentes que conforman la blokchain como es Hyperledger Explorer.

Asumimos que se cuenta con un ambiente de Hyperledger Composer para desarrollo instalado siguiendo con la documentación oficial: https://hyperledger.github.io/composer/latest/installing/development-tools

A continuación vamos a describir detalladamente el proceso que tuve que realizar para poder configurar el framework Hyperledger Explorer, válido aclarar que difiere de la documentación oficial en algunos aspectos como por ejemplo en el formato del fichero de configuración.

También decidí levantar todo el ambiente en contenedores por lo qué tanto la base de datos postgesql, así como el propio explorer corren en contenedores sobre docker.

Pasos:

1. Primeramente ahí que modificar el docker-compose.yml de la instalación del Hyperledger Fabric para añadir una variable a la construcción del servicio "peer0.org1.example.com", la opción a añadir es:

- CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051

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