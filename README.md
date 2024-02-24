# CQRS (Command Query Responsibility Segregation)

## Context
In large systems, there are many hidden logics. We will encounter many problems from using only one model to manage data. (Most of which are data access objects or ORMs). For example, querying data back requires very complex joins to satisfy a single view or updating data with many validations. Ultimately, you This model will accumulate more and more problems until it becomes a bottleneck in future maintenance.

## Decision
CQRS (Command Query Responsibility Segregation) is a control system concept that focuses on separating read operations (Query) and write operations (Command) in which data and operations are performed.

CQRS Framework is a framework and library for implementing and performing CQRS operations on web projects, various Command and Query based components. and the order of communication between the various parts.

The CQRS framework can be described as follows:

- Command Processing: The Framework collects and controls commands coming from users or external systems. Command Processing verifies commands and forwards them. to proceed further, such as writing data into technology
- Query Processing: For the reader, the data frame performs the task of sending the query in accordance with the required information. This part can be designed to work efficiently and support the management of large amounts of data.
- Event Handling: When a Command or Query is subject to a possible framework and directly affects the user or external system as appropriate.
- Infrastructure Support: The CQRS Framework requires tools or extensions to control the communication between different components. And the results are also sent to the interface and security and performance management of the system.
- Scalability and Performance Optimization: Framework system requirements or technology management system

Example normal model system

Example using CQRS

The command-side request lifecycle comes from the client sending a command through the API, and that command is processed by a command handler. This command handler then calls domain objects, such as aggregate roots in the domain layer, to process business logic. Before saving these domain objects into the database through the repository, there may be events that occur from various commands sent to the event handler that writes the changed data to the data storage on the other side for use. query only, which here is the case where we use asynchronous messaging to sync data on both sides in the case of using different data storage types. On the query side, there will be a query handler waiting to receive the query request from the API, then it will bypass the logic in The domain layer will call data access objects such as ORMs to query data from the read database and send it out to the client immediately.

Framework can do :
in actual use We will notice that since we have separate models for read and write, we can separate the data on both sides independently. This allows us to optimize scalability such as asymmetric read/write ratio, optimizing queries with materialized views to reduce aggregated data across tables or collections, as well as various use cases that may require specialized data storage, such as search that may be required Elasticsearch or use for various analytics tasks
A good starting point is that we can use a read replica with a read model.

Designing a system that uses a Data Access Model that separates read and write operations is an important point that allows it to support Advanced Use Cases, such as using Storage Technology that differs between read and write operations. such as MySQL and Elasticsearch And to improve the efficiency of queries on the read side (read), in this case there must be data sync between Write and Read Storage by using the Publish Event mechanism to come out every time the data is updated so that the Read side These events can be used to update data in themselves by using Messaging Tools such as RabbitMQ or Kafka as Event Bus to transmit these events. Therefore, the architecture of the system will look like this:
- Separate Read and Write Data Access Model: Separating the Data Access Model between read and write allows different storage technologies to be used in each process. without clause Limited or difficult to manage data
- Data Sync between Write and Read Storage: Using Mechanism to Publish Event every time data is updated will help the Read side to receive data and update the data itself continuously.
- Messaging Tool as Event Bus: RabbitMQ or Kafka is a tool suitable for use as an Event Bus to transmit events between Write and Read Storage.

On the read side, there is an event handler that receives events from the messaging channel and takes these events to create a materialized view that is ready to query immediately. Some people might call this component a denormaliser, which is a type of event handler that has the function of Receive events and create a materialised view. And because sending events through the event bus is a pub/sub format, it means that we can have multiple event handlers / denormalisers to create a materialised view to answer queries in various scenarios as needed.

Moreover, we can subscribe to events from other services to create materialized views as well, which will prevent us from having to request data via RPC in a synchronous way all the time (Event-Carried State Transfer).

## Rationale
CQRS is useful for building systems that are highly flexible and adapt well to change. Especially for systems with high complexity and many strange use cases, because we can optimize queries to support unlimited use cases without having to stick to just one model.

From the point of view of the development itself, bringing in CQRS adds another layer of separation of concern by dividing the service into two sides. Makes it easier for us to manage things in the system, optimizing the read / write part freely, especially in terms of performance, where we can choose technology and various tools to suit the situation more easily.

## Consequences
Pros 
- Efficient data analysis: Systems with SQRS data management often help in fast and efficient data analysis, such as big data processing. Creating a report or predicting various trends
- Easy data access: SQRS may help to store data systematically and provide easy access for users, for example through applications or web interfaces.
- Save time and resources: Using SQRS can help reduce data processing time and make the work process more efficient. It reduces manual work and reduces possible errors.
  
Cons
- Complexity: Building and maintaining an SQRS system can be difficult. This is due to the complexity of the structure and settings that must be adjusted according to the needs of the business.
- Cost: SQRS system development and maintenance can require a high investment budget. To make the system work efficiently and effectively
- Improvements and Upgrades: Improving and upgrading SQRS systems can be complex and time-consuming. This may cause businesses to face inconvenience during the time the system is being upgraded.

## Sample code

```
import java.util.HashMap;
import java.util.Map;

// Command side

// Define commands
interface Command {
    void execute();
}
class CreateProductCommand implements Command {
    private final String name;
    private final double price;

    public CreateProductCommand(String name, double price) {
        this.name = name;
        this.price = price;
    }

    @Override
    public void execute() {
        // Logic to create product
        System.out.println("Product " + name + " created with price " + price);
    }
}

// Query side

// Define queries
interface Query<T> {
    T execute();
}

class GetProductQuery implements Query<String> {
    private final int productId;

    public GetProductQuery(int productId) {
        this.productId = productId;
    }

    @Override
    public String execute() {
        // Logic to fetch product from database
        return "Product details for ID " + productId;
    }
}

// Command and Query Handlers

// Command handler
class CommandHandler {
    public void executeCommand(Command command) {
        command.execute();
    }
}

// Query handler
class QueryHandler {
    public <T> T executeQuery(Query<T> query) {
        return query.execute();
    }
}

public class Main {
    public static void main(String[] args) {
        // Example usage
        CommandHandler commandHandler = new CommandHandler();
        QueryHandler queryHandler = new QueryHandler();

        // Execute command to create a product
        Command createProductCommand = new CreateProductCommand("Laptop", 999.99);
        commandHandler.executeCommand(createProductCommand);

        // Execute query to get product details
        Query<String> getProductQuery = new GetProductQuery(123);
        String productDetails = queryHandler.executeQuery(getProductQuery);
        System.out.println(productDetails);
    }
}
```

Explaination
- Commands (Command and its implementations) represent actions that change the state of the system, such as creating a new product.
- Queries (Query<T> and its implementations) represent actions that retrieve data from the system without modifying state, such as fetching product details.
- Command handlers (CommandHandler) are responsible for executing commands.
- Query handlers (QueryHandler) are responsible for executing queries.
