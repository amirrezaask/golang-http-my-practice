# Goify


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
In other languages and frameworks we have the concept of controllers as the entry point for our apps but in go our application entry are handlers, handlers are usually simple functions that satisfy `http.HandlerFunc`, but this handlers are simple functions so you don't have DI, so you need to do every thing in them, even middleware functionality, of course if you use frameworks like [Echo](http://github.com/labstack/echo) they give you some syntax to make your handlers more clean but we can implement them using StdLib as well,
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
There are libraries that implement middlewares, or even routers like [gorrila/mux](http://github.com/gorrila/mux) that support middlewares outside the box, but I like to look at middlewares even simpler, middlewares are just functions that run some logic before/after request being handled by router so let's do exactly this in our code.
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

## Context is a great data container
contexts are a great a way of having propagation of goroutines but they have another great ability, they can hold data in them as well.
imagin you have a middleware that checks for user identity, so it will query database for user, and you are going too need that data later, you can simply change request and put it in request context as below:
```go
func (b *BookServer) CheckUserIdentity(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        //get user identity from db
        // changing request context to a new context with user identity
        r = r.WithContext(context.WithValue(r.Context(), "ident", /*user identity*/))
        next(w, r)
    }
}
```

## DRY is good but not always
creating abstraction over a repititive code that you write most of the times is good but in my experience some times it's easier to have a code that is copied few times, to have a abstraction that is either complex and unreadable or has overhead.

## Always stick to left
When you are reading code, it's nice to see all logic sequentialy following each other, program flow should not be indented into conditions, unless you are handling an edge case or error scenario. 

## Be liberal in what you accept, and be conservative in what you return
when writing a function always accept parameters that are generic and abstract like interfaces, because they are so much easier to mock
and always return concrete types again because they are much simpler to assert in tests.
```go
func WriteToFile(f *os.File) {} // Wrong: when testing this function creating a os.File is too expensive

func WriteToFile(w io.Writer) {} // Good: when testing this function it's really easy to create io.Pipe and pass pipeWriter to this function and assert on reader

```

another thing about passing an interface is that an interface some how shows us what this functions is going to do, forexample in above when we are passing a file we really don't know what this function is going to do with this file, maybe it's going to read content, maybe write something, maybe append something we don't know, but in second function we exactly know that this function is going to write on the interface we are passing to it.

## Initialize only when you need to
golang has multiple ways of defining and initializing variables, like `:=` and `var`, don't always use them interchangably, if you want to only define a variable but you want to intialize it later, use `var` and if you want to define and initialize use `:=`, don't use `:=` all the time.
```go
//not good
model := Model{}

err := json.Unmarshall(data, &model)
if err != nil {
    return fmt.Errorf("error in unmarshalling json: %v", err)
}

//good
var model Model

err := json.Unmarshall(data, &model)
if err != nil {
    return fmt.Errorf("error in unmarshalling json: %v", err)
}

``` 
like above example when you want to forexample pass pointer of a variable to be the destination of a data, just define it and not initialize it


## const when possible, var when necessary
constatns are one of the few ways we can approach immutability in golang, they are super fast so use them in every place you can. 

## expected errors should be expected
if you are writing a package, let's say a bank api interface for golang, always for expected error types like connection problems or domain specific errors create error types so that user of that 
package be able to assert different error types. another thing is to always define your errors as consts and not vars because they are exported and it's really easy to mess up with them so make them const and immutable.

## table driven tests


## don't name vars after what they are, name them after what they do
TBA