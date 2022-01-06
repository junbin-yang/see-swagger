# see-swagger

middleware to automatically generate RESTFUL API documentation with Swagger 2.0.

## Usage

### Start using it

1. Add comments to your API source code, [See Declarative Comments Format](https://swaggo.github.io/swaggo.io/declarative_comments_format/).
2. Download [Swag](https://github.com/swaggo/swag) for Go by using:

```sh
go get -u github.com/swaggo/swag/cmd/swag
```

3. Run the [Swag](https://github.com/swaggo/swag) at your Go project root path(for instance `~/root/go-peoject-name`),
   [Swag](https://github.com/swaggo/swag) will parse comments and generate required files(`docs` folder and `docs/doc.go`)
   at `~/root/go-peoject-name/docs`.

```sh
swag init
```

4. Download [see-swagger](https://github.com/junbin-yang/see-swagger) by using:

```sh
go get -u github.com/junbin-yang/see-swagger
go get -u github.com/swaggo/files
```

Import following in your code:

```go
import "github.com/junbin-yang/see-swagger" // see-swagger middleware
import "github.com/swaggo/files" // swagger embed files

```

### Canonical example:

1. Add Comments for apis and main function with see-swagger rules like following:

```go
// @BasePath /api/v1

// PingExample godoc
// @Summary ping example
// @Schemes
// @Description do ping
// @Tags example
// @Accept json
// @Produce json
// @Success 200 {string} Helloworld
// @Router /example/helloworld [get]
func Helloworld(g *see.Context)  {
	g.JSON(http.StatusOK,"helloworld")
}
```

2. Use `swag init` command to generate a docs, docs generated will be stored at `docs/`.
3. import the docs like this:
   I assume your project named `github.com/go-project-name/docs`.

```go
import (
   docs "github.com/go-project-name/docs"
)
```

4. build your application and after that, go to http://localhost:8080/swagger/index.html ,you to see your Swagger UI.

5. The full code and folder relatives here:

```go
package main

import (
   "github.com/junbin-yang/see"
   docs "github.com/go-project-name/docs"
   swaggerfiles "github.com/swaggo/files"
   seeSwagger "github.com/junbin-yang/see-swagger"
   "net/http"
)
// @BasePath /api/v1

// PingExample godoc
// @Summary ping example
// @Schemes
// @Description do ping
// @Tags example
// @Accept json
// @Produce json
// @Success 200 {string} Helloworld
// @Router /example/helloworld [get]
func Helloworld(g *see.Context)  {
   g.JSON(http.StatusOK,"helloworld")
}

func main()  {
   r := see.Default()
   docs.SwaggerInfo.BasePath = "/api/v1"
   v1 := r.Group("/api/v1")
   {
      eg := v1.Group("/example")
      {
         eg.GET("/helloworld",Helloworld)
      }
   }
   r.GET("/swagger/*any", seeSwagger.WrapHandler(swaggerfiles.Handler))
   r.Run(":8080")

}
```

Demo project tree, `swag init` is run at relative `.`

```
.
├── docs
│   ├── docs.go
│   ├── swagger.json
│   └── swagger.yaml
├── go.mod
├── go.sum
└── main.go
```

## Configuration

You can configure Swagger using different configuration options

```go
func main() {
	r := see.New()

	seeSwagger.WrapHandler(swaggerFiles.Handler,
		seeSwagger.URL("http://localhost:8080/swagger/doc.json"),
		seeSwagger.DefaultModelsExpandDepth(-1))

	r.Run()
}
```

| Option                   | Type   | Default    | Description                                                  |
| ------------------------ | ------ | ---------- | ------------------------------------------------------------ |
| URL                      | string | "doc.json" | URL pointing to API definition                               |
| DocExpantion             | string | "list"     | Controls the default expansion setting for the operations and tags. It can be 'list' (expands only the tags), 'full' (expands the tags and operations) or 'none' (expands nothing). |
| DeepLinking              | bool   | true       | If set to true, enables deep linking for tags and operations. See the Deep Linking documentation for more information. |
| DefaultModelsExpandDepth | int    | 1          | Default expansion depth for models (set to -1 completely hide the models). |
| InstanceName             | string | "swagger"  | The instance name of the swagger document. If multiple different swagger instances should be deployed on one router, ensure that each instance has a unique name (use the _--instanceName_ parameter to generate swagger documents with _swag init_). |