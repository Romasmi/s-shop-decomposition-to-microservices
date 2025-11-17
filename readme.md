# Decomposition to microservices

## Goal
Decompose a provided case into microservices

## Service 
I'll Have the BLT - https://nealford.com/katas/list.html

A national sandwich shop wants to enable 'fax in your order' but over the Internet instead (in addition to their current fax-in service)

Users: thousands, perhaps one day millions
### Requirements:
- Users will place their order, then be given a time to pick up their sandwich and directions 
to the shop (which must integrate with several external mapping services that include 
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

