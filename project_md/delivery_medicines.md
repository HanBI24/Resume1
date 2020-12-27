# **프로젝트 개요**
학교에서 수업을 듣는데, 배운 내용을 토대로 경진대회를 개최한다고 했다. 나는 그 중에서 블록체인을 선택했고, 프로젝트 내용은 의약품 배달 서비스로 결정했다.

의약품 배달은 가짜 약, 불법 거래 등으로 인해 현재 불법이다. 그러나 블록체인의 투명성과 불변성으로 보안성을 높이면 합법으로 바꿀 수 있지 않을 것이라는 생각에 시작한 프로젝트가 의약품 배달 서비스 프로젝트이다.  

# **프로젝트 요약**
<p align="center"><img src = ../img/expended_project_summary.png width = 1000 height = 400/></p>

### **프로젝트 처리 과정**
1. 구매자가 웹 사이트에 접속.


2. 구매할 의약품이 있다면 결제 버튼을 누르고, Meta Mask와 Ethereum이 실행.


3. 구매자가 배송 정보를 입력하면, 구매자에겐 txt 파일로 구매 정보가 다운로드 됨.


4. 결제 버튼을 누르면 JavaScript와 Solidity가 실행되며, 의약품 가격만큼 이더리움이 빠져나감.


5. 서버 관리자가 Firebase Cloud Storage를 통해 구매자의 배송 정보를 관리할 수 있음.


6. Kakao Map으로 주변 약국을 검색할 수 있음.


7. 공공 데이터 포털에서 받아온 의약품 리스트를 검색 및 구매할 수 있음.
   
# 프로젝트 구현

## **- Solidity**

### **약 값 Ethereum으로 계산**
```javascript
// adoption.sol
contract Adoption {
    address[16] public adopters;
    mapping(address => uint) private balances;

    // 약 구매 가격 처리 부분
    function adopt(uint mId) public payable returns(uint) { 
        require(mId >= 0 && mId <= 15 && balances[msg.sender] + msg.value >= balances[msg.sender]);
        balances[msg.sender] += msg.value;
        return balances[msg.sender];
    }

    // 고객 정보 얻어옴
    function getAdopters() public view returns(address[16] memory) {
        return adopters;
    }  
    
}
```

### **배송 정보 출력**

```javascript
// UserAddress.sol
// 현재는 "배송 출발" 메시지만 띄움.
// 나중에 DTO Class로 구현할 예정.
contract UserAddress {
    string public userAddress1 = "배송 출발";
    string public userAddress2 = "배송 중";
    string public userAddress3 = "배송 완료";
    //uint public cnt = 0;

    function getDeliveryInfo() public view returns (string memory) {
        // require(cnt == 0);
        return getUserAddress1();
        // require(cnt == 1);
        // return getUserAddress2();
        // require(cnt == 2);
        // return getUserAddress3();

    }

    function setUserAddress (string memory setAddress1) public {
        userAddress1 = setAddress1;
    }

    function getUserAddress1() public view returns (string memory) {
        return userAddress1;
    }

    function getUserAddress2() public view returns (string memory) {        
        return userAddress2;
    }

    function getUserAddress3() public view returns (string memory) {
        return userAddress3;
    }
}
```

## **- JS (계약에 필요한 파일)**
### **계약에 필요한 파일 생성**
```javascript
// 4_UserAddress.js
// 새로만든 UserAddress.sol 파일의 exports 시킴.
// 계약 정보에 필요한 JSON 파일 생성.
// migration을 하기 위해선 이 파일을 꼭 생성해야 함.
var UserAddress = artifacts.require("./UserAddress.sol");

module.exports = function(deployer) {
  deployer.deploy(UserAddress);
};
```

