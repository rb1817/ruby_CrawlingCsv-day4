# Day 4

## CSV 파일 W/R 코드 - 웹툰 크롤링 데이터 이용

*app.rb*

```ruby
require 'sinatra'
require 'sinatra/reloader'
require 'nokogiri'
require 'json'
require 'uri'
require 'rest-client'
require 'csv'

get '/check_file' do
 
    unless File.file?('./test.csv')
    	#내가 받아온 웹툰 데이터를 저장할 배열 생성
    	toons = []
    	#원툰 데이터를 받아올 url파악 및 요청보내기
    	url = "http://webtoon.daum.net/data/pc/webtoon/list_serialized/mon"
    	result = RestClient.get(url)
    	#응답으로 온 내용을 배열 형태로 바꾸기
   		webtoons = JSON.parse(result)
    	#해쉬에서 웹툰 리스트에 해당하는 부분 순환하기
    	webtoons["data"].each do |toon|
        	#웹툰 제목
        	title = toon["title"]
        	#웹툰 이미지 주소
        	image = toon["thumbnailImage2"]["url"]
        	#웹툰을 볼 수 있는 주소
        	link = "http://webtoon.daum.net/webtoon/view/#{toon['nickname']}"
            toons << [title,image,link]
    	end
        #CSV 파일을 새로 생성하는 코드
        CSV.open('./test.csv', 'w+') do |row|
            toons.each_with_index do |toon, index|
                row << [index+1, toon[0], toon[1], toon[2]]
            end
        end
        erb :check_file
    else
        #존재하는 CSV 파일을 불러오는 코드
        @webtoons = []
        CSV.open('./test.csv', 'r').each do |row|
         @webtoons << row
        end
        erb :webtoons
    end
        
end
```

*views/webtoons.erb*

```erb
<table class = "table table-hover">
    <thead>
        <th>글번호</th>
        <th>이미지</th>
        <th>제목</th>
        <th>링크</th>
    </thead>
    <tbody>
        <% @webtoons.each do |toon|%>
        <tr>
            <td><%=toon[0]%></td>
            <td><img src ="<%=toon[2]%>"></td>
            <td><%=toon[1]%></td>
            <td><a href="<%=toon[3]%>">보러가기</a></td>
        </tr>
        <% end %>
    </tbody>
</table>
<h1><%=@daum%><h1>
```



## layout.erb-bootstrap

* 레이아웃은 말 그대로 전체 페이지를 설계해놓은 설계도이며, 위와 같이 공통적으로 사용되는 부분을 하나의 파일에 모아놓고 여러 뷰 파일에서 사용하기 위해 만들어졌다.  
* 레이아웃의 yield는 우리가 작성한 뷰 파일로 치환된다 .

*views/layout.erb*

```erb
<html>
    <head>
        <title>멀캠해커톤 화이팅</title>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css" integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    </head>
    <body>
        <%= yield%>
    </body>
</html>
```



## CDN[Contents Delivery Network]

* 웹, 애플리케이션, 스트리밍 미디어를 비롯한 콘텐츠를 전송하도록 최적화된 전세계적으로 촘촘히 분산된 서버로 이루어진 플랫폼입니다.
* 수많은 물리적 위치와 네트워크 위치에 분산되어 있어 웹 콘텐츠에 대한 엔드유저의 요청에 직접적으로 응답하고 빠르고 안전한 미디어 전송을 보장합니다.
* **오리진이라고도 불리는 콘텐츠 서버와 엔드유저(클라이언트) 사이에서 컨텐츠를 전달하는 역할을 합니다.**

