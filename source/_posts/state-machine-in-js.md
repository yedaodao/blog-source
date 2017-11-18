---
title: Javascript中的状态机
date: 2017-02-02 23:14:45
categories: concepts
---

# Javascript中的状态机

状态机是在现代计算机领域运用极其广泛的一种编程模式。它将事物的变化抽象为离散有限个状态，并且认为事物产生变化是由于接收了事件，而事件的产生是由于事物此时已经满足了发生状态变化的条件。而状态机所管理的状态一定是有限个的，并且至少有一个状态为终态的，当状态机的当前状态为终态时，该状态机停止运行。

状态机分为两种类型：

- Moore状态机：输出与输入无关，只与状态有关。
- Mealy状态机：输出与输入及状态均有关。

## 状态机组成

在笔者看来由高级语言实现的状态机包含以下几个概念：

- State：对象在其生命周期中所处的阶段，由对象此时满足的条件所决定。
- Action：条件满足后，对象需要执行的任务。但Action不是必须的，可以在执行动作后迁移到新状态，也可不执行任何任务，直接迁移状态。
- Transition：定义状态迁移的条件。
- Guards：用于检测是否满足状态迁移的条件。
- Event：引起状态迁移的原因。

初步概括状态机的执行流程如下：

{% asset_img state-machine-flow.jpg %}

## 状态机实现

笔者在这里使用Nodejs编写，我们需要一个Factory创建状态机，实现代码如下：

```javascript
var _ = require('lodash');

var StateMachineFactory = (function () {
    var exports = {
        create: create
    };

    // 状态机类
    var StateMachine = function () {
        this.states = {};
        this.transitions = {};
        this.currentState = '';
        this.allowEvents = [];
    };

    // 激活事件，触发状态迁移
    StateMachine.prototype.emit = function (evt) {
        var self = this,
            transition = this.transitions[evt];
        if (!transition || this.currentState !== transition.from) {
            return this;
        }
        if (_.isFunction(transition.action)) {
            // 传入异步回调函数，完成Action后调用该函数设置状态
            transition.action.call(this, function () {
                self.currentState = transition.to;
            });
        } else {
            this.currentState = transition.to;
        }
        return this;
    };

    // 创建状态机实例
    function create(specObj) {
        if (!_.isPlainObject(specObj)) {
            throw new Error('First arg must be a plain obj');
        }
        var sm = new StateMachine();
        sm.currentState = specObj.initial;
        if (!specObj.states || !Object.keys(specObj.states).length) {
            throw new Error('The state property is unset');
        }
        sm.states = specObj.states;
        // 确保transition正确性
        _.forEach(specObj.transitions, function (val, key) {
            if ((sm.states[val.from]) && (sm.states[val.to])) {
                sm.allowEvents.push(key);
                val.action = sm.states[val.to];
                sm.transitions[key] = val;
            }
        });
        return sm;
    }

    return exports;
}());

var state = StateMachineFactory.create({
    initial: 'stateA',
    states: {
        'stateA': function (next) {
            console.log('to stateA');
            next();
        },
        'stateB': function (next) {
            console.log('to stateB');
            next();
        },
        'stateC': function (next) {
            console.log('to stateC');
            next();
        }
    },
    transitions: {
        'A': {from: 'stateC', to: 'stateA'},
        'B': {from: 'stateA', to: 'stateB'},
        'C': {from: 'stateB', to: 'stateC'}
    }
});

state.emit('B');
console.log(state.currentState);
state.emit('C');
console.log(state.currentState);
state.emit('A');
console.log(state.currentState);
```

这就是一个简单的状态机雏形了，我们可以定义状态，定义状态迁移的条件和规则，通过状态机，我们可以将多变复杂的UI逻辑整理清楚，归类放置。