## **- JS (메인 페이지)**
### **JSON 파일에서 필요한 의약품 데이터 로딩**
```javascript
// app.js
init: async function() {
   // 약 정보 JSON에서 불러옴
   $.getJSON('../mdcs.json', function(data) {
     var medRow = $('#medRow');
     var medTemplate = $('#med_list');

     for (i = 0; i < data.length; i ++) {
       medTemplate.find('.med_title').text(data[i].name);
       medTemplate.find('img').attr('src', data[i].picture);
       medTemplate.find('.med_buy_btn').attr('data-id', data[i].id);
       medTemplate.find('.med_info_btn').attr('data-id', data[i].id);
       medTemplate.find('.med_price').text(data[i].cost);
       medTemplate.find('.med_info').text(data[i].efc1);

       medRow.append(medTemplate.html());
     }
   });
   // 공공 데이터 포털에서 추가한 약 정보 JSON에서 불러옴
   $.getJSON('../med_info_public_data.json', function(data) {
    var medListRow = $('.medListRow');
    var medList = $('.medList');

    for (i = 0; i < data.length; i ++) {
      medList.find('.med_list_name').text(data[i].MEDICINE_NAME);
      medList.find('.med_list_info').text(data[i].MEDICINE_INFO);
      medList.find('.med_list_price').text(data[i].cost);
      medList.find('.med_list_buy_btn').attr('data-id', data[i].id);
      medListRow.append(medList.html());
    }
  });
   return await App.initWeb3();
 },
 ```
 ### **Web3 불러오고, local 서버 지정**
```javascript
   initWeb3: async function() {
    if (window.ethereum) { 
      App.web3Provider = window.ethereum; 
      try { 
      // Request account access 
      await window.ethereum.enable(); 
      } catch (error) { 
      // User denied account access... 
      console.error("User denied account access") 
      } 
      } 
      // Legacy dapp browsers... 
      else if (window.web3) { 
           App.web3Provider = window.web3.currentProvider; 
      } 
      // If no injected web3 instance is detected, fall back to Ganache 
      else { 
           App.web3Provider = new Web3.providers.HttpProvider("http://localhost:7545"); 
      } 
      web3 = new Web3(App.web3Provider); 
      

    return App.initContract();
  },
  ```
  ### **Migration한 파일들에 대해 계약서 작성을 미리 해주는 메서드**
```javascript
    // migrate한 파일들을 동기화하여 계약에 대한 정보를 수동 입력하지 않아도 된다.
  initContract: function() {
    $.getJSON('Adoption.json', function(data) { 

      var AdoptionArtifact = data; 
      App.contracts.Adoption = TruffleContract(AdoptionArtifact); 
      App.contracts.Adoption.setProvider(App.web3Provider); 
      
      $.getJSON('SimpleBank.json', function(data1) { 
        
        var SimpleBankArtifact = data1; 
        App.contracts.SimpleBank = TruffleContract(SimpleBankArtifact); 
        App.contracts.SimpleBank.setProvider(App.web3Provider); 

        // UserAddress 파일 추가
        $.getJSON('UserAddress.json', function(data2) { 
        
          var UserAddressArtifact = data2; 
          App.contracts.UserAddress = TruffleContract(UserAddressArtifact); 
          App.contracts.UserAddress.setProvider(App.web3Provider); 
          return App.markAdopted(); 
    
        });
  
      });

    }); 

    return App.bindEvents();
  },
  ```
  ### **결제 시 구매자의 Private key를 확인한 다음 Meta Mask로 결제를 진행하는 메서드**
```javascript
    // 결제시 어떤 동작 실행
  markAdopted: function() {
    var adoptionInstance; 
    var cnt = 0;
    App.contracts.Adoption.deployed().then(function(instance) { 
       adoptionInstance = instance; 
       return adoptionInstance.getAdopters.call(); 
    }).then(function(adopters) { 
       for (i = 0; i < adopters.length; i++) { 
          if (adopters[i] !== '0x0000000000000000000000000000000000000000') { 
             cnt++;
          }
       }  
    }).catch(function(err) { 
       console.log(err.message); 
    }); 
  },
  ```
