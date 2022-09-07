---
title: "Go TDD"
series: ["Go TDD"]
date: 2022-09-07T00:00:00+00:00
draft: false
author: "Masum Osman Khan"
categories:
  - blog
tags:
  - Golang
  - TDD
  - DataBase
  - API
thumbnail: golang
---

So, I'm trying to test Golang's Test Package. I am trying the default and `convey` in here. I will try to go as far as possible today. I hope to run the series on to learn deeper.

## Here are the basic Skull of Go Test:

```jsx
package main

import "testing"

func TestDeckCount(t *testing.T) {
	got := -1
	if got != 1 {
		t.Errorf("Abs(-1) = %d; want 1", got)
	}
}

/*
RUN CMD: go test
*/
```

There is the example of testing GO code. No other 3rd party library is needed like mocha, jesmine, PHPUnit for Go.

---

### Experience:

```jsx
package main

import (
	"fmt"
	"testing"
)

func TestDeckCount(t *testing.T) {
	got := 81
	if got != 1 {
		t.Errorf("Abs(-1) = %d; want 1", got)
	}
}

func TestGetNumber(t *testing.T) {
	num := getNumber()
	fmt.Println("num is: ", num)
	if num != 100 {
		t.Errorf("num = %d; want 100", num)
	}
}

/*
RUN CMD: go test

--- FAIL: TestDeckCount (0.00s)
    deck_test.go:11: Abs(-1) = 81; want 1
num is:  10
--- FAIL: TestGetNumber (0.00s)
    deck_test.go:19: num = 10; want 100
FAIL
exit status 1
FAIL    go-udemy-basics/deck    0.504s
*/
```

> In Go test, no need to import the module to access it’s functions. Because it is in the same dir and same module, and furthermore a TEST file, it has access to all the function by nature.
> 

> The Test function must named under some rules.
> 
1. The Func should start with Test key word with the Capital T in Test

If the test is passed, then you will get only notified. But if the function failed, then you will the function name in return.

---

### Work Experience:

## Beego Testing:

Just normal testing Example (Where No DB conn is needed):

 

```go
func TestGet(t *testing.T) {
	r, _ := http.NewRequest("GET", "/1/object", nil)
	w := httptest.NewRecorder()
	beego.BeeApp.Handlers.ServeHTTP(w, r)

	logs.Info("testing", "TestGet", "Code[%d]\n%s", w.Code, w.Body.String())

	Convey("Subject: Test Station Endpoint\n", t, func() {
		Convey("Status Code Should Be 200", func() {
			So(w.Code, ShouldEqual, 200)
		})
		Convey("The Result Should Not Be Empty", func() {
			So(w.Body.Len(), ShouldBeGreaterThan, 0)
		})
	})
}
```

```go
func TestPostUser(t *testing.T) {
	r, _ := http.NewRequest("POST", "/1/user", nil)
	w := httptest.NewRecorder()
	beego.BeeApp.Handlers.ServeHTTP(w, r)

	Convey("Subject: Test Object HTTP POST\n", t, func() {
		Convey("Status Code Should Not be 200", func() {
			So(w.Code, ShouldNotEqual, 200)
		})
	})
}
```

### Running Test Cases:

```go
go test -v
```

But when you need to use any Data from the DB, you should create a new connection for the test purpose specifically. No matter you have the connection before or not.

So, we have done it by thus:

```go
import (
	"net/http"
	"net/http/httptest"
	"path/filepath"
	"price_generator/models"
	_ "price_generator/routers"
	errHandler "price_generator/service/ErrHandler"
	"runtime"
	"testing"

	"github.com/beego/beego/v2/client/orm"
	"github.com/beego/beego/v2/core/logs"
	beego "github.com/beego/beego/v2/server/web"
	_ "github.com/go-sql-driver/mysql"
	. "github.com/smartystreets/goconvey/convey"
)

func dbConnection() {
	runmode, _ := beego.AppConfig.String("runmode")

	_ = orm.RegisterDataBase("default", "mysql", "root:password@tcp(127.0.0.1:3306)/my_test_db?charset=utf8")
	force := false

	err := orm.RunSyncdb("default", force, beego.BConfig.RunMode == runmode)
	errHandler.FailOnError(err, "Failed On RunSyncError")

	logs.Info("Database Connection Established")
}

func init() {
	_, file, _, _ := runtime.Caller(0)
	apppath, _ := filepath.Abs(filepath.Dir(filepath.Join(file, ".."+string(filepath.Separator))))

	logs.Info("Database Connection Establishing...")
	_ = orm.RegisterDriver("mysql", orm.DRMySQL)
	dbConnection()

	beego.TestBeegoInit(apppath)
}
```

