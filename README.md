# CST8915 Lab 8 – Algonquin Pet Store on AKS

**David Yin**

[Watch the Lab 8 demo on YouTube](https://youtu.be/5mecaMvbo88)

## Task 1 – Deploy the pet store

I built eight images from my service forks and pushed them to Docker Hub under `yanky65` with the `lab8` tag. I deployed them to a two-node AKS cluster in Mexico Central and scaled Order Service to three replicas.

| File | What it deploys |
| --- | --- |
| [aps-all-in-one-Task1.yaml](aps-all-in-one-Task1.yaml) | Store Front, Store Admin, Order Service, Product Service, Makeline Service, and AI Service, along with MongoDB and RabbitMQ. |
| [admin-tasks.yaml](admin-tasks.yaml) | Virtual Customer places orders. Virtual Worker completes them. |
| [config-maps.yaml](config-maps.yaml) | The RabbitMQ plugin configuration used by the main manifest. |

## Task 2 – Azure managed services

**MongoDB → Azure DocumentDB.** Makeline Service stores orders in the `orderdb.orders` collection. Azure DocumentDB supports the MongoDB driver the service already uses. Moving that collection to a managed database would keep order history available when AKS pods are replaced. Azure handles database patching, and paid tiers include backups. [Azure DocumentDB overview](https://learn.microsoft.com/en-us/azure/documentdb/overview)

**RabbitMQ → Azure Service Bus.** A Service Bus queue could hold incoming orders until Makeline Service is ready to process them. The consumer could lock an order, save it to the database, then acknowledge it. If the consumer stops before acknowledging the order, Service Bus can deliver it again. This moves pending orders outside AKS and removes the work of maintaining the broker. Both services would need Service Bus clients, and Makeline would need to handle duplicate deliveries. [Service Bus message handling](https://learn.microsoft.com/en-us/azure/service-bus-messaging/message-transfers-locks-settlement)