### **웹에서 구매버튼 눌렀을 때 구매자 정보 입력 창과 결제 창 띄우는 메서드**
  ```javascript
   // 구매 버튼 눌렀을 때 
  handleAdopt: function(event) {
    event.preventDefault();

    var mId = parseInt($(event.target).data('id'));

    var adoptionInstance; 
    web3.eth.getAccounts(function(error, accounts) { 
       if (error) { console.log(error); } 
       var account = accounts[0]; 

       alert("주소를 입력해주세요");
       newWin = window.open("/adress.html", "myWin", "left=300,top=300,width=800,height=300");
    
       App.contracts.Adoption.deployed().then(function(instance) { 
          adoptionInstance = instance; 
          
          // Execute adopt as a transaction by sending account 
          var price = $('.med_price').eq(mId).text();
          var med_cnt_tmp = $('.med_cnt').eq(mId).val();
          var med_cnt = parseInt(med_cnt_tmp);
          var med_cnt_price = price*med_cnt;
          var amount = parseFloat(med_cnt_price)*Math.pow(10,18);
          
          // 배송 정보 처리 
          App.getUserAddress();
          
          return adoptionInstance.adopt(mId, {value:`${amount}`, from:account, to:adoptionInstance.address});
     
       }).then(function(result) { 
          return App.markAdopted(); 
       }).catch(function(err) { 
          console.log(err.message); 
       }); 
    }); 
  },
  ```

### **최종 결제가 완료되면 배송 출발 메시지 띄우는 메서드**
```javascript
  // 배송 정보 처리
  getUserAddress: function() {
    var adoptionInstance; 
    web3.eth.getAccounts(function(error, accounts) { 
       if (error) { console.log(error); } 
       var account = accounts[0];     
       App.contracts.UserAddress.deployed().then(async function(instance) { 

          adoptionInstance = instance;         
          cnt++;
          // Solidity에서 값을 가져오는 것이 느리기 때문에 비동기로 처리
          var text1 = await adoptionInstance.getDeliveryInfo();
          alert(text1);
          return;
     
       }).then(function(result) { 
       }).catch(function(err) { 
          console.log(err.message); 
       }); 
    }); 
  },
  ```

  ## **- JS (구매자 정보 입력 페이지)**
  ### **Firebase Cloud Storage 연동**
  ```javascript
  var firebaseConfig = {
    apiKey: "AIzaSyBpkeHthpTxPgv9VTgpj60iubinssKYkFk",
    authDomain: "delivery-medicine.firebaseapp.com",
    projectId: "delivery-medicine",
    storageBucket: "delivery-medicine.appspot.com",
    messagingSenderId: "224510722855",
    appId: "1:224510722855:web:a669764f28ea316f455f02",
    measurementId: "G-2G0ZW7YHWG"
  };
  // Initialize Firebase
  firebase.initializeApp(firebaseConfig);
  firebase.analytics();
  
  var name = $('#input_name').val();
  var number = $('#input_num').val();
  var address_road = $('#input_address_road').val();
  var address_detail = $('#input_address_detail').val();

  var db = firebase.firestore();
  db.collection("주문정보").add({
    Name: name,
    CallNumber: number,
    AddressRoad: address_road,
    AddressDetail: address_detail
})
.then(function(docRef) {
    console.log("Document written with ID: ", docRef.id);
})
.catch(function(error) {
    console.error("Error adding document: ", error);
});
  alert("구매 완료");
  saveToFile_Chrome("구매 정보", name, number, address_road, address_detail);
})
```
## **구매자 정보 txt 파일로 다운로드**
```javascript
function saveToFile_Chrome(fileName, name, call_number, address_road, address_detail) {
  var blob = new Blob(["| 이름: " +name, " | 전화번호: " +call_number, " | 주소: " +address_road, ", " +address_detail+ " |"], { type: 'text/plain' });
  objURL = window.URL.createObjectURL(blob);
          
  // 이전에 생성된 메모리 해제
  if (window.__Xr_objURL_forCreatingFile__) {
      window.URL.revokeObjectURL(window.__Xr_objURL_forCreatingFile__);
  }
  window.__Xr_objURL_forCreatingFile__ = objURL;
  var a = document.createElement('a');
  a.download = fileName;
  a.href = objURL;
  a.click();
}
```

  ## **- HTML (메인 페이지)**
  ### **사진에 적용할 CSS 코드**
