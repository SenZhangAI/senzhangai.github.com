---
layout: post
title: "think of microservice"
description: ""
keywords:
category:
tags: []
---

## 前言

这是一篇关于微服务的思考，此时的背景是刚了解Java Web领域的一些设计套路和微服务的实现，对微服务还一知半解，这里写下一些思考。

## 传统架构与微服务架构

以下观点来自《Clean Architecture》，原文不去考证了，大致是如下意思：

首先不能因为赶时髦就觉得微服务高大上，就鄙视传统架构。通常一个软件开发的生命周期，早期适合单体应用架构，快速开发，更灵活。
中期为了解耦，更适合分成不同component，再往后不同应用可能对性能要求，硬件条件都不一样，拆成微服务架构。最后需求不再变化，
稳定运维期间就更倾向于合并微服务(这里可能理解有误)。

## 传统架构

参考了 <https://github.com/XiaoMi/shepher> 代码，这一套代码我认为很优秀，优秀的代码结构简单很容易理解其架构。

首先是大的层次，分为如下几个module:

### shepher-commom/

这个是底层，不依赖于任何其他module，被其他module依赖，包含两个文件夹:

1. `commom/` 定义enum和annotation，注意并不是enum都定义在这里，例如`shepher-service`中定义不同Exception种类的enum，就不在common中
2. `util/`  主要包括判断合法IP地址，encode url等工具类功能，注意例如`shepher-service`、`shepher-web`中也有util目录，但这里的util
跟业务无关，是最通用的utils

### shepher-model/
使用了`shepher-commom/` ，`shepher-service`基础，用于定义model，**model并不是数据库的表**

例如`ReviewRequest`中包含snapshotContent以及newSnapshotContent

```java
public class ReviewRequest {
    private long id;
    private String cluster;
    // 省略...

    // for view, not mapping db
    private String snapshotContent;
    private String newSnapshotContent;
    // ...
}
```

例如`Snapshot`中包含Action和ReviewStatus，这两个都是`shepher-commom/`中定义的enum

```java
public class Snapshot {
    private long id;
    private String cluster;
    // 省略..

    // not mapped to database
    private Action actionDetail;
    private ReviewStatus reviewStatus;

   public Action getActionDetail() {
        if (actionDetail == null) {
            actionDetail = Action.get(this.action);
        }
        return actionDetail;
    }

    public void setActionDetail(Action actionDetail) {
        this.actionDetail = actionDetail;
    }

    public ReviewStatus getReviewStatus() {
        if (reviewStatus == null) {
            reviewStatus = ReviewStatus.get(this.status);
        }
        return reviewStatus;
    }

    public void setReviewStatus(ReviewStatus reviewStatus) {
        this.reviewStatus = reviewStatus;
    }
}
```

这里UserTeam更接近于视图而不是数据库表，例如`private String userName`，是通过联表查`user.name`获取到

```java
public class UserTeam {
    private long id;
    private long userId;
    private long teamId;
    private int roleValue;
    private int statusValue;
    private Date time;

    //no mapping mysql fields,custom attributes for view
    private String teamName;
    private String userName;
    private Role role;
    private Status status;
}
```

#### 总结

这里的model单独作为一个module，除了service需要model，web也需要，或者这里的意思在于将model与service解耦，
这样，是从数据库里获取到model还是从redis中获取到model之类的，model并不关注，它只是对业务的建模，静态模型。

### shepher-service/

#### exception/

首先是exception目录，用于定义service中的各种类型的exception

#### common/和util/

都用作辅助，什么放在common中，什么放在util中呢？我暂时还不能判断

#### dao/

数据访问对象，包括访问数据库的mapper，和访问zookeeper的zkClient的一层包装

看上去就像是对数据访问的一层薄封装，不是纯粹的数据库增删查改，偏向了一点业务，例如:

如下的listByTeam这个函数名就包含了业务含义，而不是简单的增删查改。

