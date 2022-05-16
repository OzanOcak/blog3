---
title: "API with Golang and Sqlite"
date: 2022-05-10T00:05:06-04:00
author: "ozan ocak"
tags: ["cURL", "web development", "CRUD", "Go", "Sqlite", "Api"]
---

## Api with go and sqlite

- compiles to machine code
- garbage collected
- goroutines / green threads
- channels for thread coordination
- statically typed
- no classes or type inheritance
- interfaces
- simple language, simple tools

```console
mkdir fiberapi
cd fiberapi
go
go version
go mod init "github.com/OzanOcak/fiberapi"
go get -u gorm.io/driver/mysql
go get -u gorm.io/gorm
go get github.com/gofiber/fiber/v2
```

Fiber is a Go web framework built on top of Fasthttp, the fastest HTTP engine for Go.
Gorm is a ORM library for golang

main.go

```go
package main

import (
	"log"
	"github.com/gofiber/fiber/v2"
)

func aloha(arg *fiber.Ctx)error{
	return arg.SendString("Hello world \n")
}

func main(){
	app := fiber.New()

	app.Get("/api",aloha)
	log.Fatal(app.Listen(":8000"))
}
```

```console
go mod tidy
go run main.go
curl localhost/api -X GET -I
mkdir database
touch database/database.go
```

open database.go

```go
package database

import (
	"log"
	"os"

	//"github.com/OzanOcak/api/models"

	"gorm.io/driver/sqlite"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
)

type DbInstance struct {
	Db *gorm.DB
}

var Database DbInstance

func ConnectDb() {
	db, err := gorm.Open(sqlite.Open("api.db"), &gorm.Config{})

	if err != nil {
		log.Fatal("Failed to connect to the database! \n", err)
		os.Exit(2)
	}

	log.Println("Connected Successfully to Database")
	db.Logger = logger.Default.LogMode(logger.Info)
	log.Println("Running Migrations")

	//db.AutoMigrate(&models.User{}, &models.Product{}, &models.Order{})

	Database = DbInstance{
		Db: db,
	}
}
```

```console
	go get gorm.io/driver/sqlite
	go getgorm.io/gorm
	go get gorm.io/gorm/logger

    mkdir models
	touch models/{user.go,product.go,order.go}
```

order.go

```go
package models

import "time"

type Order struct {
	ID           uint `json:"id" gorm:"primaryKey"`
	CreatedAt    time.Time
	ProductRefer int     `json:"product_id"`
	Product      Product `gorm:"foreignKey:ProductRefer"`
	UserRefer    int     `json:"user_id"`
	User         User    `gorm:"foreignKey:UserRefer"`
}
```

product.go

```go
package models

import "time"

type Product struct {
	ID           uint `json:"id" gorm:"primaryKey"`
	CreatedAt    time.Time
	Name         string `json:"name"`
	SerialNumber string `json:"serial_number"`
}
```

user.go

```go
package models

import "time"

type User struct {
	ID        uint `json:"id" gorm:"primaryKey"`
	CreatedAt time.Time
	FirstName string `json:"first_name"`
	LastName  string `json:"last_name"`
}
```

now we can migrate models in database.go

```go
"github.com/OzanOcak/fiberapi/models"

db.AutoMigrate(&models.User{}, &models.Product{}, &models.Order{})
```

and connect database in main function

```go
database.ConnectDb()
```

click sql tool after download sql vscode plugin, add new connection

with SQLite plugin in vs code , we can see db table within vs code

---

```go
mkdir routes
touch routes/user.go
```

user.go

```go
package routes

import (
	"api/database"
	"api/models"

	"github.com/gofiber/fiber/v2"
)

type User struct {
	ID uint `json:"id"`
	FirstName string `json:"first_name"`
	LastName string `json:"last_name"`
}

func CreateResponseUser(userModel models.User) User{
	return User{ID: userModel.ID, FirstName: userModel.FirstName, LastName: userModel.LastName}
}
func CreateUser(c *fiber.Ctx) error{
  var user models.User

  if err := c.BodyParser(&user); err != nil {
      return c.Status(404).JSON(err.Error())
  }
  database.Database.Db.Create(&user)
  responseUser := CreateResponseUser(user)

  return c.Status(200).JSON(responseUser);
}

```

main.go within setupRoutes and in main connect db

```go
	app.Post("/api/users",routes.CreateUser)
```

```console
 curl localhost:8000/users -X POST -d '{"first_name":"john","last_name":"doe"}' -H 'content-type:application/json'
```

---

```go
 func GetUsers(c *fiber.Ctx) error{
	 users := []models.User{}

	 database.Database.Db.Find(& users)
	 responseUsers := []User{}
	 for _,user := range users {
		 responseUser := CreateResponseUser(user)
		 responseUsers = append(responseUsers, responseUser)
	 }
	 return c.Status(200).JSON(responseUsers)

 }

```

