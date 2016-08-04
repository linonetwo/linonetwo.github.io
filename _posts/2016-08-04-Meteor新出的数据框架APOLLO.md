---
layout: post
title: "Meteor新出的数据框架APOLLO"
date: 2016-08-04 22:29:00 +0800
image: 'blog-author.jpg'
description: 'APOLLO 可以替代 Relay 和 Redux'
main-class: 'FrontEnd'
color: '#66CCFF'
tags:
- Redux
- Relay
- React
- graphQL
categories:
twitter_text:
introduction: '介绍一个使用 GraphQL 进行通信的，将数据流和 React 界面进行干净漂亮的绑定的工具: APOLLO'
---

![prior](https://cdn-images-1.medium.com/max/800/1*QH_tgaH0Y9bY5T8Bh3FqVw.png)

GraphQL 是一个能表现出数据层次性，适合 React 这种单页应用结构的数据层模型。FaceBook 发现了 RESTful 数据层模型的不便之处，引领业界通过使用他们开发的 Relay 享受到 GraphQL 带来的 Optimistic Update（点完赞之后不用等网络传输延时直接点亮图标）、Data Diff（你随手发出了一大坨请求，Relay 自动帮你只请求真正需要更新的一小部分数据）、Declarative Data Need（数据需求就声明式地写在 React 组件旁边，前端后台都是声明式的，改需求很好改）。

![APOLLO](http://docs.apollostack.com/assets/client-diagrams/1-overview.png)  

Meteor 团队有着很丰富的数据流控制经验，他们发现了 Relay 的不便之处，引领业界通过使用他们开发的 APOLLO 享受到更简洁的接口，阅读到更易懂的教程，而且让人们敢用到生产项目里（Relay 很多人至今不敢重用）。  

![data flow 1](http://docs.apollostack.com/assets/client-diagrams/3-minimize.png)  

它的**书写**流程从 UI 开始:  

```javascript
const Feed = ({ params, feed, loading, loginToken }) => {
  const needsLogin = !loginToken && _.includes(['new', 'unread'], params.type);

  return (
    <div>
      { needsLogin && <div className="needs-login">Please log in to see this page.</div> }
      { feed.loading && <CircularProgress /> }
      { !feed.loading && !needsLogin &&
        feed.result.feed.pages.map((page) => <FeedPage page={page} />) }
    </div>
  );
}

// 然后声明式地，告诉 APOLLO 你需要什么数据

const FeedWithData = connect({
  mapQueriesToProps({ ownProps, state }) {
    return {
      feed: {
        query: `
          query getFeed($type: FeedType!) {
            feed(type: $type) {
              pages {    // 这边用花括号把 topics 括起来，意思是返回的是 {topics: {id, title}} 这样的对象的数组，保存在 pages 里
                topics { // 可以回到上面看看，pages 数据内的成员都传给了 <FeedPage page={page} />
                  id     // 这是每次 query 中不变的结构
                  title
                }
              }
            }
          }
        `,
        variables: {
          type: ownProps.params.type.toUpperCase(), // 对应 query getFeed($type: FeedType!) 中的 $type，这是 query 中可以定制的部分
        }
      }
    }
  },
  mapStateToProps(state) {
    return {
      loginToken: state.loginToken,
    };
  },
})(Feed);
```

接下来 APOLLO 会自动处理 Minimize、Augment、Fetch 的部分，只要实现告诉 APOLLO 你的服务器地址，它就会找出真正需要更新的界面对应的那一小部分请求，处理 GraphQL Schema Decorators，发给服务端。  

下面再看看服务端怎么回应我们的请求。
![data flow 2](http://docs.apollostack.com/assets/client-diagrams/4-normalize.png)  
先用 express、HAPI、Connect 或 koa 接收一下发来的 JSON 格式的请求:  

```javascript
app.use('/graphql', (req, res, next) => {
  return apolloServer({
    schema: Schema, // 里面写着我们接收什么格式的请求，返回什么结构的数据
    resolvers: resolveFunctions,
    connectors: Connectors,
    graphiql: true,
    // TODO: obviously don't do this statically. take it from req...
    context: {
      loginToken: req.headers.authorization
    },
    allowUndefinedInResolve: false,
    printErrors: true,
  })(req, res, next);
});

```
Schema 来自于这个文件，它在比较高的层次上描述了我们接收和返回的数据长什么样:  
schema.js

```javascript
const Schema = `
# A discourse Post
type Post {
  id: Int
  created_at: String // 巴拉巴拉一大堆
  cooked: String
  updated_at: String
  reply_count: Int
  reads: Int
  score: Int
  version: Int
  can_edit: Boolean
  can_delete: Boolean
  can_recover: Boolean
  can_wiki: Boolean
  raw: String
  wiki: Boolean
  user_id: Int
  category_id: Int
  username: String
  topic: Topic
}
# A discourse Topic
type Topic {
  id: String
  title: String // 也巴拉巴拉一大堆，略。具体要写什么，这个有趣的问题留给读者作为证明
  posts: PaginatedPostList
}

# A paginated list of posts
type PaginatedPostList {
  pages(page: Int, numPages: Int): [PostListPage]
}

# One page of discourse posts
type PostListPage {
  posts: [Post]
}

type RootQuery {
  allPosts: [Post]
  allTopics: [Topic]
  onePost(id: ID): Post
  oneTopic(id: ID): Topic
  feed(type: FeedType!, page: Int, numPages: Int): PaginatedTopicList
}

schema {
  query: RootQuery
}
`;
export default Schema;
```

resolveFunctions 来自于下面这个文件，它在比较低的层次上干脏活，实际去数据库取数据的就是它们:  

```javascript
const resolvers = {
  Post: { // 可以看到跟上面的 Schema 是一一对应的
    topic: (root, args, context) => {
      return context.connectors.Discourse._fetchEndpoint({
        url: ({ id }) => `/t/${id}`,
      }, { id: root.topic_id });
    },
  },
  Topic: { // 建议先写 Schema 再写 resolveFunctions，遵循从抽象到具体的书写顺序
    posts: (root, args, context) => {
      return context.connectors.Discourse._fetchEndpoint({
        url: ({ id }) => `/t/${id}.json`,
        map: (data) => ({ posts: data.post_stream.stream, topicId: data.id }),
      }, { id: root.id });
    },
  },
  PaginatedPostList: { // 本例仅作示意，意会，禁止复制黏贴
    pages: ({ posts, topicId }, args, context) => {
      return context.connectors.Discourse.getPaginatedPosts(posts, args, topicId);
    },
  },
  PostListPage: { // 和其他所有 GraphQL Tool 实现一样，你可以在这边返回一个 Promise，只要里面装着的对象格式与 Schema 里定义的相同
    posts(list) { // 你返回一个 thenable Monad 都可以
      return list.posts;
    },
  },

  RootQuery: {
    feed(_, { type, ...args }, context){
      return context.connectors.Discourse.getPagesWithParams(`/${type.toLowerCase()}`, args);
    },
    allPosts: (_, args, context) => {
      return context.connectors.Discourse._fetchEndpoint({
        url: '/posts',
        map: (data) => data.latest_posts,
      }, args);
    },
    allTopics() {
      throw new Error('AuthenticatedQuery.oneCategory not implemented'); // 例子来自官网，作者好像跑去写 APOLLO 本体了，例子就嗷嗷待哺等 PR
    },

    onePost: (_, args, context) => {
      return context.connectors.Discourse._fetchEndpoint({
        url: ({ id }) => `/posts/${id}`,
      }, args);
    },
    oneTopic: (_, args, context) => {
      return context.connectors.Discourse._fetchEndpoint({
        url: ({ id }) => `/t/${id}`,
      }, args);
    },
  }
};
export default resolvers;
```
