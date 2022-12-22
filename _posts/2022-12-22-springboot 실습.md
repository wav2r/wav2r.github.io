---
layout: post
title: springboot 실습
tags: 
math: true
date: 2022-12-23 01:11 +0800
---

![graph](/assets/python/graph_ex3.png)

학교에서 과제로 스프링부트를 이용해 간략한 홈페이지를 만들었다.

참고서적 - 백견불여일타 스프링 부트 쇼핑몰 프로젝트 with JPA
직접 타이핑하며 코드의 각 줄이 어떤 역할을 하는지 이해하는 방식으로 공부하는 책이었다. 책의 내용을 그대로 따라하는 것만이 아니라 각 코드의 기능을 이해하고 필요한대로 변형해서 사용하려 노력했다. 아래 내용은 책에 서술된 코드를 따라가며 자신의 것으로 만들기 위해 공부해본 내용이다.

### 작성할 홈페이지 계획
#### 흐름도
![flow chart](/assets/springboot1/flow.png)

#### 화면설계
![wireframe](/assets/springboot1/wireframe.png)
html, css, js프레임워크인 '부트스트랩'에서 제공하는 navbar 컴포넌트를 활용한다.
원래 html, css 코드를 작성할 때 접속하는 기기의 화면 크기에 따라 형태가 무너지지 않도록 신경써줘야 하지만, 부트스트랩에서 반응형으로 제공되므로 그런 신경을 써줄 필요가 없다는 점이 편리하다.

#### 엔티티 설계
데이터베이스의 테이블에 대응할 클래스인 엔티티를 먼저 설계했다. 내가 계획한 건 음료 리뷰 사이트로 아래는 등록할 리뷰의 entity이다.
![capture](/assets/springboot1/capture4.png)

#### Thymeleaf
책에서 템플릿 엔진으로 Thymeleaf를 사용하여 이를 사용해 실습했다.
Thymeleaf는 스프링이 권장하는 서버 사이드 자바 템플릿 엔진이다. Thymeleaf의 가장 큰 장점은 natural templates으로, 서버 사이드 렌더링 없이도 html파일을 정상적으로 출력하기 때문에 개발자와 디자이너(혹은 퍼블리셔) 간의 소통에 용이하다.

#### 페이지 구현

##### 메인페이지 구현
![capture](/assets/springboot1/main_capture.png)

    <span class="sub">이미 회원이라면
                    <a style="
                            display:inline;
                            text-decoration: underline;"
                       class="sub" href="members/login">이쪽</a>
    </span>

링크를 연결하면 자동으로 줄바꿈이 된다. 이를 막기 위해 display:inline 을 적용했다.

![capture](/assets/springboot1/flew_array.png)
flex 배치로 화면을 다음처럼 구성했다. 앞으로 계속 배워나가며 더 효율적인 배치는 어떤 것일지 고민해보고 싶다.


##### 등록페이지 구현
![capture](/assets/springboot1/capture1.png)

