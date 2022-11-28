---
title: "Getting Started with AWS"
subtitle: "Create an AWS account and set up CLI/SDK access."
date: "2020-12-27"
---

"Cloud computing" plays a vital role in the creation of software products and services. It's also one of the most highly sought-after skills in the tech industry.

In fact, most of the projects on this site will require cloud interaction of some sort—particularly with AWS's serverless products.

## Getting Started

> To use AWS in these projects, we'll need to set up an account, the CLI, and the SDK.

### Create an account

If you don't already have an account then [sign up here](https://portal.aws.amazon.com/billing/signup#/start).

Once you are signed up, you should be able to log in to the [AWS Console](https://aws.amazon.com/console/). It might look overwhelming if you're seeing for the first time.

### Install the CLI

The AWS CLI is a command-line application that lets you interact with your AWS account from the terminal. It's available on all platforms.

If you are a proficient Python user, you can just install it with `pip`.

```bash
pip install awscli
```

Otherwise, check out the [official instructions](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html).

Once installed, you should be able to run this command from the terminal to see its version.

```bash
aws --version
```

### Create an IAM user

The CLI will access your AWS account via an "IAM user." You can create one from the **Users** page in your [IAM console](https://console.aws.amazon.com/iam).

Once the user is created, you'll need to generate access keys (passwords, essentially) for it.

* [Creating an Admin IAM User](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html)
* [Creating access keys for a user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey)

Your access keys should look something like this:

```
Access key ID: AKIAIOSFODNN7EXAMPLE
Secret access key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```


### Configure the CLI

Next you need to [configure the CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html) so that it can access your AWS account via the IAM user.

Basically, just run this command and paste in your access keys.

```bash
aws configure
```

Additionally, you'll also be asked for a [default region](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-region) and [default output format](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-format).

You may leave them empty—but generally I like to use:

```
Default region name [None]: us-east-1
Default output format [None]: json
```

### Configuration files

Once configured, the AWS CLI [saves the credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) and region/format profiles to your computer. They are typically in these locations:

```
~/.aws/credentials
~/.aws/config
```

You can open them up and edit them if you like or just run `aws configure` again to change them.

### Test the CLI!

Now you should be able to use your CLI to access AWS. For example, I should be able to see the S3 buckets I have in `us-east-1`:

```bash
aws s3 ls

2020-12-09 22:36:32 blog.pixegami.com
2020-12-27 00:04:52 cloud-archiver.5dac84a54677.archivetest
```

Generally, everything that can be done in the console can also be done in the CLI. Check out the [full reference guide here](https://docs.aws.amazon.com/cli/latest/index.html).

### AWS SDK

Finally, to use AWS directly from your application code, you need to download the [SDK](https://aws.amazon.com/tools/) for the language you work with.

The SDKs can be configured in different ways as well, but by default it usually uses the same profiles and credentials stored by your `aws configure`.


### That's it! 

You're all set to start using AWS.

## Why AWS?

When we bring "the cloud" into a project, it's usually because there's some capability we'd like to add.

* Hosting for a website or service.
* File or data storage.
* On-demand computation.
* Authentication.

And there's many viable solutions to choose from—[Azure](https://azure.microsoft.com/en-au/), [Google Cloud](https://cloud.google.com), [Firebase](https://firebase.google.com), [Digital Ocean](https://try.digitalocean.com).

So why could you choose [AWS](https://aws.amazon.com/what-is-aws/) over any of these alternatives? From a new user's perspective:

* **Largest marketshare (at 30%)** which roughly translates to lots of community resources and job opportunities.

* **Most services available (175+)** which means more tools at your disposal, well-integrated under one umbrella.

On the flip-side, the biggest drawback is its upfront complexity.

Personally though, the reason I use AWS is because it's the technology I'm most familiar with.


## Why Serverless?

**It's cheaper.** Most cloud "getting started" guides will show you how to spin up a server—a mercenary rented computer that stays online  24/7 to do your bidding.

But for most of my projects, I'm going utilize technology that doesn't require a hosted server. In particular:

| Service | Purpose |
| --- | --- |
| S3 | File storage |
| DynamoDB | Database |
| Lambda | Compute engine |

Their on-demand pricing means the cost scales with usage. There is a [free tier](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc), and it only begins to cost money if usage exceeds a certain amount.

For small projects with light traffic, this usually translates to monthly costs of **less than a dollar** (if not completely free).

In comparison, the price of hosting a server typically starts at **$5.00 per month**.

## Continue Learning

* [Official documentation](https://aws.amazon.com/getting-started/)
* [Free YouTube videos](https://www.youtube.com/watch?v=ubCNZRNjhyo)
* [Udemy Courses](https://www.udemy.com/course/aws-certified-developer-associate/)
