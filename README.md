# rigo

(Request interceptor in GO) In escense `rigo` is a reverse *Proxy* cli with the ability to intercept and modify requests and responses defined on a configuration file.

## Use cases

Rigo becomes really useful when testing apps or services that are interconnected or dependent on external services. For example:

- Mock an entire backend with custom responses. This would allow testing services with limited connectivity or ahead of development.
- Intercept specific responses that change the behavior of the system under test. This allows to modify and test the behavior of the system under test withouht changing the behavior of it's dependencies.

## Install rigo

```bash
# work in progress
```

## Running rigo

Running rigo is as simple as execute the cli app passing the flags:

```bash
./rigo -p 8400 -f rigo.yaml
```

### Flags

| Flag | Name  | Description | Default |
|------|-------|-------------| ------------|
| p    | port  | Port number to run the proxy on | 8400
| f    | file  | Configuration file to use | rigo.yml
| v    | verbose  | To make the logs more verbose (useful when dealing with CORS) | false
| record    | record  | Record request/response instead of patching | false

### Configuration file

This is a configuration file that can be used as an example:

```yaml
# url that requests will be proxied to
target_url: https://api.somerealapi.com/
# in case authentication need to be appended to the requests
# this is a real use case when requests are sent on the cookie headers
authentication:
    bearer:
        type: Bearer
        token: "sometoken"
intercept:
# network can be modified after returning from the service
# useful in case of patching, or mixing the response with the status code
  responses:
# match is how we will match the intercept with the outgoing or incoming requests
# matching:
#    should contain a uri, 
#    but also optional the methods and query params
    - match:
        uri: "*/libraries/*/books"
        methods: 
          - GET
        params: 
          - name: search
            value: book
# patch is how we will modify the intercepted call
# patching:
#    should contain the type of patch, the type could be file, string or json
#    but also option the code for the response. (default 200 if not specified)   
      patch:
        type: string
        body: 'this is the patched body'
        code: 500
# network can also be intercepted and returned withouht reaching the service
# useful when we know in advance the response type that is expected
  requests:
    - match:
        uri: "*/libraries/*/books"
        methods: 
          - GET
      patch:
        type: json
        body: |
          {
            'books': [
              {
                "createdAt": "2022-12-30T18:00:51Z"
            }]
          }
    - match:
      uri: "*/customer"
      methods: 
        - GET
        - POST
      patch:
        code: 500
    - match:
      uri: "*some-otherrequest*"
      methods: 
        - GET
        - POST
      patch:
        body: ./file.json
        type: file
```

## Recording

Note: *Only json requests are supported for now*

Rigo allows also for recording matching network requests. This is very useful when the body that we need to patch later is too big or complex, or we just want to have a copy for testing.

To enter record mode just use the flag `record`:

```bash
rigo --record
```

Rigo will create a "record file" for all the matching requests. If the intercept includes `type: file` for the patch then the same name will be used for the recorded filename, otherwise the name will be infered from the request `method` and the `url`. For example: *GET_https:_api.someurl.com_this_v1_books.json*.
