
# Overview  

> Additional information available at https://www.getasecond.com/

The vision of Second is that it should be easy for every person and business to own and protect their data and connections, and customize their UX for interacting with everything. 

Effectively, an AI for managing your data, that also gives you the custom UI you want for viewing/modifying any kind of data. 

There are a few distinct parts of Second:   
- identity: decentralized, stored on the Stellar network   
- language/types/schemas: in order to exchange data, we need to agree on what the data is     
- nodes and nodechain: every piece of data is a node with a type/language, append-only log of nodes pointed at by identity, all stored on ipfs   
- common environment/runtime/vm: basic capabilities augmented by nodes  


## Identity and Routing 
Identities (previous called usernames/namesspaces/domains) are stored on the Stellar network. A username/namespace is the SHA256 hash (32 bytes) used as the seed for an ED25519 key. 

```javascript
let StellarSdk = require('stellar-sdk');
let stellarServer = new StellarSdk.Server('https://horizon.stellar.org');

function identity(username){
	let username = tmpUser.normalize('NFKC').toLowerCase();
	let seed = crypto.createHash('sha256').update(username).digest();
	let pair = StellarSdk.Keypair.fromRawEd25519Seed(seed);
	stellarServer.loadAccount(pair.publicKey())
	  .then(function(account){
	  	// identity exists
	  	console.log(account);
	  })
	  .catch(function(){
	  	// identity does not exist on chain 
	  })
}
```

For example, the username `Tellar` results in `pair.secret() => "SBEFT2DDCK2FQ5CGJEBH2KJWLUOLN6FUMYDVQ7EGK2UGPHF33FD5RHE7"`. 

> notice that `TeLLar` and `tellar` are the same, to limit ambiguity when transmitting usernames among individuals, usernames are normalized and common symbols are removed (see the code). 

