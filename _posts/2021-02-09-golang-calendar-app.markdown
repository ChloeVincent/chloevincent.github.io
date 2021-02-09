---
layout: post
title:  "Golang: create an app using Google calendar"
date:   2021-02-08 19:00:00 +0100
categories: golang calendar http
---

When working on a go project, I need to run the following command to avoid errors about not finding the packages: `export GOPATH=<working directory>`.

The working app is available on [my GitHub page][github]. I detail here the different part I needed to create the app. It might evolve in the future but the main parts are present.

# Intro: main()
I first start the Google calendar service. 
I then use a wait group to be able to stop the [Http server gracefully][so-stopHttp], either when the user click enter in the terminal from which the application is launched, either after 15 minutes if the application was launched by double clicking the built executable (in which case the reader returns an "end of file" error).
The function `refreshOccupiedDaysList()` prints the events that are accessed in the Google calendar. 
It is called by the handler to update the calendar html page

```golang
func main() {
    //read calendar
    startCalendarService()
 
    wg := &sync.WaitGroup{}
    // start http server to display calendar
    httpServer := startHttpServer(wg)
    refreshOccupiedDaysList()

    reader := bufio.NewReader(os.Stdin)
    _, err := reader.ReadString('\n')

    if err == io.EOF {
        log.Printf("EOF error have occurred\n")
        time.Sleep(15*time.Minute)
        
    } else if err != nil {
        log.Fatal(err)
    } 
    
    stopHttpServer(wg,httpServer)
}
```



In order to debug the built app, I used a [file writer][bufio-doc]. This is a temporary tool, if used "in production", the errors on the file creation should be handled!

<details><summary>

`bufio` writer (Click to expand)</summary>
<p>

```golang
    file, _ := os.Create("./temp.txt")
    writer := bufio.NewWriter(file)
    writer.WriteString("Comment 1 \n")
    writer.WriteString("Comment 2 \n")
    writer.Flush()
```
</p>
</details>


# Accessing Google Calendar
The code is copied from the [Google Calendar documentation][calendar-doc]. 
I only extracted the function `startCalendarService()` for more readability, and the `calendarService` so that it is accessible from the handler.

On the first access, the application guides the user to allow access to the chosen calendar.
A credential file and a token file are necessary for the app to work. 
Once those files are obtained, there is no need to download them again.


<details><summary>

`Code to start Google Calendar service` (Click to expand)</summary>
<p>

```golang
// Calendar service functions: getClient, getTokenFromWeb, tokenFromFile, saveToken and startCalendarService

var calendarService  *calendar.Service;

// Retrieve a token, saves the token, then returns the generated client.
func getClient(config *oauth2.Config) *http.Client {
        // The file token.json stores the user's access and refresh tokens, and is
        // created automatically when the authorization flow completes for the first
        // time.
        tokFile := "token.json"
        tok, err := tokenFromFile(tokFile)
        if err != nil {
                tok = getTokenFromWeb(config)
                saveToken(tokFile, tok)
        }
        return config.Client(context.Background(), tok)
}

// Request a token from the web, then returns the retrieved token.
func getTokenFromWeb(config *oauth2.Config) *oauth2.Token {
        authURL := config.AuthCodeURL("state-token", oauth2.AccessTypeOffline)
        fmt.Printf("Go to the following link in your browser then type the "+
                "authorization code: \n%v\n", authURL)

        var authCode string
        if _, err := fmt.Scan(&authCode); err != nil {
                log.Fatalf("Unable to read authorization code: %v", err)
        }

        tok, err := config.Exchange(context.TODO(), authCode)
        if err != nil {
                log.Fatalf("Unable to retrieve token from web: %v", err)
        }
        return tok
}

// Retrieves a token from a local file.
func tokenFromFile(file string) (*oauth2.Token, error) {
        f, err := os.Open(file)
        if err != nil {
                return nil, err
        }
        defer f.Close()
        tok := &oauth2.Token{}
        err = json.NewDecoder(f).Decode(tok)
        return tok, err
}

// Saves a token to a file path.
func saveToken(path string, token *oauth2.Token) {
        fmt.Printf("Saving credential file to: %s\n", path)
        f, err := os.OpenFile(path, os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0600)
        if err != nil {
                log.Fatalf("Unable to cache oauth token: %v", err)
        }
        defer f.Close()
        json.NewEncoder(f).Encode(token)
}

func startCalendarService(){
    b, err := ioutil.ReadFile("credentials.json")
    if err != nil {
            log.Fatalf("Unable to read client secret file: %v", err)
    }

    // If modifying these scopes, delete your previously saved token.json.
    config, err := google.ConfigFromJSON(b, calendar.CalendarReadonlyScope)
    if err != nil {
            log.Fatalf("Unable to parse client secret file to config: %v", err)
    }
    client := getClient(config)

    calendarService, err = calendar.New(client)
    if err != nil {
            log.Fatalf("Unable to retrieve Calendar client: %v", err)
    }
}
```
</p>
</details>




