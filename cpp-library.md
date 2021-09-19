## https://www.boost.org/doc/libs/1_62_0/doc/html/boost_asio.html
boost가 쇠퇴기이긴 하지만 asio 는 Networking TS로 표준이 예정되어 있다.
c++23에서나 상정될것 같음.
experimetal로 이미 구현된 기능들도 일부 있는데 asio랑 거의 같다. (facebook c++ Korea포럼에서 발췌)
예시를 보면 알겠지만 http 리퀘스트 프로토콜을 직접 입력해줘야 하는 수고를 해야한다.
구현해야하는 라인도 많고 지금의 통검 http client 방식과 큰차이가 없어서 도입하기 꺼려짐

Asio 1.10.8 / Boost 1.62 OpenSSL 1.1.0 대응 [Asio history](https://www.boost.org/doc/libs/master/doc/html/boost_asio/history.html)

- [cpp03/http/client/sync_client.cpp](https://www.boost.org/doc/libs/1_65_1/doc/html/boost_asio/example/cpp03/http/client/sync_client.cpp)
- [cpp03/http/client/async_client.cpp](https://www.boost.org/doc/libs/1_65_1/doc/html/boost_asio/example/cpp03/http/client/async_client.cpp)
- [cpp03/ssl/client.cpp](https://www.boost.org/doc/libs/1_65_1/doc/html/boost_asio/example/cpp03/ssl/client.cpp)

## https://github.com/yhirose/cpp-httplib
- A C++11 single-file header-only cross platform HTTP/HTTPS library.
- A multi-threaded 'blocking' HTTP library.
- 설치 간편함 (헤더 하나만 추가)
- 코드 작성 시 적은 라인으로 구현가능하여 편할 듯
- 통검은 http client 를 nonblocking 방식으로 호출해야하므로 적절하지 않은 듯 함..
- async 문제를 해결한 fork 가 나왔으나 다른 블록체인 프로젝트 내부에 헤더파일만 들어가 있다. 라이센스는 GPLv3 라서 사용하기 어려울듯
  - https://github.com/skalenetwork/skaled/blob/develop/libskutils/include/skutils/http.h

client 예제
```c++
#define CPPHTTPLIB_OPENSSL_SUPPORT
#include "path/to/httplib.h"

// HTTP
httplib::Client cli("http://cpp-httplib-server.yhirose.repl.co");

// HTTPS
httplib::Client cli("https://cpp-httplib-server.yhirose.repl.co");

auto res = cli.Get("/hi");
res->status;
res->body;
```

ssl 예제 (1.1.1. 만 지원함)
```c++
// CPPHTTPLIB_OPENSSL_SUPPORT. libssl and libcrypto should be linked.
#define CPPHTTPLIB_OPENSSL_SUPPORT
#include "path/to/httplib.h"

// Server
httplib::SSLServer svr("./cert.pem", "./key.pem");

// Client
httplib::Client cli("https://localhost:1234"); // scheme + host
httplib::SSLClient cli("localhost:1234"); // host

// Use your CA bundle
cli.set_ca_cert_path("./ca-bundle.crt");

// Disable cert verification
cli.enable_server_certificate_verification(false);
```

## http://www.curlpp.org/
A C++ wrapper for libcURL
그닥 끌리지가 않는다..

- https://github.com/jpbarrette/curlpp/tree/master/examples

## https://github.com/facebook/folly
- C++14
- 설치가 무지 까다롭고 오래걸린다 (http/3 테스트 시 설치경험해보았음)

## https://github.com/facebook/proxygen
This project comprises the core C++ HTTP abstractions used at Facebook. Internally, it is used as the basis for building many HTTP servers, proxies, and clients.
예도 folly처럼 설치 까다로웠다.
HTTP/1.1, SPDY/3, SPDY/3.1, HTTP/2, and HTTP/3를 다양하게 지원함
http client 예시
- https://github.com/facebook/proxygen/tree/main/proxygen/httpclient/samples/curl

## https://pocoproject.org/

The POCO C++ Libraries are powerful cross-platform C++ libraries for building network- and internet-based applications that run on desktop, server, mobile, IoT, and embedded systems.
boost 에 비교될 정도의 c++라이브러리. POCO Pro 상용 라이브러리도 제공한다.
- https://github.com/pocoproject/poco

통검 서버의 yum 으로 받으면 1.6.1 버전이 설치된다. /usr/include/Poco
- 설치 용이성 (yum)
- 코드가 간편한 편

cmake 에 포함하려면
- https://stackoverflow.com/questions/30114662/clion-cmake-and-poco

http client 예제
- https://github.com/pocoproject/poco/blob/master/Net/samples/httpget/src/httpget.cpp
```c++
//
// httpget.cpp
//
// This sample demonstrates the HTTPClientSession and the HTTPCredentials classes.
//
// Copyright (c) 2005-2012, Applied Informatics Software Engineering GmbH.
// and Contributors.
//
// SPDX-License-Identifier:	BSL-1.0
//


#include "Poco/Net/HTTPClientSession.h"
#include "Poco/Net/HTTPRequest.h"
#include "Poco/Net/HTTPResponse.h"
#include "Poco/Net/HTTPCredentials.h"
#include "Poco/StreamCopier.h"
#include "Poco/NullStream.h"
#include "Poco/Path.h"
#include "Poco/URI.h"
#include "Poco/Exception.h"
#include <iostream>


using Poco::Net::HTTPClientSession;
using Poco::Net::HTTPRequest;
using Poco::Net::HTTPResponse;
using Poco::Net::HTTPMessage;
using Poco::StreamCopier;
using Poco::Path;
using Poco::URI;
using Poco::Exception;


bool doRequest(Poco::Net::HTTPClientSession& session, Poco::Net::HTTPRequest& request, Poco::Net::HTTPResponse& response)
{
	session.sendRequest(request);
	std::istream& rs = session.receiveResponse(response);
	std::cout << response.getStatus() << " " << response.getReason() << std::endl;
	if (response.getStatus() != Poco::Net::HTTPResponse::HTTP_UNAUTHORIZED)
	{
		StreamCopier::copyStream(rs, std::cout);
		return true;
	}
	else
	{
		Poco::NullOutputStream null;
		StreamCopier::copyStream(rs, null);
		return false;
	}
}


int main(int argc, char** argv)
{
	if (argc != 2)
	{
		Path p(argv[0]);
		std::cout << "usage: " << p.getBaseName() << " <uri>" << std::endl;
		std::cout << "       fetches the resource identified by <uri> and print it to the standard output" << std::endl;
		return 1;
	}

	try
	{
		URI uri(argv[1]);
		std::string path(uri.getPathAndQuery());
		if (path.empty()) path = "/";

		std::string username;
		std::string password;
		Poco::Net::HTTPCredentials::extractCredentials(uri, username, password);
		Poco::Net::HTTPCredentials credentials(username, password);

		HTTPClientSession session(uri.getHost(), uri.getPort());
		HTTPRequest request(HTTPRequest::HTTP_GET, path, HTTPMessage::HTTP_1_1);
		HTTPResponse response;
		if (!doRequest(session, request, response))
		{
			credentials.authenticate(request, response);
			if (!doRequest(session, request, response))
			{
				std::cerr << "Invalid username or password" << std::endl;
				return 1;
			}
		}
	}
	catch (Exception& exc)
	{
		std::cerr << exc.displayText() << std::endl;
		return 1;
	}
	return 0;
}
```

다른 http client 예제
```c++
Poco::JSON::Object obj;
obj.set("name", "blah");
obj.set("language", "english");

Poco::URI uri("http://the-uri-you-want-to-request-from");
std::string path(uri.getPathAndQuery());
if (path.empty()) path = "/";

HTTPClientSession session(uri.getHost(), uri.getPort());
HTTPRequest request(HTTPRequest::HTTP_POST, path, HTTPMessage::HTTP_1_1);
HTTPResponse response;

request.setContentType("application/json");
std::stringstream ss;
obj.stringify(ss);
request.setContentLength(ss.str().size());
std::ostream& o = session.sendRequest(request);

obj.stringify(o);

// this line is where you get your response
std::istream& s = session.receiveResponse(response);

Poco::JSON::Parser parser;
Poco::JSON::Object::Ptr ret = parser.parse(s).extract<Poco::JSON::Object::Ptr>();

// (*ret) will contain all the members in a json structure returned
if ((*ret).get("success") != true) {
    std::cout << "Failed to upload: " << (*ret).get("message").toString();
    return;
}
```

## https://github.com/Microsoft/cpprestsdk
- pretty intuative interface (like really good) .. no callback hell anymore
- they fix stuff pretty fast
- lot of overhead again

빌드시 boost 라이브러리 의존성이 있다. 1.62 로 빌드 시도했으나 다수의 에러를 발생하며 실패. 해결하는 방법 구글링해봤는데 아직 못찾음

```console
git clone https://github.com/Microsoft/cpprestsdk.git
cd cpprestsdk
git checkout 2.10.18
mkdir build.debug
cd build.debug
cmake3 .. -DCMAKE_BUILD_TYPE=Release -DBOOST_ROOT=/daum/lib/tot_search_libsrc
make # 실패
make CFLAGS=-Wno-error #실패
```

## https://github.com/cesanta/mongoose
Mongoose - Embedded Web Server / Embedded Networking Library
It implements event-driven non-blocking APIs for TCP, UDP, HTTP, WebSocket, MQTT
GPLv2 또는 커머셜라이센스로 사용 가능

사용법
Step 1. Copy mongoose.c and mongoose.h into the source code tree
뭔가 좀 만들다 만느낌이어서 설치하려다가 안함

http client 예시
- https://github.com/cesanta/mongoose/blob/master/examples/http-client/main.c
예시처럼 http 전문을 모두 다 기록하는 방식이라서 기존 통검 구현과 차이점을 못느끼겠다.

```c++
    // Send request
    mg_printf(c,
              "GET %s HTTP/1.0\r\n"
              "Host: %.*s\r\n"
              "\r\n",
              mg_url_uri(s_url), (int) host.len, host.ptr);
```

## https://docs.libcpr.org/
Curl for People
C++ Requests is a simple wrapper around libcurl inspired by the excellent Python Requests project.

python requests 처럼 가독성 있게 curl 인터페이스를 구성했다. libcurl 의존성이 있음
Asynchronous requests 를 지원 

```c++
#include <cpr/cpr.h>

int main(int argc, char** argv) {
    cpr::Response r = cpr::Get(cpr::Url{"https://api.github.com/repos/libcpr/cpr/contributors"},
                      cpr::Authentication{"user", "pass"},
                      cpr::Parameters{{"anon", "true"}, {"key", "value"}});
    r.status_code;                  // 200
    r.header["content-type"];       // application/json; charset=utf-8
    r.text;                         // JSON text string
}
```
 설치 방법은 git 으로 직접 받아서 인스톨하는 방식

```console
$ git clone https://github.com/whoshuu/cpr.git
$ cd cpr
$ mkdir build && cd build
$ cmake ..
$ make
$ make install

# g++ -std=c++11 -o main -lcpr main.cpp
```

## https://github.com/chenshuo/muduo
Muduo is a multithreaded C++ network library based on the reactor pattern.
require packages
```console
sudo yum install gcc-c++ cmake make boost-devel
```
http client 는 callback 함수를 bind 하여 구현한다.
https://github.com/chenshuo/muduo-tutorial 따라하기 튜토리얼이 있다.
multiplexer 구현
- https://github.com/chenshuo/muduo/blob/master/examples/multiplexer/multiplexer.cc

## https://github.com/Qihoo360/evpp
중국거라 패스.. boost 라이브러리 의존성도 있다.
evpp is a modern C++ network library for developing high performance network services using TCP/UDP/HTTP protocols.

```
Multi-core friendly and thread-safe
A nonblocking multi-threaded TCP server
A nonblocking TCP client
A nonblocking multi-threaded HTTP server based on the buildin http server of libevent
A nonblocking HTTP client
```
http client 예제
- https://github.com/Qihoo360/evpp/tree/master/examples/http/http_client_request