Then we are allowed to run our DB based test cases like those:

```go
func TestPriceEstimationAPIasHole(t *testing.T) {
	r, _ := http.NewRequest("GET", `/1/estimates/price/trip/?pick_lat=23.25645&drop_lat=23.25417&pick_lng=90.21546&drop_lng=90.21546`, nil)
	w := httptest.NewRecorder()
	beego.BeeApp.Handlers.ServeHTTP(w, r)

	Convey("Subject: Test Price Estimation API\n", t, func() {
		Convey("Status Code Should Be 200", func() {
			So(w.Code, ShouldEqual, 200)
		})
		Convey("The Result Should Not Empty", func() {
			So(w.Body.Len(), ShouldBeGreaterThanOrEqualTo, 0)
		})
	})
}
```

```go
func TestGetCar_type_based_cost_configByCarSize(t *testing.T) {
	costByCarSize, err := models.GetCar_type_based_cost_configByCarSize(1007)
	errHandler.FailOnError(err, "Failed to Get Cost From GetCar_type_based_cost_configByCarSize Model")

	Convey("Subject: Test Funtion GetCar_type_based_cost_configByCarSize\n", t, func() {
		Convey("Return Should Not Be Empty", func() {
			So(costByCarSize.FuelType, ShouldEqual, "diesel")
		})
	})
}
```

---

Testing the Test package:

```go
func TestGoTestingModule(t *testing.T) {
	Convey("Subject: Test Go Test Module\n", t, func() {
		Convey("145 Should Not Equal to 200", func() {
			So(145, ShouldNotEqual, 200)
		})
	})
}
```

### The Output:
### Initiate to run:
```bash
$ cd tests
$ go test -v
```
### The Result
``` bash
200 {
  "hjkhsbnmn123": {
    "ObjectId": "hjkhsbnmn123",
    "Score": 100,
    "PlayerName": "astaxie"
  },
  "mjjkxsxsaa23": {
    "ObjectId": "mjjkxsxsaa23",
    "Score": 101,
    "PlayerName": "someone"
  }
}

2022/09/07 20:41:17.548 [I] [server.go:241]  http server Running on http://:8105


  Subject: Test Station Endpoint
 
    Status Code Should Be 200 ✔
    The Result Should Not Be Empty ✔


2 total assertions

--- PASS: TestGet (0.00s)
2022/09/07 20:41:17.549 [C] [server.go:258]  ListenAndServe:  listen tcp :8105: bind: address already in use

=== RUN   TestGetObject
2022/09/07 20:41:17.549 [I] [default_test.go:66]  testing TestGet Code[%d]
200 {
  "hjkhsbnmn123": {
    "ObjectId": "hjkhsbnmn123",
    "Score": 100,
    "PlayerName": "astaxie"
  },
  "mjjkxsxsaa23": {
    "ObjectId": "mjjkxsxsaa23",
    "Score": 101,
    "PlayerName": "someone"
  }
}


  Subject: Test Object Endpoint
 
    Status Code Should Be 200 ✔
    The Result Should Not Be Empty ✔


4 total assertions

--- PASS: TestGetObject (0.00s)
=== RUN   TestGetUser
2022/09/07 20:41:17.550 [I] [default_test.go:83]  testing TestGet Code[%d]
%s 200 {
  "user_11111": {
    "Id": "user_11111",
    "Username": "astaxie",
    "Password": "11111",
    "Profile": {
      "Gender": "male",
      "Age": 20,
      "Address": "Singapore",
      "Email": "astaxie@gmail.com"
    }
  }
}


  Subject: Test Object Endpoint
 
    Status Code Should Be 200 ✔
    The Result Should Not Be Empty ✔


6 total assertions

--- PASS: TestGetUser (0.00s)
=== RUN   TestPostUser
2022/09/07 20:41:17.551 [E] [router.go:733]  Error parsing request body:missing form body


  Subject: Test Object HTTP POST
 
    Status Code Should Not be 200 ✔


7 total assertions

--- PASS: TestPostUser (0.00s)
=== RUN   TestPriceEstimationAPIasHole
&{1 1007 open 18 diesel 109 25 5 7 1 M 2022-09-05 17:18:56}
&{2 1007 18 85 union union 450 0 78 1 M 2022-09-06 11:09:03}

  Subject: Test Price Estimation API
 
    Status Code Should Be 200 ✔
    The Result Should Not Empty ✔


9 total assertions

```