Viewing the public address of `tellar` ([view](https://horizon.stellar.org/accounts/GCJ2UTBPMZJJN6YX4DIFL33Y2LJJEWZEVUEN4HHI5WNSKYEXANVXFJL3)) gives us some useful information: 


```json
thresholds: {
	low_threshold: 2,
	med_threshold: 2,
	high_threshold: 2
},
signers: [
	{
		public_key: "GBJ3R3AJL7T6JL34HSWPQJMBG73BNWXSAYVGVXULDXJG3WAJWB6PSALQ",
		weight: 1,
		key: "GBJ3R3AJL7T6JL34HSWPQJMBG73BNWXSAYVGVXULDXJG3WAJWB6PSALQ",
		type: "ed25519_public_key"
	},
	{
		public_key: "GCJ2UTBPMZJJN6YX4DIFL33Y2LJJEWZEVUEN4HHI5WNSKYEXANVXFJL3",
		weight: 1,
		key: "GCJ2UTBPMZJJN6YX4DIFL33Y2LJJEWZEVUEN4HHI5WNSKYEXANVXFJL3",
		type: "ed25519_public_key"
	}
],
data: {
	"|second": "base64-ipfs-hash",
	"|web": "base64-ipfs-hash",
	"|nodechain": "base64-ipfs-hash",
	"|licensetoken" : "base64-ipfs-hash",
	"nick|second" : "base64-ipfs-hash"
}
```


The thresholds and signers are set so that two keys are needed in order to make any changes; both the username/seed, and a private seed that you control. 

The `data` contains fields similar to DNS entries. In each data field, the `key` identifies the type of record (or the name+type), and the value is an IPFS hash of a Node with connection information. 

### root identity 

Your root identity "owns" anything that starts with a pipe (`|`) character. 


#### Types used in Identity Data  

__second__ - points to an External Identity Node for connecting to a Second  
```json
{
	type: "external_identity:Qmf2389hf9sdsfkdjn9fh8w9eh923",
	data: {
		publicKey: "-----BEGIN PUBLIC KEY----- MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAJMXKHxgvILqEaf/g6sZIEpylNdb2I0j wOgXOrXEesbrCdggin2o/X8YAd5+QVbCz9bwTOLexdIpp9tNkn+HlmUCAwEAAQ== -----END PUBLIC KEY-----"
	},
	nodes: [
		{
			type: "external_identity_connect_method:Qmsdkfjewiojfiwoefjdionfsoief",
			data: {
				method: "http",
				connection: "https://teacher.second.ngrok.io/ai"
			}
		}
	]
}
```

__web__ - used by Second Browser for loading websites, replacing current domain names (adds `x-second-forward` header). 

__nodechain__ - points at the API for an append-only log of signed nodes 

__email__ - points to an alternate email address to use for communication 


> TODO: change types and names to use an IPFS hash that is always a "string" type. Allows for any-length key names, instead of trying to fit inside "name|type". Matching is harder, but works both directions (easy to get IPFS hash for node type of a string) 


### namespaced identities 

You can have many data records on an identity. 

When I want to lookup the subuser `nick` on the main identity `acmecorp` (aka `nick@acmecorp`) I would look up the key `data['nick|second']`



## Language / Types / Schemas


- All data is a `node`. Logic/code is just a form of data. 
- A `node` has a `type` and `data` where the `data` follows a jsSchema defined by the `type`. 
- `type`/schemas are defined on the blockchain -> type includes the transaction hash that has the schema stored 
  - or store on ipfs (not sure yet) 
- A `node` is simply a tree structures (or graph database) with a single `parent` and multiple child `nodes`. 
- The tree/list concept is easily understood by humans, easy to query on (things that are marked X) 
- By defining the structure of data, it is easier to understand how to handle it, and use combined programming knowledge to build a system that handles all types of data. 
- Keeping things distributed (run your own Second anywhere) and use-what-you-want (just `node`s!) 
- Downloading most new abilities for a Second is as simple as using a capability to reach "outside" the environment, and add new `node`s that include data and logic. 
- Display code is also included: a frontend Second can understand how to render 




## NodeChains 

We don't always want/need to store all of our data on a chain that requires payment for every transaction (lots of data, private data, limit access, etc.), but a verifiable append-only log is useful. The NodeChain also includes two fields (reference and version) that cab be used for easier searching (for example if I want an upated version of a code node by the same author). 




## Querying Data 


Everything is a `node` that has the following schema: 
```json
{
	type: String, // a globally-available language identifier for the data schema 
	data: Object,
	nodeId: String, 
	_id: 
	createdAt: Number, // optional 
	updatedAt: Number // optional
}
```


### `node.type` (the schema of the data) 

This property defines the schema of the data for the node. The `type` is written in the format: 

`simple name` `:`  `IPFS hash`

The IPFS hash will resolve to a node of type `language` that contains data in [jsSchema](https://github.com/molnarg/js-schema) format.   

```json
{
  "data": {
    "name": "folder",
    "description": "",
    "schema": {
      "additionalProperties": false,
      "properties": {
        "name": {
          "type": "string"
        },
      },
      "type": "object"
    },
    "version": "1"
  },
  "type": "language"
}
```


When receiving nodes, your Second should evauluate the data against the schema of the node's type/language. 


## Common Environment/Runtime/VM  


### Platform, Universe, Capabilities, Environment and Startup 

__Platform:__ ubuntu, website, windows 

A Second can run on any `platform` (server, frontend website, windows, linux, embedded devices, inside a native app, etc.). 

__Universe:__ any sort of code running a JavaScript V8 runtime (nodejs server, website, native app) 

On each `platform`, a `universe` is created that includes `capabilities` (`sandbox/context` functions). The `capabilities` get exposed to the Second, and allow it to "reach outside" its environment to do things like: make http requests, search memory, etc. All the `capabilities` and `universe` code is written OUTSIDE a Second's brain/context. Everything INSIDE Second is a `node`. 

__Environment and Startup:__ `node`-only sandboxed JavaScript V8 environment, access to capabilities  

The `universe` has access to a database of `nodes`. Some of those nodes are logic/code/javascript (`code:0.0.1:local:32498h32f2`). When told, the `universe` finds and runs a `node`'s code in the `environment`. 

The basic server is "started" (for every request) when a node of type:`incoming_web_request:0.0.1:local:29832398h4723` with a `ChildNode` of type:`code...` is run inside a sandboxed VM (the `environment`) with the `req.body` (from an expressjs server). 




#### basic ubuntu backend via express server 

__Startup:__ 

A Second is triggered via an http (or websocket) request. It can also be triggered internally using a `heartbeat` node. 

__On incoming request:__ 

Find first node with type:`incoming_web_request:0.0.1:local:29832398h4723` or with a childNode of type:`common_type:0.0.1:local:298fh29h2323f` where the value is `"incoming_web_request"`. 

This node should have a `code` childNode that will have the `req.body` (as a JSON object) passed in. `req.body` is expected to be a `node`! 

The `node` is available to the code as `INPUT`. `SELF` references the node's self (id, type, data, etc.) . 

The `incoming_web_request:0.0.1:local:29832398h4723` finds available actions, and calls the action specified by the passed-in node. 







## Directory/Folder Information (this is nick-local-only) 


- cloud_configs: configs used for testing a cloud setup locally (startup script parameters saved so dont have to re-enter)  
- dump: mongodb dumps  
- language_app: Homepage and "language" and "identity" server (getasecond.com) (empbedded in language_server/dist [TODO] on deploy)  
- language_server: NodeChain host (old: host graphql endpoints for language and identity, servers getasecond.com) 
- marketing_site: NOT USED  
- mongodata: mongodb data 
- nativeapp: second_env_nativeapp environment (react native app) 
- second_env_browser: default browser environment (does NOT contain basic.json, lookup from github instead!) 
- second_env_cloud: default cloud environment (default or teacher setup, use github instead!) 
- second_env_rpi: default raspberry pi environment (use github for default setup bundle!) 
- teacher_global_nodes: the "source of truth" for logic/code nodes 

- deprecated: 
  - base_app: original used for editing Nodes (before any editor existed) ... might have been used for editing the Browser UI ? 


Everything can be run locally, or targeted at remote/live/production instances. 

1. Run Cloud Second on Heroku (for easy reachability, dont have to run a bunch of services locally) 
- on startup you'll choose the nodes you want to use from a bundle on github 

2. Run the browser app for managing your Cloud Second 



## Getting Started 


### 1. Create a Cloud Second 

Your Cloud Second is your "always-on" digital assistant, the operations center for your IoT, and the guardian of your data. 

Cloud Install Guide: [http://github.com/secondai/env_cloud_default](http://github.com/secondai/env_cloud_default) 

Your Cloud Second will be identified/discoverable using a phrase that you define. 


### 2. Create Browser Second to connect to your Cloud Second 

Each Second is a self-contained environment, so you need a local Second that can communicate with your Cloud Second and display the information you want. Uses React components. 

Browser Install Guide: [http://github.com/secondai/env_browser_default](http://github.com/secondai/env_browser_default) 


### 3. Run Second on a Raspberry Pi 

A Pi can serve as a great base for controlling smart home components. 

[http://github.com/secondai/env_rpi_default](http://github.com/secondai/env_rpi_default) 


### 4. Run Second as a Native Mobile App 

Control your Cloud Second while on the go. 

[http://github.com/secondai/env_nativeapp_default](http://github.com/secondai/env_nativeapp_default) 





## Local Development 

```sh
./second.sh run services
```




&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;



## Overview - Old 

Second is an intermediary that protects your data and makes it accessible to the world on your terms, and also organizes the world's information and presents it how you prefer. 

When I want to know what data you have, I need to know where I can send you the request (your identity and location), and the data I receive is the format I'm expecting (or at least a format I understand). I need to trust that we have a similar understanding of data, otherwise you say "left" and I hear "right" and disaster strikes. 

Usually before giving you information, a Second wants to know who the requestor is, so the most common action/node that is passed in is `run this sequence, starting with 'hello my name is IDENTIFIER', then give me some data`. A Second can handle authentication in a variety of ways (basic auth, jwt, custom, etc.). 





