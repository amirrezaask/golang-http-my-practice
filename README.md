# Golang notes
Following notes come from my personal experience writing golang web services.


## App struct
Always avoid using global state for any thing. use a type like server or app to hold your global state, connections and every thing you are going too need in requests, and always try to use abstract implementation(interface) instead of concrete types, If you need something in only for example 2 of your handlers, don't put them in App struct, and initialize them in controllers or create a new struct for those 2 handlers, don't make a mess in your main app struct.
```go
type App struct {
    db someDbInterface
    logger someLogger
    emailer someEmailer
    notificationService someNotificationService
}
```
Remember some times your app struct becomes too fat, you can break it into simpler and smaller structs
```go
type PeopleServer struct {
    db peopleDbInterface
    logger somelogger
}
type OrderServer struct {
    db orderDbInterface
    logger somelogger
    bankService bankInterface
}
```


## Controllers vs Handlers
In other languages and frameworks we have the concept of controllers as the entry point for our apps but in go our application entry are handlers, handlers are usually simple functions that satisfy `http.HandlerFunc`, but this handlers are simple functions so you don't have DI, so you need to do every thing in them, even middleware functionality, of course if you use frameworks like (Echo)[http://github.com/labstack/echo] they give you some syntax to make your handlers more clean but we can implement them using StdLib as well,
In my opinion Controllers in golang are methods on the server struct which would return handlers and handlers are `http.HandlerFunc`.
### Example:
```go
func (b *BookServer) GetBook() http.HandlerFunc {
    // you can initialize vars here....
    // do loging or anything
    // notice that all code you write here calls every time you access this controller
    return func(w http.ResponseWriter, r *http.Request) {
        //your logic
    }
} 
```
## Middlewares
There are libraries that implement middlewares, or even routers like (gorrila/mux)[http://github.com/gorrila/mux] that support middlewares outside the box, but I like to look at middlewares even simpler, middlewares are just functions that run some logic before/after request being handled by router so let's do exactly this in our code.
```go
func (b *BookServer) IsAdmin(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        //query for user and check permission
        if !userIsAdmin {
            w.WriteHeader(401)
            fmt.Fprintln(w, "Access Denied")
        }
        next(w,r)
    }
}
```