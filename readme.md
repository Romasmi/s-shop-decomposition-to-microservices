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

#### UC-1 Place order
- Customer gets catalog of sandwiches (SH-1)
- Customer gets list of promotionals/specials (SH-1)
- Customer makes order, chooses a delivery (optionally) and pays (SH-1, SH-2, SH-5)
- (optionally) Customer gets location of a shop to pick up order (SH-2) 
- Customer receives order confirmation (analog of fax) (SH-1)

#### UC-2 Deliver order

- Shop dispatches a courier (SH-2)
- Courier process order - pick up, ship, deliver (SH-3)

#### UC-3 Manager franchise.
- Administrator creates a new franchiser (SH-4)