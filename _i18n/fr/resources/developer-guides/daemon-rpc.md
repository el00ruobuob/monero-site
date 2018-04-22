## Introduction

Voici une liste des appels de procédures distantes (RPC) du démon, leurs entrées et sorties, et des exemples pour chacun d'eux.

De nombreux appels RPC utilisent l'interface JSON RPC du démon, alors que d'autres utilisent leurs propres interfaces, comme démontré plus bas.

Remarque : "unité atomique" réfère à la plus petite fraction de 1 XMR selon l'implémentation monerod. **1 XMR = 1e12 unités atomiques**

### [Méthodes JSON RPC](#methodes-json-rpc):

* [getblockcount](#getblockcount)
* [on_getblockhash](#on_getblockhash)
* [getblocktemplate](#getblocktemplate)
* [submitblock](#submitblock)
* [getlastblockheader](#getlastblockheader)
* [getblockheaderbyhash](#getblockheaderbyhash)
* [getblockheaderbyheight](#getblockheaderbyheight)
* [getblock](#getblock)
* [get_connections](#get_connections)
* [get_info](#get_info)
* [hard_fork_info](#hard_fork_info)
* [setbans](#setbans)
* [getbans](#getbans)

### [Autres Méthodes RPC](#autres-appels-rpc-du-demon):

* [/getheight](#getheight)
* [/gettransactions](#gettransactions)
* [/is_key_image_spent](#is_key_image_spent)
* [/sendrawtransaction](#sendrawtransaction)
* [/get_transaction_pool](#get_transaction_pool)
* [/stop_daemon](#stop_daemon)


---

## Méthodes JSON RPC

La majorité des appels RPC de monerod utilisent l'interface `json_rpc` du démon pour demander des bribes d'information. Ces méthodes suivent toutes une structure similaire, par exemple :

```
IP=127.0.0.1
PORT=18081
METHOD='getblockheaderbyheight'
PARAMS='{"height":912345}'
curl \
    -X POST http://$IP:$PORT/json_rpc \
    -d '{"jsonrpc":"2.0","id":"0","method":"'$METHOD'","params":'$PARAMS'}' \
    -H 'Content-Type: application/json'
```

Certaines méthodes comportent des paramètres, alors que d'autres non. Ci-dessous des exemples de chaque méthode JSON RPC.

### **getblockcount**

Recherche combien de blocs sont dans la plus longue chaîne connue du nœud.

Entrées : *Aucune*.

Sorties :

* *count* - entier non-signé; Nombre de blocs dans la chaîne la plus longue vue par le nœud.
* *status* - chaîne; Code erreur général RPC. "OK" signifie que tout à l'air correct.

Exemple :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"getblockcount"}' -H 'Content-Type: application/json'  

{  
  "id": "0",  
  "jsonrpc": "2.0",  
  "result": {  
    "count": 993163,  
    "status": "OK"  
  }  
}  
```


### **on_getblockhash**

Recherche le hachage d'un bloc par sa hauteur.

Entrées :

* hauteur de bloc (tableau d'un seul entier)

Sorties :

* hachage de bloc (chaîne)

Exemple :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"on_getblockhash","params":[912345]}' -H 'Content-Type: application/json'

{
  "id": "0",
  "jsonrpc": "2.0",
  "result": "e22cf75f39ae720e8b71b3d120a5ac03f0db50bba6379e2850975b4859190bc6"
}
```


### **getblocktemplate**

Entrées :

* *wallet_address* - chaîne; Adresse du portefeuille pour recevoir les transaction de la base de la pièce si le bloc est miné avec succès.
* *reserve_size* - entier non-signé; Taille réservée.

Sorties :

* *blocktemplate_blob* - chaîne; Forme floue sur laquelle tenter de miner un nouveau bloc.
* *difficulty* - entier non-signé; Difficulté du prochain bloc.
* *height* - entier non-signé; Hauteur à laquelle miner.
* *prev_hash* - chaîne; Hachage du bloc le plus récent sur lequel miner le prochain bloc.
* *reserved_offset* - entier non-signé; Décalage réservé.
* *status* - chaîne; Code erreur général RPC. "OK" signifie que tout à l'air correct.

Exemple :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"getblocktemplate","params":{"wallet_address":"44GBHzv6ZyQdJkjqZje6KLZ3xSyN1hBSFAnLP6EAqJtCRVzMzZmeXTC2AHKDS9aEDTRKmo6a6o9r9j86pYfhCWDkKjbtcns","reserve_size":60}' -H 'Content-Type: application/json'

{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "blocktemplate_blob": "01029af88cb70568b84a11dc9406ace9e635918ca03b008f7728b9726b327c1b482a98d81ed83000000000018bd03c01ffcfcf3c0493d7cec7020278dfc296544f139394e5e045fcda1ba2cca5b69b39c9ddc90b7e0de859fdebdc80e8eda1ba01029c5d518ce3cc4de26364059eadc8220a3f52edabdaf025a9bff4eec8b6b50e3d8080dd9da417021e642d07a8c33fbe497054cfea9c760ab4068d31532ff0fbb543a7856a9b78ee80c0f9decfae01023ef3a7182cb0c260732e7828606052a0645d3686d7a03ce3da091dbb2b75e5955f01ad2af83bce0d823bf3dbbed01ab219250eb36098c62cbb6aa2976936848bae53023c00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001f12d7c87346d6b84e17680082d9b4a1d84e36dd01bd2c7f3b3893478a8d88fb3",
    "difficulty": 982540729,
    "height": 993231,
    "prev_hash": "68b84a11dc9406ace9e635918ca03b008f7728b9726b327c1b482a98d81ed830",
    "reserved_offset": 246,
    "status": "OK"
  }
}
```


### **submitblock**

Soumettre un bloc miné au réseau.

Entrées :

* Données floues du bloc - chaîne

Sorties :

* *status* - chaîne; Statut de soumission du bloc.


### **getlastblockheader**

Les informations d'entête de bloc pour le bloc le plus récent sont facilement récupérables avec cette méthode. Aucune entrée n'est nécessaire.

Entrées : *Aucune*.

Sorties :

* *block_header* - Une structure contenant les informations de l'entête de bloc :
  * *depth* -  entier non-signé; Le nombre de bloc succédant à ce bloc sur la chaîne de blocs. Un nombre plus élevé signifie un bloc plus ancien.
  * *difficulty* - entier non-signé; La force du réseau Monero basé sur la puissance d'extraction minière.
  * *hash* - chaîne; Le hachage de ce bloc.
  * *height* - entier non-signé; Le nombre de blocs précédant ce bloc sur la chaîne de blocs.
  * *major_version* - entier non-signé; La verison majeure du protocol monero à cette hauteur de bloc.
  * *minor_version* - entier non-signé; La version mineure du protocol monero à cette hauteur de bloc.
  * *nonce* - entier non-signé; Un nombre cryptographique aléatoire utilisé dans l'extraction minière d'un bloc Monero.
  * *orphan_status* - booléen; Habituellement `faux`. Si `vrai`, ce bloc ne fait pas parti de la chaîne la plus longue.
  * *prev_hash* - chaîne; Le hachage du bloc précédant immédiatement ce bloc dans la chaîne.
  * *reward* - entier non-signé; La quantité de nouvelles unités atomiques générées dans ce bloc et données en prime au mineur. Remarque : 1 XMR = 1e12 unités atomiques.
  * *timestamp* - entier non-signé; L'heure à laquelle le bloc a été enregistré sur la chaîne de blocs.
* *status* - chaîne; Code erreur général RPC. "OK" signifie que tout à l'air correct.

Dans cette exemple, le bloc le plus récent (990793 à ce moment là) est retourné :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"getlastblockheader"}' -H 'Content-Type: application/json'

{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "block_header": {
      "depth": 0,
      "difficulty": 746963928,
      "hash": "ac0f1e226268d45c99a16202fdcb730d8f7b36ea5e5b4a565b1ba1a8fc252eb0",
      "height": 990793,
      "major_version": 1,
      "minor_version": 1,
      "nonce": 1550,
      "orphan_status": false,
      "prev_hash": "386575e3b0b004ed8d458dbd31bff0fe37b280339937f971e06df33f8589b75c",
      "reward": 6856609225169,
      "timestamp": 1457589942
    },
    "status": "OK"
  }
}
```


### **getblockheaderbyhash**

Les informations d'entête de bloc peuvent être renvoyées en utilisant soit un hachage, soit une hauteur de bloc. Cette méthode inclue un hachage de bloc comme paramètre d'entrée pour récupérer les informations basiques à propos du bloc.

Entrées :

* *hash* - chaîne; Le hachage sha256 du bloc.

Sorties :

* *block_header* - Une structure contenant les informations d'entête de bloc. Voir [getlastblockheader](#getlastblockheader).

Dans cet exemple, le bloc 912345 est recherché par son hachage :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"getblockheaderbyhash","params":{"hash":"e22cf75f39ae720e8b71b3d120a5ac03f0db50bba6379e2850975b4859190bc6"}}' -H 'Content-Type: application/json'

{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "block_header": {
      "depth": 78376,
      "difficulty": 815625611,
      "hash": "e22cf75f39ae720e8b71b3d120a5ac03f0db50bba6379e2850975b4859190bc6",
      "height": 912345,
      "major_version": 1,
      "minor_version": 2,
      "nonce": 1646,
      "orphan_status": false,
      "prev_hash": "b61c58b2e0be53fad5ef9d9731a55e8a81d972b8d90ed07c04fd37ca6403ff78",
      "reward": 7388968946286,
      "timestamp": 1452793716
    },
    "status": "OK"
  }
}
```


### **getblockheaderbyheight**

Comme pour `getblockheaderbyhash` ci-dessus, cette méthode inclue une hauteur de bloc comme paramètre d'entrée pour récupérer les informations basiques à propos du bloc.

Entrées :

* *height* - entier non-signé; La hauteur de bloc.

Sorties :

* *block_header* - Une structure contenant les informations d'entête de bloc. Voir [getlastblockheader](#getlastblockheader).

Dans cet exemple, le bloc 912345 est recherché par sa hauteur (remarquez que l'information renvoyée est la même que dans l'exemple précédent) :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"getblockheaderbyheight","params":{"height":912345}}' -H 'Content-Type: application/json'

{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "block_header": {
      "depth": 78376,
      "difficulty": 815625611,
      "hash": "e22cf75f39ae720e8b71b3d120a5ac03f0db50bba6379e2850975b4859190bc6",
      "height": 912345,
      "major_version": 1,
      "minor_version": 2,
      "nonce": 1646,
      "orphan_status": false,
      "prev_hash": "b61c58b2e0be53fad5ef9d9731a55e8a81d972b8d90ed07c04fd37ca6403ff78",
      "reward": 7388968946286,
      "timestamp": 1452793716
    },
    "status": "OK"
  }
}
```


### **getblock**

Les informations complètes d'un bloc peuvent être récupérées soit par hauteur de bloc, soit par hachage, comme pour les appels d'entêtes de bloc ci-dessus. Pour les informations complètes d'un bloc, les deux recherches utilisent la meme méthode, mais avec des paramètres d'entrée différents.

Entrées (choisir l'une de celle-ci) :

* *height* - entier non-signé; La hauteur de bloc.
* *hash* - chaîne; Le hachage du bloc.

Sorties :

* *blob* - chaîne; Forme floue hexadécimale des informations du bloc.
* *block_header* - Une structure contenant les informations d'entête de bloc. Voir [getlastblockheader](#getlastblockheader).
* *json* - chaîne json; Détails du bloc au format JSON :
  * *major_version* - Comme dans l'entête du bloc.
  * *minor_version* - Comme dans l'entête du bloc.
  * *timestamp* - Comme dans l'entête du bloc.
  * *prev_id* - Comme `prev_hash` dans l'entête du bloc.
  * *nonce* - Comme dans l'entête du bloc.
  * *miner_tx* - Informations de la transaction du mineur :
    * *version* - Numéro de version de la transaction.
    * *unlock_time* - La hauteur de bloc à laquelle la base de la pièce pourra être dépensée.
    * *vin* - Liste des entrées de la transaction :
      * *gen* - Les transactions du mineur sont les transaction de la base de la pièce, ou "gen".
        * *height* - Cette hauteur de bloc, c'est à dire lorsque la base de la pièce est générée.
    * *vout* - Liste des sorties de la transaction outputs. Chaque sortie contient :
      * *amount* - Le montant de la sortie, en unités atomiques.
      * *target* -
        * *key* -
    * *extra* - Habituellement intitulé "ID de transaction" mais peut être utilisé pour insérer n'importe quelle chaîne aléatoire de 32 octets / 64 caractères hexadécimaux.
    * *signatures* - Contient les signatures des signataires de la transaction. Les transactions de la base de la pièce n'ont pas de signatures.
  * *tx_hashes* - Liste des hachages des transactions n'appartenant pas à la base de la pièce. S'il n'y a pas d'autre transactions, cette liste peut être vide.
* *status* - chaîne; Code erreur général RPC. "OK" signifie que tout à l'air correct.

**Look up by height:**

Dans l'exemple suivant, le bloc 912345 est recherché par sa hauteur. Notez que le bloc 912345 n'a pas de transactions n'appartenant pas à la base de la pièce. (regardez l'exemple d'après pour un bloc contenant des transactions supplémentaires) :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"getblock","params":{"height":912345}}' -H 'Content-Type: application/json'

{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "blob": "...",
    "block_header": {
      "depth": 80694,
      "difficulty": 815625611,
      "hash": "e22cf75f39ae720e8b71b3d120a5ac03f0db50bba6379e2850975b4859190bc6",
      "height": 912345,
      "major_version": 1,
      "minor_version": 2,
      "nonce": 1646,
      "orphan_status": false,
      "prev_hash": "b61c58b2e0be53fad5ef9d9731a55e8a81d972b8d90ed07c04fd37ca6403ff78",
      "reward": 7388968946286,
      "timestamp": 1452793716
    },
    "json": "{\n  \"major_version\": 1, \n  \"minor_version\": 2, \n  \"timestamp\": 1452793716, \n  \"prev_id\": \"b61c58b2e0be53fad5ef9d9731a55e8a81d972b8d90ed07c04fd37ca6403ff78\", \n  \"nonce\": 1646, \n  \"miner_tx\": {\n    \"version\": 1, \n    \"unlock_time\": 912405, \n    \"vin\": [ {\n        \"gen\": {\n          \"height\": 912345\n        }\n      }\n    ], \n    \"vout\": [ {\n        \"amount\": 8968946286, \n        \"target\": {\n          \"key\": \"378b043c1724c92c69d923d266fe86477d3a5ddd21145062e148c64c57677008\"\n        }\n      }, {\n        \"amount\": 80000000000, \n        \"target\": {\n          \"key\": \"73733cbd6e6218bda671596462a4b062f95cfe5e1dbb5b990dacb30e827d02f2\"\n        }\n      }, {\n        \"amount\": 300000000000, \n        \"target\": {\n          \"key\": \"47a5dab669770da69a860acde21616a119818e1a489bb3c4b1b6b3c50547bc0c\"\n        }\n      }, {\n        \"amount\": 7000000000000, \n        \"target\": {\n          \"key\": \"1f7e4762b8b755e3e3c72b8610cc87b9bc25d1f0a87c0c816ebb952e4f8aff3d\"\n        }\n      }\n    ], \n    \"extra\": [ 1, 253, 10, 119, 137, 87, 244, 243, 16, 58, 131, 138, 253, 164, 136, 195, 205, 173, 242, 105, 123, 61, 52, 173, 113, 35, 66, 130, 178, 250, 217, 16, 14, 2, 8, 0, 0, 0, 11, 223, 194, 193, 108\n    ], \n    \"signatures\": [ ]\n  }, \n  \"tx_hashes\": [ ]\n}",
    "status": "OK"
  }
}
```

**Look up by hash:**

Dans l'exemple suivant, le bloc 993056 est recherché par son hachage. Remarquez que que le bloc 993056 possède trois transactions n'appartenant pas à la base de la pièce :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"getblock","params":{"hash":"510ee3c4e14330a7b96e883c323a60ebd1b5556ac1262d0bc03c24a3b785516f"}}' -H 'Content-Type: application/json'

{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "blob": "...",
    "block_header": {
      "depth": 12,
      "difficulty": 964985344,
      "hash": "510ee3c4e14330a7b96e883c323a60ebd1b5556ac1262d0bc03c24a3b785516f",
      "height": 993056,
      "major_version": 1,
      "minor_version": 2,
      "nonce": 2036,
      "orphan_status": false,
      "prev_hash": "0ea4af6547c05c965afc8df6d31509ff3105dc7ae6b10172521d77e09711fd6d",
      "reward": 6932043647005,
      "timestamp": 1457720227
    },
    "json": "{\n  \"major_version\": 1, \n  \"minor_version\": 2, \n  \"timestamp\": 1457720227, \n  \"prev_id\": \"0ea4af6547c05c965afc8df6d31509ff3105dc7ae6b10172521d77e09711fd6d\", \n  \"nonce\": 2036, \n  \"miner_tx\": {\n    \"version\": 1, \n    \"unlock_time\": 993116, \n    \"vin\": [ {\n        \"gen\": {\n          \"height\": 993056\n        }\n      }\n    ], \n    \"vout\": [ {\n        \"amount\": 2043647005, \n        \"target\": {\n          \"key\": \"59e9d685b3484886bc7b47c133e6099ecdf212d5eaa16ce19cd58e8c3c1e590a\"\n        }\n      }, {\n        \"amount\": 30000000000, \n        \"target\": {\n          \"key\": \"4c5e2f542d25513c46b9e3b7d40140a22d0ae5314bfcae492ad9f56fff8185f0\"\n        }\n      }, {\n        \"amount\": 900000000000, \n        \"target\": {\n          \"key\": \"13dd8ffdac9e6a2f71e327dad65328198dc879a492d145eae72677c0703a3515\"\n        }\n      }, {\n        \"amount\": 6000000000000, \n        \"target\": {\n          \"key\": \"62bda00341681dccbc066757862da593734395745bdfe1fdc89b5948c86a5d4c\"\n        }\n      }\n    ], \n    \"extra\": [ 1, 182, 145, 133, 28, 240, 87, 185, 195, 2, 163, 219, 202, 135, 158, 28, 186, 76, 196, 80, 97, 202, 85, 170, 166, 224, 60, 220, 103, 171, 158, 69, 80, 2, 8, 0, 0, 0, 12, 97, 127, 223, 22\n    ], \n    \"signatures\": [ ]\n  }, \n  \"tx_hashes\": [ \"79c6b9f00db027bde151705aafe85c495883aae2597d5cb8e1adb2e0f3ae1d07\", \"d715db73331abc3ec588ef07c7bb195786a4724b08dff431b51ffa32a4ce899b\", \"b197066426c0ed89f0b431fe171f7fd62bc95dd29943daa7cf3585cf1fdfc99d\"\n  ]\n}",
    "status": "OK",
    "tx_hashes": ["79c6b9f00db027bde151705aafe85c495883aae2597d5cb8e1adb2e0f3ae1d07","d715db73331abc3ec588ef07c7bb195786a4724b08dff431b51ffa32a4ce899b","b197066426c0ed89f0b431fe171f7fd62bc95dd29943daa7cf3585cf1fdfc99d"]
  }
}
```


### **get_connections**

Récupère des informations concernant les connexions entrantes et sortantes de votre nœud.

Entrées : *Aucune*.

Sorties :

* *connections* - Liste de toutes les connexions et leurs infos :
  * *avg_download* - entier non-signé; Octets de données téléchargés en moyenne par le nœud.
  * *avg_upload* - entier non-signé; Octets de données uploadés en moyenne par le nœud.
  * *current_download* - entier non-signé; Octets actuellement téléchargés par le nœud.
  * *current_upload* - entier non-signé; Octets actuellement uploadés par le nœud.
  * *incoming* - booléen; Est-ce que le nœud récupère des informations depuis votre nœud ?
  * *ip* - chaîne; L'adresse IP du nœud.
  * *live_time* - entier non-signé
  * *local_ip* - booléen
  * *localhost* - booléen
  * *peer_id* - chaîne; L'ID du nœud sur le réseau.
  * *port* - chaîne; Le port utilisé par le nœud pour se connecter au réseau.
  * *recv_count* - entier non-signé
  * *recv_idle_time* - entier non-signé
  * *send_count* - entier non-signé
  * *send_idle_time* - entier non-signé
  * *state* - chaîne

Voici un exemple de `get_connections` et de sa réponse :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"get_connections"}' -H 'Content-Type: application/json'

{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "connections": [{
      "avg_download": 0,
      "avg_upload": 0,
      "current_download": 0,
      "current_upload": 0,
      "incoming": false,
      "ip": "76.173.170.133",
      "live_time": 1865,
      "local_ip": false,
      "localhost": false,
      "peer_id": "3bfe29d6b1aa7c4c",
      "port": "18080",
      "recv_count": 116396,
      "recv_idle_time": 23,
      "send_count": 176893,
      "send_idle_time": 1457726610,
      "state": "state_normal"
    },{
    ...
    }],
    "status": "OK"
  }
}
```


### **get_info**

Récupère des informations générales concernant l'état de votre nœud et du réseau.

Entrées : *Aucune*.

Sorties :

* *alt_blocks_count* - entier non-signé; Nombre de blocs alternatifs jusqu'à la chaîne principale.
* *difficulty* - entier non-signé; Difficulté du réseau (similaire à la force du réseau).
* *grey_peerlist_size* - entier non-signé; Taille de la liste de pairs gris (déconnectés)
* *height* - entier non-signé; Longueur de la chaîne la plus longue connue par le démon.
* *incoming_connections_count* - entier non-signé; Nombre de pairs connecté et tractés par votre nœud.
* *outgoing_connections_count* - entier non-signé; Nombre de pairs auxquels vous êtes connectés et depuis lesquels vous récupérez des informations.
* *status* - chaîne; Code erreur général RPC. "OK" signifie que tout à l'air correct.
* *target* - entier non-signé; Cible actuelle pour la prochaine preuve de travail.
* *target_height* - entier non-signé; La hauteur du prochain bloc dans la chaîne.
* *testnet* - booléen; Indique si le noeud est sur testnet (vrai) ou sur mainnet (faux).
* *top_block_hash* - chaîne; Hachage du plus haut bloc de la chaîne.
* *tx_count* - entier non-signé; Quantité totale de transaction n'appartenant pas aux bases des pièces dans la chaîne.
* *tx_pool_size* - entier non-signé; Nombre de transactions qui ont été diffusées mais pas insérées dans un bloc.
* *white_peerlist_size* - entier non-signé; Taille de la liste de pairs blanc (connectés)

Ci-dessous un exemple d'appel `get_info` et de ses réponses :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"get_info"}' -H 'Content-Type: application/json'

{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "alt_blocks_count": 5,
    "difficulty": 972165250,
    "grey_peerlist_size": 2280,
    "height": 993145,
    "incoming_connections_count": 0,
    "outgoing_connections_count": 8,
    "status": "OK",
    "target": 60,
    "target_height": 993137,
    "testnet": false,
    "top_block_hash": "",
    "tx_count": 564287,
    "tx_pool_size": 45,
    "white_peerlist_size": 529
  }
}
```


### **hard_fork_info**

Rechercher des information concernant le vote et la disponibilité de la mise à jour du réseau.

Entrées : *Aucune*.

Sorties :

* *earliest_height* - entier non-signé; Hauteur de bloc à laquelle la mise à jour du réseau sera activée si elle est votée.
* *enabled* - booléen; Indique si la mise à jour du réseau est appliquée.
* *state* - entier non-signé; Etat actuel de la mise à jour du réseau : 0 (Il est possible qu'il y ait une mise à jour du réseau), 1 (une mise à jour est nécessaire pour mettre à jour le réseau), ou 2 (tout va bien).
* *status* - chaîne; Code erreur général RPC. "OK" signifie que tout à l'air correct.
* *threshold* - entier non-signé; Pourcentage minimum de votes pour déclencher la mise à jour du réseau. 8à par défaut.
* *version* - entier non-signé; la version majeure de bloc pour la mise à jour du réseau.
* *votes* - entier non-signé; Nombre de votes en faveur de la mise à jour du réseau.
* *voting* - entier non-signé; Statut du vote de la mise à jour du réseau.
* *window* - entier non-signé; Nombre de blocs sur lesquels les votes actuels sont lancés. 10080 blocs par défaut.

Exemple :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"hard_fork_info"}' -H 'Content-Type: application/json'

{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "earliest_height": 1009827,
    "enabled": false,
    "state": 2,
    "status": "OK",
    "threshold": 0,
    "version": 1,
    "votes": 7277,
    "voting": 2,
    "window": 10080
  }
}
```


### **setbans**

Bannir l'IP d'un autre nœud.

Entrées :

* *bans* - Une liste de nœuds à bannir :
  * *ip* - entier non-signé; Adresse IP à bannir, au format entier.
  * *ban* - booléen; Configurer à `vrai` pour bannir.
  * *seconds* - entier non-signé; Duré du bannissement du nœud en secondes.

Sorties :

* *status* - chaîne; Code erreur général RPC. "OK" signifie que tout à l'air correct.

Exemple :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"setbans","params":{"bans":[{"ip":838969536,"ban":true,"seconds":30}]}}' -H  'Content-Type: application/json'

{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "status": "OK"
  }
}
```


### **getbans**

Entrées : *Aucune*.

Sorties :

* *bans* - Liste des nœuds bannis :
  * *ip* - entier non-signé; Adresse IP banni, au format entier.
  * *seconds* - entier non-signé; Heure locale Unix jusqu'à laquelle l'IP est banni.
* *status* - chaîne; Code erreur général RPC. "OK" signifie que tout à l'air correct.

Exemple :

```
$ curl -X POST http://127.0.0.1:18081/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"getbans"}' -H 'Content-Type: application/json'

{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "bans": [{
      "ip": 838969536,
      "seconds": 1457748792
    }],
    "status": "OK"
  }
}
```


---

## Autres Appels RPC du Démon

Tous les appels RPC du démon n'utilisent pas l'interface JSON_RPC. Cette rubrique fournit des exemples de ces appels.

La structure des données de ces appels est différente de celle des appels JSON RPC. Alors que les méthodes JSON RPC étaient appelées en utilisant l'extension `/json_rpc` et en spécifiant une méthode, ces méthodes sont appelées par leurs propres extensions. Par exemple :

    IP=127.0.0.1
    PORT=18081
    METHOD='gettransactions'
	PARAMS='{"txs_hashes":["d6e48158472848e6687173a91ae6eebfa3e1d778e65252ee99d7515d63090408"]}'
	curl \
		-X POST http://$IP:$PORT/$METHOD \
		-d $PARAMS \
		-H 'Content-Type: application/json'

Remarque : Il est recommandé d'utiliser JSON RPC lorsque cette alternative existe, plutôt que les méthodes suivantes. Par exemple, la manière recommandée d'obtenir la hauteur d'un nœud est via les méthodes JSON RPC [get_info](#getinfo) ou [getlastblockheader](#getlastblockheader), plutôt que [getheight](#getheight) ci-dessous.


### **/getheight**

Obtenir la hauteur actuelle d'un nœud.

Entrées : *Aucune*.

Sorties :

* *height* - entier non-signé; Longueur actuelle, ou plus longue chaîne connue du démon.

```
$ curl -X POST http://127.0.0.1:18081/getheight -H 'Content-Type: application/json'

{
  "height": 993488,
  "status": "OK"
}
```


### **/gettransactions**

Recherche une transaction par hachage.

Entrées :

* *txs_hashes* - liste de chaîne; Liste des hachages de transaction à rechercher.
* *decode_as_json* - booléen; Optionnel. Si positionné à `vrai`, les informations de transaction renvoyées seront décodées plutôt qu'en binaire.

Sorties :

* *status* - Code erreur général RPC. "OK" signifie que tout à l'air correct.
* *txs_as_hex* - chaîne; Informations complètes de la transaction en chaîne hexadécimale.
* *txs_as_json* - chaîne json; (Optionnelle - renvoyée si positionné en entrée.) Liste des informations de transaction :
  * *version* - Version de la transaction
  * *unlock_time* - Si différent de 0, cela indique que la sortie de la transaction peut être dépensée.
  * *vin* - Liste d'entrées dans la transaction :
    * *key* - La clef publique de la précédente sortie dépensée dans cette transaction.
      * *amount* - Le montant de l'entrée, en unités atomiques.
      * *key_offsets* - Une liste d'entiers de décalages de l'entrée.
      * *k_image* - L'image clef pour l'entrée donnée
  * *vout* - Liste de sorties dans la transaction :
    * *amount* - Le montant de la sortie de la transaction, en unités atomiques.
    * *target* - Information de destination de la sortie :
      * *key* - La clef publique furtive du récipiendaire. Quiconque possède la clef privée associée à cette clef contrôle la sortie de la transaction.
  * *extra* - Habituellement appelé "l'ID de Paiement" mais peut être utilisé pour insérer n'importe quels 32 bits aléatoires.
  * *signatures* - Liste de signatures utilisées dans la signature de cercle pour masquer l'origine réelle de la transaction.
Exemple 1 : Renvoyer les informations de transaction au format binaire.

```
$ curl -X POST http://127.0.0.1:18081/gettransactions -d '{"txs_hashes":["d6e48158472848e6687173a91ae6eebfa3e1d778e65252ee99d7515d63090408"]}' -H 'Content-Type: application/json'

{
  "status": "OK",
  "txs_as_hex": ["..."]
}
```


Exemple 2 : Decoder les informations de transaction renvoyées au format JSON. Remarque : la liste de "vout" a été tronquée dans la réponse affichée pour des raisons d'espace.

```
$ curl -X POST http://127.0.0.1:18081/gettransactions -d '{"txs_hashes":["d6e48158472848e6687173a91ae6eebfa3e1d778e65252ee99d7515d63090408"],"decode_as_json":true}' -H 'Content-Type: application/json'

{
  "status": "OK",
  "txs_as_hex": ["..."],
  "txs_as_json": ["{\n  \"version\": 1, \n  \"unlock_time\": 0, \n  \"vin\": [ {\n      \"key\": {\n        \"amount\": 70000000, \n        \"key_offsets\": [ 35952\n        ], \n        \"k_image\": \"d16908468dff9438a9814fe96bdaa575f06fe8da85772b72e54926428712293d\"\n      }\n    }, {\n      \"key\": {\n        \"amount\": 400000000000000, \n        \"key_offsets\": [ 6830\n        ], \n        \"k_image\": \"c7a7024b763df1181ae6fe821b70669735e38a68162ac02362e33acbe829b605\"\n      }\n    }\n  ], \n  \"vout\": [ {\n      \"amount\": 50000, \n      \"target\": {\n        \"key\": \"f6be43f7be4f06adcb1d06f4a07c637c7359e009cf3e57bb32b8c9ea636509c3\"\n      }\n    }, {\n      \"amount\": 200000, \n      \"target\": {\n        \"key\": \"b0a7a8e32f2b5302552bcd8d85112c62838b1f56cccd844eb9b63e0a732d0353\"\n      }\n    },  ...  \n  ], \n  \"extra\": [ 1, 225, 240, 98, 34, 169, 73, 47, 237, 117, 192, 30, 192, 60, 155, 47, 4, 115, 20, 21, 11, 13, 252, 219, 129, 13, 174, 37, 36, 78, 191, 141, 109\n  ], \n  \"signatures\": [ \"e6a3be8003d481d2855c8127f56871de3d28a4fb52385b999eb986c831c5cc08361c126b0db24a21b6c4299b438ee2be201d44d57a371230b9cd04395ab8c400\", \"8309851abaf2cf2a7091e0cdb9c83704fa7d68838a7a8ef8c178bb55a1e93a038dd18bb4a7549dc056b7a70e037cabd80911a03f427e36f712756d4c00f38f0b\"]\n}"]
}
```


### **/is_key_image_spent**

Vérifie si les sorties ont été dépensées en utilisant l'image clef associée à cette sortie.

Entrées :

* *key_images* - liste de chaîne; Liste de chaîne hexadécimale d'image clef à vérifier.

Sorties :

* *spent_status* - liste d'entiers non-signés; Liste de statuts pour chaque image vérifiée. Les statuts sont les suivants : 0 = non dépensé, 1 = dépensé dans la chaine de bloc, 2 = dépensé dans la cagnotte de transaction
* *status* - chaîne; Code erreur général RPC. "OK" signifie que tout à l'air correct.

Exemple :

```
$ curl -X POST http://127.0.0.1:18081/is_key_image_spent -d '{"key_images":["8d1bd8181bf7d857bdb281e0153d84cd55a3fcaa57c3e570f4a49f935850b5e3","7319134bfc50668251f5b899c66b005805ee255c136f0e1cecbb0f3a912e09d4"]}' -H 'Content-Type: application/json'

{
  "spent_status": [1,2],
  "status": "OK"
}
```


### **/sendrawtransaction**

Transmet une transaction brut sur le réseau.

Entrées :

* *tx_as_hex* - chaîne; Information de transaction complète sous forme de chaîne hexadécimale.
* *do_not_relay* - booléen; Arrête de relayer les transactions aux autres nœuds (par défaut à `faux` maintenant; était à `vrai` en version 0.11.1).

Sorties :

* *status* - chaîne; Code erreur général RPC. "OK" signifie que tout à l'air correct. Any other value means that something went wrong.
* *double_spend* - booléen; La transaction est une double dépense (`vrai`) ou non (`faux`).
* *fee_too_low* - booléen; Les frais sont trop faibles (`vrai`) ou OK (`faux`).
* *invalid_input* - booléen; L'entrée est invalide (`vrai`) ou valide (`faux`).
* *invalid_output* - booléen; La sortie est invalide (`vrai`) ou valide (`faux`).
* *low_mixin* - booléen; Nombre de Mixin trop faible (`vrai`) ou OK (`faux`).
* *not_rct* - booléen; La transaction n'est pas une transaction de cercle (`vrai`) ou est une transaction de cercle (`false`).
* *not_relayed* - booléen; La transaction n'a pas été relayée (`vrai`) ou a été relayée (`faux`).
* *overspend* - booléen; La transaction utilise plus d'argent que disponible (`vrai`) ou non (`faux`).
* *reason* - chaîne; Information complémentaire. Actuellement vide ou "non relayé" si la transaction a été acceptée mais pas relayée.
* *too_big* - booléen; La taille de la transaction est trop grande (`vrai`) ou OK (`faux`).


Exemple (Pas d'information incluses en retour ici.) :


```
$ curl -X POST http://127.0.0.1:18081/sendrawtransaction -d '{"tx_as_hex":"de6a3...", "do_not_relay":false}' -H 'Content-Type: application/json'
```


### **/get_transaction_pool**

Affiche les informations concernant les transactions valides vue par le nœud mais pas encore minées dans un bloc, de même que les informations d'image de clef de dépense en mémoire dans le nœud.

Entrées : *Aucune*.

Sorties :

* *spent_key_images* - Liste d'images de clefs de dépense :
  * *id_hash* - chaîne; Hachage de l'ID d'image de clef.
  * *txs_hashes* - lsite de chaîne; hachages d'image de clef de transaction.
* *status* - chaîne; Code erreur général RPC. "OK" signifie que tout à l'air correct.
* *transactions* - Liste de transaction dans pool mémoire qui n'ont pas été incluses dans un bloc :
  * *blob_size* - entier non-signé; La taille de la forme floue de la transaction au complet.
  * *fee* - entier non-signé; Le montant de frais d'extraction minière inclus dans la transaction, en unités atomiques.
  * *id_hash* - chaîne; Le hachage de l'ID de transaction.
  * *kept_by_block* - booléen; Nous n'acceptons pas de transactions qui se terminent dans le passé, sauf si définit à `vrai`.
  * *last_failed_height* - entier non-signé; Si la transaction a précédemment expiré, ceci indique à quelle hauteur cela c'est produit.
  * *last_failed_id_hash* - chaîne; Comme les précédent, ceci indique le hachage de l'ID de transaction précédent.
  * *max_used_block_height* - entier non-signé; Indique la hauteur du bloc le plus récent ayant une sortie utilisé dans cette transaction.
  * *max_used_block_hash* - chaîne; Indique le hachage du bloc le plus récent ayant une sortie utilisé dans cette transaction.
  * *receive_time* - entier non-signé; L'heure Unix à laquelle la transaction a été vue pour la première fois sur le réseau par le nœud.
  * *tx_json* - chaîne json; Structure JSON de toutes les informations dans la transaction :
    * *version* - Versioon de la transaction
    * *unlock_time* - Si différent de 0, cela indique quand la sortie d'une transaction peut être dépensée.
    * *vin* - Liste d'entrées dans la transaction :
      * *key* - La clef publique de la précédente sortie dépensée dans cette transaction.
        * *amount* - Le montant de l'entrée The amount of the input, in atomic units.
        * *key_offsets* - Une liste d'entiers de décalage depuis l'entrée.
        * *k_image* - L'image de clef pour l'entrée donnée.
    * *vout* - Liste de sorties de la transaction :
      * *amount* -  Montant de la sortie de la transaction en unités atomiques.
      * *target* - Information de destination de la sortie :
        * *key* - Adresse furtive du récipiendaire. Quiconque possède la clef privée associée à cette clef contrôle la sortie de cette transaction.
    * *extra* - Habituellement appelé "l'ID de Paiement" mais peut être utilisé pour insérer n'importe quels 32 bits aléatoires.
    * *signatures* - Liste de signatures utilisées dans la signature de cercle pour masquer l'origine réelle de la transaction.

Exemple (Remarque : Certaines listes dans les informations renvoyées ont été tronquées pour des raisons d'espace) :

```
$ curl -X POST http://127.0.0.1:18081/get_transaction_pool -H 'Content-Type: application/json'

{
  "spent_key_images": [{
    "id_hash": "1edb9ecc39602040282d326070ad22cb473c952c0d6280c9c4c3b853fb34f3bc",
    "txs_hashes": ["409911b2be02e3f0e930b326c67ab9e74675885ce23d71bb3bd28b62bc3e7803"]
  },{
    "id_hash": "4adb4bb63b3397027340ca4e6c45f4ce2147dfb3a4e0fafdec18834ae594a05e",
    "txs_hashes": ["946f1f4c52e3426a41959c93b60078f314813bc4bdebcf69b8ee11d593b2bd14"]
  },
  ...],
  "status": "OK",
  "transactions": [{
    "blob_size": 25761,
    "fee": 290000000000,
    "id_hash": "11d4cff23e610fac6a2b89187ad61d429a5e226652693dcac5d83d506eb92b96",
    "kept_by_block": false,
    "last_failed_height": 0,
    "last_failed_id_hash": "0000000000000000000000000000000000000000000000000000000000000000",
    "max_used_block_height": 954508,
    "max_used_block_id_hash": "03f96b374778bc059e47b96e2beec2e6d4d9e0ad39afeabdbcd77e1bd5a62f81",
    "receive_time": 1457676127,
    "tx_json": "{\n  \"version\": 1, \n  \"unlock_time\": 0, \n  \"vin\": [ {\n      \"key\": {\n        \"amount\": 70000000000, \n        \"key_offsets\": [ 63408, 18978, 78357, 16560\n        ], \n        \"k_image\": \"7319134bfc50668251f5b899c66b005805ee255c136f0e1cecbb0f3a912e09d4\"\n      }\n    },  ...  ], \n  \"vout\": [ {\n      \"amount\": 80000000000, \n      \"target\": {\n        \"key\": \"094e6a1b187385533665f89db741149f42d95fdc50bdd2a2384bcd1dc5209c55\"\n      }\n    },  ...  ], \n  \"extra\": [ 2, 33, 0, 15, 56, 190, 21, 169, 77, 13, 182, 209, 51, 35, 54, 96, 89, 237, 96, 23, 24, 107, 240, 79, 40, 86, 64, 68, 45, 166, 119, 192, 17, 225, 23, 1, 31, 159, 145, 15, 173, 255, 165, 192, 55, 84, 127, 154, 163, 25, 85, 204, 212, 127, 147, 133, 118, 218, 166, 52, 78, 188, 131, 235, 9, 159, 105, 158\n  ], \n  \"signatures\": [ \"966e5a67fbdbf72d7dc0364b705121a58e0ced7e2ab45747b6b154c05a1afe04fac4aac7f64faa2dc6dd4d51b8277f11e2f2ec7729fac225088befe3b8399c0b71a4cb55b9d0e20f93d305c78cebceff1bcfcfaf225428dfcfaaec630c88720ab65bf5d3399dd1ac82ea0ecf308b3f80d9780af7742fb157692cd60515a7e2086878f082117fa80fff3d257de7d3a2e9cc8b3472ef4a5e545d90e1159523a60f38d16cece783579627124776813334bdb2a2df4171ef1fa12bf415da338ce5085c01e7a715638ef5505aebec06a0625aaa72d13839838f7d4f981673c8f05f08408e8b372f900af7227c49cfb1e1febab6c07dd42b7c26f921cf010832841205\",  ...  ]\n}"
  },
  ...]
}
```


### **/stop_daemon**

Envoie une commande au démon pour le déconnecter proprement et l'arrêter.

Entrées : *Aucune*.

Sorties :

* *status* - chaîne; Code erreur général RPC. "OK" signifie que tout à l'air correct.

Exemple :

```
$ curl -X POST http://127.0.0.1:18081/stop_daemon -H 'Content-Type: application/json'

{
  "status": "OK"
}
```
