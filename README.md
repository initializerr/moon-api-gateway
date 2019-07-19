# Moon-API-gateway

[![Build Status](https://travis-ci.org/longcoding/moon-api-gateway.svg?branch=master&maxAge=2592000)](https://travis-ci.org/longcoding/moon-api-gateway.svg?branch=master)
[![codecov](https://codecov.io/gh/longcoding/moon-api-gateway/branch/master/graph/badge.svg?maxAge=2592000)](https://codecov.io/gh/longcoding/moon-api-gateway/branch/master/graph/badge.svg)
[![Release](https://img.shields.io/github/release/longcoding/moon-api-gateway.svg?maxAge=2592000)](https://img.shields.io/github/release/longcoding/moon-api-gateway.svg)
[![HitCount](http://hits.dwyl.io/longcoding@gmail.com/longcoding/moon-api-gateway.svg)](http://hits.dwyl.io/longcoding@gmail.com/longcoding/moon-api-gateway.svg)
[![LastCommit](https://img.shields.io/github/last-commit/longcoding/moon-api-gateway.svg)](https://img.shields.io/github/last-commit/longcoding/moon-api-gateway.svg)
[![TotalCommit](https://img.shields.io/github/commit-activity/y/longcoding/moon-api-gateway.svg)](https://img.shields.io/github/commit-activity/y/longcoding/moon-api-gateway.svg)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?maxAge=2592000)]()

### Other Language Guide
|Korean|[README_ko](README_ko.md)|
|:---:|---|

![feature](https://user-images.githubusercontent.com/3271895/51427833-6d132780-1c3f-11e9-8f73-7112a7f0da0c.png)

## Introduction
Asynchronous API Gateway with Spring boot 2.1, Servlet 4, jetty 9 client <br />
The Gateway is a network gateway created to provide a single access point for real-time web based protocol elevation that supports load balancing, clustering, and several validations. It is designed to make the best performance to deliver open API.

* Lightweight API Gateway
* High Performance/Scalability

## New Feature(June-2019)
* Management API
    - Service, Service Contract API
* Supported Server Cluster
    - Improved cluster synchronization

## Features
Moon API Gateway offers a powerful, yet lightweight feature.

* **Request Validation** - You can use various verification features in your request. It's also easy to add or remove new features.
    - Header, Query, Path Param
* **Rate Limiting** - Provides strong rate limiting per API users. Redis-based Cluster servers share rate limiting information on a per-key basis.
    - App Daily Rate Limiting
    - App Minutely  Rate Limiting
* **Service Capacity** - Manages the capacity of the service linked to the API gateway to ensure stable operation.
    - Service Daily Capacity
    - Service Minute Capacity
* **Service Contract(agreement)** - (Optional) API, App Users can only call APIs that have agreed to the contract relationship or terms.
* **Request Transform** - (Optional) It supports the change of Header, Query, Path Param as well as URI. This will change the user's request appropriately to the request of the service associated with the moon api gateway.
* **IP Whitelisting** - Block access to non-trusted IP addresses for more secure interactions on a per-key basis
* **Management API** - Provides powerful Rest API to manage API Gateway.
    - API Add/Delete/Change
    - APP Add/Delete/Change
    - IP Whitelist Add/Delete
    - Key Expiry/Regenerate
* **Supported Server Cluster** - API Gateway Cluster can be configured. With the Management API, the changes take effect on all servers. Rate Limiting, and Service Capacity information are all shared.

## Dependency
* Spring Boot 2.1
* Servlet 4
* Ehcache 3
* Jetty 9 client
* Jedis 3.0

## Configuration
There are required settings to run moon-api-gateway.
You do not need to use initialization with the management API.

### Step 1
```
- Please set the global application first in application.yaml

moon:
  service:       
    ip-acl-enable: false
    cluster:
      enable: false
      sync-interval: 300000       
    proxy-timeout: 20000

jedis-client:      
  host: '127.0.0.1'
  port: 6379
  timeout: 1000
  database: 0
```    
- ip-acl-enable: Enable the ip whitelisting feature. It operates on APP basis.
- cluster/enable: If you enable a server cluster, the daemon thread continues to fetch services, apps, and apis information.
- cluster/sync-interval: This option allows you to set the interval for the cluster synchronization operation.
- proxy-timeout: This option allows you to set the timeout for the Rotating service.
- **jedis-client**: Redis is an essential infrastructure for moon-api-gateway.
- jedis-client/host: Host information for Redis.
- jedis-client/port: Port information for Redis.

### Step 2
```
- Please set the initial application registration in application-apps.yaml
- (These settings are optional)

init-apps:
  init-enable: true
  apps:
    -
      app-id: 0
      app-name: TestApp
      api-key: 1000-1000-1000-1000
      app-minutely-ratelimit: 2000
      app-daily-ratelimit: 10000
      app-service-contract: [1, 2, 3]
      app-ip-acl: ['192.168.0.1', '127.0.0.1']
    -
      app-id: 1
      app-name: BestApp
      api-key: e3938427-1e27-3a37-a854-0ac5a40d84a8
      app-minutely-ratelimit: 1000
      app-daily-ratelimit: 50000
      app-service-contract: [1, 2]
      app-ip-acl: ['127.0.0.1']
```
- init-enable: The initial registration setting is not used.
- app-service-contract: A list of API services that app has permission to use.
- app-ip-acl: The whitelist of ip that can use this API Key.
- app minutely/daily ratelimit: The amount of APIs available to the app.

### Step 3
```
- Set up service and API specification configurations in application-apis.yml
- The API Gateway obtains Service and API information through the APIExposeSpecLoader.

api-spec:
  init-enable: true
  services:
    -
      service-id: 1
      service-name: stackoverflow
      service-minutely-capacity: 10000
      service-daily-capacity: 240000
      service-path: /stackoverflow
      outbound-service-host: api.stackexchange.com
      apis:
        -
          api-id: 101
          api-name: getInfo
          protocol: http, https
          method: get
          inbound-url: /2.2/question/:first
          outbound-url: /2.2/questions
          header: page, votes
          header-required: ""
          query-param: version, site
          query-param-required: site
        -
          api-id: 202
          api-name: getQuestions
          protocol: https
          method: put
          inbound-url: /2.2/question/:first
          outbound-url: /2.2/questions
          header: page, votes
          header-required: ""
          query-param: version, site
          query-param-required: site
    -
      service-id: 2
      service-name: stackoverflow2
      service-minutely-capacity: 5000
      service-daily-capacity: 100000
      service-path: /another
      outbound-service-host: api.stackexchange.com
      apis:
        -
          api-id: 201
          api-name: transformTest
          protocol: http, https
          method: get
          inbound-url: /2.2/haha/question/:site
          outbound-url: /:page/:site
          header: page, votes
          header-required: ""
          query-param: version, site
          query-param-required: site
          transform:
            page: [header, param_path]
            site: [param_path, header]
    -
      service-id: '03'
      service-name: service3
      service-minutely-capacity: 5000
      service-daily-capacity: 100000
      service-path: /service3
      outbound-service-host: api.stackexchange.com
      only-pass-request-without-transform: true
```
- init-enable: The initial registration setting is not used.
- service-path: URL The first parameter in the Path. The API is routed to the service registered in that parameter.
- service minutely/daily capacity: The total amount of APIs that the service can route.
- outbound-service-host: The APIs of the service are routed to that domain.
- apis/inbound-url: Externally exposed API URL Path specification. ':?' Is a variable.
- apis/outbound-url: The actual url path of the service connected to the api-gateway.
- apis/header: This is the header that can be received when requesting API.
- apis/header-required: This header is mandatory for API requests.
- apis/query-param: This is the url query parameter that can be received when requesting API.
- apis/query-param-required: This url query parameter is mandatory for API requests.
- transform: The param that is received at the time of request is transformed into another variable area at the time of routing.
    - Possible options: **header**, **param_path**, **param_query**, **body_json**
    - usage: [source, destination] like [header, param_path]
    - To use the body_json type, the content-type must be the application/json.
- only-pass-request-without-transform: All APIs are routed to services connected to the gateway without any analysis or transformation.


Moon-api-gateway supports the following protocol and method.

* Protocol
    - http, https
* Method
    - Get, Post, Put, Delete

## API Gateway Cluster
Moon API Gateway supports clusters. Each node synchronizes API, APP, IP Whitelist and App Key (= API Key) information in near real time.
Cluster nodes also work together to calculate the correct API Ratelimiting, Service Capacity.

- **Service, API, APP, IP Whitelist Interval Sync**

![feature](https://user-images.githubusercontent.com/3271895/51427837-7b614380-1c3f-11e9-82f3-5a668ba63f00.png)


## Management REST API
The Management API helps manage a single gateway or cluster group.

**APP Management**
  - **APP Register** - [POST] /internal/apps
  - **APP UnRegister** - [DELETE] /internal/apps/{appId}
  - **APP Update** - [Future Feature]
  - **API Key Expiry** - [DELETE] /internal/apps/{appId}/apikey
  - **API Key Regenerate** - [PUT] /internal/apps/{appId}
  - **Add NEW IP Whitelist** - [POST] /internal/apps/{appId}/whitelist
  - **Remove IP Whitelist** - [DELETE] /internal/apps/{appId}/whitelist

**API Management**
  - **New API Register** - [POST] /internal/apis
  - **API UnRegister** - [DELETE] /internal/apis/{apiId}
  - **API Update** - [PUT] /internal/apis/{apiId}

**Service Group API Will be updated**



## Usage/Test

##### API - stackoverflow API.

Service And API Expose Specification for stackoverflow


    service-id: '01'
          service-name: stackoverflow
          service-minutely-capacity: 10000
          service-daily-capacity: 240000
          service-path: /stackoverflow
          outbound-service-host: api.stackexchange.com
          apis:
            -
              api-id: '0101'
              api-name: getInfo
              protocol: http, https
              method: get
              inbound-url: /2.2/question/:first
              outbound-url: /2.2/questions
              header: page, votes
              header-required: ""
              query-param: version, site
              query-param-required: site


- 1) The api service path we want to call is '/stackoverflow'
- 2) The inbound url path following the service path is '/2.2/question/:first'
- 3) ': first' is the path parameter and we declare it 'test'.
- 4) The calling protocol supports http and https, and we will call http.
- 5) Set the header and url query parameters.
- 6) When you call the API, the gateway will route the call to api.stackexchange.com set to outbound-service-host.
- 7) When calling the API, the domain is api.stackexchange.com and the destination url path is '/2.2/questions' set to outbound-url.

##### 1) Use Test Case - Run moon-api-gateway by gradle

    ./gradlew test

##### 2) Use rest-client like Postman.
To set method and scheme.

    GET, http

Input URL.

    http://localhost:8080/stackoverflow/2.2/question/test

Input URL parameter. ( site is mandatory query parameter )

    site = stackoverflow

OR you can input URL like below.

    http://localhost:8080/stackoverflow/2.2/question/test?site=stackoverflow

and then input header fields. ( apikey is mandatory header.(or Query Parameter) )

    apikey, 1000-1000-1000-1000
    page, 5
    votes, 1

Execute request and check response code and content.

##### 3) Use cUrl.

    curl -X GET -H "Content-type: application/json" -H "apikey: 1000-1000-1000-1000" -H "page: 5" -H "votes: 1" http://localhost:8080/stackoverflow/2.2/question/test?site=stackoverflow

## Future update
* Authentication for Private API
* Docker-Compose
    - Easy To Run

## Contact
For any inquiries, you can reach me at longcoding@gmail.com

## License
moon-api-gateway is released under the MIT license. See LICENSE for details.
