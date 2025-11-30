# Decomposition to microservices

## Goal
Decompose a provided case into microservices

## Service 
I'll Have the BLT - https://nealford.com/katas/list.html

A national sandwich shop wants to enable 'fax in your order' but over the Internet instead (in addition to their current fax-in service)

Users: thousands, perhaps one day millions
### Requirements:
- Users will place their order, then be given a time to pick up their sandwich and directions 
to the shop (which must integrate with several external mapping services that includes 
traffic information) 
- if the shop offers a delivery service, 
dispatch the driver with the sandwich to the user  
- mobile-device accessibility  
- offer national daily promotionals/specials 
- offer local daily promotionals/specials
- accept payment online or in person/on delivery
### Additional Context:
- Sandwich shops are franchised, each with a different owner.
- Parent company has near-future plans to expand overseas.
- Corporate goal is to hire inexpensive labor to maximize profit.

## To do
- Use cases diagram
- C4 model - containers diagram
    - https://c4model.com/
    - https://c4model.com/diagrams/deployment
- Bounded Context diagram for each microservice
- Contract for each microservice

Examples
- https://github.com/team7katas/sysopsquad?tab=readme-ov-file#process-views
- https://github.com/tekiegirl/Archangels/blob/main/2.SolutionBackground/Conceptual.md

## Solution

### DDD 
First of all, let's use DDD to decompose the system into bounded contexts

