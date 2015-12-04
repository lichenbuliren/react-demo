## React 入门练习实践总结

### 1、模板使用

``` html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <script src="../lib/react.js"></script>
        <script src="../lib/react-dom.js"></script>
        <script src="../lib/browser.min.js"></script>
        <title>react-demo06</title>
    </head>
    <body>
        <div id="example"></div>
        <script type="text/babel">

        </script>
    </body>
    </html>
```

<!-- more -->

### 2、JSX 语法

遇到 `HTML` 标签（以 < 开头），就用 `HTML` 规则解析；遇到代码块（以 { 开头），就用 `JavaScript` 规则解析。

``` javascript
    var names = ['Alice','Emily','Kate'];
    ReactDOM.render(
        <div>
        {
            names.map(function(name,index){
                return <div key={index}>Hello, {name}!</div>
            })
        }
        </div>,
        document.getElementById('example')
    );
```

### 3、自定义组件写法

`React` 允许将代码封装成组件（component），然后像插入普通 HTML 标签一样，在网页中插入这个组件。React.createClass 方法就用于生成一个组件类

``` javascript
    var HelloMessage = React.createClass({
        render: function(){
            return <h1>Hello {this.props.name}</h1>
        }
    });

    ReactDOM.render(
        <HelloMessage name="Heaven" />,
        document.getElementById('example')
    );
```


### 4、this.props.children

`this.props` 对象的属性与组件的属性一一对应，但是有一个例外，就是 `this.props.children` 属性。它表示组件的所有子节点。

``` javascript
    var NotesList = React.createClass({
        render: function(){
            return (
                <ol>
                {
                    React.Children.map(this.props.children, function(child,index){
                        return <li key={index}>{child}</li>;
                    })
                }
                </ol>
            );
        }
    });

    ReactDOM.render(
        <NotesList>
            <span>hello</span>
            <span>world</span>
        </NotesList>,
        document.getElementById('example')
    );
```


### 5、PropTypes

组件的属性可以接受任意值，字符串、对象、函数等等都可以。有时，我们需要一种机制，验证别人使用组件时，提供的参数是否符合要求。组件类的 PropTypes 属性，就是用来验证组件实例的属性是否符合要求

``` javascript
    var MyTitle = React.createClass({
        // 设置默认属性
        getDefaultProps: function(){
            return {
                title: 'Hello world'
            };
        },
        propTypes:{
            title: React.PropTypes.string.isRequired
        },
        render: function(){
            return <h1>{this.props.title}</h1>
        }
    });

    // var data = 124; 用数子会有报错，title属性为必须的且是string类型
    var data = 'hello world';
    ReactDOM.render(
        <MyTitle/>,
        document.getElementById('example')
    );
```


### 6、获取真实的 DOM 节点

组件并不是真实的 DOM 节点，而是存在于内存之中的一种数据结构，叫做虚拟 DOM （virtual DOM）。只有当它插入文档以后，才会变成真实的 DOM 。根据 React 的设计，所有的 DOM 变动，都先在虚拟 DOM 上发生，然后再将实际发生变动的部分，反映在真实 DOM上，这种算法叫做 DOM diff ，它可以极大提高网页的性能表现。但是，有时需要从组件获取真实 DOM 的节点，这时就要用到 ref 属性

``` javascript
    var MyComponent = React.createClass({
        handleClick: function(){
            this.refs.myTextInput.focus();
        },
        render: function() {
            return (
                <div>
                    <input type="text" ref="myTextInput"/>
                    <input type="button" value="Focus the text input" onClick={this.handleClick}/>
                </div>
            );
        }
    });

    ReactDOM.render(
        <MyComponent/>,
        document.getElementById('example')
    );
```

上面代码中，组件 MyComponent 的子节点有一个文本输入框，用于获取用户的输入。这时就必须获取真实的 DOM 节点，虚拟 DOM 是拿不到用户输入的。为了做到这一点，文本输入框必须有一个 ref 属性，然后 this.refs.[refName] 就会返回这个真实的 DOM 节点。
需要注意的是，由于 this.refs.[refName] 属性获取的是真实 DOM ，所以必须等到虚拟 DOM 插入文档以后，才能使用这个属性，否则会报错。上面代码中，通过为组件指定 Click 事件的回调函数，确保了只有等到真实 DOM 发生 Click 事件之后，才会读取 this.refs.[refName] 属性。


### 7、this.state

组件免不了要与用户互动，React 的一大创新，就是将组件看成是一个状态机，一开始有一个初始状态，然后用户互动，导致状态变化，从而触发重新渲染 UI。


``` javascript
    var LikeButton = React.createClass({
        getInitialState: function(){
            return {liked: false};
        },
        handleClick: function(event){
            this.setState({liked: !this.state.liked});
        },
        render: function() {
            var text = this.state.liked ? 'like': 'haven\'t liked';
            return (
                <p onClick={this.handleClick}>
                You {text} this. Click to toggle.
                </p>
            );
        }
    });

    ReactDOM.render(
        <LikeButton />,
        document.getElementById('example')
    );
```

上面代码是一个 LikeButton 组件，它的 getInitialState 方法用于定义初始状态，也就是一个对象，这个对象可以通过 this.state 属性读取。当用户点击组件，导致状态变化，this.setState 方法就修改状态值，每次修改以后，自动调用 this.render 方法，再次渲染组件。
由于 this.props 和 this.state 都用于描述组件的特性，可能会产生混淆。一个简单的区分方法是，this.props 表示那些一旦定义，就不再改变的特性，而 this.state 是会随着用户互动而产生变化的特性。


### 8、表单

用户在表单填入的内容，属于用户跟组件的互动，所以不能用 this.props 读取

``` javascript
    var Input = React.createClass({
        getInitialState: function(){
            return {value: 'Hello!'};
        },
        handleChange: function(event){
            this.setState({value: event.target.value});
        },
        render: function() {
            var value = this.state.value;
            return (
                <div>
                    <input type="text" value={value} onChange={this.handleChange}/>
                    <p>{value}</p>
                </div>
            );
        }
    });

    ReactDOM.render(<Input />,document.getElementById('example'));
```

上面代码中，文本输入框的值，不能用 this.props.value 读取，而要定义一个 onChange 事件的回调函数，通过 event.target.value 读取用户输入的值。textarea 元素、select元素、radio元素都属于这种情况。

### 9、组件的生命周期

组件的生命周期分成三个状态：

- Mounting: 已插入真实 DOM
- Updating：正在被重新渲染
- Unmounting：已移除真实 DOM

React 为每个状态都提供了两种处理函数，will 函数在进入状态之前调用，did 函数在进入状态之后调用，三种状态共计五种处理函数。

- componentWillMount()
- componentDidMount()
- componentWillUpdate(object nextProps, object nextState)
- componentDidUpdate(object prevProps, object prevState)
- componentWillUnmount()


此外，React 还提供两种特殊状态的处理函数。
- componentWillReceiveProps(object nextProps)：已加载组件收到新的参数时调用
- shouldComponentUpdate(object nextProps, object nextState)：组件判断是否重新渲染时调用

``` javascript
    var Hello = React.createClass({
        getInitialState: function(){
            return {
                opacity: 1.0,
                direction: true
            }
        },
        componentDidMount: function(){
            this.timer = setInterval(function(){
                var opacity = this.state.opacity;
                if(this.state.direction){
                    opacity -= 0.02;
                }else{
                    opacity += 0.02;
                }

                if(opacity < 0){
                    this.state.direction = false;
                }else if(opacity >= 1.0){
                    this.state.direction = true;
                }

                this.setState({
                    opacity: opacity
                });
            }.bind(this),100);
        },
        render: function() {
            return (
                <div style={{opacity: this.state.opacity}}>
                    Hello {this.props.name}
                </div>
            );
        }
    });

    ReactDOM.render(
        <Hello name="world"/>,
        document.getElementById('example')
    );
```

上面代码在 Hello 组件加载以后，通过 componentDidMount 方法设置一个定时器，每隔 100 毫秒，就重新设置组件的透明度，从而引发重新渲染。同时设置了一个方向变量来决定 opacity 是增加还是减少。

另外，组件的 `style` 属性的设置方式也值得注意，不能写成
``` css
    style="opacity: this.state.opacity"
```
而要写成

``` javascript
    style={{opacity: this.state.opacity}}
```

这是因为 [React 组件样式](https://facebook.github.io/react/tips/inline-styles.html)  是一个对象，所以第一重大括号表示这是 JavaScript 语法，第二重大括号表示样式对象。

### 10、Ajax

组件的数据来源，通常是通过 Ajax 请求从服务器获取，可以使用前面提到的 `componentDidMount`  方法设置 `Ajax` 请求，等到请求成功，再用 `this.setState` 方法重新渲染 UI。

``` javascript
    var UserGist = React.createClass({
        getInitialState: function(){
            return {
                username: '',
                lastGistUrl: ''
            };
        },

        componentDidMount: function(){
            $.get(this.props.source, function(result){
                var lastGist = result[0];
                if(this.isMounted()){
                    this.setState({
                        username: lastGist.owner.login,
                        lastGistUrl: lastGist.html_url
                    });
                }
            }.bind(this));
        },

        render: function() {
            return (
                <div>
                   {this.state.username} last gist is
                   <a href={this.state.lastGistUrl}>here</a>.
                </div>
            );
        }
    });

    ReactDOM.render(
        <UserGist source="https://api.github.com/users/octocat/gists"/>,
        document.getElementById('example')
    );
```