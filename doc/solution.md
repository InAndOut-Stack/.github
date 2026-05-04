# Solution

## Technical architecture of the system

The typical **Customer** workflow for generating an itinerary: he can optionally apply localization filters such as coordinates, a radius-based area lookup, or specific store or brand names to narrow down the search. Next, the user selects a store and chooses the desired products. The system then displays the available offers, with the option to filter and view only the products that are available in the selected store. After making the necessary selections, the user presses the “**Generate Route**”\*\* \*\*button, and the system produces a final image-based graph representing the route.

## Actors

- (Online, not physically present) **Customer**
  - The actual physical client entering the store
  - Makes product selections accounting their details, prices and offers
- **Employee**
  - His primary responsibility is to manage the location of products (plans the article stand locations in advance, updates them into the system, and finally makes the physical changes required)
  - Also manages offers and discounts
- **Store Owner**
  - Provides the store structure (floors, navigation aisles, etc.) and details (name, operating hours, etc.)
  - Also manages offers and discounts
  - The main attribution is to correctly define the store spaces, both in measurements and directions
- **Brand Owner**
  - Manages the Brand profile (name, logo, etc.)
  - Manages the articles registered within his business

## Resources

- Provided by the Customer:
  - **Route** = Optimal in-store navigation path for visiting all the selected articles
- Provided by the Employee:
  - **Node** = Navigation point acting as an intersection between two edges
  - **Edge** = Navigation aisle open to Customers
  - **Floor** = Undirected graph having nodes and edges representing a map component of a store
  - **Stand** = Array of shelves alongside an edge where multiple articles can be placed
- Provided by the Store Owner:
  - **Store** = physical unit with a brand identity, location, and operating schedule
  - **Article** = the brand-specific commercial definition of a product (Price, Currency, Brand)
  - **Offer** = promotional logic linked to articles or stores
- Provided by the Brand Owner:
  - **Brand** = the legal entity
  - **Product** = Simple product record with details like name, category, or vendor
    - acts as an information cache preventing duplicate **Article** information across multiple businesses 
    - not directly managed by Brand Owners but they can request to create new records
    - example:
      - the Zuzu milk is sold by both Kaufland and Lidl,
      - brands should reference the same product and not create two identical ones

### Relationships

- 1 Brand <-> 0/∞ Stores
- 1 Store <-> 0/∞ Floors
- 1 Floor <-> 0/∞ Nodes
- 1 Floor <-> 0/∞ Edges

## APIs

To support a "one-click" onboarding experience for Owners, we provide composite operations. When a store registers a new Stand, the system automatically upserts the underlying Product and Article records if they do not already exist.

To satisfy complex intents - such as de-listing an item from a shelf without deleting it from the brand catalog; we expose Lower-Level APIs. Articles and Products exist independently of Stands. Removing a Stand entry does not destroy the Article record, allowing it to be easily re-placed or utilized by other stores of the same brand.

### Services

#### 1. Route Service

- Actors: Customers
- Resources: Routes
- **/routes** - POST, GET, DELETE
- Store and stand selection interface

#### 2. Map Service

- Actors: Employee
- Resources: Nodes, Edges, Floors, Stands
- **/stores/{storeId}/floors** - POST, GET, PUT, DELETE
- **/stores/{storeId}/stands** - POST, GET, LIST, PUT, DELETE
- Floor modelling interface

#### 3. Business Service

- Actors: Store/Brand Owners
- Resources: Brands, Stores, Articles, Offers
- **/brands** = POST, GET, PATCH, DELETE
- **/brands/{brandId}/stores** = POST, GET, LIST, PATCH, DELETE
- **/brands/{brandId}/articles** - POST, GET, PATCH, DELETE
- **/brands/{brandId}/offers** - POST, GET, LIST, PATCH, DELETE

### Repositories

1. InAndOut-API-modelling
2. InAndOut-Route-Service
3. InAndOut-Mapping-Service
4. InAndOut-Business-Service
5. InAndOut-Customer-Interface
6. InAndOut-Mapping-Interface

## Authorization

The roles with higher ownership should inherit the permissions of those with less. For example a Store Owner should be able to edit the structure of the store.

### Authentication

### Rate Limiting

### Resource Deletion
