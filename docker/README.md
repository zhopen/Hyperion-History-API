The version is used For cmchain.testnet.pub
 
- eosio-node iamges: boscore/bos:v3.0.8  
- use fixed ip with links in docker-compose.yml insteal of DNS, because of DNS reponse is too slow.  
- add "sleep 20" in ./scripts/run-nodeos.sh. because of missing "nc" in eosio-node's image.  modified docker-compose.yml. 'true' means not to call wait for  
``` 
$ vi ./scripts/run-nodeos.sh   
sleep 20
if [ "$2" = "" ]
then
  nodeos --genesis-json /home/eosio/config/genesis.json --disable-replay-opts --data-dir /home/eosio/data --config-dir /home/eosio/config &
else
  nodeos --delete-all-blocks --snapshot /home/eosio/data/snapshots/$2 --disable-replay-opts --data-dir /home/eosio/data --config-dir /home/eosio/config &
fi
```  
```
    command: bash -c "/home/eosio/scripts/run-nodeos.sh true  ${SNAPSHOT:-""}"

```
