# GO JWT 中间件

[![GoDoc](https://pkg.go.dev/badge/github.com/authok/go-jwt-middleware.svg)](https://pkg.go.dev/github.com/authok/go-jwt-middleware/v2)
[![License](https://img.shields.io/github/license/authok/go-jwt-middleware.svg?style=flat-square)](https://github.com/authok/go-jwt-middleware/blob/master/LICENSE)
[![Release](https://img.shields.io/github/v/release/authok/go-jwt-middleware?include_prereleases&style=flat-square)](https://github.com/authok/go-jwt-middleware/releases)
[![Codecov](https://img.shields.io/codecov/c/github/authok/go-jwt-middleware?style=flat-square&token=fs2WrOXe9H)](https://codecov.io/gh/authok/go-jwt-middleware)
[![Tests](https://img.shields.io/endpoint.svg?url=https%3A%2F%2Factions-badge.atrox.dev%2Fauth0%2Fgo-jwt-middleware%2Fbadge%3Fref%3Dmaster&style=flat-square)](https://github.com/authok/go-jwt-middleware/actions?query=branch%3Amaster)
[![Stars](https://img.shields.io/github/stars/authok/go-jwt-middleware.svg?style=flat-square)](https://github.com/authok/go-jwt-middleware/stargazers)
[![Contributors](https://img.shields.io/github/contributors/authok/go-jwt-middleware?style=flat-square)](https://github.com/authok/go-jwt-middleware/graphs/contributors)

---

Golang 中间件 用于检查和验证 [JWTs](jwt.io).

-------------------------------------

## 大纲

- [GO JWT 中间件](#go-jwt-中间件)
  - [大纲](#大纲)
  - [安装](#安装)
  - [使用](#使用)
  - [作者](#作者)
  - [许可](#许可)

-------------------------------------

## 安装

```shell
go get github.com/authok/go-jwt-middleware/v2
```

[[table of contents]](#table-of-contents)

## 使用

```go
package main

import (
	"context"
	"encoding/json"
	"log"
	"net/http"

	"github.com/authok/go-jwt-middleware/v2"
	"github.com/authok/go-jwt-middleware/v2/validator"
)

var handler = http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
	claims := r.Context().Value(jwtmiddleware.ContextKey{}).(*validator.ValidatedClaims)

	payload, err := json.Marshal(claims)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	w.Write(payload)
})

func main() {
	keyFunc := func(ctx context.Context) (interface{}, error) {
		// Our token must be signed using this data.
		return []byte("secret"), nil
	}

	// Set up the validator.
	jwtValidator, err := validator.New(
		keyFunc,
		validator.HS256,
		"https://<issuer-url>/",
		[]string{"<audience>"},
	)
	if err != nil {
		log.Fatalf("failed to set up the validator: %v", err)
	}

	// Set up the middleware.
	middleware := jwtmiddleware.New(jwtValidator.ValidateToken)

	http.ListenAndServe("0.0.0.0:3000", middleware.CheckJWT(handler))
}
```

After running that code (`go run main.go`) you can then curl the http server from another terminal:

```
$ curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJpc3MiOiJnby1qd3QtbWlkZGxld2FyZS1leGFtcGxlIiwiYXVkIjoiZ28tand0LW1pZGRsZXdhcmUtZXhhbXBsZSJ9.xcnkyPYu_b3qm2yeYuEgr5R5M5t4pN9s04U1ya53-KM" localhost:3000
```

That should give you the following response:

```
{
  "CustomClaims": null,
  "RegisteredClaims": {
    "iss": "go-jwt-middleware-example",
    "aud": "go-jwt-middleware-example",
    "sub": "1234567890",
    "iat": 1516239022
  }
}
```

The JWT included in the Authorization header above is signed with `secret`.

To test how the response would look like with an invalid token:

```
$ curl -v -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.yiDw9IDNCa1WXCoDfPR_g356vSsHBEerqh9IvnD49QE" localhost:3000
```

That should give you the following response:

```
...
< HTTP/1.1 401 Unauthorized
< Content-Type: application/json
{"message":"JWT is invalid."}
...
```

[[table of contents]](#table-of-contents)

[[table of contents]](#table-of-contents)

## 作者

[Authok](https://authok.cn/)

[[table of contents]](#table-of-contents)

## 许可

本项目基于 MIT 许可. 参考 [LICENSE](LICENSE).

[[table of contents]](#table-of-contents)
