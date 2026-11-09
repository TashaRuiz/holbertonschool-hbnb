# High-Level Package Diagram for Holberton HBnB (Task-0)

---

## What is the HBnB Project?
>HBnB is a simplified version of Airbnb, is for the Holberton student make a replica of it like a final project.

**Users can:**
- Create accounts
- List places (apartments, houses, etc.)
- Add amenities
- Write reviews
- Search and view places

## The Three-Layer Architecture
The application is organized into 3 layers. Each layer has a clear responsibility and should not do the work of another layer:

- **Presentation Layer-** Talks to the outside world (users / frontend)
- **Business Logic Layer-** Contains the real rules and entities of the app
- **Persistence Layer-** Saves and loads data

# Responsibilities of Each Layer:

  ## Presentation Layer
  - Receives HTTP requests
  - Validates input format
  - Calls the Facade
  - Returns JSON responses + status codes
  - Should never contain business rules

  ## Business Logic Layer
  - Contains the models (User, Place, Review, Amenity)
  - Contains business rules (example: a user cannot review their  own place)
  - Knows the relationships between entities

  ## Persistence Layer
  - Responsible only for saving and retrieving data
  - Can be:
    - File storage (JSON)
    - Database (SQLAlchemy / MySQL)
  - The rest of the application should not care which one is used

    ---

    # High-Level Package Diagram

```mermaid
classDiagram
    direction TB

    class PresentationLayer {
        <<layer>>
        +API Endpoints
        +Services / Controllers
        +Request / Response Handling
        +Input Validation
    }

    class Facade {
        <<Facade Pattern>>
        +create_user()
        +get_user()
        +get_user_by_email()
        +create_place()
        +get_place()
        +get_all_places()
        +create_review()
        +get_reviews_by_place()
        +create_amenity()
        +get_all_amenities()
    }

    class BusinessLogicLayer {
        <<layer>>
        +User
        +Place
        +Review
        +Amenity
        +Business Rules
        +Validation Logic
    }

    class PersistenceLayer {
        <<layer>>
        +Repository
        +FileStorage / DBStorage
        +CRUD Operations
    }

    PresentationLayer --> Facade : uses
    Facade --> BusinessLogicLayer : coordinates
    BusinessLogicLayer --> PersistenceLayer : persists / retrieves
```
  
---

# Business Logic Layer for Holberton HBnB (Task-1)

---

## Project Overview
>The objective of this project is to design a detailed UML Class Diagram representing the **Business Logic Layer** of the HBnB application.
    **BaseModel** is the parent class inherited by all four main entities. It provides common attributes and methods, avoiding code duplication.

The diagram illustrates the internal structure of the application, focusing on its four main entities:

* **User:** Represents registered users of the application.

* **Place:** Represents accommodation listings created by users.

* **Review:** Represents feedback and ratings submitted by users.

* **Amenity:** Represents facilities and features available at places.

## The Core Entities

1. **User**
    - Represents registered users (guests or hosts).
    - Key attributes: unique id (UUID4), email (unique), password (hashed in real implementations), names, is_admin flag, and the mandatory created_at / updated_at timestamps.
    - Core methods: CRUD operations (create, update, delete), serialization (to_dict), and validation.
    - Role: Owns places and authors reviews.

2. **Place**
    - Core entity representing a rental listing.
    - Key attributes: title/description, price, geolocation (latitude/longitude), capacity (max_guests, rooms, bathrooms), reference to the owner (owner_id), and timestamps.
    - Methods include standard CRUD + helpers for managing the many-to-many relationship with amenities.
    - Role: Belongs to one User (owner), can receive many Reviews, and can have many Amenities.

3. **Review**
    - User-generated feedback for a place.
    - Key attributes: review text, numeric rating (typically 1–5), foreign keys to User and Place, and timestamps.
    - Methods: standard CRUD + validation (e.g., rating range, ownership rules).
    - Role: Links a User to a Place. Business rules usually enforce: one review per user per place, and owners cannot review their own places.

4. **Amenity**
    - Catalog of features (WiFi, Pool, Parking, etc.).
    - Simple entity with id, name, and timestamps.
    - Role: Shared across places via a many-to-many association (implemented with a junction table such as place_amenity in the persistence layer).

---

# Business Logic Layer: Class Diagram

