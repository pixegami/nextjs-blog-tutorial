---
title: "Building a serverless SaaS product"
subtitle: "A full-stack SaaS project with authentication and payments."
date: "2021-07-12"
---

The [SaaS (software as a service)](https://en.wikipedia.org/wiki/Software_as_a_service) model underpins many of today's successful new businesses. Knowing how to build one from start to finish is probably a useful addition to any software developer's skill set.

But even when you strip a SaaS product of its business logic, there's still a non-trivial amount of work and trade-offs to consider.

In this project, my goal was to build a fully serverless SaaS web-app with authentication and payments — the two vital organs of any business.

My implementation is opinionated (as you'll see), and intended to serve as a starting point for new SaaS ideas in the future. Here's what's included:

- [Authentication](#authentication)
- [Payments (Stripe)](#payments-stripe)
- [Frontend (React)](#frontend-react)
- [Backend API](#backend-api)
- [Serverless architecture](#serverless-architecture)
- [Infrastructure as code](#infrastructure-as-code)
- [CRUD operations](#crud-operations)
- [Lessons Learnt](#lessons-learnt)

You can view the example at https://saas-starter-stack.com/app/ and the source on [GitHub](https://github.com/pixegami/saas-starter). In this post, I'll be reflecting on my choices and experience for each of the above features.

### Authentication

**Don't roll your own auth!** It's hard, and mistakes can be devastating to a business. With that said, I did it anyway — mostly to learn from it. Here's also some [discussion on Hackernews](https://news.ycombinator.com/item?id=22001918) on why you might want to build your own auth.

I used [bcrypt](https://codahale.com/how-to-safely-store-a-password/) and [JSON Web Tokens](https://jwt.io/), and stored credentials on DynamoDB. That part wasn't so bad. The real grind came from building things like exponential back-offs for failed attempts, account verification and reset mechanisms, and patching all the security edge cases.

I got it to a roughly working state, and then called it a day. If this was a production system, I'd probably look into something like [Cognito](https://aws.amazon.com/cognito/), [Firebase](https://firebase.google.com/products/auth) or [Okta](https://www.okta.com/).

### Payments (Stripe)

From payments integration, [Stripe](https://stripe.com) was an easy choice. No prominent alternative come to mind, and I've heard high praises about Stripe's developer onboarding experience.

I set up [subscription payment](https://stripe.com/en-au/billing) integration with the project, and I think the developer experience lives up to expectations. The tutorials were well structured and concise.

But the little thing that impressed me the most was when I typed in 'test card' in a [search box](https://stripe.com/docs/testing), it actually just straight up gave me a card-number I could copy straight to my clipboard. Whoever thought of that just saved me a click, and I'm grateful.

### Frontend (React)

The frontend is a responsive web-app build with [React](https://reactjs.org/). It seems like React is still the dominant technology is the area, although I've yet to try its main competitors like [Vue](https://vuejs.org/) or [Svelte](https://svelte.dev/).

I used [TailWindCSS](https://tailwindcss.com/) for styling, and prefer to anything I've tried in the past (Boostrap CSS, Semantic UI and just vanilla CSS).

I then used [Gatsby](https://www.gatsbyjs.com/) to optimize the static site rendering — but I'm not sure if the extra steps are worth it at this stage. It's better for SEO and performance, but costs extra development cycles.

Overall though, I was quite satisfied with this stack for the frontend, and would be happy to use it for production.

### Backend API

The backend is a serverless REST API implemented in Python and hosted as [Lambda functions](https://aws.amazon.com/lambda/) behind API Gateway.

My main challenge here was to abstract away the lower level things (like CORS, HTTP response formatting, database access) as much as possible. I did this via [Lambda layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html), which allowed me to group a bunch of Python packages and common scripts together.

This allowed me to implement handlers that are quite short and readable, which is think is key to maintainability.

### Serverless architecture

Why serverless? I think for a lot of businesses it simply wins out from a cost and scaling perspective. I could probably serve north of 500k API requests for [less than a dollar](https://aws.amazon.com/lambda/pricing/).

However, this implies that the choice of database must be serverless as well. I chose [DynamoDB](https://aws.amazon.com/dynamodb/) just for the ease of integration. But if I had different data modeling requires (for which the DynamoDB architecture might be unfit), I might look into [Aurora](https://aws.amazon.com/rds/aurora/) or [Fauna](https://fauna.com/).

### Infrastructure as code

Configuring infrastructure is time-consuming and error prone. If I want to be able to deploy a copy of this service quickly, I'd have to [model it as code (IaC)](https://en.wikipedia.org/wiki/Infrastructure_as_code). In keeping theme with my AWS integration so far, I've modeled this project with [AWS CDK](https://docs.aws.amazon.com/cdk/latest/guide/home.html) in Typescript.

With this the entire frontend and backend can be deployed to a brand new account or domain in less than 30 minutes with just a [few configuration changes](https://github.com/pixegami/saas-starter/blob/master/saas-infrastructure/service.config.json).

### CRUD operations

Finally, I've added some simple Twitter-like posting capabilities to the project just as a stub for the actual business logic. It has ways to interact with the authentication API, and find out whether a user is verified, and if they are a paying subscriber.

## Closing Thoughts

Honestly, I'm so tired of this project already. It was a lot more complex than I expected — especially for an app that really doesn't do _anything_! But I did learn a lot along the way though, and will probably be faster the second time around.

My top three takeaways are:

- Don't build your own auth.
- You'll probably rebuild the project at least once or twice, so design things to be flexible.
- Having integration tests really paid off.
