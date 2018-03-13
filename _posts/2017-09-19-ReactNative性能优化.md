---
title: RN组件性能优化
date: 2017-09-19
categories:
- RN
tag: 
- react native
- stateless组件
---

### 调用stateless组件方式

写一个头像组件

```js
class Avator extends React.Component {
  render() {
    return <img src={this.props.url} />;
  }
}

```

一次优化，将其变成无状态组件

```jsx
const Avator = (props) => <img src={props.url} />;

ReactDOM.render(<Avator url='http://baidu.com' />, mountNode)

```

再次优化

```jsx
const Avator = (props) => <img src={props.url} />;

ReactDOM.render(Avator({ url: 'http://baidu.com' }), mountNode)

```
<!-- more -->
### 缓存






[stateless component](https://medium.com/missive-app/45-faster-react-functional-components-now-3509a668e69f)