```css
    <style>
      .bd-placeholder-img {
        font-size: 1.125rem;
        text-anchor: middle;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
      }

      @media (min-width: 768px) {
        .bd-placeholder-img-lg {
          font-size: 3.5rem;
        }
      }
    </style>
```

### **검색 기능에 사용할 CSS**
```css
<style type="text/css">
body{
  margin: 0;
  padding: 0;
  font-family: "Montserrat";
}
.searchbox{
  width: 90%;
  margin: 10px auto;
}
.header{
  background: #00000059;
  overflow: hidden;
  padding: 20px;
  text-align: center;
}
.header h1{
  text-transform: uppercase;
  color: white;
  margin: 0;
  margin-bottom: 8px;
}
<!--중략-->
```

### 카카오 맵 css
```css
<!--Kakao Map CSS-->
    <style>
      .map_wrap, .map_wrap * {margin:0;padding:0;font-family:'Malgun Gothic',dotum,'돋움',sans-serif;font-size:12px;}
      .map_wrap a, .map_wrap a:hover, .map_wrap a:active{color:#000;text-decoration: none;}
      .map_wrap {position:relative;width:100%;height:700px;}
      #menu_wrap {position:absolute;top:0;left:0;bottom:0;width:250px;margin:10px 0 30px 10px;padding:5px;overflow-y:auto;background:rgba(255, 255, 255, 0.7);z-index: 1;font-size:12px;border-radius: 10px;}
      .bg_white {background:#fff;}
      <!--중략...-->
```

### **약 리스트를 출력하는 부분**
```html
<!--JS에서 반복문을 통하여 여래 개를 추가함-->
<div id="medRow" class="row"> 
    </div>
    <div class="row" id="med_list">
      <div class="col-md-3">
        <div class="card mb-4 shadow-sm">
          <img class="bd-placeholder-img med_img" src="media/med_13.png" alt="타이레놀 500mg (해열진통제)" width="100%" height="500">
          <div class="card-body">
            <strong class="med_title">test_title</strong> <hr>
            <b>가격 :</b> <span class="med_price">test_price</span> ETH <br>
            <strong class="med_info">test_info</strong><br>
            <b>수량</b>&nbsp;&nbsp; <input class="med_cnt" size="1" value="1">
            <div class="d-flex justify-content-between align-items-center">
              <div class="btn-group">
                <button type="button" class="btn btn-sm btn-outline-secondary med_buy_btn">구매</button>
                <button type="button" class="btn btn-sm btn-outline-secondary med_info_btn">성분</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>  
  </main>
  ```

  ### **카카오 맵을 이용하여 주변 약국 검색하는 기능**
  ```html
    <div class="map_wrap">
    <div id="map" style="width:100%;height:100%;position:relative;overflow:hidden;"></div>

    <div id="menu_wrap" class="bg_white">
        <div class="option">
            <div>
                <form onsubmit="searchPlaces(); return false;">
                    키워드 : <input type="text" value="춘천 약국" id="keyword" size="15"> 
                    <button type="submit">검색하기</button> 
                </form>
            </div>
        </div>
        <hr>
        <ul id="placesList"></ul>
        <div id="pagination"></div>
    </div>
</div>
```

### **카카오 맵 기능 구현**
```javascript
<script type="text/javascript" src="//dapi.kakao.com/v2/maps/sdk.js?appkey=&libraries=services"></script>
<script>
// 마커를 담을 배열입니다
var markers = [];

var mapContainer = document.getElementById('map'), // 지도를 표시할 div 
    mapOption = {
        center: new kakao.maps.LatLng(37.566826, 126.9786567), // 지도의 중심좌표
        level: 3 // 지도의 확대 레벨
    };  

// 지도를 생성합니다    
var map = new kakao.maps.Map(mapContainer, mapOption); 

// 장소 검색 객체를 생성합니다
var ps = new kakao.maps.services.Places();  
// 중략...

```

