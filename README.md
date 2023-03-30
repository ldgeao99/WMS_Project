# :pushpin: WMS_Project(창고 재고관리 시스텀)

</br>

## 1. 제작 기간 & 참여 인원
- 22.10.24 ~ 22.11.24 (4주)
- 팀프로젝트(이대곤 외 7명)
<details>
<summary><b>팀플 이미지</b></summary>
<div markdown="1">
	<img src="https://user-images.githubusercontent.com/106065178/207888739-03680068-5ce0-4c65-94ad-1c12f728883b.png" width="400" height="300">
	<img src="https://user-images.githubusercontent.com/106065178/207888770-120dc403-70b1-41d0-9060-54a495ce42e5.png" width="400" height="300">
</div>
</details>

</br>

## 2. 사용 기술
  - Java
  - Spring
  - Maven
  - AWS EC2, AWS RDS, Jenkins, Github-Webhook, Oracle Cloud, Ngrok 
  - Oracle
  - Tomcat
  - HTML, CSS, Javascript, Jsp, Bootstrap, jQuery
  - RestAPI
  - Generic
  - WebSoket
  - Cookies

</br>

## 3. ERD 설계
![WMS_ERD 최종](https://user-images.githubusercontent.com/106065178/207878543-8710d855-3ee9-4ab2-8f19-0ec27a03972a.png)

## 3.1 시퀀스 다이어그램
<p align="center">
  <img src="https://github.com/ldgeao99/WMS_Project/blob/main/sequence%20diagram.PNG" width="500" height="600">
  </p>
  <br>


## 4. 핵심 기능 
이 서비스의 핵심 기능은 **창고재고관리 및 채팅**입니다.  
사용자는 이 서비스를 이용하기 위해 기초 정보를 등록하고 그 등록한 정보를 토대로 **재고를 발주,수주,이동을 통하여 재고관리**를 할 수 있습니다.

<details>
<summary><b>핵심 기능 설명 펼치기</b></summary>
<div markdown="1">

### 4.1. AWS-Jenkins-Github_Webhook
  <p align="center">
  <img src="https://user-images.githubusercontent.com/106065178/207881230-a1f44baf-5716-4a31-8a0d-ecfadc7248e0.png">
  </p>
  <br>

- 이전 프로젝트에서는 Spring에서 **War 파일을 추출하여 FileZilla로 프로젝트를 배포**하였으나 이렇게 진행할 시 <br>
**관리자는 매번 프로젝트를 다시 올려야하고 사용자는 관리자가 배포하는 기간동안 이용할 수 없는 번거럽고 치명적인 문제점 발생**

- 이것을 해결하기 위해 **AWS EC2에 Jenkins**를 설치하여 사용자는 서버1을 사용하고 있다가 관리자가 Github에 Push할 때마다 **Github_Webhook**로 신호를 보내주고 그 신호를 받은 Jenkins가 서버2에 자동적으로 배포를 하고 배포가 완료됨과 동시 사용자는 서버2를 사용하게 되는 것입니다.
- 이로 인해 **관리자는 지속적인 배포의 번거로움을 없애고 사용자는 끊김없는 서버를 사용**할 수 있게 됩니다.<br>
<a href="https://ldgeao99-developer.tistory.com/category/%EA%B0%9C%EC%9D%B8%20%EA%B3%B5%EB%B6%80/%28CI%20%7C%20CD%29%20Jenkins">[&nbsp;블로그 정리&nbsp;]</a>
<br>
	
### 4.2. Websoket 실시간 채팅
  <p align="center">
  <img src="https://user-images.githubusercontent.com/106065178/207891192-33383516-f9a5-4cab-a2eb-0eb8b8bc2d47.png">
  </p>
  <br>

- 이전 프로젝트에서는 Spring의 버전이 낮아 WebSoket을 사용하기 적절하지 않아 Ajax를 통한 reload 새로고침을 사용하여 구현하였습니다.

- 이번 프로젝트에서는 구현을 시작하기 앞서 Spring의 버전을 높히고 **Websoket을 활용**하였습니다.

- **Websoket을 활용**하여 새로고침없는 (끊김없는) **실시간 채팅**을 구현하였습니다.
<br>

### 4.3. Oracle 함수
  ~~~java
create or replace FUNCTION GENERATE_LOT
(V_ITEM_NO ITEM.NO%TYPE)
RETURN VARCHAR
IS
    V_LOT_CODE LOT.CODE%TYPE; -- 가장 최근 로트코드
    VR_LOT_CODE LOT.CODE%TYPE; -- 만들어진 로트코드
    V2_ITEM_NO ITEM.NO%TYPE; -- 가장 최근 로트코드의 품목번호
    V_INPUT_DATE VARCHAR2(8);
    V_SEQ VARCHAR2(3);

BEGIN
    WITH A AS (
        SELECT NO, CODE, ITEM_NO, REG_DATE, ROW_NUMBER() OVER (ORDER BY NO DESC) AS LEV
        FROM LOT
    )
    SELECT CODE
    INTO V_LOT_CODE
    FROM A
    WHERE LEV = 1;

    V_INPUT_DATE := SUBSTR(V_LOT_CODE, 0, 8);
    V_SEQ := SUBSTR(V_LOT_CODE, -3);

    IF TO_CHAR(SYSDATE, 'YYYYMMDD') != V_INPUT_DATE THEN
        V_INPUT_DATE := TO_CHAR(SYSDATE, 'YYYYMMDD');
        V_SEQ := LPAD('1', 3, '0');
    ELSE
        V_SEQ := LPAD(TO_NUMBER(V_SEQ) + 1, 3, '0');
    END IF;

    VR_LOT_CODE := V_INPUT_DATE || V_ITEM_NO || V_SEQ;

    RETURN VR_LOT_CODE;
END;	
~~~

- 이번 프로젝트에서는 상품마다 부여되는 **로트번호를 생성하기 위해 함수를 사용**하여 같은 코드를 여러번 반복하지 않고 간결하게 보다 **Clean한 코드**를 작성하였습니다.

<br>
	
### 4.4. Rest API

~~~java
	@GetMapping(
			value={"/pages/{pageNum}/{amount}", "/pages/{pageNum}/{amount}/{whatColumn}", "/pages/{pageNum}/{amount}/{whatColumn}/{keyword}"}, 
			produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
	public ResponseEntity<PageDTO<ItemVO>> clientlist(
			@PathVariable("pageNum") int pageNum,
			@PathVariable("amount") int amount,
			@PathVariable(value="whatColumn", required = false) String whatColumn,
			@PathVariable(value="keyword", required = false) String keyword) {				
		

		//System.out.println(pageNum + " " + amount + " " + whatColumn + " " + keyword);
		
		Criteria cri = new Criteria(pageNum, amount, whatColumn, keyword);
		
		return new ResponseEntity<>(service.getListPage(cri), HttpStatus.OK);		
	}
	
~~~

- **Rest API를 사용**하여 메시지를 읽는 것 만으로도 메시지가 **의도하는 바를 명확하게 파악**할 수 있도록 진행하였습니다.
<br>

### 4.5.  Generic

~~~java
@Data
@AllArgsConstructor
public class PageForWareHouseDTO<T> {
	
	private int totalCount;
	private List<T> list;
	private CriteriaForWareHouse cri;
	
}
	
~~~

- **Generic 방식**을 사용하여 비슷한 코드의 재사용성을 높혔으며, 후에 관리하기가 쉽도록 구현하였습니다.
<br>

### 4.6.  Redirect 객체 넘기기[RedirectAttributes]

	
<details>
<summary><b>기존 코드</b></summary>
<div markdown="1">
	
~~~java
	@RequestMapping(value = command , method = RequestMethod.POST)
	public String updateMember(MemberBean mb,Model model) {
		
		return "redirect:mypage.mb?select=6";
	}
	
~~~
	
</div>
</details>

	
<details>
<summary><b>개선된 코드</b></summary>
<div markdown="1">

~~~java
	@PostMapping("/update")
	public String update(ClientVO VO,SearchVO searchvo,RedirectAttributes rttr) {
		
		service.update(VO);
		rttr.addFlashAttribute("searchvo",searchvo);
		
		return redirect;
	}
	
~~~

</div>
</details>

</br>	
	
- 이전 프로젝트에서는 변수를 하나하나 받아와서 redirect 주소 뒤에 변수를 붙혀 넘기는 방식을 사용했었습니다.
	
- 이번 프로젝트에서는 개선하여 **RedirectAttributes클래스**를 사용하여 사용할 변수들을 객체에 담아 **객체를 바로 넘길 수 있도록** 하여 **Clean한 코드**를 작성하였습니다. 
<br>
	

### 4.7.  JSON Parsing

	
<details>
<summary><b>기존 코드</b></summary>
<div markdown="1">
	
~~~java
	 $.ajax({
	 	type : 'post',
		url : "allchatting.mb",
		contentType: "application/x-www-form-urlencoded; charset=UTF-8",
		success : function(data) {
			var roomlist = data.split("|");
			
			for(var i=0 in roomlist){
				var roomlists = roomlist[i].split(",");
				$('#lists').append(
			              '<div class="card-body navbar-light px-0" data-simplebar>'+
			                '<div class="navbar-nav">'+
			                  '<a onClick="detailmsg('+roomlists[1]+')" class="nav-link d-flex align-items-center px-3 gap-3">'+
			                  '</a>'+
			                '</div>'+
			              '</div>'+
			            '</div>'); 
			}
			}//else
		}//success 
	})//ajax
~~~
	
</div>
</details>

	
<details>
<summary><b>개선된 코드</b></summary>
<div markdown="1">

~~~java
	$.getJSON("/chat/getAll", 
	 		function(c){
				for(i=0;i<c.length;i++){
					$("#messageArea").append(
					  "<div class='chat ch1'>"+
				          "<div class='lnamed'>"+c[i].member_name+" "+c[i].rank_name+"</div><div class='textbox'>"+c[i].content+"</div></div>");
				}
		)
	
~~~

</div>
</details>

</br>	
	
- 이전 프로젝트에서는 Controller에서 객체를 문자열로 바꿔준 후 view에서 Split으로 List로 만들어 준 후 사용을 하여 번거럽고 효율적이지 못한 코드 작성이였습니다.
	
- 이번 프로젝트에서는 개선하여 **Controller에서 바로 Json형태**로 받은 후 바로 그 **Json에 담긴 변수를 사용**할 수 있도록 개선하여 **코드의 효율성**을 높혔습니다.
<br>
  
### 4.8.  세션 확인

- 이전 프로젝트에서 로그인 후 화면 이동 시 일정시간이 지나면 **세션이 풀려 에러가 발생**하는 문제 발생
- 이번 프로젝트에서는 **세션이 풀리면 알림창을 뜨게 하여 에러 발생을 방지**하였습니다.
<br>

### 4.9.  Service 방식

- 이전 프로젝트에서 한 Controller에서 여러 작업을 하여 복잡하고 알아보기 힘든 코드로 작성이 되었었습니다.

- 이번 프로젝트에서는 **Service 방식**을 활용하여 Controller에서 한가지의 작업을 하면 뒤에 있는 Service에서 한 번에 작업을 하여 **Clean 한 코드**를 작성하였습니다.
<br>

### 4.10.  페이징 Offset ~ Fetch
~~~java
	SELECT * FROM client where email like '%' ||  #{keyword} || '%'
		ORDER BY no desc OFFSET #{beginRow} ROWS FETCH NEXT #{pageSize} ROWS ONLY
~~~

-  다른 페이징 방식보다 빠르고 **간편한 페이징, 속도가 가장 빨라서 효율적인 방식**으로 코드를 작성하였습니다.
<br>


</div>
</details>

</br>


## 5. 시연 영상
 <p align="center"><img src="https://user-images.githubusercontent.com/106065178/207879068-86cee319-dd15-4d5b-afe7-e88f7d0210be.gif" width="600" height="350">
<br><a href="https://www.youtube.com/watch?v=gVzC5DC1zgE&t=420s">[&nbsp;Youtube&nbsp;]</a><br><br>
</p>



## 6. 핵심 트러블 슈팅
### 6.1. literal does not match format string [데이터 타입이 맞지 않다는 의미]
- 모든 프로젝트를 마무리하고 로컬에서의 테스트를 문제없이 진행하고 AWS에 배포를 하여 테스트를 하는데 <br>
  배포할 때만 발생하는 문제를 발견하게 됐습니다.

- AWS의 EC2의 Unbuntu Lang은 UTF-8 형식을 사용하여 프로젝트에서 사용하는 ko_KR 형식과 달라 날짜 형식을 읽지 못하여 발생하는 문제였습니다.

- 이전 프로젝트에서는 프로젝트의 모든 Date 타입을 쓴 sql문들을 UTF-8 형식으로 바꿔주는 번거러운 작업을 하였습니다.
- 이번 프로젝트에서는 번거러운 작업을 방지하기위해 **AWS EC2의 Lang을 oracle sql에 맞는 ko_KR 로 바꿔**주었습니다.<br>
  <a href="https://ldgeao99-developer.tistory.com/614">[&nbsp;블로그 정리&nbsp;]</a>

### 6.2. AWS EC2 ram 용량 부족  [Jenkins 최소 사양 부족]
- **Jenkins에서 배포 시** 특정 부분도 아닌 계속 여러 부분에서 멈추는 현상이 발생하였습니다.<br>

- **Jenkins를 구동하기 위한 최소한의 사양**이 있는데 AWS EC2의 무료 버전인 micro를 사용하게 되면 **Ram 용량이 부족**하게 되는 현상을 발견하였습니다.

- 이를 해결하기위해 EC2 내에 **swap 공간을 활용**하여 Ram 사양을 키운 듯한 효과를 내었습니다.<br>
  <a href="https://ldgeao99-developer.tistory.com/600">[&nbsp;블로그 정리&nbsp;]</a>

</br>

## 7. 회고 / 느낀점
>프로젝트 개발 회고 :

- 이전 프로젝트보다 훨씬 **효율적**이고 **Clean 한 코드**를 작성할 수 있어 **발전**할 수 있었습니다. 다음도 그 다음도 전보다 훨씬 나은 코드 작성을 할 것입니다.
- 팀원 모두가 팀프로젝트를 통하여 하나라도 더 얻어갈 수 있도록 **서로 돕고 지원**하였습니다.
- 다양한 방법을 하고 싶었고 처음 접하는 것들이 많아 에러가 무수히 발생하였지만 더 많은 것들을 배울 수 있었기에 재미있었습니다.
- **사용자 입장에서 사용**해보고 불편한 점이 있는 지 확인해가면서 프로젝트를 진행하였습니다.
- 이번에 사용한 기술을 다음 프로젝트에서는 더욱 **효율적으로 코드**를 작성할 수 있는 방법이 있는 지 알아보고 배워가며 적용해보고 싶습니다.

<br>

- 이번 프로젝트에서는 팀장 역할을 맡게 되어 모든 이들이 하나라도 더 얻어갈 수 있도록 도움을 주고 팀의 좋은 분위기를 형성하였습니다.
- 팀원 각자에게 업무를 주고 맡은 업무 외에도 서로 돕고 지원하였습니다.

