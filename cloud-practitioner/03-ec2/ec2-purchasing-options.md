# Purchasing Options

## 1. On-Demand Instances

- Pay for what you use:
  Linux or windows: biling per second, after the first minute
  All other OS: billing per hour
- Has the highest cost but no upfront payment
- No long-term commitment
  -> Recommend for short-term and uninterrupted workloads, where you can't predict how the application will behave

## 2. Reserved Instances ( 1 or 3 years )

- Up to 72% discount compared to On-demand
- You reserve a specific instance attributes ( Instance Type, Region, Tenancy, OS )
- Reservation Period - 1 year ( + discount ) or 3 years ( +++ discount )
- Payment Options - No Upfront ( + ), Partial Upfront ( ++ ), All Upfront ( +++ )
- Reserved Instance's Scope - Regional or Zonal ( reserve capacity in an AZ )
  -> Recommended for steady-state usage applications ( like databases )
- You can buy and sell in the Reserved Instance Marketplace
- Convertible Reserved Instances
  Can change the EC2 instance type, instance family, OS, scope and tenancy
  Up to 66% discount

## 3. Instance Savings Plan ( 1 or 3 years )

- Get a discount based on long-term usage ( same as RIs )
- Commit to a certain type of usage ( $10/hour for 1 or 3 years )
- Usage beyond EC2 Savings Plan is billed at the On-Demand price
- Locked to a specific instance family and AWS region
- Flexible across:
  Instance size
  OS
  Tenancy

## 4. Spot Instances

- Can get a discount of up to 90% compared to On-demand
- Instances that you can ""lose"" at any point of time if your max price is less than the current spot price
- The most cost-efficient instances in AWS
- Useful for workloads that are resilient to failure
  Batch jobs
  Data analysis
  Image processing
  Any distributed workloads
  Workloads with a flexible start and end time
- Not suitable for critical jobs or database

## 5. Dedicated Hosts

- A physical server with EC2 instance capacity fully dedicated to your use
- Allows you address compliance requirements and use your existing server-bound software licenses ( per-socket, per-core, pe-VM software licenses)
- Purchasing options:
  On-demand - pay per second for active Dedicated Host
  Reserved: 1 or 3 year ( No upfront, Partial Upfront, All Upfront)
- The most expensive option
- Useful for software that have complicated licensing model ( BYOL - Bring your own license ) or for companies that have strong regulatory or compliance needs

## 6. Dedicated Instances

- Instances run on hardware that's dedicted to you
- May share hardware with other instances in same account
- No control over instance placement ( can move hardware after Stop / Start )

## 7. Capacity Reservations

- Reserve On-Demand instances capacity in a specific AZ for any duration
- You always have access to EC2 capacity when you need it
- No time commitment ( create / cancel anytime ), no billing discounts
- Combine with Regional Reserved Instances and Savings Plans to benefit from billing discounts
- You're charged at On-Demand rate whether you run instances or not
- Suitable for short-term, uninterrupted workloads that needs to be in a specific AZ"
