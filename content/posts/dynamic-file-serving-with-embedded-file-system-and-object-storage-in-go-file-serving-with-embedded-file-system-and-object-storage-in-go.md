---
date: '2024-08-14T20:36:36-08:00'
draft: false 
title: 'Dynamic File Serving With Embedded File System and Object Storage in Go File Serving With Embedded File System and Object Storage in Go'
---
## Introduction

*The references to a blog in this post are for my older blog that was built from
scratch using Go. My new blog is built using Hugo and the Papermod.*

In my current project, this blog, I have a folder of static files embedded directly into the server’s binary. While this is nice to have, I am concerned with scaling, so I would also like to keep these static files in an object storage container like an S3 bucket.

Since these files will be stored in two separate places, I need to create a Go interface that allows my application to switch between these storage types based on a runtime flag.

---

## What is Needed

- An interface that I can use to point to the locations of my static files.
- A flag to direct my program to where it should obtain my static files from.
- A URL for my object storage location and embedded static files.
- An environment variable to store the URL of my object storage.

## The Static Files

Within the `head` element of each of my pages are some static files like

- Logo images
- Stylesheets
- JavaScript files

The goal is to  be able to load the static files either the embedded file system or an Object Storage location.

- Embedded File System: `/static/dist`
- Cloud Storage: `https://mybucketlocation.com/static/dist`o

This will be possible by redirecting requests to either use the embedded file system or the object storage whenever a GET request is made to `/static/`.

Static file URLs  have the following format.

```text
"/static/some/path/static.file"
```

---

# The Plan

## 1. Create a flag to to toggle between embedded and object storage based serving

In the `main.go` file there is a `config` struct which has been assigned to `cfg` where the application will store if the object storage should be used. This config struct is included in our `application` struct which handles the state of our application.

If `serveStaticObjectStorage` is set to true an environment variable named `objectStorageURL` will be validated and set to the `cfg`.

```go
type application struct {
 //...
 cfg               *config
 //...
}

type config struct {
 //...
 objectStorage objectStorageConfig
 //...
}

type objectStorageConfig struct {
 objectStorageURL         string
 serveStaticObjectStorage bool
}

func main() {
 var cfg config
 //...

 // Check flags
 flag.BoolVar(&cfg.objectStorage.serveStaticObjectStorage, "object-storage", false, "Serve static files from object storage")

 flag.Parse()

 // Validate Object Storage URL
 if cfg.objectStorage.serveStaticObjectStorage {
  osURL := os.Getenv("OBJECT_STORAGE_URL")
  if osURL == "" {
   log.Fatal("OBJECT_STORAGE_URL must be set when object storage is enabled")
  }
  targetFile := fmt.Sprintf("%s/static/dist/js/form-prevent.js", osURL)
  resp, err := http.Get(targetFile)
  if err != nil {
   log.Fatal("Unable to connect to object storage")
  }
  if resp.StatusCode != http.StatusOK {
   log.Fatal("Unable to connect to object storage")
  }
  cfg.objectStorage.objectStorageURL = osURL
 }

 //...
 // Further into the main function the application
 // assigns `cfg` to an application struct.
}

```

## 2. Abstract the FileServer Logic

Currently, static files are being served from by a [Handler](https://pkg.go.dev/net/http#Handler) returned from the [FileServerFS](https://pkg.go.dev/net/http#FileServerFS) function. The logic needs to be abstracted by creating a custom `FileServer` interface which can be used for two types `EmbeddedFileServer` and `ObjectStorageFileServer` which I will create.

```go
type FileServer interface {
 ServeHTTP(w http.ResponseWriter, r *http.Request)
}
```

Now that I have created the `FileServer` interface  I can create my two types and their `ServeHTTP` method to satisfy the interface.

### EmbeddedFileServer

```go
// internal/fileserver/embedded.go

type EmbeddedFileServer struct {
 FS http.FileSystem
}

func (efs *EmbeddedFileServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
 http.FileServer(efs.FS).ServeHTTP(w, r)
}
```

### ObjectStorageFileServer

```go
// internal/fileserver/object_storage.go
type ObjectStorageFileServer struct {
 objectStorageURL string
}

func (osfs *ObjectStorageFileServer) ServeHTTP(w http.ResponseWriter, r *http.Request){
 fileURL := fmt.Sprintf("%s%s", osfs.objectStorageURL, r.URL.Path)
 http.Redirect(w, r, fileURL, http.StatusFound)
}
```

Now that the new types have been created and both satisfy the interface they can be implemented into the router.

## 3. Update the Router

### Router Update

The router is currently serving the embedded static files with the following code

```go
func (app *application) router {
 mux := http.NewServeMux()

 fileServer := http.FileServerFS(ui.Files)
 mux.Handle("/static/", fileServer)
 // ...
}
```

Now static files can be served from either the embedded file system or object storage by checking if the `serveStaticObjectStorage` is true or false.

```go
func (app *application) router {
 mux := http.NewServeMux()
 
 var fileSvr http.Handler
 if app.cfg.objectStorage.serveStaticObjectStorage {
  fileSvr = &fileserver.ObjectStorageFileServer{ObjectStorageURL: app.cfg.objectStorage.objectStorageURL}
 } else {
  fileSvr = &fileserver.EmbeddedFileServer{FS: http.FS(ui.Files)}
 }

 mux.Handle("/static/", fileSvr)
 // ...
}
```

---

# Conclusion

With this setup, my blog is now flexible enough to serve static files from either an embedded file system or an object storage service, depending on the needs at runtime. This approach makes it easier to scale and adapt the application as it grows, whether it’s to handle more traffic or to streamline deployments. By simply flipping a flag, I can switch between these storage options, ensuring my blog remains both efficient and ready for future expansions. As I continue to iterate on this blog, I'll keep exploring ways to enhance its performance and scalability, ensuring that it remains both maintainable and future-proof.
