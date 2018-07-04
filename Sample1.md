- 자바스크립트의 비동기 처리 방식의 문제점 해결을 위해 콜백함수 사용

  ~~~javascript
  function getData() {
  	var tableData;
  	$.get('https://domain.com/products/1', function (response) {
  		tableData = response;
  	});
  	return tableData;
  }
  
  console.log(getData()); // undefined
  
  ~~~

  ~~~javascript
  function getData(callbackFunc) {
  	$.get('https://domain.com/products/1', function (response) {
  		callbackFunc(response); // 서버에서 받은 데이터 response를 callbackFunc() 함수에 넘겨줌
  	});
  }
  
  getData(function (tableData) {
  	console.log(tableData); // $.get()의 response 값이 tableData에 전달됨
  });
  ~~~

- 콜백지옥의 해결책 => Promise 또는 Async

  

- ### [Promise](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise)

  ~~~javascript
  function getData(callback) {
    // new Promise() 추가
    return new Promise(function (resolve, reject) {
      $.get('url 주소/products/1', function (response) {
        // 데이터를 받으면 resolve() 호출
        resolve(response);
      });
    });
  }
  
  // getData()의 실행이 끝나면 호출되는 then()
  getData().then(function (tableData) {
    // resolve()의 결과 값이 여기로 전달됨
    console.log(tableData); // $.get()의 reponse 값이 tableData에 전달됨
  });
  ~~~

- promise의 3가지 상태(states)

  - Pending (대기) : 비동기 처리 로직이 아직 완료되지 않은 상태

    ~~~javascript
    //new Promise()를 호출하면 대기 상태
    new Promise();
    ~~~

    ~~~javascript
    new Promise(function(resolve, reject){
        
    });
    ~~~

  - Fulfilled (이행) : 비동기 처리가 완료되어 프로미스가 결과값을 반환한 상태 

    ~~~javascript
    //콜백 함수의 인자 resolve를 실해하면 fulfilled상태가 된다.
    new Promise(function(resolve, reject){
        resolve();
    });
    ~~~

    ~~~javascript
    //이행상태가 되면 then()을 이용하여 처리 결과값을 받을 수 있다.
    function getData(){
        return new Promise(fuction(resolve,reject){
          var data = 100;
          resolve(data);                 
        });
    }
    
    getData().then(fucntion(resolvedData){
             console.log(resolvedData);   //100  
    });
    ~~~

  - Rejected (실패) : 비동기 처리가 실패하거나 오류가 발생한 상태

    ~~~javascript
    //new Promise 객체를 생성하면 콜백 인자로 resolve와 reject를 사용할 수 있다.
    //여기서 reject 인자로 reject() 메서드를 실행하면 Rejected(실패) 상태가 된다.
    
    new Promise(function(resolve, reject){
        reject();
    });
    ~~~

    ~~~javascript
    //실패 상태가 되면 실패한 이유(실패 처리의 결과값)를 catch()로 받을 수 있다. 
    function getData(){
        return new Promise(function(resolve, reject){
            reject(new Error("fail"));
        });
    }
    
    getData().then().catch(fucntion(err){
            console.log(err);             
    });
    ~~~



- ### Web RTC sample1 - [Basic getUserMedia demo](https://webrtc.github.io/samples/src/content/getusermedia/gum/)

  - `navigator.getUserMedia()`를 사용하면 플러그인 없이 웹캠과 마이크에 접근할 수 있다. 카메라에 접근하기 위해 무엇인가를 설치할 필요가 없다. 왜? 브라우저 안에 이미 포함되어 있으니까!

    

  - 입력장치의 접근권한 얻기

    - 웹캠과 마이크에 접근하기 위한 권한 요청.
    -  `getUserMedia()` 의 첫번째 인자 => 접근하려는 미디어 별 상세와 요구사항을 나타내는 객체

    ~~~javascript
    var constraints = window.constraints = {    //마이크는 사용하지 않고 카메라만 사용
      audio: false,  
      video: true
    };
    
    navigator.mediaDevices.getUserMedia(constraints).
        then(handleSuccess).catch(handleError);
    ~~~

    

  - `console` 창

    ~~~javascript
    //처음 웹사이트 들어갔을 때, pending(대기)상태 
    > navigator.mediaDevices.getUserMedia(constraints)
    
    Promise {<pending>}
      __proto__: Promise
      [[PromiseStatus]]: "pending"
      [[PromiseValue]]: undefined
    ~~~

    ~~~javascript
    //카메라 권한 허용 버튼 눌렀을 때
    //비동기 처리가 완료되어 프로미스가 결과값(MediaStream)을 반환한 상태 
    > navigator.mediaDevices.getUserMedia(constraints)
    
    Promise {<pending>}
      __proto__: Promise
      [[PromiseStatus]]: "resolved"   
      [[PromiseValue]]: MediaStream
    ~~~

    ~~~javascript
    //권한 허용 안할 경우
    //비동기 처리가 실패하거나 오류가 발생한 상태, 실패 처리 결과값 반환
    > navigator.mediaDevices.getUserMedia(constraints)
    
    Promise {<pending>}
    __proto__: Promise
    [[PromiseStatus]]: "rejected"
    [[PromiseValue]]: DOMException: Permission denied
    
    ~~~

    

  - `main.js`

    ~~~javascript
    function handleSuccess(stream) {
      var videoTracks = stream.getVideoTracks();
      console.log('Got stream with constraints:', constraints);
      console.log('Using video device: ' + videoTracks[0].label);
      stream.oninactive = function() {
        console.log('Stream inactive');
      };
      window.stream = stream; // make variable available to browser console
      video.srcObject = stream;  // add the MediaStream to the video element.
    }
    
    function handleError(error) {
       ...
    }
    
    navigator.mediaDevices.getUserMedia(constraints).
    then(handleSuccess).catch(handleError);
    ~~~

    

  



참고 : [자바스크립트 Promise 쉽게 이해하기](https://joshua1988.github.io/web-development/javascript/promise-for-beginners/),  [mdn docs](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise), 

​          [Capturing Audio and Video in HTML5](https://www.html5rocks.com/ko/tutorials/getusermedia/intro/#toc-webaudio-api),  [Google Developers - Web RTC](https://codelabs.developers.google.com/codelabs/webrtc-web/#3)





