# REST

<br>

## 정의
* 자원을 이름(표현)으로 구분하여 자원의 상태(정보)를 주고 받는 것
* 자원의 표현에 의한 상태 전달
* `HTTP URI`를 통해 자원을 명시하고 `HTTP Method(POST, GET, PUT, DELETE)`를 통해 해당 자원에 대한 `CRUD` 연산을 적용
  * Create: POST, Read: GET, Update: PUT, Delete: DELETE
* 웹 사이트의 이미지, 텍스트, DB 등의 모든 자원에 고유한 ID인 HTTP URI를 부여
* HTTP 프로토콜의 인프라를 그대로 사용하므로 REST API 사용을 위한 별도의 인프라를 구축할 필요가 없음

<br>

## 구성 요소
* 자원: URI
  * 모든 자원에는 고유의 `ID(HTTP URI)` 존재
* 행위: HTTP Method(POST, GET, PUT, DELETE)
* 표현: JSON, XML 등

<br>

## 특징
* 서버-클라이언트 구조: 서버가 자원을 소유, 클라이언트가 자원을 요청
* Stateless(무상태)
  * 클라이언트의 Context(쿠키, 세션)를 서버에 저장하지 않음
  * 서버는 각 요청을 별개로 처리 -> 요청들이 서로에 영향X
* Cacheable
* 계층화

<br>

## RESTful
* REST 아키텍처를 구현하는 웹 서비스
* REST 원리를 따르는 시스템
* 목적: 이해하기 쉽고 사용하기 쉬운 REST API 제작, 일관성을 통한 이해도 및 호환성 향상
* RESTful하지 않은 것: CRUD를 모두 POST로 처리, Route에 Resource, ID외의 정보가 사용되는 경우