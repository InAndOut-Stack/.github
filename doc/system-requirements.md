# System Requirements

## Customer Requirements

- Device having a minimal display being able to navigate our website and check results
- Internet access

## Physical Requirements

- Location and its map
- Articles and their mapping

## Software Requirements

- Backend code
- Infrastructure and servers (backend and database)
- Frontend interfaces for all actors

## Scalability Requirements

### Architecture

Any information should be available for anyone in the world but region specific information should be loaded faster for those in proximity.

The architecture of the system, specific to a particular region (e.g. us-east-1): ![Architecture Diagram](./InAndOut.drawio.png)

It encapsulates all the necessary machines needed to make the system work for a specific region. According to [AWS](https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions.html) there are 34 regions, so the plan is to replicate this architecture for each of those (Each region has its own brain, there is no central brain).

The itinerary of a client requst before hitting our services:

#### 1. CDN Interface

Client interfaces will be running on the CDN servers (here referred as CDN Interface) caching media, javascript, and responses from our backend services. In case of a cache miss, these will redirect the request to our Region gateway.

#### 2. Region Gateway

Region Gateway is a simple request forwarding: it call the API Gateway specific to the requested region. It is expected that most of the time the requested region and the "Region Gateway" region to correspond and therefore the latency added between these two steps is minimal. It could also be possible to deploy both gateways on the same machine to achive 0ms latency.

#### 3. API Gateway

The API Gateway does what its name says: service orientated traffic distribution. For example "Route" related APIs are redirected to servers running the routing service.

Some other concerns addressed by either Region Gateway or API Gateway:

- Authentication
- Rate Limiting
- Monitoring

#### 4. Load Balancer

Before finally hitting our service server, a load balancer will be placed in front of it to ensure no deployment is overwhelmed. Since the routing is the main feature of our system and it is computationally expensive, multiple instances of `Route Service` are expected to be accepting requests.

### TSP Caching

This operation is computationally intensive and may generate significant costs. CDN caches avoid expensive recomputations and will be used. The CDN machine could also host a Redis instance.

Challenges:

- CPU Expense: High CPU overhead is required to recompute the same solution repeatedly.
- Solution Invalidation: Employees might update the store map, invalidating the cached solutions.

Redis Data Structure:

- Key Structure - `tsp:{storeId}:{mappingVersion}:{standsHash}`
- Value Structure - the actual output of the operation

Hash stand ids algoritms:

```
1.  Sort: Take the list of Stand ids and sort them numerically: [10, 2, 5] -> [2, 5, 10].
2.  Stringify: Join the sorted IDs with a delimiter: "2,5,10".
3.  Hash: Apply a fast hashing algorithm (like MD5 or SHA-1) to the string to produce a fixed-length suffix: a1b2c3d4.
```

Example:

`tsp:101:v4:a1b2c3d4` =

```json
{
  "routeId": "9426b826-1f72-449c-9b95-d9c5c7d88274",
  "storeId": "2d4ec91c-cd88-4138-bfa7-f42773b5a11c",
  "mappingVersion": "1",
  "standIdList": [
    "20c423f3-b8d6-454b-8c3c-94f0a7a66a78",
    "7dc2a857-e33f-4314-bf74-6d33c13d8fb7",
    "7b8facca-2bc7-414b-ace8-45edc72a3220",
    "801a699c-9720-4a4c-9298-2e07b5e241f2"
  ],
  "solutionList": []
}
```
