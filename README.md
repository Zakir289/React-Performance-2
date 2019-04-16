# React-Performance-2

  Please go through React Performance part 1 as well. 
 
1.Should React Update The Component?
2.Debounce Input Handlers
3.Memoize React Components
4.Avoid Async Initialization in componentWillMount()
5.Using Optimizing tools like  why-did-you-update
6.Beware of Object Literals in JSX
7.Problems with mutating the state directly 
---
### 1.Should React Update The Component?
when the application is growing, attempting to re-render and compare the entire virtual DOM at every action will eventually slow down.
React provides a weapon to take a call whether to update the component or not. This is where the shouldComponentUpdate method comes into play.
```
function shouldComponentUpdate(nextProps, nextState) {
    return true;
} 
```

When this function returns true for any component, it allows the render-diff process to be triggered.
This gives the you a simple way of controlling the render-diff process. Whenever you need to prevent a component from being re-rendered at all, simply return false from the function. Inside the function, you can compare the current and next set of props and state to determine whether a re-render is necessary:
```
function shouldComponentUpdate(nextProps, nextState) {
    return nextProps.id !== this.props.id;
}
```

##### Using a React.PureComponent
To ease and automate a bit this optimization technique, React provides what is known as “pure” component. A React.PureComponent is exactly like a React.Component that implements a shouldComponentUpdate()function with a shallow prop and state comparison.
A React.PureComponent is more or less equivalent to this:
```
class MyComponent extends React.Component {
    shouldComponentUpdate(nextProps, nextState) {
        return shallowCompare(this.props, nextProps) && shallowCompare(this.state, nextState);
    }
    …
}
```
It performs shallow comparison, So, It works perfectly with primitive data types, when coming to objects unless you avoid mutating the object, you can’t take full use of object. 
This will cause the render() function to create a new function on every render. 
 ---
### 2. Debounce Input Handlers
This concept is not specific to React, or any other front-end library.
 Debouncing has long been used in JavaScript to successfully run expensive tasks without hindering performance.

 A debounce function can be used to delay certain events so that it doesn’t get fired up every millisecond. This helps you limit the number of API calls, DOM updates, and time consuming tasks. 
For instance, when you’re typing something into the search menu, there is a short delay before all of the suggestions pop up. The debounce function limits the calls to onChange event handler.
 In many cases, the API request is usually made after the user has stopped typing.
Let me focus more on the debouncing strategy for input handlers and DOM updates. Input handlers are a potential cause of performance issues.

 Let me explain how:

##### Without Debounce

```
export default class SearchBar extends React.Component {
  constructor(props) {
    super(props);
    this.onChange = this.onChange.bind(this);
  }
  onChange(e) {
    this.props.onChange(e.target.value);
  }
  render () {
    return (
      <label> Search </label>
      <input onChange={this.onChange}/>
    );
  }
}
 ```
On every input change, we’re calling this.props.onChange(). What if there is a long-running sequence of actions inside that callback? That’s where debounce comes it handy. 
##### With Debounce:
```
import {debounce} from 'throttle-debounce';

export default class SearchBarWithDebounce extends React.Component {
  constructor(props) {
    super(props);
    this.onChange = this.onChange.bind(this);
    this.onChangeDebounce = debounce( 300, 
         value => this.props.onChange(value) //onChange will be called only once for 300 ms
    );
  }
  onChange(e) {
    this.onChangeDebounce(e.target.value);
  }
  render () {
    return (
      <input onChange={this.onChange}/>
    );
  }
}
 ```
The DOM update happens after the user has finished typing, which is exactly what we need. If you’re curious about debouncing AJAX calls, you can do something like this:
```
import {debounce} from 'throttle-debounce';

export default class Comp extends Component {
constructor(props) {
    super(props);
    this.callAPI = debounce(500, this.callAPI);onChange will be called only once for 500 ms
  }
  onChange(e) {
    this.callAPI(e.target.value);
  }
  callAjax(value) {
    console.log('value :: ', value);
    // AJAX call here
  }
  render() {
    return (
      <div>
        <input type="text" onKeyUp={this.onChange.bind(this)}/>
      </div>
    );
  }
}

```
---
### 3. Memoize React Components
Memoization is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls and returning the cached result when the same inputs occur again.
 A memoized function is usually faster because if the function is called with the same values as the previous one then instead of executing function logic it would fetch the result from cache.

