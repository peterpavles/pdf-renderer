# PDF-RENDERER

This app runs in conjunction with a headless chrome browser. It listens for requests on port 9766. Upon receiving a render request, it will go to a URL, render the page and respond with a zip file. The zip file will contain a json file and a zip file. The json file will be a list of network requests made by the page that was rendered. The pdf file will be a print-out of the page. 

## Features:

#### Network Request Polling
The service will wait until no requests are being made before printing the page.

#### QoS Reporting
In order to provide some insight into how well the page was rendered, the zip file generated by this service includes information about every network response and the status code.

#### Custom Header Passthrough
You can provide custom headers that will be used by headless chrome when making all network requests.

#### Request Correlation
If your request timed out, this will allow you to make a request again with the same correlation id and possibly get a stored version of the pdf instead of having to generate it again. The stored version is encrypted at rest and will be eligible for deletion by the service after some time.

## Configuration

You can configure the app via environment variables.

#### DEBUG
Default Value: (bool) `false`  
Description: when "true", outputs debug log messages and enables other debugging features (ie memory usage logging)

#### PDF_RENDERER_KEY
Default Value: (string) `JKNV29t8yYEy21TO0UzvDsX2KgiWrOVy`  
Description: defines the key used to encrypt the pdf files while at rest

#### PDF_RENDERER_STORAGE_DIRECTORY
Default Value: (string) `/tmp/pdf-renderer`  
Description: defines the directory in which the encrypted pdfs are temporarily stored

#### FILE_RETENTION_DURATION
Default Value: (string) `1h`  
Description: defines the minimum age at which a temporarily stored file becomes eligible for deletion

#### REQUEST_POLL_RETRIES
Default Value: (int) `10`  
Description: defines the number of times to poll the browser for new network requests/responses when no requests are pending before assuming the page is done rendering

#### REQUEST_POLL_INTERVAL
Default Value: (time.Duration) `1s`  
Description: defines the amount of time between each poll for new network requests/responses

#### PRINT_DEADLINE
Default Value: (time.Duration) `5m`  
Description: defines the maximum amount of time to spend on any given render request before simply printing whatever is there 

## Usage
```
cd $GOPATH
go get github.com/rapid7/pdf-renderer
cd src/github.com/rapid7/pdf-renderer
go build
./pdf-renderer
```

```
curl -X POST \
  http://localhost:9766/render \
  -H 'Content-Type: application/json' \
  -d '{
	"correlationId": "7667e2de-2f21-4ab7-9afb-402de2a6468f",
	"targetUrl": "https://example.com",
	"headers": {
		"Cookie": "cookiename: cookievalue"
	},
	"orientation": "Portrait",
	"printBackground": true,
	"marginTop": 0.4,
	"marginRight": 0.4,
	"marginBottom": 0.4,
	"marginLeft": 0.4
}'
```

## TODO
* add support for other headless browsers
* upload to s3
