## About

ZipStreamer is a golang project for building and streaming zip files from a series of S3 objects. For example, if you have 200 files on S3, and you want to download a zip file of them, you can do so in 1 request to this server.

It was forked from [https://github.com/scosman/zipstreamer](https://github.com/scosman/zipstreamer) and modified to work on Cloud foundry and to read from bound S3 service only.

Highlights include:

 - Low memory: the files are streamed out to the client immediately
 - Low CPU: the default server doesn't compress files, only packages them into a zip, so there's minimal CPU load (configurable)
 - High concurrency: the two properties above allow a single small server to stream hundreds of large zips simultaneous
 - It includes a HTTP server, but can be used as a library (see zip_streamer.go).

## HTTP Endpoints

**POST /download**

This endpoint takes a post, and returns a zip file.

It expects a JSON body defining which files to include in the zip. The `ZipPath` is the path and filename in the resulting zip file (it should be a relative path).

Example body:

```
{
  "entries": [
    {"S3Path":"image1.jpg","ZipPath":"image1.jpg"},
    {"S3Path":"image2.jpg","ZipPath":"in-a-sub-folder/image2.jpg"}
  ]
}
```

**POST /create_download_link**

This endpoint creates a temporary link which can be used to download a zip via a GET. This is helpful as on a webapp it can be painful to POST results, and trigger a "Save File" popup with the result. With this, you can create the link in a POST, then open the download link in a new window.

*Important*:

 - This stores the link in an in memory cache, so it's not suitable for deploying to a cluster.
 - These links are only live for 60 seconds. They are expected to be used immediately and are not long living.

It expects the same body format as `/download`.

Here is an example response body:

```
{
  "status":"ok",
  "link_id":"b4ecfdb7-e0fa-4aca-ad87-cb2e4245c8dd"
}
```

**GET /download_link/{link_id}**

Call this endpoint with a `link_id` generated with `/create_download_link` to download that zip file.
