# High Availability, Scalability, Elasticity

## Scalability

- Scalability means that an application / system can handle greater loads by adapting
- There are two kinds of scalability:
  Vertical scalability
  Horizontal scalability ( = elasticity )
- Scalability different to High Avalibility

## Vertical Scalability ( = scale up / down )

- Increasing the size of the instance
- Very common for non distributed systems, such as a database
- There's usually a limit to how much you can vertically scale ( hardware limit )

## Horizontal Saclability ( = scale out / in )

- Increasing the number of instances / systems for your application
- Implies distributed systems
- This is very common for web applications / modern applications

## High Availability

- Usually goes hand in hand with horizontal scaling
- Means running your application / system in at least 2 AZs
- The goal of high availability is to survive a data center loss ( disaster )

## Scalability vs Elasticity vs Agility

- Scalability
  Ability to handle increased workloads by upgrading hardware (Vertical Scaling) or adding more instances (Horizontal Scaling).
- Elsaticity
  Automatically scale resources up/down based on demand → Optimize costs, pay-per-use.
- Agility
  Quickly provision IT resources in minutes instead of weeks."