# Launching the Http server
I followed [golang documentation][web-app-doc], which I modified slightly to be able to [stop gracefully][so-stopHttp]. The [Http doc][http-doc] can also be useful.

The handler follows the following pattern:
```golang
func handler(w http.ResponseWriter, r *http.Request) {
	 ....
    t, _ := template.New("template.html").Funcs(fmap).ParseFiles("template.html")

    t.Execute(w, TemplateArgs)
```


<details><summary>

`Code to start Http server` (Click to expand)</summary>
<p>

```golang
// start and stop Http server
func startHttpServer(wg *sync.WaitGroup) *http.Server{
    log.Printf("main: starting HTTP server")

    wg.Add(1)
    httpServer := &http.Server{Addr: ":8080"}

    http.Handle("/resources/", http.StripPrefix("/resources/", http.FileServer(http.Dir("resources")))) 
    http.HandleFunc("/", handler)

    go func() {
        defer wg.Done() // let main know we are done cleaning up

        // always returns error. ErrServerClosed on graceful close
        if err := httpServer.ListenAndServe(); err != http.ErrServerClosed {
            // unexpected error. port in use?
            log.Fatalf("ListenAndServe(): %v", err)
        }
    }()
    return(httpServer)

}

func stopHttpServer(wg *sync.WaitGroup, httpServer *http.Server){
    log.Printf("main: stopping HTTP server")

    // now close the server gracefully ("shutdown")
    // timeout could be given with a proper context
    // (in real world you shouldn't use TODO()).
    if err := httpServer.Shutdown(context.TODO()); err != nil {
        panic(err) // failure/timeout shutting down the server gracefully
    }

    // wait for goroutine started in startHttpServer() to stop
    wg.Wait()

    log.Printf("main: done. exiting")
}
```
</p>
</details>




# Displaying the events on a Html page

The handler first refresh the data needed for the calendar (occupied days and events). 
The refresh function is described below. 

I use `template.FuncMap` ([template][template-doc]) to make three functions (`iterate`, `contains` and `getColor`) accessible to the html template (see below `template.html`). 
When executing the template created by parsing the html file, I can pass arguments.

<details><summary>

`handler` (Click to expand)</summary>
<p>

```golang

// Handler functions: iterate, contains and getColor
func iterate(count int) []int {
    var i int
    var Items []int
    for i = 0; i < (count); i++ {
        Items = append(Items, i+1)
    }
    return Items
}

func contains(s []OccupiedDay, e int) bool {
    for _, a := range s {
        //fmt.Printf("a: %v ; e: %v", a, e)
        if a.DayNumber == e {
            return true
        }
    }
    return false
}

func getColor(s []OccupiedDay, e int) string {
    for _, a := range s {
        //fmt.Printf("a: %v ; e: %v", a, e)
        if a.DayNumber == e {
            return a.Color
        }
    }
    return ""
}


// Runs server
func handler(w http.ResponseWriter, r *http.Request) {
    OccupiedDaysList, events := refreshOccupiedDaysList()

    fmap:= template.FuncMap{"Iterate": iterate,
                            "IsOccupied": func(day int, monthStr string) bool {
                                        var month, _= strconv.Atoi(monthStr)
                                        return contains(OccupiedDaysList[month], day)},
                            "DayColor": func(day int, monthStr string) string {
                                        var month, _= strconv.Atoi(monthStr)
                                        return getColor(OccupiedDaysList[month], day)},
                            }
    
    t, _ := template.New("template.html").Funcs(fmap).ParseFiles("template.html")

    t.Execute(w, TemplateArgs{MonthsDefinition, events})
}


```
</p>
</details>


