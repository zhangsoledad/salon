---
layout: post
title:  "React getDefaultProps"
date:   2015-03-30 19:57:45
categories: react
---
用react时遇到的一点困惑
========

前几天在用react的时候出现了一个很神奇的问题，用下面小例子来重现下：

    var SelectBox = React.createClass({
      getDefaultProps: function() {
        return {
          value: []
        };
      },
      getInitialState: function() {
        return {checked: this.props.value,count: 1};
      },
      handleClick: function() {
        var opt = this.state.checked;
        opt.push({id: this.state.count, name: this.state.count});
        this.setState({checked: opt, count: this.state.count + 1});
      },
      render: function() {
        var that = this;
        var options = this.state.checked.map(item => <option key={'checked-' + that.props.name + item.id} value={item.id}>{item.name}</option>);
        return (
          <div>
            <select multiple={true}>
              {options}
            </select>
            <button type="button" onClick={this.handleClick}>add</button>
          </div>
        );
      }
    });
    React.render(
      <SelectBox name='one'/>,
      document.getElementById('one')
    );
    React.render(
      <SelectBox name='two'/>,
      document.getElementById('two')
    );


上面代码看起来没什么问题，运行一下看看。

先点击'one'的`button`，一切正常：

![click 'one' button](http://7fvk4m.com1.z0.glb.clouddn.com/FuolwoqBU7F8eGF69pYsUHrWHP6D)

再点击'two'的`button`:

![click 'two' button](http://7fvk4m.com1.z0.glb.clouddn.com/FiPIRZ75qZ_jmoPL-wtYw7WepSgx)

发现不对了，两个组件应该是相互独立的，却表现得像共享了state一样。

其实，问题出在`getDefaultProps`这个地方，官方文档是这样描述的：
>Invoked once and cached when the class is created. Values in the mapping will be set on this.props if that prop is not specified by the parent component (i.e. using an in check).

>This method is invoked before any instances are created and thus cannot rely on this.props. In addition, be aware that any complex objects returned by getDefaultProps() will be shared across instances, not copied.

`getDefaultProps`这个方法只会被调用一次，而且结果会被缓存起来，其返回的复杂对象，如上面代码中的`value`是个数组,
是传引用，而非复制，且在各个实例共享，这个地方不知道为什么要这样设计，有点全局变量的味道。
所以正确的写法应该是：


    getInitialState: function() {
      var tmp = this.props.value.concat();
      return {checked: tmp, count: 1};
    },