```mermaid
classDiagram
    direction TB

    class BaseModel {
        <<abstract>>
        +UUID id
        +DateTime created_at
        +DateTime updated_at
        +save()
        +update(data)
        +delete()
    }

    class User {
        +String first_name
        +String last_name
        +String email
        +String password
        +Boolean is_admin
        +register()
        +update_profile(data)
        +delete_account()
        +create_place()
        +create_review()
    }

    class Place {
        +String title
        +String description
        +Float price
        +Float latitude
        +Float longitude
        +update_details(data)
        +add_amenity(amenity)
        +remove_amenity(amenity)
        +get_reviews()
        +calculate_average_rating()
    }

    class Review {
        +String text
        +Integer rating
        +update_review(data)
        +delete_review()
        +validate_rating()
    }

    class Amenity {
        +String name
        +String description
        +update_amenity(data)
    }

    BaseModel <|-- User : inherits
    BaseModel <|-- Place : inherits
    BaseModel <|-- Review : inherits
    BaseModel <|-- Amenity : inherits

    User "1" --> "0..*" Place : hosts
    User "1" --> "0..*" Review : writes
    Place "1" --> "0..*" Review : receives
    Place "0..*" --> "0..*" Amenity : includes
```
---

# Sequence Diagrams for Holberton HBnB (Task-2)

---

## What for the Sequence Diagrams? 
>Illustrate the interactions between the Presentation, Business Logic, and Persistence layers of the HBnB application.

   - Each diagram demonstrates how an API request is received, validated, processed, and communicated with the database before returning a response to the client.
   - These diagrams also demonstrate the importance of separating responsibilities between the layers, making the application easier to maintain, test, and extend.
   - Important implementation note: These diagrams represent the intended architectural flow. Adapt service and model method names to match your actual HBnB implementation.

Below are four UML sequence diagrams for the HBnB application using Mermaid.js. They illustrate how the Presentation Layer, Business Logic Layer, and Persistence Layer interact to process API requests.

## 1. User Registration
**Objective:** Illustrate how a new user creates an account in HBnB.

**API Endpoint:** POST /api/v1/users/

### Description:
1. The user submits registration information (first name, last name, email, and password).
2. The API receives the request and forwards the information to the User model through the business logic layer.
3. The business logic validates the information and checks whether the email is already registered.
4. The database searches for an existing user with the same email.
5. If the email is available, the password is hashed and the new user is saved.
6. The API returns a successful response with the new user's information.

### Diagram:

```mermaid
---
title: User Registration
---

   sequenceDiagram
    autonumber
    actor User
    participant API as User API
    participant Service as User Service
    participant Model as User Model
    participant DB as Database

    User->>API: POST /api/v1/users/
    API->>Service: register_user(user_data)
    Service->>Model: validate_user(user_data)
    Model-->>Service: Validation successful

    Service->>DB: Check if email exists
    DB-->>Service: Email availability

    alt Email already exists
        Service-->>API: Email already registered
        API-->>User: 409 Conflict
    else Email is available
        Service->>Model: Hash password
        Model-->>Service: Hashed password
        Service->>DB: INSERT new user
        DB-->>Service: User created
        Service-->>API: Return user information
        API-->>User: 201 Created
    end
```

## 2. Place Creation
**Objective:** Illustrate how an authenticated user creates a new place listing.

**API Endpoint:** POST /api/v1/places/

### Description:
1. The user submits place information, including title, description, price, and location.
2. The API receives the request and verifies the user's authentication.
3. The business logic validates the submitted information.
4. The Place model processes the data and associates the listing with its owner.
5. The database saves the new place.
6. The API returns the newly created place information.

### Diagram:

```mermaid
---
title: Place Creation
---

sequenceDiagram
    autonumber
    actor User
    participant API as Place API
    participant Auth as Authentication Service
    participant Service as Place Service
    participant Model as Place Model
    participant DB as Database

    User->>API: POST /api/v1/places/
    API->>Auth: Verify authentication token
    Auth-->>API: User authenticated

    API->>Service: create_place(place_data, user_id)
    Service->>Model: validate_place(place_data)
    Model-->>Service: Validation successful

    Service->>Model: Assign owner(user_id)
    Model-->>Service: Place object created

    Service->>DB: INSERT new place
    DB-->>Service: Place saved

    Service-->>API: Return place information
    API-->>User: 201 Created

    alt Authentication failed
        Auth-->>API: Invalid or missing token
        API-->>User: 401 Unauthorized
    end

```

