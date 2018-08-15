# 浅谈react中组件懒加载


> 参考文章: https://doc.webpack-china.org/guides/lazy-load-react/


## 1. 导入懒加载组件

```jsx
import React from 'react';

class LazilyLoad extends React.Component {

  constructor() {
    super(...arguments);
    this.state = {
      isLoaded: false,
    };
  }

  componentDidMount() {
    this._isMounted = true;
    this.load();
  }

  componentDidUpdate(previous) {
    const shouldLoad = !!Object.keys(this.props.modules).filter((key)=> {
        return this.props.modules[key] !== previous.modules[key];
    }).length;
    if (shouldLoad) {
        this.load();
    }
  }

  componentWillUnmount() {
    this._isMounted = false;
  }

  load() {
    this.setState({
      isLoaded: false,
    });

    const { modules } = this.props;
    const keys = Object.keys(modules);

    Promise.all(keys.map((key) => modules[key]()))
      .then((values) => (keys.reduce((agg, key, index) => {
        agg[key] = values[index];
        return agg;
      }, {})))
      .then((result) => {
        if (!this._isMounted) return null;
        this.setState({ modules: result, isLoaded: true });
      });
  }

  render() {
    if (!this.state.isLoaded) return null;
    return React.Children.only(this.props.children(this.state.modules));
  }
}

LazilyLoad.propTypes = {
  children: React.PropTypes.func.isRequired,
};

export const LazilyLoadFactory = (Component, modules) => {
  return (props) => (
    <LazilyLoad modules={modules}>
      {(mods) => <Component {...mods} {...props} />}
    </LazilyLoad>
  );
};

export const importLazy = (promise) => (
  promise.then((result) => result.default)
);

export default LazilyLoad;
```

> 暴露出一个组件及其高阶组件接口的 LazilyLoad 组件模块的源代码（如下），（里面有一个）让 ES2015 的定义更自然的 importLazy 函数。



