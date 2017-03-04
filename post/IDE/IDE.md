## [轉載] Rain & Water Effect Experiments [Back](./../post.md)

> - Author: [sskyy](https://github.com/sskyy)
- Origin: http://www.cnblogs.com/sskyy/p/6496287.html
- Time: Mar, 3rd, 2017

### 引言

我有个非常犀利的朋友，在得知我要去做可视化的页面搭建工具时问了我一个问题： “你自己会用这样的工具吗？” 同时带着意味深长的笑。

然而这个问题并没有如他所愿改变我的想法。早在 jquery ui、bootstrap 盛行的时代，就有过无数这样的工具，我没有用过，也不会去用。原因有一万个:

- 业务需求太灵活，工具都是基于已有的组件库，个性化的东西搭不出来。
- 前端技术发展太快，工具整合没有那么迅速。
- 有学习成本，不如手写快，灵活性没有手写高。 

在包括我的很多前端看来，这条路上尸骨累累，甚至有很多连痕迹都没有留下。但是失败者最多的路，并不一定是死路。如果都没有抛开过头脑里的成见，没有进行过独立思考就放弃了，未免太盲目。这篇文章就当做我在求生之路上的记录。也请读者暂且忘掉所有的经验，轻装上阵，这趟旅途不会让你失望。对具体设计不感兴趣的读者可以直接阅读《生门》一章，读完那一章后或许你会迫不及待再从头读起。

### 起点和方向

在接下来的两章中，我们将从项目背景一直讨论到关键技术的实践。这其中既会包括各种技术也会包括产品和交互的思考。 项目的背景是，公司业务迅速扩张，有大量对内的系统页面需要搭建。而前端人力是瓶颈，所以我们希望能以服务化的方式输出前端能力，让公司内所有非前端出身但有编程能力的人都能使用这种服务快速地开发出较高质量的页面。

从产品角度来说，它的目标已经很明确了：

- 使用人群：非前端的开发者 
- 要提供的服务：能以中上等开发速度开发出中上等可维护性页面的集成开发环境(以下简称开发环境) 

有了这个目标，我们就可以开始设计产品形态了。

页面分为视图和逻辑两部分，在目前组件化的大背景下，视图基本上可以等同于组件树。首先，什么样的页面编辑方式学习成本最低同时最快速？当然是所见即所得，拖拽或者编辑树型结构的数据这两种方式都可以接受。实际测试中拖拽最容易上手，熟悉了快捷键的情况下则编辑组件树更快。

接着，怎样让用户编写页面逻辑既能学习成本低，又能保障质量？学习成本低意味着概念要少，或者都是用户已知的概念。保障质量这个概念比较大，我们可以从开发的两个阶段来考虑：

- 一是开发时最好有保障，例如前端开发时的 eslint 加上编辑器提示就能很好地提前避免一些低级错误。
- 二是在开发完之后，代码应该“有迹可循”，无论是排查问题，还是扩展需求，都要让用户在头脑里第一时间就知道应该怎样写逻辑，写在哪里。也就意味着概念要完善，职责分明。同时，工具层面也可以有些辅助功能，例如传统编辑器的变量搜索等。

为了给读者一个更直观的影响，我们暂且来看一张两张图。

页面编辑:

![页面编辑](https://zos.alipayobjects.com/rmsportal/blelrkgUdVBVTucDuuoT.png)

逻辑编辑:

![逻辑编辑](https://zos.alipayobjects.com/rmsportal/sgarfasRwViOjDvvePUv.png)

接下来分部分细化形态，梳理关系，来得到一个明确的架构图。

目前看来可先拆分成三个部分:

- 一个编辑页面和逻辑的工具，以下暂称 IDE。 - 搭建页面所需的基础组件。
- 运行时框架(以下简称框架)，由它将页面的组件树、和页面逻辑结合在一起渲染成最终的页面。

很容易发现这三者的关系并不是平行的。首先，IDE 在这三者中是直接给用户使用的产物，它代表着我们最终想要呈现给用户什么样的东西。对其他部分来说，它算是需求来源。

来看它和页面以及组件的的关系。我们最终希望用户在点击页面上的某个组件或者组件树上的节点时，就能查看、配置这个组件上的属性，逻辑绑定到它触发的事件上。

![组建与属性](https://zos.alipayobjects.com/rmsportal/uMMIizInErTFLUCCSRgQ.png)

因此它对组件的需求是：组件必须暴露出自己的所有属性和事件，让外部可读。

再看 IDE 和框架的关系。用户在编写逻辑时，需要理解的概念都是属于框架的，IDE 只是编辑工具。当然 IDE 可以提供很多辅助功能，例如语法校验，例如可视化地展示逻辑与组件的绑定关系。框架为主，IDE 为辅。

最后，框架和组件的关系。这里很有意思，按技术发展的现状来说，一直都是先有组件库，才有上层应用框架。然而，组件规范其实应该是应用框架规范的一部分。举个实际例子，如果应用框架要建立全局数据源(方便做回滚等高级功能)，来保存所有状态。那么组件就不再需要内部状态，只要渲染就够了，实现上简单很多。这种上层建筑与基础设施的关系，很像高楼与砖瓦。摩天大楼需要钢筋混凝土，负责烧土砖的工人一开始是想不到的。所以实施中，框架和组件库之间通常还会有适配层。优秀的架构能力就体现在一开始就看到了足够多的上层需求，提前避免了发展中的人力损耗。

理清了所有关系后，来看看整体架构：

![整体架构](https://zos.alipayobjects.com/rmsportal/QRMIwUZyhvduxcXkxkOu.png)

这其中将 IDE 底层和业务层进行了拆分，IDE 底层提供窗口、快捷键、Tab 等常用功能，IDE 上业务层才用来处理和可视化相关的内容。其中也包括为了提供更好体验，却又不适合放到组件、和应用框架中的胶水代码，例如组件属性的说明，示例等等。IDE 的架构设计将会在另一篇文章中介绍。

### 龙骨

整体的架构有了后，接下来就是关键技术——运行时框架的设计了。 在数据驱动的大背景下，应用框架处理的问题实际上只有一个：数据管理。其中“数据”既包括组件数据也包括业务数据，而“管理”既包括如何保存数据，也包括以何种方式让用户来读写数据。我们仍然从使用场景出发，来分析出数据管理的应用场景，最后再考虑设计实现。在前端领域内，用户对交互的需求是渐进增长的，业务的需求是渐进的，因此应用的复杂度整体看来也是渐进的。所以我们只需要明确出最简单和最复杂的情况，就可以勾勒出框架需要支持的范围了：

- 按业务经验，最简单的情况无非就是纯展示的“详情页” 或 “列表页”。最符合本能的逻辑写法应该是:
- 拼好组件树。如果是静态的数据，直接在每个组件上的属性里设置好即可，流程结束。 
- 如果是动态数据，那么使用 ajax 获取到数据。将获取的数据格式化成组件所能接受的格式，然后使用 api set 进去。

在这个场景中用户需要了解两件事情:

- 组件的数据格式，实际上就是组件的属性，在 IDE 中已经是直接暴露出来的。
- 设置数据的 api 。 再接着看最复杂的场景，我所接触过的最复杂的前端应用都是业务关联极强的工具，例如云计算平台的控制台，客服系统的控制台，包括这个 IDE 也算。

这类产品的复杂体现在两个方面:

- 有大量的交互细节，例如组件状态要和权限结合(例如 按钮的 disable 状态)、组件要根据需求动态显示或隐藏，表单的校验，异步状态的提示或管理(例如发送请求后，按钮上出现loading)。
- 除了组件数据，还有大量的业务数据要管理，并且是其中有很多联动关系。例如在云计算控制台里面有 ECS、LBS 等概念，ECS 和 LBS 有关联关系，ECS如果改名了。不仅要更新ECS自己的详情显示，还要自动更新关联的LBS的显示等。

有了这两个端点，就找到了要提供的能力的上限和下限，接下来就是框架设计中最有意思也最困难的部分了——如何提供渐进式地开发体验。这几乎也是所有优秀框架的共有的一个品质。渐进式的体验意味着用户只要了解最基本的功能就能马上开始工作，当要处理更高级的需求时才需要再学习高级的功能。更进一步话，最好这些高级功能也是用一种可扩展的机制来实现的，如中间件，学习一次机制，即可解决无限的问题。

在最简场景里可以看到，用户所需的最基本的功能就是一个可读写的，包含所有组件数据的数据源即可(以下简称组件数据源)。为了便于让用户理解，这个数据源的数据格式最好与组件树存在类似的对应关系。举个注册页面的例子，我们的组件树可能长这样:

```html
<div>
    <title>注册</title>
    <input label="姓名">
    <input label="密码" type="password">
    <button text="提交"></button>
</div>
```

那么组件数据源可表述为:

```json
{
    0: { text: '注册', size: 'large' },
    1: { value: '', label: '姓名', type: 'text' },
    2: { value: '', label: '密码', type: 'password' },
    3: { text: '提交', type: 'normal' }
}
```

用户的读写操作可以设计成这样:

```js
// 借用 redux 中的 store 作为数据源的名字
store.get('1.value') // 读取第一个 Input 的值
store.set('3.type', 'loading') // 将 Button 设为 loading 状态
```

这个写法可以实现需求，但有两个问题:
- 用组件的位置作为索引不友好，不能适应变化。例如组件的位置调整了一下顺序，代码里就得相应改动。
- 在用户的业务逻辑中，并不是所有组件的数据用户都需要，例如Title。

为何不让用户自己给想要数据的组件取名？这可以一次性解决这两个问题。

```html
<div>
    <title>注册</title>
    <input bind="name" label="姓名">
    <input bind="password" label="密码" type="password">
    <button bind="submit" text="提交"></button>
</div>
```

得到的数据源:

```json
{
    name: { value: '', label: '姓名', type: 'text' },
    password: { value: '', label: '密码', type: 'password' },
    submit: { text: '提交', disabled: false }
}
```

再看看用户的提交逻辑如何写(这个逻辑绑定在 Button 的 onClick 事件上):

```js
// 通过注入的方式把数据源管理对象交给用户
function({store}) {
    store.set('submit', {disabled: true}) // 为了防止重复提交
    ajax({
        url : 'xxx',
        data: {
            name: store.get('name').value,
            password: store.get('password').value
        }
    }).finally(() => {
        store.set('submit', {disabled: false})
    })
}
```