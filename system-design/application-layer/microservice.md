# 微服务

微服务架构是一种开发软件系统的方式，微服务专注于构建多个单一功能的模组，其中包含大量接口和操作。使用微服务构建软件是如今的趋势，它使软件开发变得更敏捷，同时更容易转向DevOps并进行持续测试。

微服务拥有简单的形式，它通过一些小服务来构建应用，这些服务都在自己的进程中运行，并且可独立部署。这些服务可以使用不同的语言进行编写，也可以使用不同的数据储存方式，总之各个服务只需要关心自己的实现。在开发灵活可扩展的系统时，系统需要具有动态改善的能力，微服务通常使用API进行链接，可以利用RESTful和Web服务生态系统中增长的许多相同工具和解决方案。测试这些API可以帮助验证整个微服务部署中的数据和信息流。

## 微服务的好处

| **Simpler To Deploy** | Deploy in literal pieces without affecting other services. |
| :--- | :--- |
| **Simpler To Understand** | Follow code easier since the function is isolated and less dependent. |
| **Reusability Across Business** | Share small services like payment or login systems across the business. |
| **Faster Defect Isolation** | When a test fails or service goes down, isolate it quickly with microservices. |
| **Minimized Risk Of Change** | Avoid locking in technlogies or languages - change on the fly without risk. |

## 微服务的六个特性

### **多组建**

Software built as microservices can, by definition, be broken down into multiple component services.  Why?  So that each of these services can be deployed, tweaked, and then redeployed independently without compromising the integrity of an application.  As a result, you might only need to change one or more distinct services instead of having to redeploy entire applications.  But this approach does have its downsides, including expensive remote calls \(instead of in-process calls\), coarser-grained remote APIs, and increased complexity when redistributing responsibilities between components.

### 专为业务打造

