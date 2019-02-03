# gosip - SharePoint authentication client for Golang

## Main features

`gosip` allows you to perform SharePoint unattended (without user interaction) http authentication with Go (Golang) using different authentication strategies.

Supported SharePoint versions:

- SharePoint Online (SPO)
- On-Prem: 2019, 2016, and 2013

Authentication strategies:

- SharePoint 2013, 2016, 2019:
  - ADFS user credentials
  - Form-based authentication (FBA)
  - Forefront TMG authentication
- SharePoint Online:
  - Addin only permissions
  - SAML based with user credentials
  - ADFS user credentials

## Installation

```bash
go get github.com/koltyakov/gosip
```

## Usage samples

### Addin Only Permissions

```golang
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"

	"github.com/koltyakov/gosip/auth/addin"
)

func main() {
	auth := &addin.AuthCnfg{
		SiteURL:      "https://contoso.sharepoint.com/sites/my_site",
		ClientID:     "[CLIENT_ID]",
		ClientSecret: "[CLIENT_SECRET]",
	}

	authToken, err := auth.GetAuth()
	if err != nil {
		fmt.Printf("unable to authenticate: %v", err)
		return
	}

	apiEndpoint := auth.SiteURL + "/_api/web?$select=Title"
	req, err := http.NewRequest("GET", apiEndpoint, nil)
	if err != nil {
		fmt.Printf("unable to create a request: %v", err)
		return
	}

	req.Header.Set("Accept", "application/json;odata=minimalmetadata")
	req.Header.Set("Authorization", "Bearer "+authToken)

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		fmt.Printf("unable to request api: %v", err)
		return
	}
	defer resp.Body.Close()

	data, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("unable to read a response: %v", err)
		return
	}

	type apiResponse struct {
		Title string `json:"Title"`
	}

	results := &apiResponse{}

	err = json.Unmarshal(data, &results)
	if err != nil {
		fmt.Printf("unable to parse a response: %v", err)
		return
	}

	fmt.Println("=== Response from API ===")
	fmt.Printf("Web title: %s\n", results.Title)
}
```

## Tests

### Run automated tests

```bash
go test ./...
```

### Run manual test

Modify `cmd/gosip/main.go` to include required scenarios and run:

```bash
go run cmd/gosip/main.go
```