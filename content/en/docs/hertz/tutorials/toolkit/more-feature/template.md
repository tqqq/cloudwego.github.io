---
Title: "hz custom template use"
Date: 2023-02-20
Weight: 2
keywords: ["hz client", "layout", "package"]
description: "hz custom template use."
---

Hertz provides command-line tools (hz) that support custom template features, including:

- Customize the layout template (i.e., the directory structure for generating code, which is independent of specific idl definitions and can be directly generated without the need for idl)
- Custom package templates (i.e. the code structure related to the definition of idl, including handler, model, router, etc.)

Taking the code structure in [The structure of the generated code](/docs/hertz/tutorials/toolkit/layout/) as an example (integrating hz and hz client generated code), introduce the default template provided by hz:

```
.
├── biz
│   ├── handler                          // biz/handler is the default handler_dir, which can be modified through --handler_dir
│   │   ├── hello                        // handler related code, related to idl, package template generation
│   │   │   └── example
│   │   │       └── hello_service.go
│   │   └── ping.go                      // layout template generation
│   ├── model                            // biz/model is the default model_dir, which can be modified through --model_dir
│   │   └── hello
│   │       └── example
│   │           └── hello.go             // generated by hz calling thriftgo, without involving layout and package templates
│   │           └── hello_service        // call the hz client command to obtain, related to idl, package template generation
│   │               └── hello_service.go
│   │               └── hertz_client.go
│   └── router                           // biz/router is the default router_dir, which can be modified through --router_dir
│       ├── hello                        // router related code, related to idl, package template generation
│       │   └── example
│       │       ├── hello.go
│       │       └── middleware.go
│       └── register.go                  // when idl is not specified, it is generated by the layout template; when specifying idl, generated by the package template
├── go.mod                               // go.mod file, layout template generation
├── main.go                              // program entry, layout template generation
├── router.go                            // user defined routing methods other than idl, layout template generation
└── router_gen.go                        // route registration code generated by hz, used to call user-defined routes and routes generated by hz, generated by layout templates
├── .hz
├── build.sh                             // program compilation script, layout template generation
├── script
│   └── bootstrap.sh                     // program running script, layout template generation
└── .gitignore                           // layout template generation
```

Users can provide their own templates and rendering parameters, and combine the capabilities of hz to complete custom code generation structures.

## Layout template

hz utilizes go template support to define layout templates in the format of "yaml" and uses "json" to define templates for rendering data。

Users can modify or rewrite according to the default template to meet their own needs.

