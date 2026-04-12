# Solution

## Technical architecture of the system

The typical **Customer** workflow for generating an itinerary: he can optionally apply localization filters such as coordinates, a radius-based area lookup, or specific store or brand names to narrow down the search. Next, the user selects a store and chooses the desired products. The system then displays the available offers, with the option to filter and view only the products that are available in the selected store. After making the necessary selections, the user presses the “**Generate Route**”\*\* \*\*button, and the system produces a final image-based graph representing the route.

# APIs

To support a "one-click" onboarding experience for Owners, we provide composite operations. When a store registers a new Stand, the system automatically upserts the underlying Product and Article records if they do not already exist.

To satisfy complex intents - such as de-listing an item from a shelf without deleting it from the brand catalog; we expose Lower-Level APIs. Articles and Products exist independently of Stands. Removing a Stand entry does not destroy the Article record, allowing it to be easily re-placed or utilized by other stores of the same brand.

## Services

### 1. Route Service

- Actors: Customers
- Resources: Routes
- **/routes** - POST, GET, DELETE
- Store and stand selection interface

### 2. Mapping Service

- Actors: Employee
- Resources: Stands, Floors, Nodes, Edges
- **/stores/{storeId}/stands** - POST, GET, LIST, PUT, DELETE
- **/stores/{storeId}/floors** - POST, GET, PUT, DELETE
- Floor modelling interface

### 3. Business Service

- Actors: Brand/Store Owners
- Resources: Brands, Stores, Offers
- **/stores** = POST, GET, LIST, PATCH, DELETE
- **/brands** = POST, GET, PATCH, DELETE
- **/brands/{brandId}/articles** - POST, GET, PATCH, DELETE
- **/products** - POST, GET, PATCH, DELETE
- **/offers** - POST, GET, LIST, PATCH, DELETE

## Repositories

1. InAndOut-API-modelling
2. InAndOut-Route-Service
3. InAndOut-Mapping-Service
4. InAndOut-Business-Service
5. InAndOut-Customer-Interface
6. InAndOut-Mapping-Interface

# Authorization

The roles with higher ownership should inherit the permissions of those with less. For example a Store Owner should be able to edit the structure of the store.

## Employee/Owner Authentication

## Resource Deletion

## Rate limiting

# TSP

The final operation in the workflow is computationally intensive and may generate significant costs. To handle this more efficiently, the system could use a POST request that returns a job id instead of processing everything synchronously. Once the job is completed, the client will be able to query and inspect the result using that job identifier.

## Persistence and Caching Strategy

This issue outlines the data structure, technical challenges, and implementation strategy for managing Traveling Salesperson Problem (TSP) solutions.

Each TSP solution object is composed of the following attributes:

| Name         | Type             | Description                                                               |
| ------------ | ---------------- | ------------------------------------------------------------------------- |
| storeId      | UUID             | Unique identifier for the specific store.                                 |
| standIds     | List[String]     | The collection of stands included in the optimization.                    |
| result       | List[List[Node]] | An array of routes containing node sequences to satisfy all requirements. |
| storeVersion | Integer          | The version snapshot used to generate the solution.                       |

The nature of TSP optimization presents three primary challenges:

- Computational Expense: High CPU overhead is required to recompute the same solution repeatedly.
- Storage Footprint: Dwhigh-speed RAM cache (e.g., Redis) for near-instant retrieval.
- Version Tracking: Every cached solution is tagged with a storeVersion.

1.  Request: Retrieve the current storeVersion and check the cache for the corresponding storeId.
2.  Validation: \* If cachedVersion == currentVersion: Return the cached solution immediately.
    - If cachedVersion != currentVersion (or no cache exists): Trigger a recomputation.

## Caching Deterministic Versioning

To store complex TSP solutions in Redis we use a hashing and serialization pattern. This ensures that the same set of stands on a specific floor layout always maps to the same pre-computed result.

## 1. Key Structure

The key must be unique to the layout version and the specific set of stands requested.

Format: tsp:{storeId}:{mappingVersion}:{standsHash}

### StandsHash Generation

To ensure the key remains the same regardless of the order in which the user selects the stands, follow this deterministic process:

1.  Sort: Take the list of Stand IDs and sort them numerically: [10, 2, 5] -> [2, 5, 10].
2.  Stringify: Join the sorted IDs with a delimiter: "2,5,10".
3.  Hash: Apply a fast hashing algorithm (like MD5 or SHA-1) to the string to produce a fixed-length suffix: a1b2c3d4.

## 2. Value Structure (JSON)

Since the value is a list of node lists (solutions), we serialize the entire collection into a single JSON string before storing it in Redis.

Example value for key: tsp:101:v4:a1b2c3d4

```json
[
  [
    { "id": 1, "name": "Entrance" },
    { "id": 12, "name": "Aisle 1" }
  ],
  [
    { "id": 12, "name": "Aisle 1" },
    { "id": 45, "name": "Milk Stand" }
  ]
]
```

Websockets for when the job is done?

OpenPriceMap? Queries for multiple store shopping upon?

# References

- [https://petstore3.swagger.io/](https://petstore3.swagger.io/)
- [https://dbdiagram.io/d/InAndOut-693154e7d6676488ba8f6f4d](https://dbdiagram.io/d/InAndOut-693154e7d6676488ba8f6f4d)
- [https://github.com/apache/commons-math/blob/master/commons-math-examples/examples-sofm/tsp/src/main/java/org/apache/commons/math4/examples/sofm/tsp/TravellingSalesmanSolver.java](https://github.com/apache/commons-math/blob/master/commons-math-examples/examples-sofm/tsp/src/main/java/org/apache/commons/math4/examples/sofm/tsp/TravellingSalesmanSolver.java)
- [https://github.com/TheAlgorithms/Java/blob/master/src/main/java/com/thealgorithms/graph/TravelingSalesman.java](https://github.com/TheAlgorithms/Java/blob/master/src/main/java/com/thealgorithms/graph/TravelingSalesman.java)
- [https://github.com/swagger-api/swagger-petstore/blob/master/src/main/resources/openapi.yaml](https://github.com/swagger-api/swagger-petstore/blob/master/src/main/resources/openapi.yaml)
- [https://github.com/aws/aws-parallelcluster/blob/e6e636ac3550d7a2aabf10ee0245f0ddc9bdc5af/api/spec/smithy/model/parallelcluster.smithy#L8](https://github.com/aws/aws-parallelcluster/blob/e6e636ac3550d7a2aabf10ee0245f0ddc9bdc5af/api/spec/smithy/model/parallelcluster.smithy#L8)
- [https://dev.routific.com/use-cases/travelling-salesman-problem](https://dev.routific.com/use-cases/travelling-salesman-problem)