In the template, I can access arguments by using `{ { . }}`.
In my case the argument is a struct that has two properies: `Months` and `Events`, that I can access with ` { {.Months} }`.
The functions defined in the FuncMap can be accessed with the double brackets, for instance `{ { IsOccupied 1 2 }}` will print the return value of the function `isOccupied(1,2)`.

For loops are defined as follow, where `.Events` is a list
```html
{ { range $event := .Events  } }
  <p>  { { $event } }</p>
{ { end  } }
```

If-else statements are defined as follow, where `isOccupied(val, index)` returns a bool and `getColor(val, index)` (mapped to "DayColor") returns a string of a color. The class css are defined in `calendrier.css`
```html
{ { if IsOccupied $val $month.Index  } }
    <span class="day occupied" style="border-color: { { DayColor $val $month.Index } };">{ {  $val  } } </span>
{ { else } }
    <span class="day">{ {  $val  } } </span>
{ { end } }
```

<details><summary>

`template.html` (Click to expand)</summary>
<p>
The space between the double brackets is added here otherwise the content does not appear in the article. 

```html
<!DOCTYPE html>
<html>
<head>
  <title>Calendrier Toulon</title>
  <link rel="stylesheet" style="text/css" href="resources/calendrier.css">
</head>
<body>
  <h1>Calendrier Toulon Indivision</h1>
  <p></p>

{ { range $month := .Months } }
    <div class="month">
      { { $month.Name } }  
      <div class="days">
        { { range $val := Iterate $month.StartDay  } }
          <span class="day">  </span>
      { { end  } }

          { { range $val := Iterate $month.Days  } }
              { { if IsOccupied $val $month.Index  } }
            <span class="day occupied" style="border-color: { { DayColor $val $month.Index } };">{ {  $val  } } </span>
          { { else } }
            <span class="day">{ {  $val  } } </span>
          { { end } }
      { { end  } }
    </div>
    </div>
{ { end } }

{ { range $event := .Events  } }
  <p>  { { $event } }</p>
{ { end  } }

</body>
</html>

```
</p>
</details>

<details><summary>

`calendrier.css` (Click to expand)</summary>
<p>
 
```css

.days {
    border-style: solid;
    border-width: 3px;
    border-color: #00d4ce;
    color:black;
    width: 210px;
}

.day {
	width: 25px;
	color: black;
	display: inline-block;
}

.occupied {
	color: #808080;
	border-width: 2px;
	border-style: solid;
}

.month{
	color:#00d4ce;
	display: inline-block;
	height: 160px;
	vertical-align: top;

}

```
</p>
</details>


The following code allow to get *future* events from the Google Calendar (past events are not considered).
`OccupiedDaysList` maps each month with the occupied days for this month and `EventList` is a list of events description to print below the calendar.


<details><summary>

`refresh functions` (Click to expand)</summary>
<p>

