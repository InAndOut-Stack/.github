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

# Actors

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
- **System Admin**
  - Manages product caching and route generations

# Resources

## System resources

- Provided by the Customer:
  - **Route** = Optimal in-store navigation path for visiting all the selected articles
- Provided by the Employee:
  - **Floor** = Undirected graph having nodes and edges representing a map component of a store
  - **Node** = Navigation point acting as an intersection between two edges
  - **Edge** = Navigation aisle open to Customers
  - **Stand** = Array of shelves alongside an edge where multiple articles can be placed
- Provided by the Store Owner:
  - **Article** = the brand-specific commercial definition of a product (Price, Currency, Brand)
  - **Product** = Simple product record with details like name, category, or vendor
  - **Store** = physical unit with a brand identity, location, and operating schedule
  - **Offer** = promotional logic linked to articles or stores
  - Provided by the Brand Owner:
    - **Brand** = the legal entity

## Relationships

- 1 Brand &lt;-> 0/∞ Stores
- 1 Store &lt;-> 0/∞ Floors
- 1 Floor &lt;-> 0/∞ Nodes
- 1 Floor &lt;-> 0/∞ Edges
