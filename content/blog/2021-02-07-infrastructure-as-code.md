+++
title = "Infrastructure as Code"
description = "Notes from the book"
date = 2021-02-07
updated = 2025-03-12

[taxonomies]
tags = ["Infrastructure", "Automation", "Quality"]

[extra]
social_media_card = "img/social_cards/blog_infrastructure_as_code.jpg"
+++

# Introduction

This is a summary of the book **Infrastructure as Code (2nd edition)** by Kief Morris, where I state the things that stood out the most to
me.

# What is Infrastructure as Code?

Organizations are becoming increasingly "digital" [^1], and as they do, the IT infrastructure becomes more and more complex: more services,
more users, more business activities, suppliers, products, customers, stakeholders... and the list goes on and on.

[^1]: short for 'software systems are essential for our business'

Infrastructure automation tools help manage this complexity by keeping the entire infrastructure as code. This will help by optimizing for
change. People say: we don’t make changes often enough to justify automating them; we should build first, automate later; we must choose
between speed and quality. These all lead to a "Fragile Mess"; changing the infrastructure becomes a cumbersome error-prone process.

When prioritizing quality, many organizations put in place complex processes to change even the tiniest detail of their infrastructure;
which eventually are forgotten due to an approaching deadline. Other companies prioritize speed (move fast and break things); infrastructure
that "just works, but no one knows how" is a very dangerous thing to have. Making changes to it becomes more an obscure art than a clear
process. The only sustainable way of maintaining infrastructure is by prioritizing equally both quality and speed; this may seem like an
unattainable ideal, but it is where we find high performers.

