Title: REST and File Uploads/Attachments
Date: 2013-07-23 13:44
Category: Software
Tags: REST, web, software, code
Slug: rest-uploads
Author: Ben Reilly
Summary: File uploads in a REST application
Status: published

Your web application will support uploading files. At first glance, this is an action and you might consider working with it as an RPC endpoint rather than REST. The upload could refer to a verb rather than a noun.

There isn’t anything really wrong with this, but I would argue there are significant advantages with going with it as a noun (REST resource). Here are a few:

### Staging an upload to external datastore

An upload may not be directly to you, and it might not be used by the requesting client – signed S3 forms, one-time URL endpoints, other protocols like Bittorrent, and other mechanisms that allow direct client uploads.

Example: You `POST` to create an `upload` resource.

```json
{
    "upload_to_url": "https://example.com/one/time/endpoint/hashhashhash",
    "signed_token": "blahblahblah",
    "expires": "2013-07-12T19:10:19.491Z",
    "etc": "..."
}

```

...and it returns the `upload` resource you created. It's not the raw file, but rather useful metadata about it (including the actual raw file location).

### Tracking/auditing – both internally and externally

What if a user wants to see what uploads are currently in progress? All of the successful ones? The failures? Those are all also useful metrics internally as well. But as above you have an `upload` resource now, so you can retrieve it with a `GET`:

```json
{
    "createuser": "https://example.com/user/1234",
    "modifieduser": "https://example.com/user/1234",
    "createdate": "2013-07-12T19:10:19.491Z",
    "modifieduser": "2013-07-12T19:10:19.491Z"
}
```

### Attaching additional resources as a means of post-upload action

The file being uploaded is unlikely to exist in a vacuum. You will have related resources and possibly related actions. You can stick that stuff on here too. Consider, for instance, that you want to send alerts to some people when the upload is complete:

```json
{
    "subscribers": [
      "https://example.com/user/1234",
      "https://example.com/user/288",
      "https://example.com/user/3"
    ],
    "etc": "..."
}
```

### Explicit vs. implicit

Bottom line – your upload has state information. You are probably capturing it anyway in logs or other resources. If you have some subscribers as above, you want to make that information explicit, and in many cases, client controlled.