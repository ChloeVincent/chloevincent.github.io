---
layout: post
title:  "Deploy Go app to App Engine"
date:   2021-02-11 11:00:00 +0100
categories: go AppEngine Google Cloud
---

I previously created a go app accessing a google calendar. 

In this post I describe how to deploy it in App Engine. 
I follow [Google Cloud documentation][ae-doc].

# Deploy a Go App

I already had installed Cloud SDK in order to deploy my Django app. I updated the installation using `sudo apt-get update` and `sudo apt-get upgrade`.

I created my project "indivision-toulon" by running `gcloud projects create indivision-toulon --set-as-default`. I then checked that everything went well with `gcloud projects describe indivision-toulon`. Finally I initialized it with a region (I am not sure which one I should choose, and how much it matters in term of pricing between London, Belgium and Zurich): `gcloud app create --project=indivision-toulon`.

Before going on to follow the [doc to build a go app on App Engine][go-app-doc], I install the components that will allow to deploy the Go app: `sudo apt-get install google-cloud-sdk-app-engine-go`.

I added an `app.yaml` next to my existing go app, containing the following configuration:
```yaml
runtime: go115
```

I modified my app so that it does not access Google Calendar for the time being.


<details><summary>

`Hello World` app (Click to expand)</summary>
<p>

```golang
package main

import (
    "fmt"
    "log"
    "net/http"
    "os"
)


// indexHandler responds to requests with our greeting.
func indexHandler(w http.ResponseWriter, r *http.Request) {
    if r.URL.Path != "/" {
        http.NotFound(w, r)
        return
    }
    fmt.Fprint(w, "Hello, World!")
}


func main() {
    http.HandleFunc("/", indexHandler)

    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
        log.Printf("Defaulting to port %s", port)
    }

    log.Printf("Listening on port %s", port)
    if err := http.ListenAndServe(":"+port, nil); err != nil {
        log.Fatal(err)
    }
}
```
</p>
</details>

Important: it is better to add a `.gcloudignore` file: similar to a `.gitignore` file, it will ignore all the files not needed to deploy the app. 
It is created in the first steps of the deployment, which I stopped to complete it.
In my case it looks like the following: 

<details><summary>

`.gcloudignore` (Click to expand)</summary>
<p>

```
# This file specifies files that are *not* uploaded to Google Cloud Platform
# using gcloud. It follows the same syntax as .gitignore, with the addition of
# "#!include" directives (which insert the entries of the given .gitignore-style
# file at that point).
#
# For more information, run:
#   $ gcloud topic gcloudignore
#
.gcloudignore
# If you would like to upload your .git directory, .gitignore file or files
# from your .gitignore file, remove the corresponding line
# below:
.git
.gitignore

# Binaries for programs and plugins
*.exe
*.exe~
*.dll
*.so
*.dylib
# Test binary, build with `go test -c`
*.test
# Output of the go coverage tool, specifically when used with LiteIDE
*.out


## Added by Chloe on February 11th 2021

# go build appli
appli

# Packages that are stored locally
pkg/
src/

# Credentials and token files
*credentials.json
*token.json
```
</p>
</details>


Finally I can deploy using `gcloud app deploy`.

After a first fail due to "Unable to retrieve P4SA: \<service\> from GAIA. Could be GAIA propagation delay or request from deleted apps.", I openned the Google Cloud console and tried again. 
This time I got an "Access Not Configured" error, with a link to enable billing for the Cloud Build API.
Which I did and retried, and it worked !


# Deploy my Calendar App

I spent hours trying to understand why, when deploying my calendar app, I got errors related to not finding packages: context, oauth2 api/calendar. 

The solution is to use a [module file][mod-doc] to manage dependencies. 
The first step is to run `go mod init example.com/m`. 
On the next build (`go build appli.go`), the file "go.mod", that was created in the first step, is updated with the needed dependencies.

In this process I also started using the `appengine` package instead of `context` (to install run `go get google.golang.org/appengine`).
So my "go.mod" file looks like: 

```
module example.com/m

go 1.15

require (
    golang.org/x/oauth2 v0.0.0-20210210192628-66670185b0cd // indirect
    google.golang.org/appengine v1.6.7 // indirect
)
```