```go
 	app.Get("/api/users",routes.GetUsers)
```

```console
curl localhost:8000/api/users -X GET
```

---

```go
func findUser(id int, user *models.User) error{
    database.Database.Db.Find(&user, "id=?",id)
	if user.ID == 0 {
		return errors.New("user does not exist ")
	}
	return nil
}

func GetUser(c *fiber.Ctx) error{
	id, err := c.ParamsInt("id")
	var user models.User

	if err !=nil {
		return c.Status(400).JSON("please enure that :id is an integer")
	}
	if err := findUser(id, &user); err != nil {
		return c.Status(400).JSON(err.Error())
	}

	responseUser := CreateResponseUser(user)

	return c.Status(200).JSON(responseUser)
}
```

and call the function in main function

```go
	app.Get("/api/users/:id",routes.GetUser)
```

```console
curl localhost:8000/users/2 -X GET
```

---

routes/user.go

```go
func UpdateUser(c *fiber.Ctx) error {
	id, err := c.ParamsInt("id")

	var user models.User

	if err != nil {
		return c.Status(400).JSON("Please ensure that :id is an integer")
	}

	err = findUser(id, &user)

	if err != nil {
		return c.Status(400).JSON(err.Error())
	}

	type UpdateUser struct {
		FirstName string `json:"first_name"`
		LastName  string `json:"last_name"`
	}

	var updateData UpdateUser

	if err := c.BodyParser(&updateData); err != nil {
		return c.Status(500).JSON(err.Error())
	}

	user.FirstName = updateData.FirstName
	user.LastName = updateData.LastName

	database.Database.Db.Save(&user)

	responseUser := CreateResponseUser(user)

	return c.Status(200).JSON(responseUser)

}
```

main.go

```go
app.Put("/users/:id",routes.UpdateUser)
```

```console
curl localhost:8000/users/3 -X PUT -d '{"first_name":"tom","last_name":"cruse"}' -H 'content-type:application/json'
```

---

```go
func DeleteUser(c *fiber.Ctx) error {
	id, err := c.ParamsInt("id")

	var user models.User

	if err != nil {
		return c.Status(400).JSON("Please ensure that :id is an integer")
	}

	err = findUser(id, &user)

	if err != nil {
		return c.Status(400).JSON(err.Error())
	}

	if err = database.Database.Db.Delete(&user).Error; err != nil {
		return c.Status(404).JSON(err.Error())
	}
	return c.Status(200).JSON("Successfully deleted User")
}
```

```go
app.DELETE("/users/:id",routes.DeleteUser)
```

```console
curl localhost:8000/users/3 -X DELETE
```

---

### routes/product.go

models/product.go

```go
package models

import "time"

type Product struct {
	ID           uint `json:"id" gorm:"primaryKey"`
	CreatedAt    time.Time
	Name         string `json:"name"`
	SerialNumber string `json:"serial_number"`
}
```

routes/product.go

```go
package routes

import (
	"github.com/OzanOcak/fiberapi/database"
	"github.com/OzanOcak/fiberapi/models"
	"github.com/gofiber/fiber/v2"
)

type Product struct {
	ID           uint   `json:"id"`
	Name         string `json:"name"`
	SerialNumber string `json:"serial_number"`
}

func CreateResponseProduct(product models.Product) Product {
	return Product {ID: product.ID, Name: product.Name, SerialNumber: product.SerialNumber}
}

func CreateProduct(c *fiber.Ctx) error {
	var product models.Product

	if err := c.BodyParser(&product); err != nil {
		return c.Status(400).JSON(err.Error())
	}

	database.Database.Db.Create(&product)
	responseProduct := CreateResponseProduct(product)
	return c.Status(200).JSON(responseProduct)
}
```

main.go

```go
	app.Post("/products",routes.CreateProduct)
```

terminal

```console
 curl localhost:8000/products -X POST -d '{"serial_number":"0002316","name":"Letrange"}' -H 'content-type:application/json'
```

---

product.go

```go
func GetProducts(c *fiber.Ctx) error {
	products := []models.Product{}
	database.Database.Db.Find(&products)
	responseProducts := []Product{}
	for _, product := range products {
		responseProduct := CreateResponseProduct(product)
		responseProducts = append(responseProducts, responseProduct)
	}

	return c.Status(200).JSON(responseProducts)
}
```

```go
app.GET("/products",routes.GetProducts)
```

```console
curl localhost:8000/products
```

---

```go
	app.Post("/orders",routes.CreateOrder)
```

```console
curl localhost:8000/orders -X POST -d '{"user_id":4,"product_id":1}' -H 'content-type:application/json'
```

```go
	app.Get("/orders",routes.GetOrders)
```

```console
curl localhost:8000/orders
```

```go
	app.Get("/orders/:id",routes.GetOrder)
```

```console
curl localhost:8000/orders/1
```
