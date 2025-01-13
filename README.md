# Gin SCS Adapter

The Gin SCS Adapter allows you to seamlessly use the SCS session manager within a Gin web framework. It provides a simple way to manage session states while adhering to best practices for handling session cookies and data.

## Installation

To use this package, import it in your Go application:

```bash
go get github.com/39george/scs_gin_adapter
```

## Getting Started

You can create a new session manager and set up the Gin adapter as follows:

```go
import(
  	"github.com/alexedwards/scs/v2"
   	"github.com/gin-gonic/gin"
    gin_adapter "github.com/39george/scs_gin_adapter"
)

func main() {
    sessionManager := scs.New()
    sessionManager.Lifetime = 24 * time.Hour // Set the session lifetime
    sessionAdapter := gin_adapter.New(sessionManager)

    r := gin.Default()
    r.Use(sessionAdapter.LoadAndSave) // Load session for each request

    r.GET("/put", func(c *gin.Context) {
        sessionAdapter.Put(c, "foo", "bar") // Store a value in the session
        c.JSON(http.StatusOK, gin.H{"foo": "bar"}) // Respond with the stored value
    })

    // Start the server
    r.Run()
}
```

## Important Details

### Session Handling in Gin

Due to the way Gin handles response writing, it is not possible to use the SCS session manager's LoadAndSave middleware directly without encountering issues. Once the response body or headers are written in a Gin handler, attempting to modify the headers will result in an error stating that headers have already been written.

The Gin SCS Adapter serves as a wrapper around essential SCS functions. It ensures that the session is committed and response headers are written at the appropriate time, within the request handler, rather than within the middleware. This approach avoids conflicts with Gin's response writing mechanism.
Middleware Functionality

The LoadAndSave middleware is responsible solely for loading the session data and injecting the relevant session context into the Gin context. It does not perform any additional actions.

### Session Methods

The Gin adapter provides methods that correspond to operations that can be performed with sessions:

- Put: Adds a key-value pair to the session data. Existing values for the key will be replaced, and the session status will be marked as modified.
- Get: Retrieves the value associated with a given key from the session data. The retrieved value will need to be type asserted as it returns an interface{}.
- Destroy: Deletes session data, marking the session as destroyed. Any subsequent operations within the same request cycle will create a new session.
- RenewToken: Regenerates the session token while retaining existing session data. This is particularly important for mitigating session fixation attacks.
- RememberMe: Configures whether the session cookie is persistent (retained across browser sessions).
