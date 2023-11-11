---
layout: post
title: Belajar Go Fiber
modified: 2023-01-22T21:08:21+07:00
categories:
description: belajar go fiber
tags: [Go-Lang, gofiber, fiber, framework]
image:
    background: abstract-2.png
comments: true
share: true
date: 2023-01-22T21:08:21+07:00
---

Pada artikel [instalasi perlengkapan coding golang](https://rizkimufrizal.github.io/instalasi-perlengkapan-coding-golang), penulis sudah membahas mengenai bagaiman cara instalasi golang. Pada artikel ini, penulis akan membahas salah satu framework yang lumayan banyak digunakan yaitu [gofiber](https://gofiber.io) atau biasa juga disebut fiber dan penulis akan memberikan contoh bagaimana penggunaan gofiber.

## Apa itu GoFiber ?

>>GoFiber merupakan salah satu framework Go-Lang yang dibangun dengan Fasthttp dimana Fasthttp ini merupakan salah satu library atau package yang dibuat untuk keperluan server side.

GoFiber sendiri dikembangan berdasarkan inspirasi dari framework [express js](https://expressjs.com). Jadi bagi teman - teman yang pernah melakukan development dengan express js maka tidak akan asing lagi dengan framework gofiber ini.

## Membuat Project

Untuk membuat project, silahkan buat sebuah folder yaitu `belajar-gofiber` lalu jalankan perintah

```shell
go mod init github.com/RizkiMufrizal/belajar-gofiber
```

Teman - teman bisa sesuaikan dengan nama project yang telah dibuat. Pada artikel ini, kita akan membuat restful api CRUD sederhana, pasti nya menggunakan ORM dari [GORM](https://gorm.io). Untuk melakukan instalasi gofiber silahkan jalankan perintah berikut.

```shell
go get github.com/gofiber/fiber/v2
```

Lalu untuk gorm, silahkan jalankan perintah berikut.

```shell
go get -u gorm.io/gorm
```

Untuk driver mysql, silahkan jalankan perintah berikut.

```shell
go get -u gorm.io/driver/mysql
```

Lalu untuk konfigurasi .env, silahkan jalankan perintah berikut.

```shell
go get -u github.com/joho/godotenv
```

## Standard Struktur Folder

Sebenarnya di dalam gofiber ini tidak ada suatu standard struktur folder melainkan dari masing - masing developer dapat menentukan standard nya.

```shell
➜  belajar-gofiber git:(master) ✗ tree -c
.
├── .env
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── README.md
├── go.mod
├── go.sum
├── main.go
├── configuration
├── controller
├── entity
├── middleware
├── model
├── exception
├── repository
│   └── impl
└── service
    └── impl
```

1. .env : digunakan untuk menyimpan parameter seperti koneksi ke database, pooling dan sebagainya.
2. .gitignore : digunakan untuk list file atau folder yang akan di ignore atau tidak ikut commit.
3. Dockerfile : digunakan untuk konfigurasi docker.
4. docker-compose.yml : digunakan untuk konfigurasi docker compose, biasa nya digunakan untuk menjalankan aplikasi pihak ketiga yang dibutuhkan oleh aplikasi kita seperti database, message broker dan lain sebagai nya.
5. README.md : digunakan untuk informasi dari sebuah project.
6. go.mod dan go.sum : adalah go module sebagai dependency management pada golang.
7. main.go : digunakan untuk main file yang akan di jalankan di golang.
8. configuration : digunakan untuk menyimpan konfigurasi seperti konfigurasi ke database, logger dan lain sebagain nya.
9. controller : digunakan untuk menghadle request yang masuk ke aplikasi.
10. entity : digunakan untuk mendefinisikan entity yang merepresentasikan ke table pada database.
11. middleware : digunakan untuk processing yang berkaitan pada pre request maupun pre response. Biasa nya digunakan untuk pengecekan security, logging dan sebagainya.
12. model : digunakan sebagai DTO (data transfer object), dimana biasa nya pada package ini terdapat object - object yang dibutuhkan untuk request dan response.
13. exception : digunakan untuk kumpulan exception.
14. repository : digunakan untuk akses ke database (repository pattern).
15. repository/impl : digunakan untuk akses ke database (implementasi dari repository)
16. service : digunakan untuk bisnis logic
17. service/impl : digunakan untuk bisnis logic (implementasi dari service)

## Membuat Konfigurasi

Untuk proses membuat konfigurasi ada beberapa tahapan yaitu

### Membuat Konfigurasi .env

Pada file .env terdapat beberapa konfigurasi yaitu server port, database, database pooling dan user basic auth. Silahkan buka file `.env` lalu tambahkan code berikut

```properties
SERVER.PORT=0.0.0.0:9999

#Database Config
DATASOURCE_USERNAME=root
DATASOURCE_PASSWORD=root
DATASOURCE_HOST=localhost
DATASOURCE_PORT=3306
DATASOURCE_DB_NAME=belajar_gofiber

#Pool Config
DATASOURCE_POOL_MAX_CONN=10
DATASOURCE_POOL_IDLE_CONN=5
DATASOURCE_POOL_LIFE_TIME=30000

#User Basic Auth
username=admin
password=admin
```

### Membuat Konfigurasi Pada Package configuration dan exception

Pada package exception, silahkan buat file `error.go` lalu masukkan code berikut

```go
package exception

func PanicLogging(err interface{}) {
    if err != nil {
        panic(err)
    }
}
```

Code diatas berfungsi sebagai global error, jadi kita tidak perlu secara manual membuat panic, cukup panggil function tersebut.

Pada package configuration, silahkan buat 2 file yaitu `config.go` dan `database.go`. Silahkan buka file `config.go` lalu masukan konfigurasi seperti berikut.

```go
package configuration

import (
    "github.com/RizkiMufrizal/belajar-gofiber/exception"
    "github.com/joho/godotenv"
    "os"
)

type Config interface {
    Get(key string) string
}

type configImpl struct {
}

func (config *configImpl) Get(key string) string {
    return os.Getenv(key)
}

func New(filenames ...string) Config {
    err := godotenv.Load(filenames...)
    exception.PanicLogging(err)
    return &configImpl{}
}
```

Code diatas berfungsi untuk memanggil file .env, sehingga kita bisa menggunakan parameter - parameter yang terdapat pada file .env.

Lalu pada file `database.go` silahkan isi code berikut.

```go
package configuration

import (
    "github.com/RizkiMufrizal/belajar-gofiber/exception"
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "gorm.io/gorm/logger"
    "log"
    "math/rand"
    "os"
    "strconv"
    "time"
)

func NewDatabase(config Config) *gorm.DB {
    username := config.Get("DATASOURCE_USERNAME")
    password := config.Get("DATASOURCE_PASSWORD")
    host := config.Get("DATASOURCE_HOST")
    port := config.Get("DATASOURCE_PORT")
    dbName := config.Get("DATASOURCE_DB_NAME")
    maxPoolOpen, err := strconv.Atoi(config.Get("DATASOURCE_POOL_MAX_CONN"))
    maxPoolIdle, err := strconv.Atoi(config.Get("DATASOURCE_POOL_IDLE_CONN"))
    maxPollLifeTime, err := strconv.Atoi(config.Get("DATASOURCE_POOL_LIFE_TIME"))
    exception.PanicLogging(err)

    loggerDb := logger.New(
        log.New(os.Stdout, "\r\n", log.LstdFlags),
        logger.Config{
            SlowThreshold:             time.Second,
            LogLevel:                  logger.Info,
            IgnoreRecordNotFoundError: true,
            Colorful:                  true,
        },
    )

    db, err := gorm.Open(mysql.Open(username+":"+password+"@tcp("+host+":"+port+")/"+dbName+"?parseTime=true"), &gorm.Config{
        Logger: loggerDb,
    })
    exception.PanicLogging(err)

    sqlDB, err := db.DB()
    exception.PanicLogging(err)

    sqlDB.SetMaxOpenConns(maxPoolOpen)
    sqlDB.SetMaxIdleConns(maxPoolIdle)
    sqlDB.SetConnMaxLifetime(time.Duration(rand.Int31n(int32(maxPollLifeTime))) * time.Millisecond)

    //autoMigrate
    return db
}

```

Pada bagian autoMigrate, nanti nya akan kita gunakan agar table nya otomatis di migrate sehingga kita tidak perlu create secara manual.

## Membuat Entity dan Model

Pada bagian sebelum nya sudah dijelaskan bahwa entity akan berkaitan langsung dengan database sedangkan model akan berkaitan dengan DTO (data transfer object).

### Membuat Entity

Pada artikel ini, kita hanya membuat 1 table saja yaitu table product, silahkan buat sebuah file `product.go` di dalam folder entity lalu masukkan code berikut.

```go
package entity

type Product struct {
    Id       int32  `gorm:"primaryKey;auto_increment;column:product_id"`
    Name     string `gorm:"index;column:name;type:varchar(100)"`
    Price    int64  `gorm:"column:price"`
    Quantity int32  `gorm:"column:quantity"`
}

func (Product) TableName() string {
    return "tb_product"
}
```

Lalu silahkan buka file `database.go` kembali, pada bagian `//autoMigrate` silahkan tambahkan code berikut

```go
//autoMigrate
err = db.AutoMigrate(&entity.Product{})
exception.PanicLogging(err)
```

## Membuat Model

Pada package model, silahkan buat 1 buat file yaitu `product_model.go` lalu masukkan code berikut

```go
package model

type ProductModel struct {
    Id       int32  `json:"id"`
    Name     string `json:"name"`
    Price    int64  `json:"price"`
    Quantity int32  `json:"quantity"`
}

type ProductCreateOrUpdateModel struct {
    Name     string `json:"name" validate:"required"`
    Price    int64  `json:"price" validate:"required"`
    Quantity int32  `json:"quantity" validate:"required"`
}
```

Dapat dilihat pada product model, kita mempunya 2 struct, dimana struct ProductModel digunakan untuk menampilkan product, sedangkan struct ProductCreateOrUpdateModel digunakan hanya untuk create dan update product saja. Lalu silkahkan buat sebuah file `general_response.go` untuk standarisasi response. lalu masukkan code berikut.

```go
package model

type GeneralResponse struct {
    Code    int         `json:"code"`
    Message string      `json:"message"`
    Data    interface{} `json:"data"`
}
```

## Membuat Repository

Repository disini bertugas untuk mengakses ke database. Pada repository ini, kita akan membuat sebuah interface yang berfungsi sebagai mendeklarasikan function - function yang akan kita gunakan. Lalu interface ini nanti nya akan di implementasikan. Silahkan buat file `product_repository.go` pada package repository, lalu masukkan code berikut.

```go
package repository

import (
    "context"
    "github.com/RizkiMufrizal/belajar-gofiber/entity"
)

type ProductRepository interface {
    Insert(ctx context.Context, product entity.Product) entity.Product
    Update(ctx context.Context, product entity.Product) entity.Product
    Delete(ctx context.Context, product entity.Product)
    FindById(ctx context.Context, id int32) (entity.Product, error)
    FindAl(ctx context.Context) []entity.Product
}
```

Lalu pada package repository/impl, silahkan buat file `product_repository_impl.go` lalu masukkan code berikut

```go
package impl

import (
    "context"
    "errors"
    "github.com/RizkiMufrizal/belajar-gofiber/entity"
    "github.com/RizkiMufrizal/belajar-gofiber/exception"
    "github.com/RizkiMufrizal/belajar-gofiber/repository"
    "gorm.io/gorm"
)

func NewProductRepositoryImpl(DB *gorm.DB) repository.ProductRepository {
    return &productRepositoryImpl{DB: DB}
}

type productRepositoryImpl struct {
    *gorm.DB
}

func (repository *productRepositoryImpl) Insert(ctx context.Context, product entity.Product) entity.Product {
    err := repository.DB.WithContext(ctx).Create(&product).Error
    exception.PanicLogging(err)
    return product
}

func (repository *productRepositoryImpl) Update(ctx context.Context, product entity.Product) entity.Product {
    err := repository.DB.WithContext(ctx).Where("product_id = ?", product.Id).Updates(&product).Error
    exception.PanicLogging(err)
    return product
}

func (repository *productRepositoryImpl) Delete(ctx context.Context, product entity.Product) {
    err := repository.DB.WithContext(ctx).Delete(&product).Error
    exception.PanicLogging(err)
}

func (repository *productRepositoryImpl) FindById(ctx context.Context, id int32) (entity.Product, error) {
    var product entity.Product
    result := repository.DB.WithContext(ctx).Unscoped().Where("product_id = ?", id).First(&product)
    if result.RowsAffected == 0 {
        return entity.Product{}, errors.New("product Not Found")
    }
    return product, nil
}

func (repository *productRepositoryImpl) FindAl(ctx context.Context) []entity.Product {
    var products []entity.Product
    repository.DB.WithContext(ctx).Find(&products)
    return products
}
```

## Membuat Service

Pada bagian package service, code yang akan kita tulis lebih ke fungsi logic bisnis yang diperlukan. Silahkan buat file `product_service.go` pada package service lalu masukkan code berikut.

```go
package service

import (
    "context"
    "github.com/RizkiMufrizal/belajar-gofiber/model"
)

type ProductService interface {
    Create(ctx context.Context, model model.ProductCreateOrUpdateModel) model.ProductCreateOrUpdateModel
    Update(ctx context.Context, productModel model.ProductCreateOrUpdateModel, id int32) model.ProductCreateOrUpdateModel
    Delete(ctx context.Context, id int32)
    FindById(ctx context.Context, id int32) model.ProductModel
    FindAll(ctx context.Context) []model.ProductModel
}
```

Bisa dilihat dari code diatas, pada bagian service, kita akan menggunakan model, bukan entity untuk memunculkan atau menerima data product. Lalu buat sebuah file `product_service_impl.go` pada package service/impl, lalu masukkan code berikut

```go
package impl

import (
    "context"
    "github.com/RizkiMufrizal/belajar-gofiber/entity"
    "github.com/RizkiMufrizal/belajar-gofiber/exception"
    "github.com/RizkiMufrizal/belajar-gofiber/model"
    "github.com/RizkiMufrizal/belajar-gofiber/repository"
    "github.com/RizkiMufrizal/belajar-gofiber/service"
)

func NewProductServiceImpl(productRepository *repository.ProductRepository) service.ProductService {
    return &productServiceImpl{ProductRepository: *productRepository}
}

type productServiceImpl struct {
    repository.ProductRepository
}

func (service *productServiceImpl) Create(ctx context.Context, productModel model.ProductCreateOrUpdateModel) model.ProductCreateOrUpdateModel {
    product := entity.Product{
        Name:     productModel.Name,
        Price:    productModel.Price,
        Quantity: productModel.Quantity,
    }
    service.ProductRepository.Insert(ctx, product)
    return productModel
}

func (service *productServiceImpl) Update(ctx context.Context, productModel model.ProductCreateOrUpdateModel, id int32) model.ProductCreateOrUpdateModel {
    product := entity.Product{
        Id:       id,
        Name:     productModel.Name,
        Price:    productModel.Price,
        Quantity: productModel.Quantity,
    }
    service.ProductRepository.Update(ctx, product)
    return productModel
}

func (service *productServiceImpl) Delete(ctx context.Context, id int32) {
    product, err := service.ProductRepository.FindById(ctx, id)
    if err != nil {
        panic(exception.NotFoundError{
            Message: err.Error(),
        })
    }
    service.ProductRepository.Delete(ctx, product)
}

func (service *productServiceImpl) FindById(ctx context.Context, id int32) model.ProductModel {
    product, err := service.ProductRepository.FindById(ctx, id)
    exception.PanicLogging(err)

    return model.ProductModel{
        Id:       product.Id,
        Name:     product.Name,
        Price:    product.Price,
        Quantity: product.Quantity,
    }
}

func (service *productServiceImpl) FindAll(ctx context.Context) (responses []model.ProductModel) {
    products := service.ProductRepository.FindAl(ctx)
    for _, product := range products {
        responses = append(responses, model.ProductModel{
            Id:       product.Id,
            Name:     product.Name,
            Price:    product.Price,
            Quantity: product.Quantity,
        })
    }
    if len(products) == 0 {
        return []model.ProductModel{}
    }
    return responses
}
```

Pada bagian `exception.NotFoundError` pasti error karena kita belum membuat error handling tersebut, silahkan buat file `not_found_error.go` pada package exception lalu masukkan code berikut.

```go
package exception

type NotFoundError struct {
    Message string
}

func (notFoundError NotFoundError) Error() string {
    return notFoundError.Message
}
```

## Membuat Middleware

Sebelum lanjut ke controller, kita akan membuat middleware pada artikel ini. Middleware yang dibuat pada artikel disini hanya sebatas security dengan menggunakan basic authentication dan credentials nya sementara akan di hardcode di file .env. Silahkan buat file `basic_auth.go` di dalam package middleware lalu masukkan code berikut.

```go
package middleware

import (
    "encoding/base64"
    "github.com/RizkiMufrizal/belajar-gofiber/configuration"
    "github.com/RizkiMufrizal/belajar-gofiber/exception"
    "github.com/RizkiMufrizal/belajar-gofiber/model"
    "github.com/gofiber/fiber/v2"
    "strings"
)

func BasicAuth(config configuration.Config) fiber.Handler {
    return func(ctx *fiber.Ctx) error {
        username := config.Get("USERNAME")
        password := config.Get("PASSWORD")

        basicAuth := ctx.Get("Authorization")

        if basicAuth == "" {
            return ctx.
                Status(fiber.StatusBadRequest).
                JSON(model.GeneralResponse{
                    Code:    404,
                    Message: "Bad Request",
                    Data:    "Header Not Found",
                })
        }

        basicAuthDecode, err := base64.StdEncoding.DecodeString(strings.Split(basicAuth, " ")[1])
        exception.PanicLogging(err)
        basicAuthDecodeString := string(basicAuthDecode)
        basicAuthUsername := strings.Split(basicAuthDecodeString, ":")[0]
        basicAuthPassword := strings.Split(basicAuthDecodeString, ":")[1]

        if username != basicAuthUsername && password != basicAuthPassword {
            return ctx.
                Status(fiber.StatusUnauthorized).
                JSON(model.GeneralResponse{
                    Code:    401,
                    Message: "Unauthorized",
                    Data:    "Invalid Credentials",
                })
        }

        return ctx.Next()
    }
}
```

## Membuat Controller

Semua yang terdapat di dalam controller akan menghadle request yang datang dari user / client. Silahkan buat file `product_controller.go` pada package controller, lalu masukkan code berikut.

```go
package controller

import (
    "github.com/RizkiMufrizal/belajar-gofiber/configuration"
    "github.com/RizkiMufrizal/belajar-gofiber/exception"
    "github.com/RizkiMufrizal/belajar-gofiber/middleware"
    "github.com/RizkiMufrizal/belajar-gofiber/model"
    "github.com/RizkiMufrizal/belajar-gofiber/service"
    "github.com/gofiber/fiber/v2"
)

type ProductController struct {
    service.ProductService
    configuration.Config
}

func NewProductController(productService *service.ProductService, config configuration.Config) *ProductController {
    return &ProductController{ProductService: *productService, Config: config}
}

func (controller ProductController) Route(app *fiber.App) {
    app.Post("/v1/api/product", middleware.BasicAuth(controller.Config), controller.Create)
    app.Put("/v1/api/product/:id", middleware.BasicAuth(controller.Config), controller.Update)
    app.Delete("/v1/api/product/:id", middleware.BasicAuth(controller.Config), controller.Delete)
    app.Get("/v1/api/product/:id", middleware.BasicAuth(controller.Config), controller.FindById)
    app.Get("/v1/api/product", middleware.BasicAuth(controller.Config), controller.FindAll)
}

func (controller ProductController) Create(c *fiber.Ctx) error {
    var request model.ProductCreateOrUpdateModel
    err := c.BodyParser(&request)
    exception.PanicLogging(err)

    response := controller.ProductService.Create(c.Context(), request)
    return c.Status(fiber.StatusCreated).JSON(model.GeneralResponse{
        Code:    200,
        Message: "Success",
        Data:    response,
    })
}

func (controller ProductController) Update(c *fiber.Ctx) error {
    var request model.ProductCreateOrUpdateModel
    id, err := c.ParamsInt("id")
    err = c.BodyParser(&request)
    exception.PanicLogging(err)

    response := controller.ProductService.Update(c.Context(), request, int32(id))
    return c.Status(fiber.StatusOK).JSON(model.GeneralResponse{
        Code:    200,
        Message: "Success",
        Data:    response,
    })
}

func (controller ProductController) Delete(c *fiber.Ctx) error {
    id, err := c.ParamsInt("id")
    exception.PanicLogging(err)

    controller.ProductService.Delete(c.Context(), int32(id))
    return c.Status(fiber.StatusOK).JSON(model.GeneralResponse{
        Code:    200,
        Message: "Success",
    })
}

func (controller ProductController) FindById(c *fiber.Ctx) error {
    id, err := c.ParamsInt("id")
    exception.PanicLogging(err)

    result := controller.ProductService.FindById(c.Context(), int32(id))
    return c.Status(fiber.StatusOK).JSON(model.GeneralResponse{
        Code:    200,
        Message: "Success",
        Data:    result,
    })
}

func (controller ProductController) FindAll(c *fiber.Ctx) error {
    result := controller.ProductService.FindAll(c.Context())
    return c.Status(fiber.StatusOK).JSON(model.GeneralResponse{
        Code:    200,
        Message: "Success",
        Data:    result,
    })
}
```

## Membuat Exception Handler

Yang tidak kalah penting nya adalah membuat exception handler, Dimana exception ini agar aplikasi tidak berhenti dan dapat mengeluarkan error secara proper. Silahkan buat file `error_handler.go` pada package exception lalu isi code berikut

```go
package exception

import (
    "github.com/RizkiMufrizal/belajar-gofiber/model"
    "github.com/gofiber/fiber/v2"
)

func ErrorHandler(ctx *fiber.Ctx, err error) error {
    _, notFoundError := err.(NotFoundError)
    if notFoundError {
        return ctx.Status(fiber.StatusNotFound).JSON(model.GeneralResponse{
            Code:    404,
            Message: "Not Found",
            Data:    err.Error(),
        })
    }

    return ctx.Status(fiber.StatusInternalServerError).JSON(model.GeneralResponse{
        Code:    500,
        Message: "General Error",
        Data:    err.Error(),
    })
}
```

Lalu buat sebuah file `fiber.go` di dalam configuration lalu isi code berikut

```go
package configuration

import (
    "github.com/RizkiMufrizal/belajar-gofiber/exception"
    "github.com/gofiber/fiber/v2"
)

func NewFiberConfiguration() fiber.Config {
    return fiber.Config{
        ErrorHandler: exception.ErrorHandler,
    }
}
```

Code pada fiber.go nanti nya akan dipanggil senagai error handling, sehingga diharapkan aplikasi dapat mengeluarkan response code yang proper.

## Membuat Main

Silahkan buat file `main.go` di root folder, file ini bertugas sebagai main object yang akan dijalankan. Silahkan masukkan code nya seperti berikut.

```go
package main

import (
    "github.com/RizkiMufrizal/belajar-gofiber/configuration"
    "github.com/RizkiMufrizal/belajar-gofiber/controller"
    "github.com/RizkiMufrizal/belajar-gofiber/exception"
    repository "github.com/RizkiMufrizal/belajar-gofiber/repository/impl"
    service "github.com/RizkiMufrizal/belajar-gofiber/service/impl"
    "github.com/gofiber/fiber/v2"
    "github.com/gofiber/fiber/v2/middleware/cors"
    "github.com/gofiber/fiber/v2/middleware/recover"
)

func main() {
    //setup configuration
    config := configuration.New()
    database := configuration.NewDatabase(config)

    //repository
    productRepository := repository.NewProductRepositoryImpl(database)

    //service
    productService := service.NewProductServiceImpl(&productRepository)

    //controller
    productController := controller.NewProductController(&productService, config)

    //setup fiber
    app := fiber.New(configuration.NewFiberConfiguration())
    app.Use(recover.New())
    app.Use(cors.New())

    //routing
    productController.Route(app)

    //start app
    err := app.Listen(config.Get("SERVER.PORT"))
    exception.PanicLogging(err)
}
```

## Membuat Dockerfile dan Docker Compose

Pada tahap terakhir, kita perlu membuat dockerfile jika nanti suatu saat perlu di deploy ke kubernetes atau openshift. Silahkan buat file `Dockerfile` di root folder lalu masukkan code berikut.

```shell
# Get Go image from DockerHub.
FROM golang:1.19.5 AS api

# Set working directory.
WORKDIR /compiler

# Copy dependency locks so we can cache.
COPY go.mod go.sum ./

# Get all of our dependencies.
RUN go mod download

# Copy all of our remaining application.
COPY . .

# Build our application.
RUN CGO_ENABLED=0 GOOS=linux go build -o server ./main.go

# Use 'scratch' image for super-mini build.
FROM scratch AS prod

# Set working directory for this stage.
WORKDIR /app

# Copy our compiled executable from the last stage.
COPY --from=api /compiler/server .

# Run application and expose port 9999.
EXPOSE 9999
CMD ["./server"]
```

Untuk proses build docker image, teman - teman bisa lihat cara nya di [belajar docker](https://rizkimufrizal.github.io/belajar-docker).

Sedangkan untuk docker compose, silahkan buat file `docker-compose.yml` pada root folder lalu masukkan code berikut

```yaml
version: '3.1'

services:

  db:
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: belajar_gofiber
    ports:
      - "3306:3306"

  adminer:
    image: adminer
    restart: always
    ports:
      - "8080:8080"
```

Lalu jalankan docker compose dengan perintah

```shell
docker compose up
```

## Menjalankan, Build dan Test Aplikasi

Setelah selesai membuat code nya, tahapan selanjut nya adalah menjalankan aplikasi nya. Cara menjalankan silahkan gunakan perintah berikut.

```shell
go run ./main.go
```

Jika sudah muncul output berikut

```shell
┌───────────────────────────────────────────────────┐ 
│                   Fiber v2.41.0                   │ 
│               http://127.0.0.1:9999               │ 
│       (bound on host 0.0.0.0 and port 9999)       │ 
│                                                   │ 
│ Handlers ............ 16  Processes ........... 1 │ 
│ Prefork ....... Disabled  PID ............. 32394 │ 
└───────────────────────────────────────────────────┘
```

Berarti aplikasi gofiber nya sudah berjalan sebagai mesti nya. Untuk melakukan build, teman - teman cukup menjalankan perintah berikut.

```shell
go build ./main.go
```

Untuk melakukan test aplikasi, teman - teman bisa menggunakan postman atau perintah curl. Contoh nya disini penulis menggunakan perintah curl seperti berikut

```shell
curl --location --request POST 'http://localhost:9999/v1/api/product' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "product 1",
    "price": 1000,
    "quantity": 5
}'
```

dan hasil nya

```json
{"code":200,"message":"Success","data":{"name":"product 1","price":1000,"quantity":5}}
```

Atau jika teman - teman ingin menampilkan semua nya bisa menggunakan perintah berikut.

```shell
curl --location --request GET 'http://localhost:9999/v1/api/product' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Content-Type: application/json'
```

dan berikut hasil nya

```json
{"code":200,"message":"Success","data":[{"id":1,"name":"product 1","price":1000,"quantity":5}]}
```

Akhirnya selesai juga artikel mengenai belajar go fiber :). Untuk source code diatas dapat anda akses di [belajar gofiber](https://github.com/RizkiMufrizal/belajar-gofiber) atau jika teman - teman ingin yang lebih advance bisa lihat source nya di [gofiber clean architecture)](https://github.com/RizkiMufrizal/gofiber-clean-architecture). Sekian artikel mengenai Belajar gofiber, jika ada saran dan komentar silahkan isi dibawah dan terima kasih :).