---
title: "Hello Hugo World"
date: 2021-10-24
tags: ["first"]
author: "Rafael Toguko"
draft: false
hidemeta: false
comments: false
description: "Hello world of Papermod with Hugo."
canonicalURL: "https://canonical.url/to/page"
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
editPost:
    URL: "http://github.com/toguko/toguko.github.io/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

I'm migrating my old Pelican Blog to a Hugo Blog engine, so this is just a test before starting to put my posts here, and also on medium and [LinkedIn](https://www.linkedin.com/in/rafaeldias1/).

---

Below is just to test of the PaperMod typography, image, code syntax highlight, etc.

# Header 1
Text below header 1

## Header 2
Text below header 2

### Header 3
Text below header 3

### Topics:
Topics with header 3
- Headers
- Table Test
- Code highlight
  - Python Code highlight
  - Go code highlight
- Images
  - PNG image
  - JPG image

### Table Test
Table test with all rows centralized

|Aligment Left  | Description  | Test Text     |
|   :----       |    :----:    |     ----:     |
| Header        | center       | Here's right alignment  |
| Paragraph     | CENTER       | And here is a big text to show how it will be on different screen sizes      |


## Python Code
```python3
# Solve the quadratic equation ax**2 + bx + c = 0

# import complex math module
import cmath

a = 1
b = 5
c = 6

# calculate the discriminant
d = (b**2) - (4*a*c)

# find two solutions
sol1 = (-b-cmath.sqrt(d))/(2*a)
sol2 = (-b+cmath.sqrt(d))/(2*a)

print('The solution are {0} and {1}'.format(sol1,sol2))
```

## Golang Code
```go
// simple for loop in go
package main
import "fmt"

func main() {  
var i int
for i = 1; i <= 5; i++ {
fmt.Println(i)
    }
}
```



### Image Test

Goku PNG
![Goku SS](/images/goku.png)

Dragon Ball Z poster in JPG
![Dragon Ball Z](/images/dbz.jpg)

