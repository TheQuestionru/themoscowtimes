TheQuestion Server API for The Moscow Times
===========================================

Table of Contents
-----------------

General:
- [Overview](#overview)
- [Types](#types)

Services:
- [Authorization](#authorization)
- [Answers](#answers)
- [Questions](#questions)
- [Accounts](#accounts)


Overview
--------

All requests are sent over HTTP. POST requests must be encoded as
`multipart/form-data` or `application/x-www-form-urlencoded`.
OK responses are sent as `application/json`. Error responses are sent as `text/plain`.
Data is UTF8-encoded.


**Headers**

Authorization header with an access token.
```
Authorization: token <token>
```


**Status codes**

- `200`, OK.
- `204`, No content, OK.
- `401`, Unauthorized, a valid access token required.
- `403`, Forbidden, a user does not have enough privileges to access/modify an object.
- `404`, Not found, an object is not found.
- `422`, Business exception with a human-readable message.
- `503`, Service unavailable, retry later.
- `500`, Internal server error.


**OK Response**
```
HTTP/1.1 200 OK
Server: nginx
Content-Type: application/json; charset=utf-8
Content-Length: N

{
    "bool0": false,
    "short0": 13481,
    "int0": 781856377,
    "long0": -2826246807606628400,
    "float0": 0.36959887,
    "double0": 0.3665312172141054,
    "string0": "Привет, мир",
    "enum0": "one",
    "date0": "2015-01-14T14:59:08Z",
    "list": ["a", "b", "c"],
    "map": {
        "a": 1,
        "b": 2,
        "c": 3
    }
}
```


**Error Response**
```
HTTP/1.1 422 Unprocessable Entity
Server: nginx
Content-Type: text/plain; charset=utf-8

Пользователь с таким логином уже существует.
```


Types
-----

- `int64`, a signed 64-bit integer.
- `string`, a unicode string, `string(255)` is a unicode string with a max len.
- `datetime`, a datetime encoded as a `yyyy-MM-dd'T'HH:mm:ss'Z'` string.


Authorization
-------------
**Types**
```yaml
TmtAccount:
    id:             int64
    firstName:      string
    lastName:       string
   
AccountToken:
    token:          string
    account:        TmtAccount
```
**Methods**
```yaml
signIn:
   # sign in via Facebook.
   path: GET /tmt/sign_in
   args:
        url string
   result:  redirect to  <uri>?code=:code
   
queryAccount:
    # returns AccountToken token by code
    path: POST /tmt/token
    args:
        code string
    result: AccountToken
```


Questions
---------
**Types**
```yaml
TmtQuestionForm:
    text: string
    topics: list<string>

TmtQuestion:
   id:              int64            
   createdAt:       datetime     
   author:          TmtAccount
   text:            string           
   topic:           string           
   answersIds:      list<TmtAnswer> # 25 last answers  
```
**Methods**
```yaml
questionCreate:
   # Creates a new question in "The Moscow Times" and additional topics
   path: POST /tmt/create_question/account/:account_id
   args:
       account_id: int64
   request: TmtQuestionForm
   result:  TmtQuestion

queryAll:
   # Query questions from "The Moscow Times Topic"
   path: GET /tmt/questions
   args:
       limit:      int32
       # These params are optional.
       offset:     int32           # Default is 0.
   result: list<TmtQuestion>
   
queryTopic:
   # Query questions from "The Moscow Times" and one more topic
   path: POST /tmt/questions/topic/:topic
   args:        
       # These params are optional.
       limit:      int32  
       offset:     int32           # Default is 0.
   request:
       topic:      string
   result: list<TmtQuestion>    

```

Answers
-------
**Types**
```yaml
TmtAnswerForm:
    text:           string

TmtAnswer:
    id:             int64
    questionId:     int64
    author:         TmtAccount
    createdAt:      datetime
    text:           string
```
**Methods**
```yaml
answerCreate:
   # Creates a new answer
   path: POST /tmt/create_answer/question/:questionId/account/:account_id
   args:
       question_id: int64
       account_id: int64
   request: TmtAnswerForm
   result:  TmtAnswer
   
queryAnswers:
    # Query answers for question
    path: GET /tmt/answers/question/:question_id
    args: 
        question_id: int64
        limit:       int32           
        # These params are optional.
        offset:      int32           # Default is 0.
    result: list<TmtAnswer>
```