NOTE that I am not sure about "example.com/m" which should probably be replaced with something that make more sense.

#### Connect with oauth2

The connection with [oauth2][oauth2-doc] has to be changed since, when deployed, I can not access the command line to enter the token.
I use [http][http-doc] pages to display messages with [appengine][appengine-go-doc], get the authentication code and redirect to the main page. 

I followed the process described for [Authenticating as an end user][auth-end-user], using the example given in the [overview][oauth2-ex].
Make sure to define the scope of your oauth configuration correctly, using the [Scopes for Google APIs][scopes].

I got the authentication code by reading the [url][url-doc] query values returned after the authentication to google.

The oauth protocol is described in the page [Using OAuth 2.0 to Access Google APIs][oauth-protocol]. 
It helped me understand the whole process.

I modified step by step the [calendar API quickstart example][calendar-qs] to display messages in html pages instead of command line.

The main modification to the code base were as follow: 

<details><summary>

In the `handler` function (Click to expand), I first try to get the OAuth2 authentication link, which will be displayed (see below). 
After following the link and authenticating, the app user will be automatically redirected to the handler with an authentication code in the http request, which will allow to connect to the calendar API and start the service.
Once the service is started, I can access the events as I did before with the function `refreshOccupiedDaysList()`.

</summary>
<p>

```golang
func handler(w http.ResponseWriter, r *http.Request) {

    if !tokenRequested{
        getOAuth2Link(w,r)
        return
    }
    if !calendarServiceStarted{
        startCalendarService(w,r)
        return
    }

    if calendarService == nil{
        log.Fatal("Calendar service was not initialized properly.")
    }

    OccupiedDaysList := make(map[int][]OccupiedDay)
    events:= []string{"event1", "event2"}
    if (ctx != nil){
        OccupiedDaysList, events = refreshOccupiedDaysList()
    } else{
        log.Fatal("The background context was not initialized properly")
    }

```
</p></details>


<details><summary>

`getOAuth2Link()` function (Click to expand)
</summary>

<p>

```golang
func getOAuth2Link(w http.ResponseWriter, r *http.Request) {
    redirectURL := os.Getenv("OAUTH2_CALLBACK")
    if redirectURL == "" {
            //redirectURL = "https://indivision-toulon.ew.r.appspot.com/"
            redirectURL = "http://localhost:8080/"
            // note that the redirect url has to change depending on the environment (local test or appengine)
    
    }

    oauth2Config = &oauth2.Config{
        ClientID:     "(redacted).apps.googleusercontent.com",
        ClientSecret: "(redacted)",
        RedirectURL:  redirectURL,
        Scopes:       []string{"https://www.googleapis.com/auth/calendar.events.readonly"},
        Endpoint:     google.Endpoint,
    }

    authURL := oauth2Config.AuthCodeURL("state-token", oauth2.AccessTypeOffline)

        
    fmt.Fprint(w, "Go to the following link and sign in to your account: \n", authURL)
    tokenRequested = true
}
```
</p></details>

<details><summary>

`startCalendarService()` function (Click to expand)

</summary>
<p>

```golang
func startCalendarService(w http.ResponseWriter, r *http.Request){
    fmt.Println("Start Calendar Service")

    if r.FormValue("code") == "" {
        fmt.Fprint(w, "Please reload the page and follow the link provided")
        tokenRequested = false
        return
    }

    var err error
    
    var tok *oauth2.Token
    fmt.Println("About to exchange authorization code for token")
    if ctx ==nil{
        fmt.Println("context is nil!")
    }

    authCode := r.FormValue("code")
    tok, err = oauth2Config.Exchange(ctx, authCode)
    if err != nil {
        fmt.Fprint(w, "Error with authorization code exchange : " +err.Error())
        fmt.Fprint(w, "\nAuthorization code is : "+authCode)
        fmt.Fprint(w, "Please reload the page and follow the link provided")
        tokenRequested = false
        return
    }
    
    fmt.Println("About to create client")
    client:= oauth2Config.Client(ctx, tok)
    fmt.Println("Client created")

    calendarService, err = calendar.New(client)
    if err != nil {
            log.Fatalf("Unable to retrieve Calendar client: %v", err)
            fmt.Fprint(w, "error getting calendar")
    }
     fmt.Println("Calendar service started")

    calendarServiceStarted = true
    http.Redirect(w, r, "/", http.StatusSeeOther)
}
```
</p></details>