```golang
type OccupiedDay struct {  
    DayNumber int
    Color string
}

type Month struct {  
    Name string
    Index string
    Days  int
    StartDay int
    //OccupiedDays []int
}

type TemplateArgs struct {
    Months []Month
    Events []string
}

var calendarService  *calendar.Service;


var MonthsDefinition = []Month {
    Month{"jan","01",31, 4},
    Month{"feb","02", 28, 0},
    Month{"mar","03",31, 0},
    Month{"avr","04", 30,3},
    Month{"may","05", 31,5},
    Month{"jun","06", 30,1},
    Month{"jul","07", 31,3},
    Month{"aug","08", 31,6},
    Month{"sep","09", 30,2},
    Month{"oct","10", 31,4},
    Month{"nov","11", 30,0},
    Month{"dec","12", 31,2}}

var colorIdDict = map[string]string{
    "11": "red",
    "6": "orange",
    "":"blue",
    "1":"blue",
    "2":"green",
    "3":"blue",
    "4":"blue",
    "5":"blue",
    "7":"blue",
    "8":"blue",
    "9":"blue",
    "10":"blue",
}


// refreshOccupiedDaysList functions: getYMD (Year Month Day) and appendODL (OccupiedDaysList)
func getYMD(startDate string)(int, int, int){
    ymd := strings.Split(startDate, "-")
    y,_ := strconv.Atoi(ymd[0])
    m,_ := strconv.Atoi(ymd[1])
    d,_ := strconv.Atoi(ymd[2])
    return y,m,d
}

func appendODL(OccupiedDaysList map[int][]OccupiedDay, month int, startDay int, endDay int, colorID string) {
    for day := startDay; day < endDay; day ++{
        OccupiedDaysList[month]= append(OccupiedDaysList[month], OccupiedDay{day, colorIdDict[colorID]})
    }
}

func refreshOccupiedDaysList() (map[int][]OccupiedDay, []string) {
    OccupiedDaysList := make(map[int][]OccupiedDay)
    var EventList []string
    t := time.Now().Format(time.RFC3339)
    events, err := calendarService.Events.List("primary").ShowDeleted(false).
        SingleEvents(true).TimeMin(t).MaxResults(10).OrderBy("startTime").Do()
    if err != nil {
        log.Fatalf("Unable to retrieve next ten of the user's events: %v", err)
    }

    fmt.Println("Upcoming events:")
    if len(events.Items) == 0 {
            fmt.Println("No upcoming events found.")
    } else {
        for _, item := range events.Items {
            colorID := item.ColorId
            
            startDate := item.Start.Date
            if startDate == "" {
                fmt.Printf("**Should add all-day events in calendar, on %v \n", item.Start.DateTime)
            }
            fmt.Printf("# %v (%v)\n", item.Summary, startDate)

            y, m, d := getYMD(startDate)            
            OccupiedDaysList[m]= append(OccupiedDaysList[m], OccupiedDay{d, colorIdDict[colorID]})
            fmt.Printf("\tSTART DATE: year: %v, month: %v, day: %v \n", y, m, d)

            endDate :=item.End.Date
            if endDate != startDate {
                endY, endM,endD := getYMD(endDate)
                fmt.Printf("\tENDDATE: year: %v, month: %v, day: %v \n", endY, endM, endD)
                
                if m == endM{
                    appendODL(OccupiedDaysList, m, d, endD, colorID)
                    
                } else{ 
                    appendODL(OccupiedDaysList,m, d, 32, colorID)
                    for month := m+1; month < endM; month ++{
                        appendODL(OccupiedDaysList, month, 0, 32, colorID)
                    }
                    appendODL(OccupiedDaysList, endM, 0, endD, colorID)
                }    
            }

            fmt.Printf("\tcolorID: '%v' aka %v \n",colorID, colorIdDict[colorID])

            EventList = append(EventList, "Du "+startDate+" au "+endDate+ " : "+ item.Summary)
            // end foreach items
        }
    }

    fmt.Println("-------------\nClick Enter to exit\n")
    return OccupiedDaysList, EventList
}

```
</p>
</details>



[github]: https://github.com/ChloeVincent/CalendrierToulonApp
[calendar-doc]: https://developers.google.com/calendar/quickstart/go
[so-stopHttp]: https://stackoverflow.com/questions/39320025/how-to-stop-http-listenandserve
[web-app-doc]: https://golang.org/doc/articles/wiki/
[http-doc]: https://golang.org/pkg/net/http/
[bufio-doc]: https://golang.org/pkg/bufio/
[template-doc]: https://golang.org/pkg/text/template/