[Dora](https://www.devops-research.com/research.html) identified four key metrics in their _Accelerate_ research:

1. **Delivery lead time**: The elapsed time it takes to implement, test, and deliver changes to the production system
2. **Deployment frequency**: How often you deploy changes to production systems
3. **Change fail percentage**: What percentage of changes either cause an impaired service or need immediate correction, such as a rollback
   or emergency fix
4. **Mean Time to Restore (MTTR)**: How long it takes to restore service when there is an unplanned outage or impairment Organizations that
   perform well against

# Core Practices

- **Define everything as code**: Enables reusability, consistency and transparency
- **Continuously test and deliver all work in progress**: Build quality in instead of trying to test quality in
- **Build small, simple pieces that you can change independently**: The larger a system is, the harder it is to change, and the easier it is
  to break.

# Principles

- **Assume systems are unreliable**: Cloud scale infrastructure has so many moving parts, that even when using reliable hardware, systems
  fail
- **Make everything reproducible**: It removes the fear and risk of making changes
- **Create disposable things**: It should be possible to add, remove, start, stop, change and move parts of the the system. The cloud
  abstracts resources (storage, compute, networking) from physical hardware. The system should be able to dispose faulty parts and heal
  itself.
- **Minimize variation**: Keep the minimum number of different _types_ of pieces possible; It’s easier to manage one hundred identical
  servers than five completely different servers.
- **Ensure you can repeat any process**: This will help make things reproducible

Of course, to use code to define and manage infrastructure, a dynamic infrastructure platform is required. The platform should expose its
functionality to provision resources via APIs or something of the sorts, think Amazon Web Services, Microsoft Azure, Google Cloud Platform,
Digital Ocean, even VMWare.

# Infrastructure code

If I want to create a server, it is easier, and faster to go to the platform (AWS) and click through the GUI instead of writing a script for
it. On the other hand, provisioning it through code will make the process reusable, consistent and transparent. It is obvious that defining
this server as code, is the better approach. Defining everything as code enables you to leverage speed to improve quality, much like Agile
uses speed to improve software quality by having tight feedback loops and iterating on that feedback.

Infrastructure code can use declarative or imperative languages[^2].

[^2]: Imperative code is a set of instructions that specifies how to make a thing happen. Declarative code specifies what you want, without
specifying how to make it happen.

# Infrastructure stacks

An infrastructure stack is a group of resources that are defined, provisioned and updated as a unit. For example: A stack may include a
virtual machine, a disk volume and a subnet.

Examples of stack management tools include:

- [HashiCorp Terraform](https://www.terraform.io/)
- [AWS CloudFormation](https://aws.amazon.com/cloudformation/)
- [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview)
- [Google Cloud Deployment Manager](https://cloud.google.com/deployment-manager/docs/)
- [OpenStack Heat](https://wiki.openstack.org/wiki/Heat)
- [Pulumi](https://www.pulumi.com/)
- [Bosh](https://bosh.io/docs/)

## Patterns and Antipatterns for Structuring Stacks

One challenge with infrastructure design is deciding how to size and structure stacks.

### Antipatterns

- **Monolithic stack**: An entire system into one stack. Changing a large stack is riskier than changing a smaller stack. More things can go
  wrong. Larger stacks take longer to provision and change. Because of the risk and slowness, people make less changes to it, and with less
  frequency. This increases the levels of technical debt

### Patterns

- **Application group stack**: Multiple, related pieces of a system into stacks. It's common for application group stacks to grow into
  monolithic stacks; but it can work well when a single team owns the infrastructure of all the pieces of an application.
- **Service stack**: Infrastructure for a single application into a single stack. Service stacks align the boundaries of infrastructure to
  the software that runs on it. There could be an unnecessary duplication of code, which can encourage inconsistency. Reusing shareable
  modules is encouraged.
- **Micro stack**: Breaks the infrastructure for a given application or service into multiple stacks. Different parts of a service’s
  infrastructure may change at different rates. Although parts are smaller, having many of them increases complexity.

# Building Environments with Stacks

Environments and stacks are both collections of resources. What makes environments different is that they are organized around a particular
purpose (support the testing phase, provide service in a geographical region). An environment is composed by one or multiple stacks.
Although possible, a stack should not provision multiple environments.

Consistency across environments is one of the main drivers of Infrastructure as Code. If the testing environment is not the same as the one
in production, you may find that some things behave differently; you may even push broken code that "works" in the testing environment. How
many time have we heard "it runs okay in my local environment"?

## Antipattern: Multi-Environment Stack

It defines and manages multiple environments in a single stack instance. Every time you change a testing environment, you risk breaking the
production if there was a mistake, a bug in the tools or somewhere in the pipeline.

## Antipattern: Copy-Paste Environment

It uses separate source-code projects to manage each environment. Because environments should be identical, people resort to copy-pasting
the code from the modified stack into the other ones. As you can imagine, this can get messy pretty fast. Maybe some tweaks to reduce costs
are incompatible with the production environment. This causes configuration drift, and makes it confusing for people looking at the code for
the first time.

## Pattern: Reusable Stack

Here, you maintain a single source-code project, with enough configuration parameters to provision all the required environments. However,
it could be too rigid for situations where environments must be heavily customized.

The reusable stack should be the foundation for any new environment. There are several ways to configure the configuration parameters it
requires to be unique, let's look at those.

## Antipattern: Manual Stack Parameters

The most natural approach to provide values for a stack instance is to type the values on the command line manually.

```bash
stack up environment=production --source my-stck/src
# FAILURE: No such directory 'my-stck/src'

stack up environment=production --source my-stack/src
# SUCCESS: new stack 'production' created

stack destroy environment=production --source my-stack/src
# SUCCESS: stack 'production' destroyed

stack up environment=production --source my-stack/src
# SUCCESS: existing stack 'production' modified
```

## Pattern: Stack Environment Variables

The stack environment variables pattern involves setting parameter values as environment variables for the stack tool to use. This pattern
is often combined with another pattern to set the environment variables.

```bash
export STACK_ENVIRONMENT=test
export STACK_CLUSTER_MINIMUM=1
export STACK_CLUSTER_MAXIMUM=1
export STACK_SSL_CERT_PASSPHRASE="correct horse battery staple"
```

## Pattern: Scripted Parameters

Scripted parameters involves hardcoding the parameter values into a script that runs the stack tool. You can write a separate script for
each environment or a single script that includes the values for all of your environments

```ruby
if ${ENV} == "test"
  stack up cluster_maximum=1 env="test"
elsif ${ENV} == "staging"
  stack up cluster_maximum=3 env="staging"
elsif ${ENV} == "production"
  stack up cluster_maximum=5 env="production"
end
```

## Pattern: Stack Configuration Files

We can manage the parameters for each stack instance in separate files.

For example:

```text
├── src/
│   ├── cluster.tf
│   ├── host_servers.tf
│   └── networking.tf
├── environments/
│   ├── test.tfvars
│   ├── staging.tfvars
│   └── production.tfvars
└── test/
```

This pattern is simple and very useful when the environments don't change often, as it requires to create and commit a new configuration
file per environment. It is also slower to reach the stable production environments, since these files have to progress through the various
stages before being committed to the main branch.

## Pattern: Wrapper Stack

Let's say we write our infrastructure code as reusable modules, kind of like a library. A wrapper stack is a code project that imports and
uses those modules, passing the appropriate parameters to them. In that sense, every environment is its own code project. We can set up
independent repositories for them using git. The only thing we must make sure does not get into these repos are secrets, for those we can
leverage other patterns, like using environment variables, or untracked configuration files.

There's a catch, though. There is added complexity by having to manage the reusable modules and the projects that use them. Furthermore,
having separate projects tempt people into adding specific logic to one of them to customize it without including it upstream.

## Pattern: Pipeline Stack Parameters

Delivery pipelines allow us to set at the very least environment variables via some admin panel, a CLI or even an API. We can hook up our
project in one such pipeline, and let it configure our project. An added bonus is that the entire team can use it without having to share
parameters or even secrets. The catch is that we become dependant on the pipeline. If for some reason, the pipeline is offline, or simply
not accessible, we cannot make changes to our infrastructure.

Also, the first thing attackers look for when gaining access to a network are CI/CD servers because they are full of admin-level
credentials.

## Pattern: Stack Parameter Registry

We can store all our parameters in a centralized registry, this can range from simple files in some file server, to a Consul cluster or a
SQL database. Of course, we have to use another pattern to set the connection parameters to our registry; but once plugged it, changing it
becomes a matter of executing a query, or modifying a file.

The main issue with this pattern is the fact that we are adding something more to manage, the registry itself. This registry is an extra
moving piece and a potential point of failure.

# Testing

If good testing yields great results for application development, we can assume it will be very useful while defining our infrastructure as
code.

Testing infrastructure code is not easy. Many frameworks use declarative languages, which makes unit tests for these declarations somewhat
useless. If we declare that we want a server with 1 CPU and 2 GB of RAM, the test that we write might check for these attributes. We are
simply re-stating what we declared in the code. With this test, we are testing that the provider and the tool works as promised, not that
our code does what we intend. Unit testing is reserved, then, for those sneaky pieces of code that are influenced by configuration
parameters, and why not, for testing around known issues with the tools of platforms we use.

The heavy burden will lie on integration tests. If we want three subnets: `subnet-a` is public, `subnet-b` is private, `subnet-c` is private
and does not have access to the other subnets; we can test for exactly that, that we can reach `subnet-a` through internet, that `subnet-b`
can access `subnet-a`, and that `subnet-c` cannot access the other subnets. This is much more useful, since we are testing several moving
pieces. Provisioning networks include access rules, routing tables, subnet masks, internet gateways, nat gateways, and some other pieces
that provisioned together will provide a useful network; we might as well test that it works like we want it to.

Integration tests are slower than unit tests, since we must provision actual resources on real platforms. The slowness gets worse when we
make end-to-end tests. When we provision our entire infrastructure to test, that is.

The key thing is to identify risks, in our case, our risk is that we may accidentally expose `subnet-c` to the outside world; therefore we
test to ensure that risk does not become a reality.