##### 이미지 저장
![capture](/assets/springboot1/capture5.png)
상품 이미지는 이미지 자체를 데이터베이스에 저장하는 것이 아닌 [이미지 파일명, 원본 이미지 파일명, 이미지 조회 경로]를 저장하여 파일시스템에 저장된 이미지를 불러올 수 있도록 한다. 이미지 자체를 데이터베이스에 저장하는 것은 불가능한 일은 아니지만 이미지 파일의 크기가 큰 경우 저장이 힘들고 효율이 좋지 않아 일반적으로 전자의 방식을 사용한다. (가변길이를 가져 큰 파일크기를 저장하는데 유용한 BLOB 데이터 유형을 사용하는 방식이 다음 블로그에 서술되어 있었다. https://www.thecreativedev.com/how-to-store-a-file-in-database/)

##### 수정페이지 구현
![capture](/assets/springboot1/capture3.png)
리뷰 아이디를 통해서 리뷰 수정 페이지로 진입하도록 개발한다.


![capture](/assets/springboot1/modify_code3.png)
↑ service
controller - service - repository 중 service에서 상품을 조회하기 위한 코드이다.

controller에서 getDrinkDtl을 실행하면 service의 트랜잭션이 실행된다. service에선 Repository를 통해 데이터베이스를 조회하여 리뷰 정보를 가져오고, 해당 리뷰의 이미지를 조회한다. 이 이미지 리스트와 조회한 리뷰 정보를 drinkFormDto에 담아 controller로 return한다.

![capture](/assets/springboot1/modify_code2.png)
↑ service

![capture](/assets/springboot1/modify_code1.png)
↑ controller

controller의 try-catch문에서 EntityNotFoundException 예외가 발생할 경우(입력된 리뷰 아이디가 존재하지 않을 경우) key:"errorMessage", value:"존재하지 않는 상품입니다."를 모델에 담아서 상품 등록 페이지로 진입한다. 그럼 아래같은 에러 메시지가 발생한다.

![error message](/assets/springboot1/error1.png)

상품이 존재할 경우 dto를 controller가 모델의 형태로 뷰에 전달하여 수정하려는 데이터가 화면에 출력된다.


DrinkImgService 클래스에 추가되는 수정을 위한 코드이다.(변경 감지 기능을 활용)
![drinkImg entity](/assets/springboot1/drinkImgService_modify.png)

DrinkService 클래스에 리뷰를 수정하기 위한 코드를 추가한다. 마찬가지로 변경 감지 기능을 사용한다.

    updateItem(DrinkFormDto drinkFormDto, List<MultipartFile> drinkImgFileList) throws Exception{ ... }

리뷰를 업데이트하는데 updateItem 함수를 사용한다. 여기서 updateDrink는 entity에서 정의되어있다.
![entity1](/assets/springboot1/entity_update.png)


##### 검색페이지 구현 (조회 기능)
![search view](/assets/springboot1/capture_search.png)

조회를 위한 프레임워크 Querydsl을 사용한다. Maven > compile을 실행시켜 QDomain을 생성할 수 있다.

Querydsl을 Spring Data Jpa와 함께 사용하기 위해선 아래의 3단계의 과정으로 사용자 정의 리포지토리를 정의해야 한다.
1) 사용자 정의 인터페이스 작성
 - ItemRepositoryCustom.java
2) 사용자 정의 인터페이스 구현
 - ItemRepositoryCustomImpl.java
3) Spring Data Jpa 리포지토리에서 사용자 정의 인터페이스 상속
 - ItemRepository 인터페이스에서 ItemRepositoryCustom 인터페이스를 상속
 
 html 코드
 
    <th:block layout:fragment = "script">
        <script th:inline="javascript">
            $(document).ready(function(){
                $("#searchBtn").on("click", function(e){
                    e.preventDefault();
                    page(0);
                });
            });

            function page(page){
                var searchDrinkType = $("#searchDrinkType").val();
                var searchBy = $("#searchBy").val();
                var searchQuery = $("#searchQuery").val();

                location.href="search/" + page + "?searchDrinkType=" + searchDrinkType
                + "&searchBy=" + searchBy + "&searchQuery=" + searchQuery;
            }
        </script>
    </th:block>

prevenDefault()를 사용하면 검색 버튼을 클릭할 때 form 태그의 전송을 막아 페이지가 새로고침되는 것을 방지해준다.

![search view](/assets/springboot1/capture_search2.png)
location 객체에 속한 href 프로퍼티는 현재 접속중인 페이지의 정보를 가지는데 이를 가져와 검색결과를 표시한다.

content 영역

