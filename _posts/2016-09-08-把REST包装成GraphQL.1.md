---
layout: post
title: "把REST包装成GraphQL.1"
date: 2016-09-08 22:10:20 +0800
image: 'blog-author.jpg'
description: '用更舒心的API'
main-class: 'FrontEnd'
color: '#66CCFF'
tags:
- Redux
- Relay
- React
- graphQL
categories: Review
twitter_text:
introduction: '介绍如何将现有的 RESTful API 包装成一个适合单页面应用的 GraphQL API'
---

# 偷偷在前端使用 GraphQL


## 摘要
GraphQL 是一个支持积极更新数据（Optimistic Update）、在 React 组件旁边声明式地取数据、数据所见即所得的数据层范式。由 Facebook 所推广的它，比起 RESTful API 有很多先进之处。本文以一个生产环境中的例子相伴，介绍了如何在不影响后端开发人员的情况下将现有的 RESTful API 包装成便于前端使用的 GraphQL API。

## 背景  
在一个从外包人员处接手的前端项目中，我见到了许多有趣的遗迹，例如页面间复制黏贴的无用 import、许多后缀名加上 .bak 但与另一个页面无甚区别的古老页面、富有童趣的组件命名等。其中对 API 的使用尤为令人动容，大概是这样子：  
```javascript
import * as Mapi from '../../lib/Mapi';

// ... 略去无关代码

  componentWillMount() {
    Mapi.districtPie(this.props.id)
      .then(json => {
        if (json.data.code === 0 && json.data.data !== undefined) {
          this.setState({
            piedata: json.data.data,
          });
        }
      });
  }
```
我遇到的第一个问题是数据经常需要在使用时检查它到底可不可用，错误码有时候是 -1 ，有时是返回一个重定向请求，这使得错误的定位比较消耗体力。  
还有当我需要多个数据时，我常常也开始从别的页面复制黏贴：  
```javascript
  componentWillMount() {
    Mapi.pie()
      .then(json => {
        if (json.data.code === 0) {
          this.setState({
            piedata: json.data.data,
          });
        }
      });
    Mapi.whoami()
      .then(json => {
        this.setState({ user: json.data.data });
      });
  }
```
因为这些数据在不同页面都有需求。  
此外的坑还有很多，虽然使用 Redux 等数据流框架能解决大半，但我最希望解决的其实是数据的可见性问题：  
「我希望在写 UI 的时候能看到数据长啥样，而且最好能和我 UI 的结构长得一模一样，让绑定数据到 UI 上的过程轻松愉快。」  
  
同时使用 Redux 和 GraphQL 解决了所有问题。  
  
##声明数据格式  
在 REST 中，我们按照业务将数据区分到不同的路径下，相当于给每一堆数据取一个通俗易懂的名字，希望在每个业务中使用一个：  
```javascript
export function whoami() {
  return apiget('/api/account/whoami');
}

export function pie() {
  return apiget('/api/data/index/pie');
}

export function entry() {
  return apiget('/api/info/entry');
}

export function districtPie(id) {
  return apiget(`/api/data/district/${id}/pie`);
}

export function siteOverview(id) {
  return apiget(`/api/data/site/${id}/overview`);
}

export function sitePie(id) {
  return apiget(`/api/data/site/${id}/pie`);
}

export function cabinetsSwitches(id) {
  return apiget(`/api/data/site/${id}/cabinets/switches`);
}
```
在 GraphQL 中，我们将数据表示成一棵树，并能直接请求其中的任何一片叶子。  
与 REST 类似的是我们也能起通俗易懂的名字，不同的是现在我们有了更小粒度的数据，类似于这样：  
```graphql

# /api/account/whoami
type UserType {
  logined: Boolean
  username: String
  password: String
  token: String

  id: Int!
  name: String

  companyId: Int
  companyName: String
  departmentId: Int
  departmentName: String
  role: String
}


# /api/info/entry  /api/data/site/{id}/overview
# 厂区信息，说明还有它有哪些变电站或子厂区，厂区有自己的饼图数据
# 同时也能表示变电站的数据，可以包括饼图和进线列表等数据
type PowerEntityType {
  id: Int!
  name: String!

  # ↓ 比较无关紧要的信息 
  address: String
  areaType: AreaType!
  coordinate: String
  companyId: Int
  districtId: Int
  siteId: Int
  gatewayID: Int

  alarmInfos(pagesize: Int, pageIndex: Int, orderBy: OrderByType, fromTime: String, toTime: String, filterAlarmCode: String): [AlarmInfoType]
  unreadAlarm: Int
  pie: PieGraphType
  infos(siteID: Int!): [InfoType] # 显示一些「本日最大负荷」、「本月最大负荷」、「告警数量」等信息
  wires(siteID: Int!): [WireType]
  cabinets(siteID: Int!): [CabinetType]
  children(areaType: AreaType, id: Int): [PowerEntityType]
}
```
我们可以从注释中看到每个数据类型和原有的 REST 端点之间的关系，一个数据类型可以由多个原有的 RESTful 数据源拼凑而来（例如集成多个微服务），也可以根据常识把一个 REST 数据源拆成多个数据类型，以方便三个月后重读这份代码的自己理解。  
  
这样变换之后有两个直接好处: 一是你能直接看到数据源长啥样了！再也不用因为记不清每个数据端点有哪些字段而不停地用 Postman 发请求抓数据了。 二是它提供了静态类型检查，你能在写类型的时候想好每个类型都应该是什么样，然后把检查数据可用性的工作抛开，把精力集中在业务上……  
  
上面我们用于声明类型的语言就是 GraphQL，一般写在一个 schema.js 文件中，大概长这样：  
```javascript
// schema.js
export const typeDefinitions = `schema {
# /api/account/whoami
type UserType {
  logined: Boolean
  username: String
  password: String
  token: String

  id: Int!
  name: String

  companyId: Int
  companyName: String
  departmentId: Int
  departmentName: String
  role: String
}
# ...略去其他类型声明
`
```
它写在一个首行放在第一行的多行文本内，以方便看行号 debug，并 export 出去。  
在写完数据类型声明后，我们还要在 schema.js 中说明每个字段都要怎么获取：  
```javascript
// schema.js
export const resolvers = {
  // ...略去其他解析函数，代码以实物为准
  UserType: {
    logined({ token }, args, context) {
      return context.User.getLoginStatus(token);
    },
    username({ token }, args, context) {
      return context.User.getUserName(token);
    },
    password({ token }, args, context) {
      return context.User.getPassWord(token);
    },
    token({ token }, args, context) {
      return token;
    },
    id({ token }, args, context) {
      return context.User.getMetaData('id', token);
    },
    name({ token }, args, context) {
      return context.User.getMetaData('name', token);
    },
    companyId({ token }, args, context) {
      return context.User.getMetaData('companyId', token);
    },
    companyName({ token }, args, context) {
      return context.User.getMetaData('companyName', token);
    },
    departmentId({ token }, args, context) {
      return context.User.getMetaData('departmentId', token);
    },
    departmentName({ token }, args, context) {
      return context.User.getMetaData('departmentName', token);
    },
    role({ token }, args, context) {
      return context.User.getMetaData('role', token);
    },
  },
  // ...略去其他解析函数，代码以实物为准
}
```
我们可以看到，其实我们是为每个可能需要的字段都提供了一个函数作为数据源，而当我们在 UI 中需要这个字段时，我们用 GraphQL 发出的请求就会触发这里的函数，从而返回我们需要的那些字段，然后经过类型检查返回到我们发出请求的地方。  
  
整个系统使用起来大概是这种感觉：  
```javascript

```