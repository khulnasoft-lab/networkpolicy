# networkpolicy

[![License](https://img.shields.io/github/license/khulnasoft-lab/networkpolicy)](LICENSE.md)
![Go version](https://img.shields.io/github/go-mod/go-version/khulnasoft-lab/networkpolicy?filename=go.mod)
[![Release](https://img.shields.io/github/release/khulnasoft-lab/networkpolicy)](https://github.com/khulnasoft-lab/networkpolicy/releases/)
[![Checks](https://github.com/khulnasoft-lab/networkpolicy/actions/workflows/build-test.yml/badge.svg)](https://github.com/khulnasoft-lab/networkpolicy/actions/workflows/build-test.yml)
[![GoDoc](https://pkg.go.dev/badge/khulnasoft-lab/networkpolicy)](https://pkg.go.dev/github.com/khulnasoft-lab/networkpolicy)



The package acts as an embeddable configurable container handling allow/deny verdicts over a series of conditions including
- IPs
- CIDRs
- Ports
- Schemes (eg `https, http, ftp`)

## General usage as allow/deny
The following program prevents the http client to follow targets belonging to the deny list:

Example - General allow/deny list
```go
package main

import (
	"errors"
	"log"
	"net/http"

	"github.com/khulnasoft-lab/networkpolicy"
)

func main() {
	var npOptions networkpolicy.Options
	// deny connections to localhost
	npOptions.DenyList = append(npOptions.DenyList, "127.0.0.0/8")

	np, err := networkpolicy.New(npOptions)
	if err != nil {
		log.Fatal(err)
	}

	customRedirectHandler := func(req *http.Request, via []*http.Request) error {
		// if at least one address is valid we follow the redirect
		if _, ok := np.ValidateHost(req.Host); ok {
			return nil
		}
		return errors.New("redirected to a forbidden target")
	}

	client := &http.Client{
		CheckRedirect: customRedirectHandler,
	}
	req, err := http.NewRequest(http.MethodGet, "http://yourtarget", nil)
	if err != nil {
		log.Fatal(err)
	}
	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	log.Println(resp)
}
```