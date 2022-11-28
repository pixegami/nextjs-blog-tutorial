---
title: "Building a QR code micro-app"
subtitle: "Build a serverless 'hello-world' QR code app with Python and AWS."
date: "2021-08-14"
---

Ever since the pandemic started, I've noticed QR codes creeping into my daily life. Venue check-ins, digital menus at restaurants, and online payments.

I thought it'd be fun to explore the technology a little bit, so I've built a micro web-app that lets you save messages and view them later using a QR code. Sort of a "Hello World" QR code project.

In this post, I'll be sharing how I built this using Python and AWS free tier.

You can try out the [app itself here](https://qr.pixegami.com/). The [source code](https://github.com/pixegami/qr-code-webapp) is also available on GitHub.

![images/qr-code-demo.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1628899108564/NGWiGc3-w.gif)

## What is a QR Code?

We've all seen and used them before, but how does a QR code actually work? The first thing to understand is that it is an [internationally standardized specification](https://www.iso.org/obp/ui/#iso:std:iso-iec:18004:ed-3:v1:en):

> This International Standard [...] specifies the QR Code symbology characteristics, data character encoding methods, symbol formats, dimensional characteristics, error correction rules [...]

As long an image respects these standard, it is a "QR code" and can be understood by most smartphone cameras. The image itself also has a kind of anatomy:

![images/qr_code_anatomy.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628900340423/NSYNLARYu.png)

It is quite interesting, but ultimately these were the three things I cared about:

- We can store up to around 4000 alpha-numeric symbols in a standard sized QR code.
- QR codes can have a "URL" format, which will prompt phones to open it up in a browser when scanned.
- QR codes have error-correction capabilities, which allow it to remain function even if some parts of it is covered or removed—_I did not know this!_

## Architecture

I want to make an app that lets me author some arbitrary content (in this case a short text message), have it persisted somewhere, and generate a QR code that loads the content when I scan it with the phone.

So after breaking these requirements into technical tasks, here is the strategy:

- **Author some arbitrary content**: I'll use a static React front-end to let the user write their message.
- **Persist the content**: I'll have a serverless API (using AWS Lambda and API Gateway) for the front-end to use. It will receive the message (a string) and put it into database for storage. I'll generate a unique ID and use this as the `tag` for the object.
- **Generate a QR Code**: I'll first come up with a URL string I want to encode into the QR image (using the `tag` I made above). Then I'll find a Python library that lets me turn this URL string into a QR image. I'll make the image accessible to the user.
- **Load the content**: Now I'll implement the URL endpoint that the QR code re-directs to. It will probably have the `tag` as a query parameter, so I'll just use that to look up the message in my table and send it back to the page.

### Overall Stack

- **Frontend**: React (Typescript)
- **Backend**: Python code with AWS Lambda and API Gateway
- **Database**: DynamoDB
- **Image Storage**: Amazon S3

## Implementation

Most of the app's 'meaty' logic lives in the [`qr-code-infrastructure/compute/api`](https://github.com/pixegami/qr-code-webapp/tree/main/qr-code-infrastructure/compute/api) folder, as a bunch
of Python functions.

### Generating a `tag` and a URL

When a user sends a message, it generates a random tag using `uuid4` (which I truncated to 12
characters to keep it a bit shorter). A URL to view this message will then be used to create a QR code.

```python

# uuid is a built-in Python library to generate random IDs with, with low chance of collision.
qr_id = uuid.uuid4().hex[:12]
qr_tag = f"qr-{qr_id}"

# We'll later have to implement this page so that it can load our message with the given tag.
content = f"https://qr.pixegami.com/view?tag={qr_tag}"
```

### Generating the QR code image

One of the things I really love about Python is how there's a library for everything. I just typed in "qr code" into PyPI and found this [library](https://pypi.org/project/qrcode/), which took a minute to install and use.

Using the `qrcode` library, I create an image. It's a one-liner.

```python
qr_image = qrcode.make(content)
```

But now we need to save this image somewhere. Since this function is running on AWS Lambda, we can't just [save it anywhere](https://aws.amazon.com/lambda/faqs/).

> Each Lambda function receives 500MB of non-persistent disk space in its own /tmp directory.

This must be saved into `/tmp` folder on Lambda
since that is the only folder that is writable (hence why we need to pass down a `path`).

```python
image_path = os.path.join(path, f"{qr_tag}.png")
qr_image.save(image_path)
```

### Showing the QR image to the user

Now in our Lambda runtime we have a `.png` file at `image_path`. We need to find a way to get it to the user. We'll do this by storing the image somewhere permanent, and then generate a URL for that image.

Let's upload it to an [S3 bucket](https://aws.amazon.com/s3/?nc2=h_ql_prod_fs_s3).

```python
bucket_name = os.environ["IMAGE_BUCKET_NAME"]
key = f"{qr_result.tag}.png"
s3client = boto3.client("s3")
s3client.upload_file(qr_result.image_path, bucket_name, key)
```

I then create a [pre-signed URL](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html) from the bucket so that I can tell the frontend where to load the
QR code image from.

> All objects by default are private. Only the object owner has permission to access these objects. However, the object owner can optionally share objects with others by creating a presigned URL, using their own security credentials, to grant time-limited permission to download the objects.

It expires in `3600` seconds, but that's fine because I don't need the image
itself to be long lived.

```python
presigned_url = s3client.generate_presigned_url(
    "get_object",
    Params={"Bucket": bucket_name, "Key": key},
    ExpiresIn=3600,
)
```

This URL is sent back to the front-end for display.

![images/qr-code.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628903696663/uU7ksdnrL.png)

### Persisting the `tag` and the message

Finally I need to write an entry into the [DynamoDB table](https://aws.amazon.com/dynamodb/) so that when a user scans a QR code, we can fetch the message later.

The only important thing to know about DynamoDB here is that it acts as a simple key-value store, and it is also serverless.

```python
item = QrItem()
item.pk = qr_result.tag  # This is the UUID we generated above.
item.message = message  # This is the message body.
self.database.put_item(item)
```

### Scanning the QR image to load the content

On the front-end, it's simple just a page for this URL we generated earlier `https://qr.pixegami.com/view?tag={qr_tag}` to look up the table value for the item with that `tag`.

I [used](https://github.com/pixegami/qr-code-webapp/blob/main/qr-code-site/src/components/pages/ViewPage.tsx#L19) a [React `useEffect` hook](https://reactjs.org/docs/hooks-effect.html), which lets me make an API call once the page loads.

I have another API on the back-end, which receives this `tag` and looks up the saved message. It then sends it back for the front-end to display.

```python
serialized_item = self.database.get_item(QrItem(tag))
item = QrItem().deserialize(serialized_item)
message = item.message
```

![images/qr-result.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628903629263/V2Jg0j8r3.png)

## Wrap Up

That's pretty much it! Now if I ever need to use QR codes as part of an application in the future I'll just dig this up 😅

And if you're keen to try this out yourself, feel free to check out the [source](https://github.com/pixegami/qr-code-webapp). It's a small project so you can probably build it out in a few hours (or a few days, if you're new to AWS as well).