> Note: When a custom template rendering file is not specified on the command line, hz will use [default rendering parameter](#template-rendering-parameter-file) to render the custom layout template. At this time, it should be ensured that the rendering parameters of the custom layout template are within the range of [default rendering parameters](#template-rendering-parameter-file).

### Default layout template

> Note: The following bodies are all go templates

```yaml
layouts:
  # The directory of the generated dal will only be generated if there are files in the directory
  - path: biz/dal/
    delims:
      - ""
      - ""
    body: ""
  # The directory of the generated handler will only be generated if there are files in the directory
  - path: biz/handler/
    delims:
      - ""
      - ""
    body: ""
  # The directory of the generated model will only be generated if there are files in the directory
  - path: biz/model/
    delims:
      - ""
      - ""
    body: ""
  # The directory of the generated service will only be generated if there are files in the directory
  - path: biz/service/
    delims:
      - ""
      - ""
    body: ""
  # project main.go file
  - path: main.go
    delims:
      - ""
      - ""
    body: |-
      // Code generated by hertz generator.

      package main

      import (
      	"github.com/cloudwego/hertz/pkg/app/server"
      )

      func main() {
      	h := server.Default()

      	register(h)
      	h.Spin()
      }
  # go.mod file, need template rendering data {{.GoModule}} {{.UseApacheThrift}} to generate
  - path: go.mod
    delims:
      - '{{'
      - '}}'
    body: |-
      module {{.GoModule}}
      {{- if .UseApacheThrift}}
      replace github.com/apache/thrift => github.com/apache/thrift v0.13.0
      {{- end}}
  # .gitignore file
  - path: .gitignore
    delims:
      - ""
      - ""
    body: "*.o\n*.a\n*.so\n_obj\n_test\n*.[568vq]\n[568vq].out\n*.cgo1.go\n*.cgo2.c\n_cgo_defun.c\n_cgo_gotypes.go\n_cgo_export.*\n_testmain.go\n*.exe\n*.exe~\n*.test\n*.prof\n*.rar\n*.zip\n*.gz\n*.psd\n*.bmd\n*.cfg\n*.pptx\n*.log\n*nohup.out\n*settings.pyc\n*.sublime-project\n*.sublime-workspace\n!.gitkeep\n.DS_Store\n/.idea\n/.vscode\n/output\n*.local.yml\ndumped_hertz_remote_config.json\n\t\t
    \ "
  # .hz file, containing hz version, is the logo of the project created by hz, no need to transfer rendering data
  - path: .hz
    delims:
      - '{{'
      - '}}'
    body: |-
      // Code generated by hz. DO NOT EDIT.

      hz version: {{.hzVersion}}
  # the ping.go file in the handler requires template rendering data {{.HandlerPkg}} to be generated
  - path: biz/handler/ping.go
    delims:
      - ""
      - ""
    body: |-
      // Code generated by hertz generator.

      package {{.HandlerPkg}}

      import (
      	"context"

      	"github.com/cloudwego/hertz/pkg/app"
      	"github.com/cloudwego/hertz/pkg/common/utils"
        "github.com/cloudwego/hertz/pkg/protocol/consts"
      )

      // Ping .
      func Ping(ctx context.Context, c *app.RequestContext) {
      	c.JSON(consts.StatusOK, utils.H{
      		"message": "pong",
      	})
      }
  # `router_gen.go` is the file that defines the route registration, need template rendering data {{.RouterPkgPath}} to generate
  - path: router_gen.go
    delims:
      - ""
      - ""
    body: |-
      // Code generated by hertz generator. DO NOT EDIT.

      package main

      import (
      	"github.com/cloudwego/hertz/pkg/app/server"
      	router "{{.RouterPkgPath}}"
      )

      // register registers all routers.
      func register(r *server.Hertz) {

      	router.GeneratedRegister(r)

      	customizedRegister(r)
      }
  # Custom route registration file, need template rendering data {{.HandlerPkgPath}} to generate
  - path: router.go
    delims:
      - ""
      - ""
    body: |-
      // Code generated by hertz generator.

      package main

      import (
      	"github.com/cloudwego/hertz/pkg/app/server"
      	handler "{{.HandlerPkgPath}}"
      )

      // customizeRegister registers customize routers.
      func customizedRegister(r *server.Hertz){
      	r.GET("/ping", handler.Ping)

      	// your code ...
      }
  # Default route registration file, do not modify it, need template rendering data {{.RouterPkg}} to generate
  - path: biz/router/register.go
    delims:
      - ""
      - ""
    body: |-
      // Code generated by hertz generator. DO NOT EDIT.

      package router

      import (
      	"github.com/cloudwego/hertz/pkg/app/server"
      )

      // GeneratedRegister registers routers generated by IDL.
      func GeneratedRegister(r *server.Hertz){
      	//INSERT_POINT: DO NOT DELETE THIS LINE!
      }
  # Compile script, need template rendering data {{.ServiceName}} to generate
  - path: build.sh
    delims:
      - ""
      - ""
    body: |-
      #!/bin/bash
      RUN_NAME={{.ServiceName}}
      mkdir -p output/bin
      cp script/* output 2>/dev/null
      chmod +x output/bootstrap.sh
      go build -o output/bin/${RUN_NAME}
  # Run script, need template rendering data {{.ServiceName}} to generate
  - path: script/bootstrap.sh
    delims:
      - ""
      - ""
    body: |-
      #!/bin/bash
      CURDIR=$(cd $(dirname $0); pwd)
      BinaryName={{.ServiceName}}
      echo "$CURDIR/bin/${BinaryName}"
      exec $CURDIR/bin/${BinaryName}
```

### Template rendering parameter file

hz uses "json" to specify rendering data, including global rendering parameters and rendering parameters for each file.

Global rendering parameters can be used in various files, and file rendering parameters can only be used for the files they belong to.

#### Global rendering parameters

The key for global rendering parameters is "\*", and hz provides the following five global rendering parameters by default:

| rendering parameter name | default       | description                                                                   |
| :----------------------- | :------------ | :---------------------------------------------------------------------------- |
| GoModule                 | -             | go module, can be specified through --module                                  |
| ServiceName              | hertz_service | service name, which can be specified through --service                        |
| UseApacheThrift          | -             | true when idl is thrift, otherwise false                                      |
| HandlerPkg               | handler       | the last level path of the handler_dir, can be modified through --handler_dir |
| RouterPkg                | router        | the last level path of the router_dir, can be modified through --router_dir   |

> Note: Except for UseApacheThrift, all other parameters can be specified through the command line. If this parameter is also specified in the custom rendering parameter file, ensure that the values of the two parameters are consistent, otherwise problems may occur. Therefore, we suggest that when using custom templates, ServiceName, HandlerPkg, and RouterPkg do not need to be specified on the command line, but can be specified in the rendering parameter file. When specifying go mod outside of GOPATH, consistency between the two should be ensured.

Users can customize global rendering parameter names and values according to their needs, but ensure that the key is "\*".

#### File rendering parameters

hz provides the following file rendering parameters by default:

```json
{
  "router_gen.go": {
    "RouterPkgPath": "{GoModule}/biz/router",
    "HandlerPkgPath": "{GoModule}/biz/handler"
  },

  "router.go": {
    "RouterPkgPath": "{GoModule}/biz/router",
    "HandlerPkgPath": "{GoModule}/biz/handler"
  }
}
```

The file rendering parameters only apply to the file they belong to, with key being the file name (based on the relative path of out_dir) and values that can be defined arbitrarily.

### Customize a layout template

> At present, the project layout generated by hz is already the most basic skeleton of a hertz project, so it is not recommended to delete the files in the existing template.
>
> However, if the user wants a different layout, of course, you can also delete the corresponding file according to your own needs (except "biz/register.go", the rest can be modified)
>
> We welcome users to contribute their own templates.

Assuming that the user only wants "main.go" and "go.mod" files, then we modify the default template as follows:

#### template

```yaml
# layout.yaml
layouts:
  # project main.go file
  - path: main.go
    delims:
      - ""
      - ""
    body: |-
      // Code generated by hertz generator.

      package main

      import (
      	"github.com/cloudwego/hertz/pkg/app/server"
        "{{.GoModule}}/biz/router"
      )

      func main() {
      	h := server.Default()

        router.GeneratedRegister(h)
        // do what you wanted
        // add some render data: {{.MainData}}

      	h.Spin()
      }

  # go.mod file, requires template rendering data {{.GoModule}} to generate
  - path: go.mod
    delims:
      - "{{"
      - "}}"
    body: |-
      module {{.GoModule}}
      {{- if .UseApacheThrift}}
      replace github.com/apache/thrift => github.com/apache/thrift v0.13.0
      {{- end}}
  # Default route registration file, no need to modify
  - path: biz/router/register.go
    delims:
      - ""
      - ""
    body: |-
      // Code generated by hertz generator. DO NOT EDIT.

      package router

      import (
      	"github.com/cloudwego/hertz/pkg/app/server"
      )

      // GeneratedRegister registers routers generated by IDL.
      func GeneratedRegister(r *server.Hertz){
      	//INSERT_POINT: DO NOT DELETE THIS LINE!
      }
```

#### render data

```json
{
  "*": {
    "GoModule": "github.com/hertz/hello",
    "ServiceName": "hello",
    "UseApacheThrift": true
  },
  "main.go": {
    "MainData": "this is customized render data"
  }
}
```

Command:

```shell
hz new --mod=github.com/hertz/hello --idl=./hertzDemo/hello.thrift --customize_layout=template/layout.yaml --customize_layout_data_path=template/data.json
```

## Package template

The package template is related to the definition of idl, including handler, model, router, etc.

Users can modify or rewrite according to the default template to meet their own needs.

### Default package template

```yaml
layouts:
  # path only indicates the template of handler.go, the specific handler path is determined by the default path and handler_dir
  - path: handler.go
    delims:
      - "{{"
      - "}}"
    body: |-
      // Code generated by hertz generator.

      package {{.PackageName}}

      import (
      	"context"

      	"github.com/cloudwego/hertz/pkg/app"
        "github.com/cloudwego/hertz/pkg/protocol/consts"

      {{- range $k, $v := .Imports}}
      	{{$k}} "{{$v.Package}}"
      {{- end}}
      )

      {{range $_, $MethodInfo := .Methods}}
      {{$MethodInfo.Comment}}
      func {{$MethodInfo.Name}}(ctx context.Context, c *app.RequestContext) {
      	var err error
      	{{if ne $MethodInfo.RequestTypeName "" -}}
      	var req {{$MethodInfo.RequestTypeName}}
      	err = c.BindAndValidate(&req)
      	if err != nil {
      		c.String(consts.StatusBadRequest, err.Error())
      		return
      	}
      	{{end}}
      	resp := new({{$MethodInfo.ReturnTypeName}})

      	c.{{.Serializer}}(consts.StatusBadRequest, resp)
      }
      {{end}}
  # path only indicates the router.go template, the specific path is determined by the default path and router_dir
  - path: router.go
    delims:
      - "{{"
      - "}}"
    body: |-
      // Code generated by hertz generator. DO NOT EDIT.

      package {{$.PackageName}}

      import (
      	"github.com/cloudwego/hertz/pkg/app/server"

      	{{range $k, $v := .HandlerPackages}}{{$k}} "{{$v}}"{{end}}
      )

      /*
       This file will register all the routes of the services in the master idl.
       And it will update automatically when you use the "update" command for the idl.
       So don't modify the contents of the file, or your code will be deleted when it is updated.
       */

      {{define "g"}}
      {{- if eq .Path "/"}}r
      {{- else}}{{.GroupName}}{{end}}
      {{- end}}

      {{define "G"}}
      {{- if ne .Handler ""}}
      	{{- .GroupName}}.{{.HttpMethod}}("{{.Path}}", append({{.HandlerMiddleware}}Mw(), {{.Handler}})...)
      {{- end}}
      {{- if ne (len .Children) 0}}
      {{.MiddleWare}} := {{template "g" .}}.Group("{{.Path}}", {{.GroupMiddleware}}Mw()...)
      {{- end}}
      {{- range $_, $router := .Children}}
      {{- if ne .Handler ""}}
      	{{template "G" $router}}
      {{- else}}
      	{	{{template "G" $router}}
      	}
      {{- end}}
      {{- end}}
      {{- end}}

      // Register register routes based on the IDL 'api.${HTTP Method}' annotation.
      func Register(r *server.Hertz) {
      {{template "G" .Router}}
      }
  # path only indicates the template of register.go, the specific path is determined by the default path and router_dir
  - path: register.go
    delims:
      - ""
      - ""
    body: |-
      // Code generated by hertz generator. DO NOT EDIT.

      package {{.PackageName}}

      import (
      	"github.com/cloudwego/hertz/pkg/app/server"
      	{{$.DepPkgAlias}} "{{$.Pkg}}"
      )

      // GeneratedRegister registers routers generated by IDL.
      func GeneratedRegister(r *server.Hertz){
      	//INSERT_POINT: DO NOT DELETE THIS LINE!
      	{{$.DepPkgAlias}}.Register(r)
      }
  - path: model.go
    delims:
      - ""
      - ""
    body: ""
  # path only indicates the template of middleware.go, the path to middleware is the same as router.go
  - path: middleware.go
    delims:
      - "{{"
      - "}}"
    body: |-
      // Code generated by hertz generator.

      package {{$.PackageName}}

      import (
      	"github.com/cloudwego/hertz/pkg/app"
      )

      {{define "M"}}
      {{- if ne .Children.Len 0}}
      func {{.GroupMiddleware}}Mw() []app.HandlerFunc {
      	// your code...
      	return nil
      }
      {{end}}
      {{- if ne .Handler ""}}
      func {{.HandlerMiddleware}}Mw() []app.HandlerFunc {
      	// your code...
      	return nil
      }
      {{end}}
      {{range $_, $router := $.Children}}{{template "M" $router}}{{end}}
      {{- end}}

      {{template "M" .Router}}
  # path only indicates the template of client.go, generated only when using hz new --client_dir or hz update --client_dir, path determined by out_dir and client_dir
  - path: client.go
    delims:
      - "{{"
      - "}}"
    body: |-
      // Code generated by hertz generator.

      package {{$.PackageName}}

      import (
        "github.com/cloudwego/hertz/pkg/app/client"
      	"github.com/cloudwego/hertz/pkg/common/config"
      )

      type {{.ServiceName}}Client struct {
      	client * client.Client
      }

      func New{{.ServiceName}}Client(opt ...config.ClientOption) (*{{.ServiceName}}Client, error) {
      	c, err := client.NewClient(opt...)
      	if err != nil {
      		return nil, err
      	}

      	return &{{.ServiceName}}Client{
      		client: c,
      	}, nil
      }
  # handler_single means a separate handler template, used to update each new handler when updating
  - path: handler_single.go
    delims:
      - "{{"
      - "}}"
    body: |+
      {{.Comment}}
      func {{.Name}}(ctx context.Context, c *app.RequestContext) {
      // this my demo
      	var err error
      	{{if ne .RequestTypeName "" -}}
      	var req {{.RequestTypeName}}
      	err = c.BindAndValidate(&req)
      	if err != nil {
      		c.String(consts.StatusBadRequest, err.Error())
      		return
      	}
      	{{end}}
      	resp := new({{.ReturnTypeName}})

      	c.{{.Serializer}}(consts.StatusOK, resp)
      }
  # middleware_single means a separate middleware template, which is used to update each new middleware_single when updating
  - path: middleware_single.go
    delims:
      - "{{"
      - "}}"
    body: |+
      func {{.MiddleWare}}Mw() []app.HandlerFunc {
      	// your code...
      	return nil
      }
  # hertz_client is generated by the hz client command. For detailed code, please refer to https://github.com/cloudwego/hertz/blob/develop/cmd/hz/generator/package_tpl.go#L271
  - path: hertz_client.go
    delims:
      - "{{"
      - "}}"
  # idl_client is generated by the hz client command. For detailed code, please refer to https://github.com/cloudwego/hertz/blob/develop/cmd/hz/generator/package_tpl.go#L862
  - path: idl_client.go
    delims:
      - "{{"
      - "}}"
```

### Template rendering parameters

> Note: Unlike the layout template, the custom package template does not provide rendering data functionality. This is mainly because these rendering data are parsed and generated by the hz tool, so the ability to write your own rendering data is temporarily not provided. You can modify the parts of the template that are not related to rendering data to meet your own needs.

Below is an introduction to the template rendering parameters provided by hz by default.

- File path rendering: The following rendering data can be used when specifying a file path

```go
type FilePathRenderInfo struct {
	MasterIDLName  string // master IDL name
	GenPackage     string // master IDL generate code package
	HandlerDir     string // handler generate dir
	ModelDir       string // model generate dir
	RouterDir      string // router generate dir
	ProjectDir     string // projectDir
	GoModule       string // go module
	ServiceName    string // service name, changed as services are traversed
	MethodName     string // method name, changed as methods are traversed
	HandlerGenPath string // "api.handler_path" value
}
```

- Single file rendering data: the rendering data used when defining a single file, all IDL information can be extracted according to the definition of "IDLPackageRenderInfo"

```go
type CustomizedFileForIDL struct {
	*IDLPackageRenderInfo
	FilePath    string
	FilePackage string
}
```

- Method level rendering data: when "loop_method" is specified, the rendering data used will be generated as a file for each method

```go
type CustomizedFileForMethod struct {
	*HttpMethod // The specific information parsed for each method definition
	FilePath    string // When the method file is generated in a loop, the path to the file
	FilePackage string // When the method file is generated in a loop, the go package name of the file
	ServiceInfo *Service // The information defined by the service to which the method belongs
}

type HttpMethod struct {
	Name            string
	HTTPMethod      string
	Comment         string
	RequestTypeName string
	ReturnTypeName  string
	Path            string
	Serializer      string
	OutputDir       string
	Models map[string]*model.Model
}
```

- Service level rendering data: when "loop_service" is specified, the rendering data will be used and a file will be generated for each service unit

```go
type CustomizedFileForService struct {
	*Service // specific information about the service, including the service name, information about the method defined in the servide, etc.
	FilePath       string // When the service file is generated in a loop, the path to the file
	FilePackage    string // When the service file is looped, the go package name of the file
	IDLPackageInfo *IDLPackageRenderInfo // Information about the IDL definition to which the service belongs
}

type Service struct {
	Name          string
	Methods       []*HttpMethod
	ClientMethods []*ClientMethod
	Models        []*model.Model // all dependency models
	BaseDomain    string         // base domain for client code
}
```

### Customize a package template

> Like layout templates, users can also customize package templates.
>
> As far as the templates provided by the package are concerned, the average user may only need to customize handler.go templates, because router.go/middleware.go/register.go are generally related to the idl definition and the user does not need to care, so hz currently also fixes the location of these templates, and generally does not need to be modified.
>
> Therefore, users can customize the generated handler template according to their own needs to speed up development; however, since the default handler template integrates some model information and package information, the hz tool is required to provide rendering data. This part of the user can modify it according to their own situation, and it is generally recommended to leave model information.

#### Overwrite default template

At present, hz itself comes with the following templates:

- handler.go
- router.go
- register.go
- middleware.go
- client.go
- handler_single.go
- middleware_single.go
- idl_client.go
- hertz_client.go

The above templates are the most basic templates for tool operation. When customizing templates:

- If a template with the same name is specified, the default content will be overwritten
- If no template with the same name is specified, the default template will be used

Therefore, when customizing templates, everyone needs to consider whether they need to be overwritten based on their actual situation.

> Note: When customizing a template, users only need to specify the file name and do not need to provide a relative path (such as handler.go) to overwrite the above template. However, when adding their own implementation, they need to provide a relative path based on out_dir.

#### Add a new template

Considering that sometimes you may need to add your own implementation for some information of IDL, such as adding a single test for each generated handler. Therefore, the hz templates allow users to customize new templates. The rendering parameters can refer to [Template rendering parameters](#template-rendering-parameters).

Template format:

```yaml
- path: biz/handler/{{$HandlerName}}.go // path+filename, support rendering data
  loop_method: bool // Whether to generate multiple files according to the method defined in idl, to be used with path rendering
  loop_service: bool // whether to generate multiple files according to the service defined in idl, to be used with path rendering
  update_behavior: // The update behavior of the file when using hz update
    type: string // update behavior: skip/cover/append
    append_key: string // Specify the appended rendering data source, method/service, in the append behavior
    insert_key: string // The "key" of the append logic in the append behavior, based on which to determine if appending is needed
    append_content_tpl: string // Specify the template for appending content in the append behavior
    import_tpl: []string // The template to be added to the import
  body: string // The template content of the generated file
```

#### Template Data Source

A simple example of a custom handler template is given below:

#### example

> example: https://github.com/cloudwego/hertz-examples/tree/main/hz/template

- Modify the content of the default handler

- Add a single test file for handler

```yaml
layouts:
  - path: handler.go
    body: |-
      {{$OutDirs := GetUniqueHandlerOutDir .Methods}}
      package {{.PackageName}}
      import (
       "context"

       "github.com/cloudwego/hertz/pkg/app"
        "github.com/cloudwego/hertz/pkg/protocol/consts"
      {{- range $k, $v := .Imports}}
       {{$k}} "{{$v.Package}}"
      {{- end}}
      {{- range $_, $OutDir := $OutDirs}}
        {{if eq $OutDir "" -}}
          "{{$.ProjPackage}}/biz/service"
        {{- else -}}
          "{{$.ProjPackage}}/biz/service/{{$OutDir}}"
        {{- end -}}
      {{- end}}
      "{{$.ProjPackage}}/biz/utils"
      )
      {{range $_, $MethodInfo := .Methods}}
      {{$MethodInfo.Comment}}
      func {{$MethodInfo.Name}}(ctx context.Context, c *app.RequestContext) {
       var err error
       {{if ne $MethodInfo.RequestTypeName "" -}}
       var req {{$MethodInfo.RequestTypeName}}
       err = c.BindAndValidate(&req)
       if err != nil {
          utils.SendErrResponse(ctx, c, consts.StatusOK, err)
          return
       }
       {{end}}
        {{if eq $MethodInfo.OutputDir "" -}}
          resp,err := service.New{{$MethodInfo.Name}}Service(ctx, c).Run(&req)
          if err != nil {
               utils.SendErrResponse(ctx, c, consts.StatusOK, err)
               return
          }
        {{else}}
          resp,err := {{$MethodInfo.OutputDir}}.New{{$MethodInfo.Name}}Service(ctx, c).Run(&req)
          if err != nil {
                  utils.SendErrResponse(ctx, c, consts.StatusOK, err)
                  return
          }
        {{end}}
       utils.SendSuccessResponse(ctx, c, consts.StatusOK, resp)
      }
      {{end}}
    update_behavior:
      import_tpl:
        - |-
          {{$OutDirs := GetUniqueHandlerOutDir .Methods}}
          {{- range $_, $OutDir := $OutDirs}}
            {{if eq $OutDir "" -}}
              "{{$.ProjPackage}}/biz/service"
            {{- else -}}
              "{{$.ProjPackage}}/biz/service/{{$OutDir}}"
            {{end}}
          {{- end}}

  - path: handler_single.go
    body: |+
      {{.Comment}}
      func {{.Name}}(ctx context.Context, c *app.RequestContext) {
       var err error
       {{if ne .RequestTypeName "" -}}
       var req {{.RequestTypeName}}
       err = c.BindAndValidate(&req)
       if err != nil {
          utils.SendErrResponse(ctx, c, consts.StatusOK, err)
          return
       }
       {{end}}
       {{if eq .OutputDir "" -}}
          resp,err := service.New{{.Name}}Service(ctx, c).Run(&req)
        {{else}}
          resp,err := {{.OutputDir}}.New{{.Name}}Service(ctx, c).Run(&req)
        {{end}}
        if err != nil {
              utils.SendErrResponse(ctx, c, consts.StatusOK, err)
              return
        }
       utils.SendSuccessResponse(ctx, c, consts.StatusOK, resp)
      }=
  - path: "{{.HandlerDir}}/{{.GenPackage}}/{{ToSnakeCase .ServiceName}}_test.go"
    loop_service: true
    update_behavior:
      type: "append"
      append_key: "method"
      insert_key: "Test{{$.Name}}"
      append_tpl: |-
        func Test{{.Name}}(t *testing.T) {
          h := server.Default()
          h.{{.HTTPMethod}}("{{.Path}}", {{.Name}})
          w := ut.PerformRequest(h.Engine, "{{.HTTPMethod}}", "{{.Path}}", &ut.Body{Body: bytes.NewBufferString(""), Len: 1},
          ut.Header{})
          resp := w.Result()
          assert.DeepEqual(t, 201, resp.StatusCode())
          assert.DeepEqual(t, "", string(resp.Body()))
          // todo edit your unit test.
        }
    body: |-
      package {{.FilePackage}}
      import (
        "bytes"
        "testing"

        "github.com/cloudwego/hertz/pkg/app/server"
        "github.com/cloudwego/hertz/pkg/common/test/assert"
        "github.com/cloudwego/hertz/pkg/common/ut"
      )
      {{range $_, $MethodInfo := $.Methods}}
        func Test{{$MethodInfo.Name}}(t *testing.T) {
        h := server.Default()
        h.{{$MethodInfo.HTTPMethod}}("{{$MethodInfo.Path}}", {{$MethodInfo.Name}})
        w := ut.PerformRequest(h.Engine, "{{$MethodInfo.HTTPMethod}}", "{{$MethodInfo.Path}}", &ut.Body{Body: bytes.NewBufferString(""), Len: 1},
        ut.Header{})
        resp := w.Result()
        assert.DeepEqual(t, 201, resp.StatusCode())
        assert.DeepEqual(t, "", string(resp.Body()))
        // todo edit your unit test.
        }
      {{end}}
```

## MVC Template Practice

Hertz provides a best practice for customizing templates for MVC, see [code](https://github.com/cloudwego/hertz-examples/tree/main/hz/template) for code details.

## Precautions

### Precautions for using package templates

Generally speaking, when users use package templates, most of them are to modify the default handler template; however, hz does not provide a single handler template at present, so when updating an existing handler file, the default handler_single template will be used to append a new handler function to the end of the handler file. When the corresponding handler file does not exist, a custom template will be used to generate the handler file.
