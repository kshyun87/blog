---
layout: post
title: React State and Lifecycle
---

지금까지 우리는 단방향으로 UI를 변경(update0하는 방법을 배웠습니다.
지금까진 만든 컴포넌트는 전달 받은 props를 이용해서 출력하도록 만드러었던 것이죠.
그런데 변수를 선언하거나 해야하는 경우는 어떻게 할까요>
 이때 필요한게 state입니다.

시계 컴포넌트를 만들어 보겠습니다.
이전에 사용했던 시간 컴포넌트입니다.
```JSX
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}
setInterval(tick, 1000);
```

이제 분리시켜서 시계 컴포넌트를 만들어보곘습니다.
```JSX
function Clock(props){
    return(
        <div>
            <h1>Hello, world!</h1>
            <h2>It is {props.date.toLocaleTimeString()}.</h2>
        </div>
    );
}
function tick(){
    ReactDOM.render(<Clock date={new Date()} />, document.getElementById('root'));
}
setInterval(tick, 1000);
```
이렇게만 해도 사용하는데는 문제 없죠.
그런데 말입니다.
굳이 Clock 컴포넌트를 호출할때 date를 넘겨줘야할까요?
그냥 불러오면 알아서 되면 안될까요?
예를 들면 <Clock /> 엘러먼트만 사용하면 시간이 출력되게 만들고 싶다면요?

이럴때 사용되는 것이 state이고 이 state가 functional이 아닌 classes로 정의된 컴포넌트를 쓰는 이유중 하나입니다.

일단 functional을 classes로 바꿔보죠
```JSX
class Clock extends ReactDOM.Component {
    render(){
        return (
            <div>
                <h1>Hello, world!</h1>
                <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
            </div>
        )
    }
}
```
구조 자체는 크게 변한 느낌이 없죠?
함수가 그냥 클래스가 된 느낌 정도입니다.
다만 props가 this.props로 변경되었습니다.

이렇게 클래스로 사용하면 몇가지 이점을 제공합니다.
local state나 lifecycle hook가 그 이점입니다.

Adding local state to a class
먼저 this.props를 this.state로 바꿉니다.
```JSX
class Clock extends Component {
    constructor(props){
        super(props);
        this.state = {date: new Date()};
    }

    render(){
        return (
            <div>
                <h1>Hello, world!</h1>
                <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
            </div>
        )
    }
}
```
constructor라는 구조체를 하나 선언해줍니다.
내용을 살피면, super()메소드로 props의 내용을 상속받습니다.
그리고 this.state에 date를 선언합니다.
this.state는 상속받은 props의 내용도 포함하면서 추가적으로 state로 선언한 date도 가진 상태 입니다.

그리고 lifecycle을 지정해줍니다.
```JSX
componentDidMount(){
}

componentWillUnmount(){
}
```

위 두개 메소드가 lifecylce관련된 메소드 입니다.
componentDidMount()는 컴포넌트 DOM으로 렌더링된 이후 동작합니다.
```JSX
componentDidMount() {
        this.timerID = setInterval(
          () => this.tick(),
          1000
        );
    }
```

인터벌 함수로 tick() 호출하도록 만들었습니다.
그런데 여기서 this는 모두 state입니다.
그럼 this.props와 this.state의 차이는 무엇일까요?
this.props는 React 자체에서 설정되고
this.state는 특별한 의미가 있지만 화면에 출력하는 것에 사용되지 않는 것을 저장해야 하는 경우, 클래스에 수동으로 필드를 추가할 수 있습니다.
간단히 말하자면, 이전에 우린 props로 설정된건 수정할 수 없다고 했었죠? 그그러나 state는 가능하다 라고 생각하면 될 것 같습니다.
클래스의render()에서 사용되는 props는 this.props라면 그대로 가져와서 사용만 하는 것이고 this.state는 추가적으로 더 정의하여 사용할 수 있다는 것입니다.
그리고 render()에서 사용되지 않는 것이라면 state로 사용할 수 없습니다.

```JSX
class Clock extends Component {
    constructor(props){
        super(props);
        this.state = {date: new Date()};
    }
    componentDidMount() {
        this.timerID = setInterval(
          () => this.tick(),
          1000
        );
    }

    componentWillUnmount() {
        clearInterval(this.timerID);
    }
    tick() {
        this.setState({
          date: new Date()
        });
      }
    render(){
        return (
            <div>
                <h1>Hello, world!</h1>
                <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
            </div>
        )
    }
}
```
componentWillUnmount()는  생명주기가 다된 타이머를 종료시킵니다.

마지막으로 tick()메소드는 this.setState()라는 메소드로 this.state값을 설정(변경)해줍니다.
즉 랜더링이 완료되면 componentDidMount()가 동작하는데 그 안에 있는 SetInsterval로 tick()을 1초마다 호출합니다.
tick()이 호출되면 this.state의 date값을 그 시간으로 변경해주고 화면에서도 그 시간으로 업데이트가 이루어져서 시계처럼 동작하게 되는 것입니다.

주의!
this.state.date = new Date();
위 방법과 같이 직접적으로 수정하면 안됩니다.

this.setState({date:new Date()});
위와 같이 setState()를 이용해서 값을 변경해야합니다.
그 이유는 setState()는 this.state를 할당하는 유일한 생성자이기 때문입니다.

이부분은 JS와는 매우 다른 모습이네요. 보통 JS에서는 직접적으로 변경해도 괜찮았는데 말이죠.

그럼 여기서 props와 state의 관계는 props가 부모고 state가 자식처럼 느껴질 것이다.
props에 있는 값을 자식에게 전달 전달 하는 방식이기에 top-down방식의 데이터 플로우이다.

React앱에서, 구성 요소가 상태 저장인지 또는 상태 비 저장인지는 시간이 지남에 따라 변경될 수 있는 구성 요소의 구현 세부 사항으로 간준된다.
Stateful 컴포넌트에서 stateless 컴포넌트를 사용할 수 있으며 그 반대의 경우도 마찬가지다.
```JSX
function App(){
    return (
        <div>
            <Clock />
            <Clock />
            <Clock />
        </div>
    )
}
```
ReactDOM.render(<App />, document.getElementById('test'));
이렇게 여러 Clock을 사용하더라도 각각은 독립적으로 동작한다.