#### Event storming in DDD
Miro: [link to miro](https://miro.com/app/board/uXjVJkF2MbQ=/?share_link_id=450228953536)

As a result there are 3 bounded contexts:
- Shop context which includes shopping, cooking and delivery;
- Shop management context;
- Corporate management context;
- Payment context.

Domain events are listed in a Miro board.
Let's consider shop context as a key context in this system.

### Stakeholders
This section describes key stakeholders of the system and their architectural concerns.

#### SH-1: Customer (Availability, performance, scalability, security)
User who places orders, pay and pick up or order delivery.

- system should be available and response time is critical
- security is important due to personal data and online payment 

#### SH-2 ShopOwner (Availability, Security)
User who gets orders, process them and ship them.

- shows should be notified about order and their changes in time
- security is important because ShopOwner works with payments, customer data and accountment 

#### SH-3 Courier (Availability, performance)
User who delivers orders.

- courier should be notified about order in time 
and use location services without any issue like delays

#### SH-4 Administrator (Scalability, security)
User who owns a parent company and manager franchisers.

- it should be easy to scale the system and expand franchise

#### SH-5 Payment processor (External system)
External service required for online payments.

### Use cases
Group by business flows

#### Customer flow
##### UC-1 Get a catalog of products and promotions (SH-1)
- Customer gets catalog of products
- Customer gets catalog of promotions

##### UC-2 Create and modify cart (SH-1)
- Customer adds products to a cart
- Customer modifies/removes products in a cart

##### UC-3 Checkout and payment (SH-1, SH-5)
- Customer gets a cart to review
- Customer gets a delivery estimation
- Customer creates an order 
- Customer pays
- Customer gets order confirmation

#### Order processing and delivery flow
##### UC-4 Shop processes order (SH-2)
- Shop gets a list of orders
- Shop changes a status to `in progress` for some order
- Shop changes s status to `completed` for some order
- Shop changes a status to `ready for pickup` for some order

##### UC-5 Customer picks up an order (SH-2, SH-1)
- Customer gets notified about order in `ready to pickup` status
- Shop changes a status to `completed` for some order

##### UC-6 Courier delivers an order (SH-3)
- Courier gets a list of orders to deliver
- Courier changes a status to `in delivery` for some order
- Courier changes s status to `completed` for some order

#### Business administration flow
##### UC-7 Administrator manages national promotions (SH-4)
- Administrator gets a list of promotions
- Administrator adds/removes promotions

##### UC-8 Shop manages local promotions (SH-3)
The same as UC-7 but a specific shop and with shop as main actor

##### UC-9 Administrator creates a new shop (SH-3, SH-4)
- Administrator gets a list of shops
- Administrator creates a new shop
- Shop gets notified about creation

##### UC-10 Administrator manages some shop (SH-3, SH-4)
- Administrator gets some shop
- Administrator updates shop data
- Shop gets notified about changes

### C4 Context diagram

https://mermaid.live/edit 
```mermaid

C4Context
    title System context diagram for online sandwich shop
    Person(customer, "Customer")
    Person(shopOwner, "Show Owner")
    Person(courier, "Courier")
    Person(administrator, "Administrator")

    Enterprise_Boundary(b0, "System") {

        System(shop, "Online Sandwich shop")
    }

    Enterprise_Boundary(b1, "External Providers") {
        System_Ext(paymentProvider, "Payment provider")
        System_Ext(emailProvider, "Email provider")
        System_Ext(smsProvider, "SMS provider")
        System_Ext(mappingProvider, "Mapping provider")
    }

    Rel(customer, shop, "Uses")
    Rel(shopOwner, shop, "Uses")
    Rel(courier, shop, "Uses")
    Rel(administrator, shop, "Uses")

    Rel(shop, paymentProvider, "withdraw money")
    Rel(shop, emailProvider, "post email")
    Rel(shop, smsProvider, "post SMS")
    Rel(shop, mappingProvider, "get directions and ETAs from")
```

### C4 Container diagram
(copy paste to mermaid live and zoom in/out)

```mermaid
C4Container
    title Container Diagram with Microservices

    Person(customer, "Customer")
    Person(shopOwner, "Shop Owner")
    Person(courier, "Courier")
    Person(administrator, "Administrator")

    System_Boundary(bSystem, "Sandwich Ordering System") {
    %% FRONTEND CLIENTS
        Container(customerMobileApp, "Customer Mobile App")
        Rel(customer, customerMobileApp, "Uses")

        Container(courierMobileApp, "Courier Mobile App")
        Rel(courier, courierMobileApp, "Uses")

        Container(webSite, "Customer Web App")
        Rel(customer, webSite, "Uses")

        Container(shopAdminWebSite, "Shop Admin Web App")
        Rel(shopOwner, shopAdminWebSite, "Uses")

        Container(franchiseAdminWebSite, "Franchise Admin Web App")
        Rel(administrator, franchiseAdminWebSite, "Uses")

        Container(apiGateway, "API Gateway", "Routes all API calls")

        Rel(customerMobileApp, apiGateway, "HTTP")
        Rel(courierMobileApp, apiGateway, "HTTP")
        Rel(webSite, apiGateway, "HTTP")
        Rel(shopAdminWebSite, apiGateway, "HTTP")
        Rel(franchiseAdminWebSite, apiGateway, "HTTP")

        Container(eventBroker, "Event Broker", "Kafka")

    %% SHOP CONTEXT
        System_Boundary(bShop, "Shop Context") {
            Container(orderService, "Order Service", "Microservice")
            ContainerDb(orderDb, "Order DB")

            Container(catalogService, "Catalog Service", "Provides read-only catalog")
            ContainerDb(catalogDb, "Catalog DB")

            Container(kitchenService, "Kitchen Service", "Tracks food preparation workflow")
            ContainerDb(kitchenDb, "Kitchen DB")

            Container(deliveryService, "Delivery Service", "Handles dispatching couriers")
            ContainerDb(deliveryDb, "Delivery DB")

            Rel(orderService, orderDb, "SQL CRUD")
            Rel(catalogService, catalogDb, "SQL CRUD")
            Rel(kitchenService, kitchenDb, "SQL CRUD")
            Rel(deliveryService, deliveryDb, "SQL CRUD")
        }

    %% SHOP MANAGEMENT CONTEXT
        System_Boundary(bShopMan, "Shop Management Context") {
            Container(menuService, "Menu Management Service")
            ContainerDb(menuDb, "Menu DB")

            Container(promotionService, "Local Promotion Service")
            ContainerDb(promoDb, "Local Promotions DB")

            Rel(menuService, menuDb, "SQL CRUD")
            Rel(promotionService, promoDb, "SQL CRUD")
        }

    %% CORPORATE CONTEXT
        System_Boundary(bCorpMan, "Corporate Management Context") {
            Container(franchiseService, "Franchise Management Service")
            ContainerDb(franchiseDb, "Franchise DB")

            Container(globalPromotionService, "Global Promotion Service")
            ContainerDb(globalPromoDb, "Global Promotions DB")

            Container(globalMenuService, "Global Menu Service")
            ContainerDb(globalMenuDb, "Global Menu DB")

            Rel(franchiseService, franchiseDb, "SQL CRUD")
            Rel(globalPromotionService, globalPromoDb, "SQL CRUD")
            Rel(globalMenuService, globalMenuDb, "SQL CRUD")
        }

        System_Boundary(bPayment, "Payment Context") {
            Container(paymentService, "Payment Service")
            ContainerDb(paymentDb, "Payment DB")

            Rel(paymentService, paymentDb, "SQL CRUD")
        }

        System_Boundary(bNotification, "Notification Context") {
            Container(notificationService, "Notification Service")
            ContainerDb(notificationDb, "Notification DB")

            Rel(notificationService, notificationDb, "SQL CRUD")
            Rel(notificationService, smsProvider, "Send SMS")
            Rel(notificationService, emailProvider, "Send EMAIL")
        }


    %% EXTERNAL SYSTEMS
        System_Ext(paymentProvider, "Payment Provider")
        System_Ext(emailProvider, "Email Provider")
        System_Ext(smsProvider, "SMS Provider")
        System_Ext(mappingProvider, "Mapping Provider")
    }

    %% REPLACE THESE THREE LINES WITH SPECIFIC SERVICE RELATIONSHIPS:
    Rel(apiGateway, orderService, "Place orders, check status")
    Rel(apiGateway, catalogService, "Get menu & promotions")
    Rel(apiGateway, deliveryService, "Courier assignments, delivery tracking")
    Rel(apiGateway, kitchenService, "Order preparation updates")
    Rel(apiGateway, menuService, "Shop menu management")
    Rel(apiGateway, promotionService, "Local promotion management")
    Rel(apiGateway, franchiseService, "Franchise administration")
    Rel(apiGateway, globalPromotionService, "Global promotion management")
    Rel(apiGateway, globalMenuService, "Global menu management")
    Rel(apiGateway, paymentService, "Payment processing")

    Rel(orderService, catalogService, "Get menu & promotions")
    Rel(orderService, paymentService, "Process payment")
    Rel(paymentService, paymentProvider, "External API call")

    Rel(orderService, eventBroker, "Publishes OrderPlaced event")
    Rel(kitchenService, eventBroker, "Subscribes to OrderPlaced")
    Rel(kitchenService, eventBroker, "Publishes OrderReady")
    Rel(deliveryService, eventBroker, "Subscribes to OrderReady")

    Rel(globalPromotionService, eventBroker, "Publishes GlobalPromotionCreated event")
    Rel(globalMenuService, eventBroker, "Publishes GlobalMenuItemCreated event")

    Rel(orderService, mappingProvider, "Requests pickup directions / traffic info")
```

Anyway the Container diagram above is pretty complex because it container microservices from
all bounderd contexts, so let's dive into a specific context and use case.

### C4 Container diagram for Shop context

```mermaid
C4Container
    title Shop Context - Order Fulfillment Workflow
    Enterprise_Boundary(c1, "Sandwich Ordering System") {
    
    Person(customer, "Customer", "Places order via app")
    
    Container(apiGateway, "API Gateway")
    Container(eventBroker, "Event Broker", "Kafka")
    
    System_Boundary(shopContext, "Shop Context") {
        Container(orderService, "Order Service", "Manages order lifecycle")
        ContainerDb(orderDb, "Order DB")
        
        Container(catalogService, "Catalog Service", "Provides menu & pricing")
        ContainerDb(catalogDb, "Catalog DB")
        
        Container(kitchenService, "Kitchen Service", "Manages food preparation")
        ContainerDb(kitchenDb, "Kitchen DB")
        
        Container(deliveryService, "Delivery Service", "Manages delivery logistics")
        ContainerDb(deliveryDb, "Delivery DB")
        
        Container(paymentService, "Payment Service", "Handles payments")
        ContainerDb(paymentDb, "Payment DB")

        Container(notificationService, "Notification Service", "Sends customer updates")
        ContainerDb(notificationDb, "Notification DB")
    }
    
    System_Ext(mappingProvider, "Mapping Provider", "Provides directions & traffic")
    System_Ext(paymentProvider, "Payment Provider", "Processes payments")
    
    %% Customer initiates order flow
    Rel(customer, apiGateway, "Submits order")
    Rel(apiGateway, orderService, "POST /orders")
    
    %% Order creation phase
    Rel(orderService, catalogService, "GET /menu/items", "Validate items & get prices")
    Rel(orderService, paymentService, "POST /payments", "Authorize payment")
    Rel(paymentService, paymentProvider, "Process payment")
    Rel(orderService, mappingProvider, "Get pickup ETA & directions")
    
    %% Event publishing - Order placed
    Rel(orderService, eventBroker, "OrderPlaced event")
    
    %% Kitchen workflow
    Rel(kitchenService, eventBroker, "Subscribes to OrderPlaced")
    Rel(kitchenService, orderService, "GET /orders/{id}", "Get order details")
    Rel(kitchenService, eventBroker, "OrderPreparing event")
    Rel(kitchenService, eventBroker, "OrderReady event")
    
    %% Delivery workflow
    Rel(deliveryService, eventBroker, "Subscribes to OrderReady")
    Rel(deliveryService, orderService, "GET /orders/{id}", "Get delivery address")
    Rel(deliveryService, mappingProvider, "Get delivery route")
    Rel(deliveryService, eventBroker, "DriverAssigned event")
    Rel(deliveryService, eventBroker, "OrderPickedUp event")
    Rel(deliveryService, eventBroker, "OrderDelivered event")
    
    %% Order completion
    Rel(orderService, eventBroker, "Subscribes to OrderDelivered")
    Rel(orderService, paymentService, "POST /payments/capture", "Capture payment")
    Rel(orderService, eventBroker, "OrderCompleted event")
    
    %% Notifications throughout the flow
    Rel(notificationService, eventBroker, "Subscribes to all order events")
    Rel(notificationService, customer, "SMS/Email/Push notifications")
    
    %% Data persistence
    Rel(orderService, orderDb, "Stores order state")
    Rel(catalogService, catalogDb, "Reads menu data")
    Rel(kitchenService, kitchenDb, "Stores preparation status")
    Rel(deliveryService, deliveryDb, "Stores delivery info")
    }
```
#### API endpoint
##### Catalog endpoints
- GET /catalog/categories - Get all menu categories
- GET /catalog/items - Get all menu items
- GET /catalog/items/{itemId} - Get specific menu item details
- GET /catalog/items?category={category} - Get items by category
- GET /catalog/search?q={query} - Search menu items
- GET /catalog/featured - Get featured/popular items 

#### Promotion endpoints
- GET /promotions - Get all promotions
- GET /promotions/{promotionId} - Get specific promotion details

##### Cart endpoints
it's only 1 cart per user.

- GET /cart - Get current user's cart
- POST /cart/items - Add item to cart
- PUT /cart/items/{itemId} - Update item quantity in cart
- DELETE /cart/items/{itemId} - Remove item from cart
- POST /cart/apply-promo - Apply promotion code
- DELETE /cart/promo - Remove promotion
- DELETE /cart - Clear/abandon cart

##### Checkout & Payment endpoints
- POST /cart/checkout - Checkout cart (validate + payment + create order)
- POST /payments/{orderId}/retry - Retry failed payment
- GET /payments/{orderId}/status - Check payment status

##### Order endpoints
- GET /orders - Get user's order history
- GET /orders/{orderId} - Get specific order details
- GET /orders/{orderId}/status - Get order status only
- POST /orders/{orderId}/cancel - Cancel order (if allowed)