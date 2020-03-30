# Go

## Notes
* **Dep** package management: https://gist.github.com/subfuzion/12342599e26f5094e4e2d08e9d4ad50d
* Channels: https://nanxiao.gitbooks.io/golang-101-hacks/content/posts/unbuffered-and-buffered-channels.html


## Stubs

### Web Scraping
``` go
package main

import (
	"bytes"
	"fmt"
	"net/http"
)

func main() {
	fmt.Println("STARTING...")
	response, err := http.Get("https://www.google.com/")
	if err != nil {
		fmt.Printf("%v\n", err)
	}
	defer response.Body.Close()

	buf := new(bytes.Buffer)
	buf.ReadFrom(response.Body)
	s := buf.String()

	fmt.Printf("Body: %v\n", s[:100])

	fmt.Println("FINISHED")
}
```

### XML Parsing
``` go
package main

import (
	"encoding/xml"
	"fmt"
	"io/ioutil"
	"os"
)

const xmlDataPath = "data/practice.xml"

func main() {
	fmt.Println("STARTING...")
	xmlFile, err := os.Open(xmlDataPath)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer xmlFile.Close()

	bytes, err := ioutil.ReadAll(xmlFile)
	if err != nil {
		fmt.Println(err)
		return
	}

	var note Note
	xml.Unmarshal(bytes, &note)

	fmt.Printf("Parsed:\n%v\n", note)
	fmt.Printf("Parsed:\n%v\n", note.XMLName)
	fmt.Printf("Parsed:\n%v\n", note.To)
	fmt.Printf("Parsed:\n%v\n", note.From)
	fmt.Printf("Parsed:\n%v\n", note.Heading)
	fmt.Printf("Parsed:\n%v\n", note.Body)

	fmt.Println("FINISHED")
}

type Note struct {
	XMLName xml.Name `xml:"note"`
	To string `xml:"to"`
	From string `xml:"from"`
	Heading string `xml:"heading"`
	Body string `xml:"body"`
}
```
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<note>
	<to>Tove</to>
	<from>Jani</from>
	<heading>Reminder</heading>
	<body>Don't forget me this weekend!</body>
</note>
```

### Wait Groups
``` go
package main
import (
    "fmt"
    "sync"
    "time"
)
func task1(a *[]string, c chan string, wg *sync.WaitGroup) {
    time.Sleep(3 * time.Second)
    *a = append(*a, "task1")
    fmt.Println("Processed task1")
    wg.Done()
}
func task2(a *[]int, c chan int, wg *sync.WaitGroup) {
    time.Sleep(2 * time.Second)
    *a = append(*a, 2)
    fmt.Println("Processed task2")
    wg.Done()
}
func main() {
    array1 := []string{}
    array2 := []int{}
    var wg sync.WaitGroup
    wg.Add(3)
    c1 := make(chan string)
    c2 := make(chan int)
    go task1(&array1, c1, &wg)
    go task2(&array2, c2, &wg)
    go task2(&array2, c2, &wg)
    wg.Wait()
    fmt.Println(array1)
    fmt.Println(array2)
}
```

### Struct Field Names Reflection
``` go
var showDataFields = [showDataFieldNumber]string{}
for i := 0; i < showDataFieldNumber; i++ {
  showDataFields[i] = reflect.Indirect(reflect.ValueOf(&showData)).Type().Field(i).Name
}
```
