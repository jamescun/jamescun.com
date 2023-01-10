---
title: Go Monorepos for Growing Teams
date: 2022-08-22T21:02:16+01:00
summary: Some thoughts on structuring Monorepos and Go Microservices.
draft: false
---

I've been working with Go monorepos and microservices for longer than I care to admit. I've seen a lot written about how to get started with them, but never anything from the trenches. You have a monorepo of microservices, great, what next?

I won't go into what monorepos or microservices are, or what they pros and cons of each are, as that has been written about at length by people better qualified than me. Instead, I will dive into how I've previously solved some of the problems of Go monorepo growth.

# Single Team, Multiple Services

Starting with the simple scenario, a small engineering team operates a monorepo consisting of Go microservices. Everyone can work on everything, mental context is readily shared and project teams are fluid. Everybody understands the system end-to-end, and what APIs are where.

If this sounds familiar, you may feel at home in the directory structure below:

<img src="/img/stage-one-monorepo.png" width="393">

I know I do, my last three jobs started their monorepo adventures similarly.

Going from the root, `pkg` contains packages that are intended to be shared between services, and `svc` containing microservices. Each microservice may have it's own `pkg` directory to split up it's own logic, but should not be consumed by any other microservice.

Pretty easy to understand and pick up for a new engineer.

However, does this scale? What happens when we grow the team, when no single engineer can maintain a clear picture of every microservice, or specialized domains develop?

# Multiple Teams, Multiple Services

To mitigate the aforementioned problems, we introduce a layer of indirection, the `prj` hierarchy (short for "projects", we are dealing with Go import paths here!) containing a sub-hierarchy relating to a team or project.

<img src="/img/stage-two-monorepo.png" width="900">

With this above structure, we logically group shared packages and microservices into the team or project that owns that area of your system. Examples could be a platform team, who owns your identity or authorization subsystems, or projects like an additional product and it's subsystems.

You may notice a curious new addition, the `proto` directory to each team. This may vary depending on what RPC framework you use, if any, but in this instance we are talking Protobuf, and specifically gRPC. This is every ***other*** team's gateway into yours. It is the contract you define with your wider engineering team about what you are going to provide them.

In combination with the review process in your monorepo, this creates a healthy team practice where you and your stakeholders can collaborate on API design. Other teams do not need to know which of your microservices handles what logic, just that the API they expect from you is available at `monorepo/prj/yourteam/proto/...` and your service mesh/service discovery/gateway/etc will handle the rest.

You also may notice we left a `pkg` directory at the root of the monorepo. While each project or team directory has it's own `pkg` directory as described previously, there will be some logic that is common to all teams or projects, for example, secrets management or service discovery. It is important to tightly scope the top-level `pkg` directory to this purpose, only elevating from project/team `pkg` directories when a clear need is defined.

And finally, while the purpose of this blog post isn't to talk about continuous deployment, if you are automatically redeploying all services for every change, which I've seen frequently in the Single Team scenario, breaking your codebase down by team or project could be the first step towards independent service deployments.

# Tradeoffs

## Import Paths

While we've taken care to keep our Go package import paths short with abbreviations, those abbreviations introduce additional mental context that needs to be communicated to new engineers, and only partially solves the problem.

## Go Modules

Where this model breaks down is dependency management with Go modules, whether Single or Multiple Team. Currently, to support this structure, the monorepo must share a single `go.mod` file at the root.

This can be painful as it introduces a point of contention when introducing or upgrading dependencies.

Since Go 1.18, [Workspaces](https://go.dev/blog/get-familiar-with-workspaces) introduced support for multiple modules, which sounds like it could help, but I've not had an opportunity to evaluate them yet.
