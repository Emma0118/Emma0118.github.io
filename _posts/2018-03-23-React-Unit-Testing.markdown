---
layout: post
title: React 单元测试
date: 2018-03-23 12:15:09
---


### 主要是使用enzyme进行组件测试

#### 1.踩坑记录

其中，使用import引入enzyme-adapter-react-16时会出现no default export的错误，
问题在于 @type/enzyme-adapter-react-16的index.d.ts文件中没有导出 Adapter

修改为
```bash
import { EnzymeAdapter } from 'enzyme';

declare class Adapter extends EnzymeAdapter {
}

declare namespace Adapter {
}

export default Adapter;

```

#### 2.基本使用

(1) 在被测试文件中
```bash
import * as React from 'react';
import { configure as enzymeConfigure } from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';
import './Hello.css';

enzymeConfigure({ adapter: new Adapter() });
```

(2)在测试文件中
```bash
import * as enzyme from 'enzyme';
import Hello from './Hello';

it('renders the correct text when no enthusiasm level is given', () => {
    const hello = enzyme.shallow(<Hello name='Daniel' />);
    expect(hello.find(".greeting").text()).toEqual('Hello Daniel!')
});
```
#### 备注

需要安装对应版本的react react-dom 如果是使用typescript开发需要安装 @type对应的版本，需要安装react-addons-test-utils