# Use local oauth authentication for tests

I extracted `startCalendarService()` and `getOAuth2Link()` to two files: one for local tests and one for app engine.
I added a `isLocal` bool that is defined in each file, so that I can switch since the call to start the calendar service is a bit different in each case.

I now either call `go run appli.go startLocalCalendarService.go` or `go run appli.go startAppEngineCalendarService.go`.

To deploy to Google Cloud, I added start**Local**CalendarService.go to `.gcloudignore`.

I do not use [build constraints][build-constraints] as I intented in the first place.

# No credentials in git
I also extracted the ClientId and ClientSecret from the OAuth2 configuration (previously "redacted"), to a file that is now ignored by git so that there is no risk of accidentally uploading the credentials to GitHub.


# Check which files are deployed
In order to check which files are deployed when running `gcloud app deploy`, you can go to the google cloud console to "App Engine" > "Versions" > "Tools" (in "Diagnose" column) > "Debug".


# Security
In order to avoid that my app holds instances of the connection to some user calendars, I need to remove global variables. 
As mentionned in the [configuration doc][structuring-ws], my "app should be "stateless" so that nothing is stored on the instance".

I created a [cookie][cookie-doc] with a [random number][rand-doc] that maps each user to their token (for the time of the session).
It is transmitted through the http request.
The Path property of the cookie must be set to `Path: "/"` given that I am using redirections (otherwise the coookie persists only for the same url, see [this page][so-cookie-redirect]).
<details><summary>

`Cookie creation` in the callback from authentication function (Click to expand). 
The tokenCookies map allows to retrieve the token later on (see below).

</summary>
<p>

```golang
    authCode := r.FormValue("code")
    ctx := appengine.NewContext(r)

    cookieId := strconv.FormatInt(rand.Int63(),10)
    cookie := http.Cookie{Name : "CookieId", 
                           Value : cookieId,
                           Path: "/"}

    http.SetCookie(w, &cookie)
    tokenCookies[cookieId], err = oauth2Config.Exchange(ctx, authCode)
```
</p></details>

<details><summary>

`Retrieve cookie` to restart the calendar service on refresh (Click to expand). 
The tokenCookies map allows to retrieve the token saved in the authentication callback.

</summary>
<p>

```golang
func getTokenFromCookie(w http.ResponseWriter, r *http.Request) *oauth2.Token {
    cookie, err := r.Cookie("CookieId")
    if err != nil{
        fmt.Println("Could not retrieve the cookie: "+err.Error())
        return nil
    } else{
        cookieId:= cookie.Value
        return tokenCookies[cookieId];
    }
}
```
</p></details>


It is good practice to use constants and not variables in the code. 
Maps however cannot be made constant for various reasons. 
I followed [this page][map-const] that shows how to use an initializer function instead.

#### Avoid Cross Site Request Forgery
Cross Site Request Forgery ([CSRF][csrf-wiki]) can be avoided by using the "state" parameter of the cookie, see [description of Oauth2 authorization framework][csrf-oauth2]. 
I initialize the state parameter with a random value, and change it everytime a connection is made.
This is probably not viable for a large application with concurrent authentication, but I assume that for my application there won't be 2 persons trying to connect at the exact same time, and if they do the session will restart.

<details><summary>

`Use a random string for the state parameter` (Click to expand). 
</summary>
<p>

```golang
var stateString string;
func initializeStateString(){
    stateString = strconv.FormatInt(rand.Int63(),10)
}

// in main
    initializeStateString()

// getting the link to authenticate
    authURL := oauth2Config.AuthCodeURL(stateString, oauth2.AccessTypeOffline)

// retrieving the authentication code to connect
    if r.FormValue("state") != stateString{
        log.Fatalf("The state is invalid, closing the session")
    } else{
        initializeStateString() //update the state parameter for the next authentication
    }

    authCode := r.FormValue("code")
    ...

```
</p></details>