### **의약품 검색**
```html
        <div class="searchbox">
          <div class="header">
            <h1>의약품 검색</h1>
            <input onkeyup="filter()" type="text" id="value" placeholder="의약품 이름 입력">
          </div>
    
          <div class="medListRow"></div>

          <div class="medList">

          <div class="container1">    
            <div class="item_med">
            <strong class="med_list_name">MedName</strong><br>
            <b>성분:</b> <strong class="med_list_info">MedInfo</strong><br>
            <b>가격 :</b> <span class="med_list_price">test_price</span> ETH <br>
            <b>수량</b>&nbsp;&nbsp; <input class="med_list_cnt" size="1" value="1">
              <button type="button" class="btn btn-sm btn-outline-secondary med_list_buy_btn">구매</button><br>
            </div>

          </div>
```

### **의약품 검색 기능**
```javascript
        <script type="text/javascript">
          function filter(){
    
            var value, name, item, i;
    
            value = document.getElementById("value").value.toUpperCase();
            item = document.getElementsByClassName("item_med");
    
            for(i=0;i<item.length;i++){
              name = item[i].getElementsByClassName("med_list_name");
              if(name[0].innerHTML.toUpperCase().indexOf(value) > -1){
                item[i].style.display = "flex";
              }else{
                item[i].style.display = "none";
              }
            }
          }
    </script>
```

## **- HTML (구매자 정보 입력 페이지)**
### **기본 입력 창 구현**
```html
<h3>배송 받으실 주소를 입력해주세요.</h3><hr><br>
        <b>받으시는 분 성함</b> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<input type="text" id="input_name" size="6" placeholder="이름"><br>
        <b>받으시는 분 전화번호</b> <input type="text" id="input_num" placeholder="전화번호"><br>
        <b>주소 (상세주소 포함)</b> &nbsp;<input type="text" id="input_address_road" size="20" placeholder="도로명 주소" readonly> &nbsp;	
        <input type="text" id="input_address_detail" size="20" placeholder="상세 주소">
        <input type="button" onClick="openDaumZipAddress();" value = "주소 찾기" />
        <br/><br><br><hr><br>
        
        <input type="button" id="finish_btn" class="finish_button" name="finish" value="완료">
```

### **카카오 주소 API 연동하여 도로명 주소 가져오는 부분**
```javascript
        <script type="text/javascript">
            function openDaumZipAddress() {    
                new daum.Postcode({    
                    oncomplete:function(data) {    
                        jQuery("#input_address_road").val(data.postcode1);    
                        jQuery("#input_address_road").val(data.postcode2);    
                        jQuery("#input_address_road").val(data.zonecode);    
                        jQuery("#input_address_road").val(data.address);    
                        jQuery("#input_address_road").focus();    
                        console.log(data);    
                    }    
                }).open();    
            }    
        </script>
```

# **프로젝트 소감**

말로만 들어봤던 블록체인을 직접 구현해 본 것은 처음이었다. 그렇기에 팀원들도 많이 헷갈려 했고, 나도 많이 어려워했다. 특히, Solidity 언어는 일반 언어와는 다르게 Contract 지향 언어라서 기본적인 부분은 비슷할지라도, 깊숙히 구현할수록 점점 복잡해진다는 것을 깨달았다.

그래서 아쉬움이 남았던 프로젝트였지만, 모든 팀원들이 열심히 해 주었고, 팀장으로써 나를 믿고 열심히 따라와 줘서 매우 고마웠다. 그 결과, 경진대회 최우수상이라는 아주 값진 경험을 할 수 있도록 도와준 팀원들에게 다시 한 번 감사하게 생각한다.