Let's consider below simple stateless UserDetails React component.
```
const UserDetails = ({user, onEdit}) => {
    const {title, full_name, profile_img} = user;

    return (
        <div className="user-detail-wrapper">
            <img src={profile_img} />
            <h4>{full_name}</h4>
            <p>{title}</p>
        </div>
    )
}
 ```
Here, all the children in UserDetails are based on props. This stateless component will re-render whenever props changes. If the UserDetailscomponent attribute is less likely to change, then it's a good candidate for using the memoize version of the component:
```
import moize from 'moize';

const UserDetails = ({user, onEdit}) =>{
    const {title, full_name, profile_img} = user;

    return (
        <div className="user-detail-wrapper">
            <img src={profile_img} />
            <h4>{full_name}</h4>
            <p>{title}</p>
        </div>
    )
}

export default moize(UserDetails,{
    isReact: true
}); 
```
This method will do a shallow equal comparison of both props and context of the component based on strict equality.


If you are using React V16.6.0 or greater version, then you can use React.memo and rewrite the above code like:
```
const UserDetails = ({user, onEdit}) =>{
    const {title, full_name, profile_img} = user;

    return (
        <div className="user-detail-wrapper">
            <img src={profile_img} />
            <h4>{full_name}</h4>
            <p>{title}</p>
        </div>
    )
}

export default React.memo(UserDetails)
```
---
### 4. Avoid Async Initialization in componentWillMount()
componentWillMount() is only called once and before the initial render. Since this method is called before render(), our component will not have access to the refs and DOM element.
Here’s a bad example:
```
function componentWillMount() {
  axios.get(`api/comments`)
    .then((result) => {
      const comments = result.data
      this.setState({
        comments: comments
      })
    })}
```

Let’s make it better by making async calls for component initialization in the componentDidMount lifecycle hook:
```
function componentDidMount() {
  axios.get(`api/comments`)
    .then((result) => {
      const comments = result.data
      this.setState({
        comments: comments
      })
    })
} 
```
The componentWillMount() is good for handling component configurations and performing synchronous calculation based on props since props and state are defined during this lifecycle method

---
### 5.Using Optimizing tools like  why-did-you-update

The why-did-you-update library detects unnecessary component renders.

 Specifically, it points out instances where a component’s render method gets called even though no changes have occurred. 

A great way to monitor your application for avoidable renders is to install an npm package called why-did-you-update which will monitor and log avoidable component renders straight into your browser console at runtime.

### 6.Beware of Object Literals in JSX
Once your components become more "pure", you start detecting bad patterns that lead to useless rerenders.
 The most common is the usage of object literals in JSX, which is "The infamous {{". Let me give you an example:
 ```
import React from 'react';
import MyTableComponent from './MyTableComponent';

const Datagrid = (props) => (
    <MyTableComponent style={{ marginTop: 10 }}>
        ...
    </MyTableComponent>
)
```

The style prop of the <MyTableComponent> component gets a new value every time the <Datagrid> component is rendered. So even if <MyTableComponent> is pure, it will be rendered every time <Datagrid> is rendered.

 In fact, each time you pass an object literal as prop to a child component, you break purity. The solution is simple:
```
import React from 'react';
import MyTableComponent from './MyTableComponent';

const tableStyle = { marginTop: 10 };
const Datagrid = (props) => (
    <MyTableComponent style={tableStyle}>
        ...
    </MyTableComponent>
)
```
This looks very basic,You can see this mistake so many times that I've developed a sense for detecting the infamous {{ in JSX. I routinely replace it with constants.

Another usual suspect for hijacking pure components is React.cloneElement(). If you pass a prop by value as second parameter, the cloned element will receive new props at every render.
```
// bad
const MyComponent = (props) => <div>{React.cloneElement(Foo, { bar: 1 })}</div>;

// good
const additionalProps = { bar: 1 };
const MyComponent = (props) => <div>{React.cloneElement(Foo, additionalProps)}</div>;

```
### 7.Problems with mutating the state directly 
So what’s wrong with mutating state directly? 

Let’s say we overwrite shouldComponentUpdate and are checking nextState against this.state to make sure that we only re-render components when changes happen in the state.
```
shouldComponentUpdate(nextProps, nextState) {
    if (this.state.users !== nextState.users) {
      return true;
    }
    return false;
  }
```
Even if changes happen in the user's array, React won’t re-render the UI as it’s the same reference.
The easiest way to avoid this kind of problem is to avoid mutating props or state. 


The explanation is already given in Should Component Update. 