# Add favicon handler
<details><summary>

`faviconHandler` (Click to expand) in needed, otherwise when the browser asks for the favicon, the regular `handler` is called. 
</summary>
<p>

```golang
func faviconHandler(w http.ResponseWriter, r *http.Request) {
   http.ServeFile(w, r, "favicon.ico")
}

    http.HandleFunc("/favicon.ico", faviconHandler)
```
</p></details>

# Avoid reconnecting everyday
In order to avoid reauthentication for every session, I decided to store tokens in cookies.
This is to be avoided!
Only the access token should be stored in cookies, the refresh token should be stored in a database. 
Given that I don't want a database for the few users I will have (literally 3 or 4), I store the token in cookies. 
If my user access the app from a common computer, anyone will be able to access the calendar events of the authenticated user, which is wrong!

This approach did work in localhost when adding the [AuthCodeOption][aco-doc] AccessTypeOffline to the exchange of the authentication code for a token.
However, when deployed, this is not a viable option, since the refresh token is only sent once on the first permission authorisation for the API. 
It is resend if we go to our google account and revoke the authorisation.
By design, I thus need to store the refresh token on my server, which will be better in terms of security so yay !

I use file storage on the cloud ([upload][cloudstorageUp-doc] and [download][cloudstorageDown-doc]) to save my refresh token. 

In order to run the app locally I need to register credential locally as described [here][authService-doc] and the bucket name (which I define in app.yaml for appengine) where the files are stored.
The [bucket][cloudstorage-doc] is created automatically by appengine.

```
export GOOGLE_APPLICATION_CREDENTIALS="/home/user/Downloads/<my-key>.json"
export BUCKET_NAME="<project-name>.appspot.com"
```

For some reason (potentially a security breach?) I can use the same refresh token for multiple accounts.




[ae-doc]: https://cloud.google.com/appengine/docs/standard/go/quickstart
[go-app-doc]: https://cloud.google.com/appengine/docs/standard/go/building-app
[mod-doc]: https://golang.org/cmd/go/#hdr-Defining_a_module
[scopes]: https://developers.google.com/identity/protocols/oauth2/scopes#calendar
[http-doc]:https://golang.org/pkg/net/http
[appengine-go-doc]:https://pkg.go.dev/google.golang.org/appengine
[oauth2-doc]:https://pkg.go.dev/golang.org/x/oauth2
[auth-end-user]:https://cloud.google.com/docs/authentication/end-user
[oauth2-ex]:https://cloud.google.com/docs/authentication#oauth-2.0-clients
[oauth-protocol]:https://developers.google.com/identity/protocols/oauth2
[url-doc]:https://golang.org/pkg/net/url
[calendar-qs]:https://developers.google.com/calendar/quickstart/go
[build-constraints]:https://blog.golang.org/appengine-gopath
[structuring-ws]: https://cloud.google.com/appengine/docs/standard/go/configuration-files#design_considerations_for_instance_uptime
[so-cookie-redirect]: https://stackoverflow.com/questions/57172415/how-to-send-cookie-through-redirection-in-golang
[rand-doc]:https://golang.org/pkg/math/rand/#Int63
[cookie-doc]:https://golang.org/pkg/net/http/#Cookie
[csrf-wiki]:https://en.wikipedia.org/wiki/Cross-site_request_forgery
[csrf-oauth2]:https://tools.ietf.org/html/rfc6749#section-10.12
[map-const]:https://qvault.io/2019/10/21/golang-constant-maps-slices/
[aco-doc]:https://pkg.go.dev/golang.org/x/oauth2#AuthCodeOption
[cloudstorageUp-doc]:https://cloud.google.com/storage/docs/uploading-objects#storage-upload-object-go
[cloudstorageDown-doc]:https://cloud.google.com/storage/docs/downloading-objects#storage-download-object-go
[authService-doc]:https://cloud.google.com/docs/authentication/production#auth-cloud-implicit-go
[cloudstorage-doc]:https://cloud.google.com/appengine/docs/standard/go/using-cloud-storage