## Extending Quick Start Templates with Bundles
The Quick Start Solution may be extended by importing RESTMAN bundles on startup. This allows us and customers to mix-and-match solutions that they intend to use with the Gateway, producing a smaller, purpose-tailored image.

### How Does it Work?
On startup, the Gateway will look in the directory `/opt/SecureSpan/Gateway/node/default/etc/bootstrap/bundle/` for files ending in `.bundle`, and loads them in natural order. For this reason, we generally name them with numbers indicating the order in which they shoulud be loaded, e.g. `60_consul.bundle`.

### How Do I Make a Bundle?
A bundle is just a RESTMAN bundle, which you can acquire using Gateway Migration Utility. For a bundle to be imported on startup, it _must not_ be encrpyted. To ensure Gateway Migration Utility does not encrypt the bundle, pass the flag indicating that encrypted secrets should be excluded.

Currently, bundles are exported by exporting the "Custom Encapsulated Assertion Fragments" folder.

An example GMU command to export the "Custom Encapsulated Assertion Fragments" folder:

  ./GatewayMigrationUtility.sh migrateOut --trustCertificate --trustHostname --encassAsPolicyDependency --excludeEncryptedSecrets --dest encass.bundle --folderName "/Quick Start Templates/Encapsulated Assertion Fragments/Custom Encapsulated Assertion Fragments" --host localhost -u admin --plaintextPassword password
  
### How Do I Configure an Encapsulated Assertion From One of These Bundles?
The same way you configure any other encapsulated assertions - you provide the input values in the JSON payload sent to /quickstart/1.0/services when publishing the service. Each encass's documentation should contain information about the input names and allowed values.

## Bundle-Specific Information ##

### consul.bundle ###
Consul Service Discovery Support
This bundle contains an encass (ConsulLookup) to support service discovery via Consul.
It retrieves a list of nodes providing a particular service, and performs routing.

Baking container image with consul template:
* The consul bundle is located under `get-started/docker-compose/add-ons/bundles`
```
cd get-started/docker-compose
docker-compose -f docker-compose-build-ssg.yml -f docker-compose.dockercloudproxy.yml up -d --build
```

Caveats:
The bundle may be modified to include the address of the Consul agent against which lookups should be performed. To make this modification, open consul.bundle as text, find the string literal CONSUL_ADDRESS_GOES_HERE (line 23 at the time of writing), and replace it with the address of the local Consul agent (e.g. http://localhost:8500).

Inputs:
*	Service name (serviceName) - the name of the service in Consul
*	URL scheme (scheme) - the scheme that should be used in accessing the service (e.g. http:// for HTTP connections, https:// for HTTPS connections). This string will be prepended to the selected node's address.
*	(optional) Path prefix (pathPrefix) - the URL prefix that should be appended to the node's address to form the service's base URL (e.g. for Consul's v1 API, this value would be /v1).
*	Routing strategy (routingStrategy) - a string specifying what routing strategy should be used. Currently, the only supported value is roundRobin.
*	(optional) Consul agent address (consul.agentAddr) - the URL of the Consul agent (excluding the /v1 prefix) which should be used for service lookup. This takes prescedence over the cluster property of the same name, but the cluster property should be preferred in general. (ex: http://10.10.0.100:8500)
*	(optional) Consul ACL token (consul.token) - a Consul ACL token to be used in discovering service instances. The token used should have the node:read and service:read permissions.
Outputs:
*	Service base URL (service.baseUrl) - A usable address that can be thought of as analogous to the targetUrlinput of the RouteHttp encass. It will include the scheme and pathPrefix from the input options (e.g. given scheme=http://, pathPrefix=/v2, service.baseUrl may be http://172.16.0.9:5400/v2).