## 3. Review Submission
**Objective:** Illustrate how a user submits a review for an existing place.

**API Endpoint:** POST /api/v1/places/{place_id}/reviews/

### Description:
1. The user submits a review containing a rating and written comment.
2. The API receives the request and verifies the user's authentication.
3. The Review Service validates the review information.
4. The business logic checks whether the requested place exists.
5. The Review model verifies the rating and associates the review with the user and place.
6. The database stores the review.
7. The API returns a successful response.

### Diagram:

```mermaid
---
title: Place Creation
---

sequenceDiagram
    autonumber
    actor User
    participant API as Review API
    participant Auth as Authentication Service
    participant Service as Review Service
    participant Model as Review Model
    participant DB as Database

    User->>API: POST /api/v1/places/{place_id}/reviews/
    API->>Auth: Verify authentication token
    Auth-->>API: User authenticated

    API->>Service: create_review(review_data, user_id, place_id)

    Service->>DB: Find place by ID
    DB-->>Service: Place information

    alt Place does not exist
        Service-->>API: Place not found
        API-->>User: 404 Not Found
    else Place exists
        Service->>Model: validate_review(review_data)
        Model-->>Service: Validation successful

        Service->>Model: Associate user and place
        Model-->>Service: Review object created

        Service->>DB: INSERT new review
        DB-->>Service: Review saved

        Service-->>API: Return review information
        API-->>User: 201 Created
    end

    alt Authentication failed
        Auth-->>API: Invalid or missing token
        API-->>User: 401 Unauthorized
    end
```

## 4. Fetching a List of Places
**Objective:** Illustrate how a user retrieves a list of places based on search criteria.

**API Endpoint:** GET /api/v1/places/

### Description:
1. The user sends a request to retrieve available places, optionally including filters such as price, location, or number of guests.
2. The API receives the request and forwards the search criteria to the Place Service.
3. The business logic validates and processes the search filters.
4. The Place model prepares the query.
5. The database retrieves matching places.
6. The business logic processes the retrieved information.
7. The API returns the list of matching places to the user.

### Diagram:

```mermaid
---
title: Fetching a List of Places
---

sequenceDiagram
    autonumber
    actor User
    participant API as Place API
    participant Service as Place Service
    participant Model as Place Model
    participant DB as Database

    User->>API: GET /api/v1/places/?price=100&location=Paris
    API->>Service: get_places(search_filters)

    Service->>Model: validate_filters(search_filters)
    Model-->>Service: Filters validated

    Service->>Model: build_search_query(filters)
    Model-->>Service: Search query

    Service->>DB: SELECT places matching filters
    DB-->>Service: Matching places

    Service->>Model: Format place results
    Model-->>Service: Processed place list

    Service-->>API: Return places
    API-->>User: 200 OK + Places JSON

    alt Invalid search filters
        Service-->>API: Invalid filter parameters
        API-->>User: 400 Bad Request
    else No matching places
        DB-->>Service: Empty result
        Service-->>API: Empty places list
        API-->>User: 200 OK + Empty list
    end
```

## The four diagrams demonstrate how the three main layers of the HBnB application communicate.

```mermaid
---
title: Architecture Overview
---

flowchart TB
    subgraph PL["PRESENTATION LAYER"]
        API["API Endpoints"]
        SERVICES["Services / Controllers"]
    end

    subgraph BL["BUSINESS LOGIC LAYER"]
        MODELS["Models"]
        VALIDATION["Validation"]
        RULES["Business Rules"]
    end

    subgraph PERSISTENCE["PERSISTENCE LAYER"]
        DB[("Database")]
    end

    User((User)) --> API
    API --> SERVICES
    SERVICES --> MODELS
    MODELS --> VALIDATION
    VALIDATION --> RULES
    RULES --> DB
    DB --> RULES
    RULES --> MODELS
    MODELS --> SERVICES
    SERVICES --> API
    API --> User

    style PL fill:#dceeff,stroke:#3976a8,color:#172b3a
    style BL fill:#e8f5e9,stroke:#43834b,color:#172b3a
    style PERSISTENCE fill:#fff0d9,stroke:#b17b32,color:#172b3a
```

---

## Author
>Developed by Tasha Arroyo, a Holberton School student 