검색창

        <form th:action="@{'/search/' +
        ${drinks.number}}" role="form" method="get" th:object="${drinks}">
            <div class="form-inline justify-content-center" th:object="${drinkSearchDto}">
                <select th:field="*{searchDrinkType}" class="form-control" style="width:auto;">
                    <option value="">종류</option>
                    <option value="COFFEE">커피</option>
                    <option value="ETC">기타</option>
                </select>
                <select th:field="*{searchBy}" class="form-control" style="width:auto;">
                    <option value="cafeNm">카페명</option>
                    <option value="menuNm">메뉴 이름</option>
                    <option value="drinkType">종류</option>
                </select>
                <input th:field="*{searchQuery}" type="text" class="form-control" placeholder="검색어를 입력해주세요">
                <button id="searchBtn" type="submit" class="btn btn-primary" style="margin-left:3px;">Search</button>
            </div>


테이블

            <table class="table">
                <thead>
                <tr>
                    <td>리뷰아이디</td>
                    <td>카페명</td>
                    <td>메뉴이름</td>
                    <td>종류</td>
                </tr>
                </thead>
                <tbody>
                <tr th:each="drink, status: ${drinks.getContent()}">
                    <td th:text="${drink.id}"></td>
                    <td>
                        <a th:href="'item/'+${drink.id}"
                        th:text="${drink.cafeNm}"></a>
                    </td>
                    <td>
                        <a th:href="'item/'+${drink.id}"
                        th:text="${drink.menuNm}"></a>
                    </td>
                    <td th:text="${drink.drinkType==T(kr.sujin.cafe.constant.DrinkType).DESSERT}? '디저트':'음료'"></td>
                    <td th:text="${drink.createdBy}"></td>
                </tr>
                </tbody>
            </table>

.getContent() 메소드를 호출해서 조회한 리뷰 데이터를 얻을 수 있다. th:each 문법을 이용하면 tr이 리스트의 사이즈만큼 반복된다.(줄row이 그려진다)


페이지 번호

            <div th:with="start=${(drinks.number/maxPage)*maxPage + 1}, end=(${(drinks.totalPages == 0) ? 1 : (start + (maxPage - 1) < drinks.totalPages ? start + (maxPage - 1) : drinks.totalPages)})" >
                <ul class="pagination justify-content-center">

                    <li class="page-item" th:classappend="${drinks.first}?'disabled'">
                        <a th:onclick="'javascript:page(' + ${drinks.number - 1} + ')'" aria-label='Previous' class="page-link">
                            <span aria-hidden='true'>Previous</span>
                        </a>
                    </li>

                    <li class="page-item" th:each="page: ${#numbers.sequence(start, end)}" th:classappend="${drinks.number eq page-1}?'active':''">
                        <a th:onclick="'javascript:page(' + ${page - 1} + ')'" th:inline="text" class="page-link">[[${page}]]</a>
                    </li>

                    <li class="page-item" th:classappend="${drinks.last}?'disabled'">
                        <a th:onclick="'javascript:page(' + ${drinks.number + 1} + ')'" aria-label='Next' class="page-link">
                            <span aria-hidden='true'>Next</span>
                        </a>
                    </li>

                </ul>
            </div>
        </form>

- th:with로 start(페이지 시작 번호) / end(페이지 끝 번호) 변수값을 정의
- 클릭 시 page함수를 호출하여 해당 페이지로 이동
- 페이지의 위치에 따라 th:classappend로 클래스를 추가하여 active 상태나 disabled 상태가 되도록 함

 
##### 상세페이지 구현
controller에 리뷰 상세 페이지로 이동하는 코드를 추가한다. 이때 수정 페이지 구현에서 만들어두었던 getDrinkDtl을 그대로 사용한다.

![detail view](/assets/springboot1/detailPage.png)


##### 도메인 연결
![detail view](/assets/springboot1/ipJar.png)
AWS EC2 인스턴스를 사용해 가상 서버를 구축하고 ubuntu를 이용해 springboot 서버를 배포했다.
AWS의 RDS에서 데이터베이스를 제공받았다.

![detail view](/assets/springboot1/domainUrl.png)
Domain을 발급받아서 연결하는 작업도 했다.