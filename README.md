# Cloud Run Reverse Proxy

`reverseproxy` is a Go package that provides a simple reverse proxy implementation with customizable routing rules for Google Cloud Run services. It allows you to route incoming HTTP requests to different backend services based on rules such as path prefixes or client IP addresses. The package also handles OIDC token generation for authenticated communication between services.

## Installation

To install the package, run:

```bash
go get -u github.com/dbut2/cloud-run-reverse-proxy
```

## Usage

### Import the package

```go
import "github.com/dbut2/cloud-run-reverse-proxy"
```

### Create a reverse proxy

Define your routing rules using the `reverseproxy.Rule` type. You can create rules based on path prefixes or client IP addresses using the `PathRule` and `IPRule` functions. Finally, create a reverse proxy instance with the defined rules using the `New` function.

```go
rules := []reverseproxy.Rule{
    // Reverse proxy rules...
}
proxy := reverseproxy.New(rules)
```

### Run the reverse proxy server

Register the reverse proxy as an HTTP handler and start the server.

```go
http.HandleFunc("/", proxy.ServeHTTP)
http.ListenAndServe(":8080", nil)
```

## Example

Below is a complete example demonstrating how to create and use a reverse proxy with customizable routing rules:

```go
package main

import (
    "log"
    "net/http"
    "os"

    "github.com/dbut2/cloud-run-reverse-proxy"
)

func main() {
	// Load environment variables
	publicURL := os.Getenv("PUBLIC_URL")
	privateURL := os.Getenv("PRIVATE_URL")
	privateClientIP := os.Getenv("PRIVATE_CLIENT_ID")
	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
	}

	// Define routing rules
	rules := []reverseproxy.Rule{
		reverseproxy.IPRule(privateClientIP, privateURL),
		reverseproxy.BaseRule(publicURL),
	}

	// Create the reverse proxy with the defined rules
	proxy := reverseproxy.New(rules)

	// Register the reverse proxy as an HTTP handler
	http.HandleFunc("/", proxy.ServeHTTP)

	// Start the HTTP server
	log.Printf("Starting reverse proxy on :%s\n", port)
	err := http.ListenAndServe(":"+port, nil)
	if err != nil {
		panic(err)
	}
}
```

Don't forget to set the environment variables `PUBLIC_URL`, `PRIVATE_URL`, `PRIVATE_CLIENT_ID`, and `PORT` before running the example.
