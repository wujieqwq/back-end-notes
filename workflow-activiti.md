## 概述

工作流(Workflow)，就是通过计算机对业务流程自动化执行管理。它主要解决的是“使在多个参与者之间按照某种预定义的规则自动进行传递文档、信息或任务的过程，从而实现某个预期的业务目标，或者促使此目标的实现”。

## 数据库

Activiti的后台是有数据库的支持，所有的表都以ACT_开头。 第二部分是表示表的用途的两个字母标识。 用途也和服务的API对应。

ACT_RE_*: 'RE'表示repository。 这个前缀的表包含了流程定义和流程静态资源 （图片，规则，等等）。

ACT_RU_*: 'RU'表示runtime。 这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。 Activiti只在流程实例执行过程中保存这些数据， 在流程结束时就会删除这些记录。 这样运行时表可以一直很小速度很快。

ACT_ID_*: 'ID'表示identity。 这些表包含身份信息，比如用户，组等等。

ACT_HI_*: 'HI'表示history。 这些表包含历史数据，比如历史流程实例， 变量，任务等等。

ACT_GE_*: 通用数据， 用于不同场景下，如存放资源文件。

表结构操作

1. 资源库流程规则表

    1) act_re_deployment 部署信息表

    2) act_re_model   流程设计模型部署表

    3) act_re_procdef   流程定义数据表

2. 运行时数据库表

    1) act_ru_execution 运行时流程执行实例表

    2) act_ru_identitylink 运行时流程人员表，主要存储任务节点与参与者的相关信息

    3) act_ru_task 运行时任务节点表

    4) act_ru_variable 运行时流程变量数据表

3. 历史数据库表

    1) act_hi_actinst 历史节点表

    2) act_hi_attachment 历史附件表

    3) act_hi_comment 历史意见表

    4) act_hi_identitylink 历史流程人员表

    5) act_hi_detail 历史详情表，提供历史变量的查询

    6) act_hi_procinst 历史流程实例表

    7) act_hi_taskinst 历史任务实例表

    8) act_hi_varinst 历史变量表

4. 组织机构表

    1) act_id_group 用户组信息表

    2) act_id_info 用户扩展信息表

    3) act_id_membership 用户与用户组对应信息表

    4) act_id_user 用户信息表

    这四张表很常见，基本的组织机构管理，关于用户认证方面建议还是自己开发一套，组件自带的功能太简单，使用中有很多需求难以满足

5. 通用数据表

    1) act_ge_bytearray 二进制数据表

    2) act_ge_property 属性数据表存储整个流程引擎级别的数据,初始化表结构时，会默认插入三条记录


## 接口

1. RepositoryService：提供管理流程部署和流程定义API。
2. RuntimeService：提供运行时流程实例进行管理与控制API。
3. TaskService：提供流程任务管理API。
4. IdentityService：提供对流程用户数据进行管理的API，包括用户组、用户及用户–组关系。
5. ManagementService：提供对流程引擎进行管理和维护的服务。
6. HistoryService：提供流程的历史数据进行操作API。
7. FormService：提供表单服务。