```java
@Mapper
public interface UserTeamMapper {

    @Insert("INSERT INTO user_team (user_id, team_id, role, status) VALUES (#{userId}, #{teamId}, #{roleValue}, #{statusValue})")
    int create(UserTeam userTeam);

    @Update("UPDATE user_team SET status = #{status} WHERE id = #{id}")
    int updateStatus(@Param("id") long id, @Param("status") int status);

    @Update("UPDATE user_team SET role = #{role} WHERE id = #{id}")
    int updateRole(@Param("id") long id, @Param("role") int role);

    @Select("SELECT role FROM user_team WHERE user_id = #{user} AND team_id = #{team} AND status = #{status}")
    Integer getRoleValue(@Param("user") long user, @Param("team") long team, @Param("status") int status);

    @Select("SELECT  user_team.id AS id, user.name AS user_name, team.name AS team_name, user_team.time AS time, " +
            "role as role_value, status as status_value, team_id, user_id " +
            "FROM user_team, user, team " +
            "WHERE team_id = #{team} AND user_team.team_id = team.id AND user_team.user_id= user.id AND user_team.status = #{status}")
    List<UserTeam> listByTeam(@Param("team") long team, @Param("status") int status);

    @Select("SELECT  user_team.id AS id, user.name AS user_name, team.name AS team_name, user_team.time AS time, " +
            "role as role_value, status as status_value, team_id, user_id " +
            "FROM user_team, user, team " +
            "WHERE user_team.id = #{id} AND user_team.team_id = team.id AND user_team.user_id= user.id")
    UserTeam get(@Param("id") long id);

    @Select("SELECT  user_team.id AS id, user.name AS user_name, team.name AS team_name, user_team.time AS time, " +
            "role as role_value, status as status_value, team_id, user_id " +
            "FROM user_team, user, team " +
            "WHERE user.id= #{user} AND user_team.team_id = team.id AND user_team.user_id= user.id AND user_team.status = #{status}")
    List<UserTeam> listByUser(@Param("user") long user, @Param("status") int status);

    @Select("SELECT user.name AS user_name FROM user_team, user WHERE user_team.team_id = #{team} AND user_team.user_id = user.id")
    List<String> listUserByTeamId(@Param("team") long team);
}
```

#### biz/

biz又是对doa层的一层封装，dao层直接跟sql语句打交道了，不关注一个参数有没有值，是否正确等，这是biz层关心的，
biz包含的作用包括：

1. 验参
2. catch exception，将其转换成service内统一定义的ShepherException，以及一些后处理校验等。
3. 更接近业务，例如dao中的create在biz中可能被封装为create和createIfNotExists两种

总之，在biz这一层，从外面看，已经没有mysql的痕迹了，包括mysql访问的异常都捕获并且转化成ShepherException这样自定义的异常了，
这一层的特征是biz与dao是一一对应的关系，biz是对dao的一层封装适配。

#### service/

对于biz层，其与dao层是一对一的关系，都是针对dao的封装适配，本质上还是单独的增删查改，
但service层则开始组织dao, 和biz，

### shepher-web/

如上的service module不包含与用户打交道的部分，只包含纯粹的业务逻辑，model是对业务的静态建模，而service则是针对model的一些业务动作

`shepher-web`则负责与用户打交道，包括:

#### controller

这是最重要的一个目录，处理核心的业务逻辑，可能包含多个service，注意service中可能包含多个biz，这里包含多个service，

controller处理`用户输入->业务逻辑->输出`， 只关注最核心的逻辑，不考虑失败。对于失败,异常抛出了, 对于异常，通过`@ControllerAdvice`统一处理所有的异常。

## 微服务

微服务一个麻烦的地方是异常，对于同一个单机服务，异常内部直接try-catch就可以了，而微服务呢？

我看到的一种做法是按照web的处理方式，设计error code，error message。 这种方法增加了设计成本，代码也更繁杂，因为都得包装成一个
`ReturnData<T>`的数据。如果一开始用的单机架构，现在改成微服务所有的返回值类型都要改成这样的包装类型，工作量极大且容易错。
毕竟Java没有方便地处理Maybe这样的数据结构。

貌似Java也只能这样处理了把？一种简化的方法是约定所有返回null都是错误，

### 我的设计思路

我猜测如果真要做微服务，首先**统一error处理**，将error单独作为一个包，这个很有必要，否则后期维护太复杂。

我猜测dubbo应该有考虑到处理exception的问题，可能有个hook，可以按照通常的写法，只要hook里面解决就好，如果dubbo没有这个那只能说设计的
不好，考虑其他微服务方案。