The microservices style is usually organized around business capabilities and priorities.  Unlike a traditional monolithic development approach—where different teams each have a specific focus on, say, UIs, databases, technology layers, or server-side logic—microservice architecture utilizes cross-functional teams.  The responsibilities of each team are to make specific products based on one or more individual services communicating via message bus.  In microservices, a team owns the product for its lifetime, as in Amazon’s oft-quoted maxim “[You build it, you run it](http://www.strehle.de/tim/weblog/archives/2010/11/09/1320)“.

### 简单路由

Microservices act somewhat like the classical UNIX system: they receive requests, process them, and generate a response accordingly.  This is opposite to how many other products such as ESBs \(Enterprise Service Buses\) work, where high-tech systems for message routing, choreography, and applying business rules are utilized.  You could say that microservices have smart endpoints that process info and apply logic, and dumb pipes through which the info flows.

### 去中心化

Since microservices involve a variety of technologies and platforms, old-school methods of centralized governance aren’t optimal. Decentralized governance is favored by the microservices community because its developers strive to produce useful tools that can then be used by others to solve the same problems. Just like decentralized governance, microservice architecture also favors decentralized data management. Monolithic systems use a single logical database across different applications. In a microservice application, each service usually manages its unique database.

### 耐故障

Like a well-rounded child, microservices are designed to cope with failure.  Since several unique and diverse services are communicating together, it’s quite possible that a service could fail, for one reason or another \(e.g., when the supplier isn’t available\).  In these instances, the client should allow its neighboring services to function while it bows out in as graceful a manner as possible. However, [monitoring microservices](https://smartbear.com/en-us/learn/performance-monitoring/monitoring-microservices/) can help prevent the risk of failure. For obvious reasons, this requirement adds more complexity to microservices as compared to monolithic systems architecture.

### 进化的

Microservices architecture is an evolutionary design and, again, is ideal for evolutionary systems where you can’t fully anticipate the types of devices that may one day be accessing your application. Many applications start based on monolithic architecture, but as several unforeseen requirements surfaced, can be slowly revamped to microservices that interact over an older monolithic architecture through APIs.

## 微服务优缺点

Microservices are not a silver bullet, and by implementing them you will expose communication, teamwork, and other problems that may have been previously implicit but are now forced out into the open. But [API Gateways in Microservices](https://smartbear.com/learn/api-design/api-gateways-in-microservices/) can greatly reduce build and QA time and effort.

One common issue involves sharing schema/validation logic across services.  What A requires in order to consider some data valid doesn’t always apply to B, if B has different needs.  The best recommendation is to apply versioning and distribute schema in shared libraries.  Changes to libraries then become discussions between teams.  Also, with strong versioning comes dependencies, which can cause more overhead.  The best practice to overcome this is planning around backward compatibility, and accepting [regression tests](https://smartbear.com/learn/automated-testing/what-is-regression-testing/) from external services/teams.  These prompt you to have a conversation _before_ you disrupt someone else’s business process, not after.

As with anything else, whether or not microservice architecture is right for you depends on your requirements, because they all have their pros and cons. 

### 优点

* Microservice architecture gives developers the freedom to independently develop and deploy services
* A microservice can be developed by a fairly small team
* Code for different services can be written in different languages \(though many practitioners discourage it\)
* Easy integration and automatic deployment \(using open-source continuous integration tools such as Jenkins, Hudson, etc.\)
* Easy to understand and modify for developers, thus can help a new team member become productive quickly
* The developers can make use of the latest technologies
* The code is organized around business capabilities
* Starts the web container more quickly, so the deployment is also faster
* When change is required in a certain part of the application, only the related service can be modified and redeployed—no need to modify and redeploy the entire application
* Better fault isolation: if one microservice fails, the other will continue to work \(although one problematic area of a monolith application can jeopardize the entire system\)
* Easy to scale and integrate with third-party services
* No long-term commitment to a technology stack

### 缺点

* Due to distributed deployment, testing can become complicated and tedious
* The increasing number of services can result in information barriers
* The architecture brings additional complexity as the developers have to mitigate fault tolerance, network latency, and deal with a variety of message formats as well as load balancing
* Being a distributed system, it can result in duplication of effort
* When the number of services increases, integration and managing whole products can become complicated
* In addition to several complexities of monolithic architecture, the developers have to deal with the additional complexity of a distributed system
* Developers have to put additional effort into implementing the mechanism of communication between the services
* Handling use cases that span more than one service without using distributed transactions is not only tough but also requires communication and cooperation between different teams

## 微服务架构工作机制

### Monoliths and Conway’s Law

To begin with, let’s explore [Conway’s Law](http://en.wikipedia.org/wiki/Conway%27s_law), which states: _“Organizations which design systems…are constrained to produce designs which are copies of the communication structures of these organizations.”_

Imagine Company X with two teams: _Support_ and _Accounting_.  Instinctively, we separate out the high-risk activities; it’s only difficult deciding responsibilities like customer refunds.  Consider how we might answer questions like “Does the Accounting team have enough people to process both customer refunds and credits?” or “Wouldn’t it be a better outcome to have our Support people be able to apply for credits and deal with frustrated customers?”  The answers get resolved by Company X’s new policy: Support can apply a _credit_, but Accounting has to _process a refund_ to return money to a customer.  The roles and responsibilities in this interconnected system have been successfully split while gaining customer satisfaction and minimizing risks.

Likewise, at the beginning of designing any software application, companies typically assemble a _team_ and create a _project_.  Over time, the team grows, and multiple projects on the same codebase are completed.  More often than not, this leads to competing projects: two people will find it difficult to work at cross purposes in the same area of code without introducing tradeoffs.  And adding more people to the equation only makes the problem worse.  As [Fred Brooks](http://en.wikipedia.org/wiki/Brooks%27s_law) puts it, nine women can’t make a baby in one month.

Moreover, in Company X or in any dev team, priorities frequently shift, resulting in management and communication issues.  Last month’s highest priority item may have caused our team to push hard to ship code, but now a user is reporting an issue, and we no longer have time to resolve it because of _this_ month’s priority.  This is the most compelling reason to adopt SOA, including the microservices variety.  Service-oriented approaches recognize the frictions involved between change management, domain knowledge, and business priorities, allowing dev teams to explicitly separate and address them.  Of course, this in itself is a tradeoff—it requires coordination—but it allows you to centralize friction and introduce efficiency, as opposed to suffering from a large number of small inefficiencies.

Most importantly, smartly implementing an SOA or microservice architecture forces you to apply the [Interface Separation Principle](http://en.wikipedia.org/wiki/Interface_segregation_principle).  Due to the connected nature of mature systems, when isolating issues of concern, the typical approach is to find a seam or communication point and then draw a dotted line between two halves of the system.  Without careful thought, however, this can lead to accidentally creating two smaller but growing monoliths, now connected with some kind of bridge.  The consequence of this can be marooning important code on the wrong side of a barrier: Team A doesn’t bother to look after it, while Team B needs it, so they reinvent it.

### Avoiding the Monoliths

We’ve named some problems that commonly emerge; now let’s begin to look at some solutions.

How do you deploy relatively independent yet integrated services without spawning accidental monoliths?  Well, suppose you have a large application, as in the sample from our Company X below, and are splitting up the codebase and teams to scale.  Instead of finding an entire section of an application to split off, you can look for something on the _edge_ of the application graph.  You can tell which sections these are because nothing depends on them.  In our example, the arrows pointing to Printer and Storage suggest they’re two things that can be easily removed from our main application and abstracted away.  Printing either a Job or Invoice is irrelevant; a Printer just wants printable data.  Turning these-Printers and Storage-into external services avoids the monoliths problem alluded to before.  It also makes sense as they are used multiple times, and there’s little that can be reinvented.  Use cases are well known from past experience, so you can avoid accidentally removing key functionality.

![MicroServices Architecture](https://smartbear.com/getattachment/all-resources/Articles/What-Is-Microservices-Architecture/microservices_image1.png?width=500&height=352)

### Service Objects and Identifying Data

So how do we go from monoliths to services?  One way is through _service objects_.  Without removing code from your application, you effectively just begin to structure it as though it were completely external.  To do that, you’ll first need to differentiate the _actions_ that can be done and the _data_ that is present as inputs and outputs of those actions.  Consider the code below, with a notion of _doing something useful_ and a _status of that task_.

```text
# A class to model a core transaction and execute it

      class Job
        def initialize
          @status = 'Queued'
        end
        
        def do_useful_work
          ....
          @status = 'Finished'
        end
        
        def finished?
          return @status == 'Finished'
        end
        
        def ready?
          return @status == 'Queued'
        end
      end
```

To prepare this to begin looking like a microservice, what’s next?

```text
# Service to do useful work and modify a status

    class JobService
      def do_useful_work(job_status)
        ....
        
        job_status.finish!
        
        return job_status
      end
    end

    # A model of our Job's status

    class JobStatus
      def initialize
        @status = 'Queued'
      end
      
      def finished?
        return @status == 'Finished'
      end
      
      def ready?
        return @status == 'Queued'
      end
      
      def finish!
        @status = 'Finished'
      end
    end
```

Now we’ve distinguished two distinct classes: one that models the data, and one that performs the operations.  Importantly, our JobService class has little or no state—you can call the same actions over and over, changing only the data, and expect to get consistent results.  If JobService somehow started taking place over a network, our otherwise monolithic application wouldn’t care.  Shifting these types of classes into a library, and substituting a network client for the previous implementation, would allow you to transform the existing code into a scalable external service.

This is [Hexagonal Architecture](http://alistair.cockburn.us/Hexagonal+architecture), where the core of your application and the coordination is in the center, and the external components are orchestrated around it to achieve your goals.

![hexagonalarchitecture.png](https://smartbear.com/getattachment/all-resources/Articles/What-Is-Microservices-Architecture/hexagonalarchitecture.png)

\(You can read more about service objects and hexagonal architecture [here](http://blog.mattwynne.net/2012/05/31/hexagonal-rails-objects-values-and-hexagons/) and [here](https://blog.engineyard.com/2014/keeping-your-rails-controllers-dry-with-services).\)

### Coordination and Dumb Pipes

Now let’s take a closer look at what makes something a microservice as opposed to a traditional SOA.

Perhaps the most important distinction is the _side effects_.  Microservices avoid them.  To see why, let’s look at an older approach: Unix pipes.

`ls | wc -l`

Above, two programs are chained together: the first lists all of the files in a directory, the second reads the number of lines in a stream of input.  Imagine writing a comparable program, then having to modify it into the below:

`ls | less`

Composing small pieces of functionality relies on repeatable results, a standard mechanism for input and output, and an exit code for a program to indicate success or lack thereof.  We know this works from observational evidence, and we also know that a Unix pipe is a “dumb” interface because it has no control statements.  The pipe applies SRP by pushing data from A to B, and it’s up to members of the pipeline to decide if the input is unacceptable.

Let’s go back to Company X’s Job and Invoice systems.  Each controls a transaction and can be used together or separately: Invoices can be created for jobs, jobs can be created without an invoice, and invoices can be created without a job.  Unlike Unix shell commands, the systems that own jobs and invoices have their own users working independently.  But without falling back to a _policy_, it’s impossible to enforce rules for either system globally.

Say we want to extract out the key operations that can be repeatedly executed—the services for sending an invoice, mutating a job status and mutating an invoice status.  These are completely separate from the task of _persisting_ data.

![illing.png](https://smartbear.com/getattachment/all-resources/Articles/What-Is-Microservices-Architecture/microservices_image-pipelines.png?width=500&height=335)

Here this allows us to wire together the discrete components into two pipelines:

**User creates a manual invoice**

* Adds data to invoice, a status _created_

  — Invokes BillingPolicyService to determine when an invoice is payable for a given customer

* Invoice is issued to the customer
* Persists to the invoice data service, status _sent_

**User finishes a job, creating an invoice**

* Validates job is completable
* Adds data to invoice, a status _created_

  —Invokes BillingPolicyService to determine when an invoice is payable for a given customer

* Invoice is issued to the customer
* Persists to the invoice data service, status _sent_

The invoice calculation related steps are idempotent, and it’s then trivial to compose a _draft invoice_ or preview the amounts payable by the customer by leveraging our new dedicated microservices.

Unlike traditional SOA, the difference here is that we have low-level details exposed via a simple interface, as compared to a high-level API call that might execute an entire business action.  With a high-level API, in fact, it becomes difficult to rewire small components together, since the service designer has removed many of the seams or choices we can take by providing a one-shot interface.

By this point, the repetition of business logic, policy, and rules leads many to traditionally push this complexity into a service bus or singular, centralized workflow orchestration tool.  However, the crucial advantage of microservice architecture is not that we _never_ share business rules/processes/policies, but that we push them into discrete packages, aligned to business needs.  Not only does this mean that policy is distributed, but it also means that _you can change your business processes without risk_.

