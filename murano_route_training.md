### 0. Go through quickstart
- [Lightbulb quickstart](http://docs.exosite.com/quickstarts/lightbulb/)
- [Solution quickstart](http://docs.exosite.com/quickstarts/solutions/exampleapp/)
	- In this quickstart, you can see some lua scripts from the [Home Automation Template](https://github.com/exosite/home-automation-example)


### 1. Create a Solution in Staging
**Link: [https://www.exosite-staging.com/business/](https://www.exosite-staging.com/business/)**

- Sign up command:

`curl -X POST https://bizapi-staging.hosted.exosite.io/api:1/key/ -d '{"email":"<Replace with email>", "name": "<Replace with a name>", "source":"signup", "company": "<Replace with a company name>"}' -H 'Content-Type: application/json'`
`export STAGING_API_URL=https://bizapi-staging.hosted.exosite.io/api:1/key/`

* command:
 
```
http POST $STAGING_API_URL email=johnsonchuang@exosite.com name=johnsonchuang source=signup company=exosite
```

* result OK(200/205?):
	* receive from the 'email' address
	* use the link in the email to verify signup and set user's password
* result NOT ok
	* 301 - user(email) already signuped

### 1.1 Create an endpoint GET /hello/world

- Using Murano web UI to create solution and add an endpoint in the 'ROUTES' tab.
- Use this solution's domain url for the rest tasks. Export as **SOLUTION_URL**
- command: `http GET /hello/world -b`
	- success: response body = `Hello World`

### 1.2 Create an endpoint GET /request/{more}
And its response body is ({more} is a variable)
"{more}"

- command: `http GET $SOLUTION_URL/request/api1234 -b`
- success: response body = api1234
- end point's script: `return request.parameters.more`

### 1.3 Create an endpoint POST /request/
And it accepts resquest body ({moreValue} is a variable)
{
  "more", "{moreValue}"
}
And its response body is
"{moreValue}"

- end point's script:

```
local more = request.body.more
return more
```
- command: `http POST $SOLUTION_URL/request/ more=moreSth -b`
- success: response body = `moreSth`

### 2.1 Create an endpoint POST /keystore/set
and It accepts request body
({stateValue} is a variable)
{
	"key": "state",
	"value": "{stateValue}"
}
and its response body is
{
	"result": "success"
}
and it should use the real **keystore**

- end point's script:

```
local body = request.body
if body['key'] == "state" then
  local val = body['value']
  if val == nil or val == '' then
    return "request 'value' not given"
  else
    return "{\"result\":\"success\"}"
  end
else
  return "reuqest 'key' not given"
end
```
- command: `http POST $SOLUTION_URL/keystore/set key=state value=off`
	- response body: `{"result":"success"}`
- command: `http POST $SOLUTION_URL/keystore/set k=state value=off`
	- response body: `request 'key' not given`
- command: `http POST $SOLUTION_URL/keystore/set value=off key=`
	- response body: `request 'key' not given`
- command: `http POST $SOLUTION_URL/keystore/set key=state v=off`
	- response body: `request 'value' not given`
- command: `http POST $SOLUTION_URL/keystore/set key=state value=`
	- response body: `request 'value' not given`

### 2.2 Create an endpoint GET /keystore/get/{key}
and It accepts a request parameter "key"
and its response header is
{
	"MUR-XXX": "MUR-URL"
}
and response body is
{
	"state": "{stateValue}"
}

- end point's scripts:

```
kii = request.parameters.key
if kii ~= nil then
  local val = Keystore.get({key=kii}).value
  response.headers["MUR-XXX"]="MUR-URL"
  return "{\"" .. kii .. "\":\"" .. val .. "\"}"
end
```
- First, set corresponding key's value in the Keystore.
- command: `http $SOLUTION_URL/keystore/get/state`
	- response header: `"MUR-XXX": "MUR-URL"`
	- response body: `{"state": "true"}` (如果先前是設定為"true"的話)
- command: `http $SOLUTION_URL/keystore/get/notSetKey`
	- response: `key : 'notSetKey' is not set`
	- status: 404

### 2.3 Create an endpoint DELETE /keystore/delete
and it accepts request body
{
	"key": "state"
}
and its response body is
{
	"result": "success"
}

- end point's scripts:

```
local target = request.body
local kii = target['key']

if kii ~= nil and kii ~= '' then
  Keystore.delete({key=kii})
  return "{\"result\":\"success\"}"
else
  response.code = 400
  return "{\"result\":\"no key\"}"
end
```
- First, set the key/value in Keystore for later deletion.
	- command: `http DELETE $SOLUTION_URL/keystore/delete key=state`
		- response: {"result":"success"}
	- command: `http DELETE $SOLUTION_URL/keystore/delete key=`
		- response: {"result":"success"}
		- status: 400

### Demo(all links not found :D)
1.1 https://jimmy-endpoint.apps.exosite-staging.io/docs/#!/default/gethelloworld
1.2 https://jimmy-endpoint.apps.exosite-staging.io/docs/#!/default/getrequestmore
1.3 https://jimmy-endpoint.apps.exosite-staging.io/docs/#!/default/postrequest
2.1 https://jimmy-endpoint.apps.exosite-staging.io/docs/#!/default/getkeystoregetkey
2.2 https://jimmy-endpoint.apps.exosite-staging.io/docs/#!/default/postkeystoreset
2.3 https://jimmy-endpoint.apps.exosite-staging.io/docs/#!/default/deletekeystoredelete

### Hint
- [Webservice Request](http://docs.exosite.com/reference/services/webservice/#request)
- [Webservice apiReply](http://docs.exosite.com/reference/services/webservice/#apireply)
- [Keystore Service](http://docs.exosite.com/reference/services/keystore/)


### Questions
- In end point scripts editor, how to response json exactly? (Lua)